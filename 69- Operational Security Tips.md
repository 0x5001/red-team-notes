* **Using the built-in tools, living off the land, increases stealthy.**
* "net" command uses SAMR protocol to query domain services on domain controller, other tools are using LDAP protocol to query the database so LDAP-based tools are most useful for OPSEC considerations
* Do not rush to get domain admin, always enumerate first. Check machines or users to determine legit or not.
* Making changes local administrator group is very noisy!
* ActiveDirectory module is stealthier than PowerView script.
* Check _pwdlastset_, _badpwdcount_, _logoncount_ attributes for a user to check whether it is a decoy user or not.
* Create tickets that is in Kerberos policy time range. (Mimikatz creates ticket for 10 years in default.)
* Staged payloads are mostly opsec friendly approach to targets.
* Less interactivity to Domain Controller, more operational security.
* `Invoke-Bloodhound -Stealth` or `SharpHound.exe --stealth` is for OPSEC friendly.
* To avoid detections like MDI: `Invoke-Bloodhound -ExcludeDCs`
* PSExec is noisy.
* EDR is fine for WinRS. but MDI generates an alert for winRS connections
* Don't use LSASS unless it is the last step to try out.
* Generate tickets from AES keys to avoid MDI detections (Micrososft EDR detection).
* Everytime try to use AES256 keys rather than RC4 key for OPTH technique.
* DCSync's not silent!
* Saving files to "temp directories" in Windows are very detectable by EDRs.