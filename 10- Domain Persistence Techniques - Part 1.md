- **As an attacker, once we have DA(domain admins) privileges, we should try to escalate our privileges to EA(enterprise admins).**
- We should use persistence techniques to protect our privileges in Active Directory once reached a user that has interesting privileges such as a domain admin user.
### Kerberos
- Kerberos is the basis of authentication in Windows Active Directory environment.
- **Clients(programs on behalf of a user) need to obtain tickets from Key Distribution Center(KDC) which is a service running on the domain controller. These tickets represent the client's credentials**.
- SPNs are used in Service Accounts, even there is not a running service.
- Domain Controllers have NTP services which converts time-zones.
- NTLM password hash is used for Kerberos RC4 encryption.
- Logon Ticket (TGT) provides user authentication to Domain Controller.
- **Kerberos policy only checked when TGT is created**.
- Domain Controller validates user account only when TGT is greater than 20 minutes. (User is valid or not.)
- Server Ticket (TGS) PAC validation is optional and rarely found in production environment.
    - Server LSASS sends PAC Validation request to DC's netlogon service.
    - If it runs as a service, PAC validation is optional.
    - If a service runs as System, it performs server signature verification on the PAC.
### Domain Persistence - Golden Ticket
* A golden ticket is signed and encrypted by the secrets of "krbtgt" account.
* The "krbtgt" user hashes and keys could be used to impersonate any user with any privileges from even a non-domain machine.
* The secret used to encrypt or decrypt the ticket is the "krbtgt" account secret. If we have the secret, we can forge our own TGT.
* **KRBTGT account's password is not changed automatically. If we can extract secrets like AES keys for KRBTGT account, we can forge a valid TGT for KRBTGT account**. 
* Since user account validation is not done by Domain Controller (KDC service) until TGT is older than 20 minutes, we can use even deleted/revoked accounts.
* **If we can extract krbtgt hash from domain controller, we can create our TGT tickets.** Because TGT tickets are using "krbtgt" user's NTLM hash for encryption key (RC4 encryption key).
- **If we create a TGT ticket for a high-privilege user such as a user in Domain Admins group, we can access this user any time that we want to access.**.
- **If we use our created TGT ticket to request a TGS ticket to access HTTP service in Domain Controller, it becomes a persistence technique via WinRM.**
* The "krbtgt" user hash could be used to impersonate any user with any privileges from even a non-domain-joined machine.
* `"lsadump::lsa /patch"`: Execute on Domain Controller as Domain Admin to get "krbtgt" hash.
* `"lsadump::dcsync /user:dcorp\krbtgt"`: **To use the DCSync feature for getting AES keys for "krbtgt" account**.
	* **Using the DCSync option is useful because no needs code execution on the target domain controller**.
	* **DCSync does not require command execution on the target domain controller server. But it needs domain admin privileges on any domain-joined machine.**
* **Pretty noisy persistence technique!**
* `"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:<user-sid> /aes256:<KRBTGT_ACCOUNT_AES256_Keys> /startoffset:0 /endin:600 /renewmax:10080 /ptt"`
* Even though passwords changed for Administrator account, we still have Administrator logon via TGT ticket. This is Golden Ticket Attack! (Make sure "Administrator" user in "Domains Admins" group!)

