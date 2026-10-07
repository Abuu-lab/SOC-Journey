# Day 05 — Commands

## 16. COMMANDS DAY 5

Masukkan ke `commands.md`.

#### PowerShell

```powershell
# Semua service
Get-Service

# Running services
Get-Service |
Where-Object {$_.Status -eq "Running"} |
Select-Object Name, Status, DisplayName

# Detail service
Get-CimInstance Win32_Service -Filter "Name = 'Spooler'" |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName

# Cari process berdasarkan PID
Get-Process -Id 1234

# Detail process berdasarkan PID
Get-CimInstance Win32_Process -Filter "ProcessId = 1234" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### CMD

```cmd
sc query
```

Service tertentu:

```cmd
sc query Spooler
```

---
