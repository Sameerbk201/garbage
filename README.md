## 8. Apply Shell Launcher (Per User)

Open PowerShell as Administrator.

There are two working ways to load the Shell Launcher class.  
Use Method A first. If it fails on your system, use Method B.

---

### Method A: Direct class path

```powershell
$ShellLauncherClass = [wmiclass]("\\localhost\root\standardcimv2\embedded:WESL_UserSetting")

$Sid = "PASTE-SID-HERE"
$Shell = '"C:\KioskApp\MyApp.exe"'

$ShellLauncherClass.SetCustomShell($Sid, $Shell, $null, $null, 0)
````

---

### Method B: Variable-based class path

Use this if Method A fails.

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"
$ShellLauncherClass = [wmiclass]"\\$COMPUTER\${NAMESPACE}:WESL_UserSetting"

$Sid = "PASTE-SID-HERE"
$Shell = '"C:\KioskApp\MyApp.exe"'

$ShellLauncherClass.SetCustomShell($Sid, $Shell, $null, $null, 0)
```

---

### What this does

* Assigns your app as the custom shell
* Applies only to the kiosk user SID
* Keeps other users unchanged
* `0` means restart the shell if the app exits

### Replace these values

* `PASTE-SID-HERE` with the real SID of your kiosk user
* `C:\KioskApp\MyApp.exe` with the full path to your kiosk app

### Example

```powershell
$COMPUTER = "localhost"
$NAMESPACE = "root\standardcimv2\embedded"
$ShellLauncherClass = [wmiclass]"\\$COMPUTER\${NAMESPACE}:WESL_UserSetting"

$Sid = "S-1-5-21-1111111111-2222222222-3333333333-1003"
$Shell = '"C:\KioskApp\truehear\TrueHear.exe"'

$ShellLauncherClass.SetCustomShell($Sid, $Shell, $null, $null, 0)
```

---

