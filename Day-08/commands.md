# Day 08 — Commands

COMMANDS DAY 8

Ini kumpulan command yang kita pelajari hari ini.

1. Current User
whoami

Fungsi:

Current user / security context
2. Current User Groups
whoami /groups

Fungsi:

Group membership dan security context
3. Semua Process + User
Get-Process -IncludeUserName |
Select-Object ProcessName, Id, UserName

Fungsi:

ProcessName
PID
UserName
4. Satu Process + User
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName

Contoh:

Get-Process -Id 6348 -IncludeUserName |
Select-Object ProcessName, Id, UserName
5. Satu Process + Full Context
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine

Contoh:

Get-CimInstance Win32_Process -Filter "ProcessId = 6348" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine


### Referensi commands dari materi/evaluasi

## 💻 DAY 7 — COMMANDS

Sesuai permintaanmu, ini **semua command utama yang kita pelajari di Day 7**:

#### 1. Process + PID + PPID

```powershell
Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId
```

#### 2. Process + PID + PPID + CommandLine

```powershell
Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId, CommandLine
```

#### 3. Process + PID + PPID + ExecutablePath + CommandLine

```powershell
Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### 🧠 DAY 7 COMMAND MAP

```text
Get-CimInstance Win32_Process
        ↓
Process investigation

Select-Object Name
        ↓
Process name

ProcessId
        ↓
PID

ParentProcessId
        ↓
PPID

ExecutablePath
        ↓
WHERE is the executable?

CommandLine
        ↓
HOW was it executed?
```

---



## 💻 COMMANDS DAY 8

Ini kumpulan command yang kita pelajari hari ini.

#### 1. Current User

```cmd
whoami
```

Fungsi:

```text
Current user / security context
```

---

#### 2. Current User Groups

```cmd
whoami /groups
```

Fungsi:

```text
Group membership dan security context
```

---

#### 3. Semua Process + User

```powershell
Get-Process -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Fungsi:

```text
ProcessName
PID
UserName
```

---

#### 4. Satu Process + User

```powershell
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Contoh:

```powershell
Get-Process -Id 6348 -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

---

#### 5. Satu Process + Full Context

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Contoh:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 6348" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

---



## 🧠 COMMAND MAP DAY 8

```text
whoami
   ↓
WHO AM I?

whoami /groups
   ↓
WHAT GROUPS / SECURITY CONTEXT?

Get-Process -IncludeUserName
   ↓
WHO IS RUNNING THE PROCESS?

Get-CimInstance Win32_Process
   ↓
PID / PPID / PATH / COMMAND
```

Jadi sekarang command toolkit kita sudah berkembang:

```text
DAY 1
hostname
whoami
systeminfo
tasklist

DAY 3
Get-CimInstance Win32_Process

DAY 4
Get-FileHash
Get-AuthenticodeSignature
Get-Item

DAY 5
Get-Service
Get-CimInstance Win32_Service
Get-Process
sc query

DAY 7
Get-CimInstance Win32_Process
    → Name
    → ProcessId
    → ParentProcessId
    → ExecutablePath
    → CommandLine

DAY 8
whoami /groups
Get-Process -IncludeUserName
```

**Day 8 selesai.** Menurut aturan kita, kita langsung boleh masuk **Day 9** pada sesi berikutnya.
