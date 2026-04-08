$ShellLauncherClass = [wmiclass]"\\localhost\root\standardcimv2\embedded:WESL_UserSetting"

$Sid = "PASTE-SID-HERE"
$Shell = '"C:\KioskApp\MyApp.exe"'

$ShellLauncherClass.SetCustomShell($Sid, $Shell, $null, $null, 0)
