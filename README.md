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
What you should change now is this:

Do not rely on SetDefaultShell("explorer.exe", 0) for your admin account.
Keep the kiosk user on a custom shell, and add explicit explorer.exe custom-shell mappings for every non-kiosk account you care about. Shell Launcher uses the user’s  ￼there is no user match does it fall back to the default shell.

Use this pattern.

1) Get the SIDs

Run as admin:

wmic useraccount get name,sid

Note the SID for:

* your kiosk user
* your admin account
* any other local user that should get normal Explorer

2) Disable Shell Launcher while editing

$ShellLauncherClass = [wmiclass]"\\localhost\root\standardcimv2\embedded:WESL_UserSetting"
$ShellLauncherClass.SetEnabled($false)

3) Remove old custom entries for the users you’re fixing

Use the real SIDs:

$kioskSid = "KIOSK-SID-HERE"
$adminSid = "ADMIN-SID-HERE"
$user2Sid  = "OTHER-USER-SID-HERE"
$ShellLauncherClass.RemoveCustomShell($kioskSid)
$ShellLauncherClass.RemoveCustomShell($adminSid)
$ShellLauncherClass.RemoveCustomShell($user2Sid)

4) Re-add them explicitly

$kioskShell = '"C:\KioskApp\MyApp.exe"'
$explorerShell = 'explorer.exe'
$ShellLauncherClass.SetCustomShell($kioskSid, $kioskShell, $null, $null, 0)
$ShellLauncherClass.SetCustomShell($adminSid, $explorerShell, $null, $null, 0)
$ShellLauncherClass.SetCustomShell($user2Sid,  $explorerShell, $null, $null, 0)

Your 0 means “restart the shell if it exits,” which is valid for Shell Launcher return behavior.

5) Only keep a default shell if you truly need a catch-all

If you want unknown users to still get Explorer, you can leave the default as Explorer:

$ShellLauncherClass.SetDefaultShell("explorer.exe", 0)

But for your admin account, do not depend on that default. The admin should have its own explicit SID mapping to explorer.exe. Shell Launcher falls back to the default only when there is no user or group custom configuration.

6) Re-enable Shell Launcher

$ShellLauncherClass.SetEnabled($true)

7) Verify

$ShellLauncherClass.IsEnabled()
$ShellLauncherClass.GetCustomShell($kioskSid)
$ShellLauncherClass.GetCustomShell($adminSid)
$ShellLauncherClass.GetCustomShell($user2Sid)
$ShellLauncherClass.GetDefaultShell()

What you want to see:

* kiosk SID → C:\KioskApp\MyApp.exe
* admin SID → explorer.exe
* other normal users → explorer.exe

Why this is better

The docs say Shell Launcher checks the signed-in user’s SID-specific custom config first, then group configs, then the default config. They also say the search order across multiple group configs is not defined. That is why explicit per-user mappings are the cleanest route for your setup.

One thing to avoid

Do not try to solve this by assigning Explorer to a broad group and the kiosk app to another broad group. Group match order is not defined, so that can create unpredictable results.

Safe recovery if something goes wrong

From an admin PowerShell session:

$ShellLauncherClass.SetEnabled($false)

Or remove just the kiosk mapping:

$ShellLauncherClass.RemoveCustomShell($kioskSid)

If you want, paste the usernames and the actual app path, and I’ll turn this into an exact ready-to-run script with your real variable names.


