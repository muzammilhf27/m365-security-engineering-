```powershell
# Powershell Script to remove windows.old folder via sccm or intune after windows updgrade
takeown /F C:\Windows.old /R /D Y
icacls C:\Windows.old /grant SYSTEM:F /T /C /Q
icacls C:\Windows.old /grant Administrators:F /T /C /Q
$emptyFolder = "C:\EmptyFolder"
if (!(Test-Path $emptyFolder)) { New-Item -ItemType Directory -Path $emptyFolder }
robocopy C:\EmptyFolder C:\Windows.old /MIR /NFL /NDL /NJH /NJS /R:1 /W:1
Remove-Item -Path C:\Windows.old -Recurse -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 5
cmd.exe /c rd /s /q C:\Windows.old
Remove-Item -Path C:\EmptyFolder -Recurse -Force -ErrorAction SilentlyContinue
```
