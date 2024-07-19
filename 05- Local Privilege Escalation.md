# Local Privilege Escalation
* Hunting for administrative privileges on computers.
* There are various ways of locally escalating privileges on Windows machines:
	- Missing patches
	- Automated deployment and AutoLogon passwords in cleartext (Kiosk Machines, still worth checking.)
	- AlwaysInstallElevated (Any user can run MSI binaries as SYSTEM)
	- Misconfigured Services
	- DLL Hijacking (via service execution)
	- NTLM Relaying and more...
- Useful tools for Local Privilege Escalation (LPE):
	- PowerUp
	- Privesc
	- winPEAS (the most comprehensive tool for local privilege escalation on Windows) (very noisy!)

* `Get-WmiObject -Class win32_service | Select pathname`: List all services and their paths.
* **Services issues using PowerUp**: 
	* `Get-ServiceUnquoted -Verbose`: Get services with unquoted paths and a space in their name. 
	* `Get-ModifiableServiceFile -Verbose`: Get services where the current user can write to its binary path or change arguments to the binary. 
	* `Get-ModifiableService -Verbose`: Get the services whose configuration current user can modify.

* sc.exe is used in order to interact with services. 
* `sc.exe sdshow <servicename>`: List the permissions of any service. 
	* `WD` means everyone has permission over the service.

* PowerUp `Invoke-AllChecks`
* Privesc: `Invoke-PrivEsc`
* WinPeas: `winpeas_any.exe`