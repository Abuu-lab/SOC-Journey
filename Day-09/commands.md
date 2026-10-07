# Day 09 — Commands

DAY 9 — COMMANDS
1. Get System Events
Get-WinEvent -LogName System -MaxEvents 10
2. Get Application Events
Get-WinEvent -LogName Application -MaxEvents 10
3. Get Security Events
Get-WinEvent -LogName Security -MaxEvents 10
4. Pilih field penting
Get-WinEvent -LogName System -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
5. Cari Event 4624
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
} -MaxEvents 10
6. Cari Event 4625
Get-WinEvent -FilterHashtable
10


### Referensi commands dari materi/evaluasi

## 💻 DAY 9 — COMMANDS

#### 1. Get System Events

```powershell
Get-WinEvent -LogName System -MaxEvents 10
```

#### 2. Get Application Events

```powershell
Get-WinEvent -LogName Application -MaxEvents 10
```

#### 3. Get Security Events

```powershell
Get-WinEvent -LogName Security -MaxEvents 10
```

#### 4. Pilih field penting

```powershell
Get-WinEvent -LogName System -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### 5. Cari Event 4624

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
} -MaxEvents 10
```

#### 6. Cari Event 4625

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} -MaxEvents 10
```

#### 7. Cari Event 4688

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4688
} -MaxEvents 10
```

#### 8. Cari Event 7045

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='System'
    Id=7045
} -MaxEvents 10
```

---



## 🧠 COMMAND MAP DAY 9

```text
Get-WinEvent
      ↓
Windows Event Investigation

-LogName
      ↓
Which Event Log?

-MaxEvents
      ↓
How many events?

-FilterHashtable
      ↓
Filter specific events

Id
      ↓
Which Event ID?

Select-Object
      ↓
Which fields do I want?
```

Dan sekarang toolkit-mu mulai terbentuk:

```text
PROCESS
Get-Process
Get-CimInstance Win32_Process

SERVICE
Get-Service
Get-CimInstance Win32_Service
sc query

FILE
Get-Item
Get-FileHash
Get-AuthenticodeSignature

USER
whoami
whoami /groups
Get-Process -IncludeUserName

EVENTS
Get-WinEvent
```

---