```
- /ptt injects the ticket in current PowerShell process.
- /ticket saves the ticket to a file.
```
### Domain Persistence - Silver Ticket
- Silver ticket is a valid TGS.
- TGS tickets are encrypted and signed with the NTLM hash of the service account.
- **If we can extract NTLM hash of the service account, we can create our TGS tickets**. Because TGS tickets are using "service" user's NTLM hash for encryption key (RC4 encryption key).
- **If we create a TGS ticket for a high-privileges user such as a user in Domain Admins group, we can access this service as our chosen user.** So it becomes a persistence technique.
- By default, service accounts' passwords reset each month.
- **A valid TGS. (We can have service account privileges via this persistence technique.)**
- **Via this method, we have access service running computer objects**.
- `"kerberos::golden /user:Administrator /domain:<domain-FQDN> /sid:<domain-SID> /target:<target-server-FQDN> /service:HOST /rc4:<ntlm-hash-service-account> /ptt"`
- Services rarely check PAC (Privileged Attribute Certificate).
- Services will allow access only to services themselves.
- Reasonable persistence period (default 30 days for computer accounts.)
- **There are multiple default services running on Windows installed computers. Some of them are CIFS/HOST/RPCSS/WSMAN, all of these windows services use machine account's password as service NTLM hash to validate a TGS ticket.**
- **We are going to target domain controller's machine account to access domain controller's services. For example, we can use HOST service to access domain controller scheduled tasks. So we can create a new service on DC to get command execution.**
- Machine accounts ends with $ sign. (For example: `DCORP-DC$`)
### Domain Persistence - Diamond Ticket
* A diamond ticket is created by decrypting a valid TGT, making changes it and re-encrypt it using the AES keys of "krbtgt" account.
- Golden ticket was a TGT forging attacks whereas diamond ticket is a TGT modification attack.
- Persistence lifetime depends on "krbtgt" account.
- A diamond ticket is better opsec friendly approach to persistence.
- We can modify: security identifier to claim other users in TGT.
* `Rubeus.exe diamond /krbkey:<krbtgt_aes_key> /user:studentx /password:studentxpassword /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\windows\system32\cmd.exe /show /ptt`
- Via this method, we can access administrator privileges to services.
### Domain Persistence - Skeleton Key
* It updates a Domain Controller's LSASS process, so that it allows access as any user with a single our chosen password.
- We manipulate the authentication mechanism in Domain Controller's LSASS process.
- All the publicly known methods are NOT persistent once DC rebooted. `"privilege::debug" "misc::skeleton"'`: To Execute skeleton key attack. `Enter-PSSession -ComputerName dcorp-dc -Credential dcorp\Administrator`: Password is `mimikatz`
- In case, LSASS is running as a protected process, we can load the mimikatz driver from the disk of the target DC. When it is a protected process, we should need filesystem access on DC to upload mimidriv.sys driver.
* It might crash ADCS service.
### Domain Persistence - DSRM
* DSRM is Directory Services Restore Mode.
* **This is by far the longest persistence technique.**
* **There is a local administrator on ever DC called "Administrator" whose password is the DSRM password once the server promoted the DC.**
- **DSRM password (SafeModePassword) is required when a server is PROMOTED to domain controller.**
- DSRM password is rarely changed.
- After altering the configuration on the DC, it is possible to run a pass the hash attack to access the DC.
* `"token::elevate" "lsadump::sam"'`: Local Administrator NTLM Hash dump including DSRM credentials.
- Since it is the local administrator of the DC, we can pass the hash to authenticate.
- But we need to change logon behavior to log in.
	- `Set-ItemProperty "HKLM:\System\CurrentControlSet\Control\lSA\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD -Verbose`: Changing logon behavior from 0 to 2 in Windows Registry.
- After that, we can execute a pass the hash attack.
	- `"sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:<NTLM-HASH-OF-DSRM-ADMINISTRATOR-PASSWORD> /run:powershell.exe"`
### Domain Persistence - Custom SSP
* A security support provider (SSP) is a DLL which provides ways for an application to obtain an authenticated connection.
* Some SSP Packages by Microsoft are:
    - NTLM
    - Kerberos
    - Wdigest
    - CredSSP
- Mimikatz provides a custom SSP - mimilib.dll. This SSP logs local logons, service account and machine account passwords in clear text on the target server.
- **Very Noisy!**
- DA's, can inject their own SSPs.
* `"misc:memssp"`
- `type C:\Windows\system32\mimilsa.log`: to read credentials after the injection.
- We can change output path to a SYSVOL directory in DLL file content, so we can access this log file via SMB fileshare. So that becomes a persistence technique.
