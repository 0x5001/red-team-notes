# Domain Enumeration
### Directory Services Enumeration
- The idea is to get as much information as we can in active directory environment by using active directory services.
- This process maps various entities, trusts, relationships and privileges for the target domain.
- **Domain controller** is a special computer that we can query domain information using installed services on a domain controller computer.
- The enumeration can be done by using native executables and .NET classes that interacts with active directory environment.

* To speed up things, as an attacker, we can use **PowerView or Active Directory module** script that contains cmdlets that are useful to enumerate active directory environment.

- We will use ActiveDirectory PowerShell Module from Microsoft.
- It is signed by Microsoft so it works even in Powershell CLM. (Constrained Language Mode)
- To use ActiveDirectory module without installing RSAT, we can use `Import-Module` for the valid ActiveDirectory module DLL.

```powershell
$ Import-Nodule C:\AD\Tools\Microosft.ActiveDirectory.Management.dll
$ Import-Nodule C:\AD\Tools\ActiveDirectory.psd1
```

- **First run invisi-shell to bypass Windows security measures.**
* **Active-Directory Module's Cmdlets:** 
	* `Get-ADDomain`: To get information about current domain.
	* `Get-ADDomain -Identity <domain-FQDN>`: To get information about another domain.
		* It is very useful once trust relationships were established.
	* `(Get-ADDomain).DomainSID` : To get domain security identifier. 
	* `Get-ADDomainController` : To get information about domain controller.
	* `Get-ADDomainController -DomainName <domain-FQDN>`: To get information about another domain's domain controller.
	* `Get-ADUser -Filter * -Properties *` : to get information about all users in the domain.
		* A user's Member Security Identifier attribute is unique in a forest.
		* MemberSID = DomainSID + UserRID
	* `Get-ADUser -Identity student1 -Properties *` : To get information about a specific user in the domain.
	* `Get-ADGroup -Filter * -Properties *`: To get all groups.
		* Groups may contain another group. (It is called as nested group.)
		* On a computer, there are also local groups which cannot be enumerated via this cmdlet.
	* `Get-ADGroupMember -Identity "Domain Admins" -Recursive`: To get all users in a specific group.
	* `Get-ADForest`: To get details about the current forest.
	* `(Get-ADForest).Domains`: To get all domains in current forest.
	* `Get-ADForest | Select -ExpandProperty GlobalCatalogs`: To get details about current forest.

* Sometimes description attribute stores clear text passwords for user objects.
	* `Get-ADUser -Filter Description -Properties Description | Select Name,Description`
* `Get-ADComputer -Filter * -Properties *`: To get information about computer objects.
	* Tip: This output just shows computer objects in current domain, it does NOT mean that every computer object has a running computer.
* Enterprise groups like "Enterprise Admins" are available in the forest's root domain.

* **PowerView Cmdlets**:
	* https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview

### Group Policy Object Enumeration
* **Group policy provides the ability to manage configuration and changes easily and centrally in AD environment**.
* Group policy(GPO) is applied to Organizational Units (OUs). 
* An organization unit are used for managing active directory objects.
* GPOs allows configuration for:
    - Security Settings
    - Registry-based policy settings
    - Group policy preferences like startup/log-on script settings
    - Software installation

- GPO can be abused for various tactics like privesc, backdoors, persistence etc.

* PowerView Cmdlets:
	* https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview
* Microsoft's Group Policy Module Cmdlets:
	* `Get-GPO -All`: To get all GPOs. 
	* `Get-ADOrganizationalUnits -Filter * -Properties *`: To get all organization units (OUs).
	* `Get-GPO -Guid <OU-GUID>` ->Get GPO applied on an organizational unit.

### Access Control List Enumeration
* **Access Control Model enables control on the ability of a process to access objects and other resources in active directory** based on;:
    - **Access Tokens** (security context of a process - identity and privs of user).
    - **Security Descriptors** (SID of the owner, Discretionary ACL (DACL) and System ACL (SACL)).
- **Access Control Entry corresponds to an individual permission or audits access**.
	- Who has permission and what can be done on the object?
- ACE stands for Access Control Entry.
- **Access Control List is a list of access control entries**.

- Two types ACLs are there:
    - DACL defines the permissions trustees (a user or group) have on an object
    - SACL logs success and failure audit messages when an object is accessed.

* Globally Unique Identifier(GUID) is used in active directory to represent in some cases principles and active directory rights.
* PowerView Cmdlets:
	* https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview
* Active Directory Module Cmdlets:
	* `Get-Acl 'AD:\<LDAP-IDENTIFIER>'`: Get ACL without resolving GUIDs.

### Trust Relationship Enumeration
* **Trust is a relationship between two domains or forests which allows users of one domain or forest to access resources in the other domain or forest**.
- Trust can be automatic(parent-child, same forest etc.) or established(forest, external).
- Trusted Domain Objects (TDOs) represent the trust relationships in a domain.

* **Trust attributes:**
	* **Trust Direction**:
		- **One-Way trust**: Unidirectional, Users in the trusted domain can access resources in the trusting domain but the reverse won't.
		    - Trusted domain can access trusting domain
			- I trust you, you can access me.
		- **Two-Way trust**: Bidirectional, Users of both domains can access resources in the other domain.
			- I trust you, you trust me. So we can access our resources.
	* **Trust Transitivity**:
		- **Transitive**: can be extended to establish trust relationships with other domain.
		    - All the default intra-forest trust relationships between domains within a same forest are transitive two-way trusts.
		- **Nontransitive**: can not be extended to other domains in the forest. Can be two-way or one-way
		    - This is the default trust between domains in different forests when forests do not have a trust relationship between each other.

- **Domain Trusts**:
    - **Default/Automatic Trusts**:
        - **Parent-Child trust**: It is created automatically between the new domain and the domain that in the namespace hierarchy, whenever a new domain is added in a tree.
	        - This trust is always two-way transitive.
	        - For example; xxx.yyy.local is a child domain of yyy.local domain. so there is a parent-child trust.
        - **Tree-root trust**: It is created automatically between whenever a new domain tree is added to a forest.
	        - This trust is always two-way transitive.
	        - For example: yyy.local domain and xxx.local has tree-root trust if they are in the same forest.
    - **ShortCut Trusts**:
        - It is used to reduce access times in complex trust scenarios.
        - Can be one-way or two-way transitive.
    - **External Trusts**:
        - Between two domains in different forests when forests do not have a trust relationship.
        - Can be one-way or two-way and it is nontransitive
- **Forest Trusts**
    - It is established between different forests' root domains.
    - Cannot be extended to a third forest. (no implicit trust)
    - Can be one-way or two-way and transitive or non-transitive.

* Active Directory Module's Cmdlets:
	`Get-ADTrust -Filter *`: Get domain trusts

* PowerView Cmdlets:
	* https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview

###  User Hunting
* It's the process where we check high-privilege users are connected to a computer.

* PowerView Cmdlets:
	* https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview

* **Note that**: For server 2019 and onwards, local administrator privileges are required to list sessions. Using `Invoke-SessionHunter -FailSafe`, we can bypass this administrative privilege to get logged-on users for each machine to determine which users are logged to the machines.
