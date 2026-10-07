# Day 13 — Commands

DAY 13 — COMMANDS
1. Sysmon Process Creation
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message
2. Membuat process chain benign
cmd.exe /c notepad.exe
3. Live Notepad investigation
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
4. User context
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
5. Parent process
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
6. File metadata
Get-Item "<Image>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
7. Digital Signature
Get-AuthenticodeSignature "<Image>"
8. SHA-256
Get-FileHash "<Image>" -Algorithm SHA256


### Referensi commands dari materi/evaluasi

## 💻 DAY 13 — COMMANDS

#### 1. Sysmon Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message
```

#### 2. Membuat process chain benign

```cmd
cmd.exe /c notepad.exe
```

#### 3. Live Notepad investigation

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### 4. User context

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

#### 5. Parent process

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### 6. File metadata

```powershell
Get-Item "<Image>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

#### 7. Digital Signature

```powershell
Get-AuthenticodeSignature "<Image>"
```

#### 8. SHA-256

```powershell
Get-FileHash "<Image>" -Algorithm SHA256
```

---



## 🧠 COMMAND MAP DAY 13

```text
Get-WinEvent
      ↓
SYSMON EVENT 1
      ↓
Historical Process Evidence
      ↓
Get-CimInstance
      ↓
Live Process Context
      ↓
Get-Process
      ↓
User Context
      ↓
Get-Item
      ↓
File Metadata
      ↓
Get-AuthenticodeSignature
      ↓
Trust / Signature
      ↓
Get-FileHash
      ↓
File Fingerprint
```

---
