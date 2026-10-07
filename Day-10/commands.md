# Day 10 — Commands

DAY 10 — COMMANDS YANG BARU
1. Event dalam 24 jam terakhir
Get-WinEvent -FilterHashtable @{
    LogName='System'
    StartTime=(Get-Date).AddDays(-1)
} -MaxEvents 20 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
2. Time Window tertentu
Get-WinEvent -FilterHashtable @{
    LogName='System'
    StartTime=(Get-Date '2026-09-28 13:00:00')
    EndTime=(Get-Date '2026-09-28 14:00:00')
} |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
3. Multiple Event IDs
Get-WinEvent -FilterHashtable
GET EVENTS
    ↓
FILTER
    ↓
READ
    ↓
SORT BY TIME
    ↓
CORRELATE
    ↓
INVESTIGATE


### Referensi commands dari materi/evaluasi

## 💻 DAY 9 — COMMANDS

Karena kamu minta command selalu dicatat, ini **commands yang dipelajari di Day 9**:

#### Get System Events

```powershell
Get-WinEvent -LogName System -MaxEvents 10
```

#### Get Application Events

```powershell
Get-WinEvent -LogName Application -MaxEvents 10
```

#### Get Security Events

```powershell
Get-WinEvent -LogName Security -MaxEvents 10
```

#### Pilih field penting

```powershell
Get-WinEvent -LogName System -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### Event 4624

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
} -MaxEvents 10
```

#### Event 4625

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} -MaxEvents 10
```

#### Event 4688

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4688
} -MaxEvents 10
```

#### Event 7045

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='System'
    Id=7045
} -MaxEvents 10
```

#### 🧠 DAY 9 COMMAND MAP

```text
Get-WinEvent
      ↓
Windows Event Investigation

-LogName
      ↓
Event Log

-MaxEvents
      ↓
Jumlah event

-FilterHashtable
      ↓
Filter event

Id
      ↓
Event ID

Select-Object
      ↓
Pilih field
```

---



## 💻 DAY 10 — COMMANDS YANG BARU

#### 1. Event dalam 24 jam terakhir

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='System'
    StartTime=(Get-Date).AddDays(-1)
} -MaxEvents 20 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### 2. Time Window tertentu

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='System'
    StartTime=(Get-Date '2026-09-28 13:00:00')
    EndTime=(Get-Date '2026-09-28 14:00:00')
} |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### 3. Multiple Event IDs

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624,4625
} -MaxEvents 20 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

#### 4. Sort berdasarkan waktu

```powershell
Get-WinEvent -LogName System -MaxEvents 20 |
Select-Object TimeCreated, Id, ProviderName, Message |
Sort-Object TimeCreated
```

---



## 🧠 DAY 10 COMMAND MAP

```text
Get-WinEvent
      ↓
ambil historical events
      ↓
FilterHashtable
      ↓
batasi log / ID / waktu
      ↓
Select-Object
      ↓
ambil field penting
      ↓
Sort-Object TimeCreated
      ↓
buat timeline
```

Ini mulai menjadi workflow investigation:

```text
GET EVENTS
    ↓
FILTER
    ↓
READ
    ↓
SORT BY TIME
    ↓
CORRELATE
    ↓
INVESTIGATE
```

---
