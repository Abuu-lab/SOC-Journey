# Day 14 — Commands

## 📌 COMMANDS LEARNED — DAY 14

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 6920} |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName
```

```powershell
Test-Path "C:\path\file.exe"
```

```powershell
Get-Process -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 1234" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 50
```

---
