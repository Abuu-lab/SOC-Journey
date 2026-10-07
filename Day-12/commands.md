# Day 12 — Commands

DAY 12 — COMMANDS
Sysmon check
Get-Command sysmon64 -ErrorAction SilentlyContinue
Get-Service Sysmon* -ErrorAction SilentlyContinue
Sysmon events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
Sysmon fields
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
Sysmon Process Creation
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
Notepad process
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
Notepad + User
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
Parent Process
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
File Metadata
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
Digital Signature
Get-AuthenticodeSignature "<ExecutablePath>"
SHA-256
Get-FileHash "<ExecutablePath>" -Algorithm SHA256


### Referensi commands dari materi/evaluasi

## 💻 DAY 12 — COMMANDS

#### Sysmon check

```powershell
Get-Command sysmon64 -ErrorAction SilentlyContinue
```

```powershell
Get-Service Sysmon* -ErrorAction SilentlyContinue
```

#### Sysmon events

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

#### Sysmon fields

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### Sysmon Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### Notepad process

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### Notepad + User

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

#### Parent Process

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### File Metadata

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

#### Digital Signature

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

#### SHA-256

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

---



## 🧠 DAY 12 COMMAND MAP

```text
SYSMON
 ↓
Get-WinEvent
 ↓
Process Creation Event
 ↓
PROCESS
 ↓
Get-CimInstance
 ↓
PID / PPID / Path / CommandLine
 ↓
Get-Process
 ↓
USER
 ↓
PARENT
 ↓
FILE
 ↓
Get-Item
 ↓
Metadata
 ↓
Get-AuthenticodeSignature
 ↓
Signature
 ↓
Get-FileHash
 ↓
SHA-256
```

---
