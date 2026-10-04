Benar. **Tidak perlu diulang di hari yang sama.** Mulai sekarang kalau ada jawaban yang masih salah, aku tandai dan **dibawa ke Active Recall Day berikutnya**. Kalau diulang nanti masih salah, baru kita kejar sampai **2× benar**.

Jawaban A–G-mu cukup untuk lanjut. Ada beberapa detail yang belum presisi, tapi masuk aturan **PASS**.

# 🛡️ DAY 12 — SYSMON & ENDPOINT TELEMETRY

### Process Creation • Parent Process • Command Line • User • File • Event Correlation

Sampai Day 11 kita banyak memakai:

```text
Windows Event Logs
        ↓
Authentication
        ↓
4624 / 4625
        ↓
Caller Process
```

Masalah yang baru saja kamu alami:

```text
4625
 ↓
Caller PID 24416
 ↓
Get-Process
 ↓
Process sudah hilang
```

Nah, hari ini kita mulai belajar **Sysmon**.

Tujuannya:

> mendapatkan endpoint telemetry yang lebih kaya untuk investigation.

Ini sangat penting karena nanti kita ingin bisa melihat aktivitas process secara lebih detail dan menghubungkannya dengan file, user, parent process, command line, dan timestamp.

---

# 1. KENAPA SYSMON?

Windows sudah punya Event Log.

Tetapi untuk SOC, kita sering membutuhkan telemetry endpoint yang lebih detail.

Secara konsep:

```text
Windows Event Logs
        ↓
basic / native telemetry

Sysmon
        ↓
richer endpoint telemetry
```

Contoh informasi yang bisa sangat berguna saat process investigation:

```text
Process
Process ID
Parent Process
CommandLine
Image / Executable
User
Timestamp
Hash
```

Jadi target kita:

```text
Alert
 ↓
Event
 ↓
Process
 ↓
Parent
 ↓
User
 ↓
CommandLine
 ↓
File
 ↓
Hash
```

---

# 2. SYSMON BUKAN ANTIVIRUS

Ini penting.

Sysmon adalah **system monitoring / telemetry tool**.

Jangan berpikir:

```text
Sysmon
=
antivirus
```

Bukan.

Sysmon terutama membantu menghasilkan dan mencatat **telemetry** yang bisa digunakan investigator dan detection engineer.

Jadi:

```text
Sysmon
→ visibility
```

bukan:

```text
Sysmon
→ automatically decides malware
```

---

# 3. MENTAL MODEL DAY 12

Mulai sekarang kita punya:

```text
EVENT
  ↓
WHAT?
  ↓
PROCESS
  ↓
WHO?
  ↓
USER
  ↓
WHO STARTED IT?
  ↓
PARENT
  ↓
HOW?
  ↓
COMMANDLINE
  ↓
WHAT FILE?
  ↓
HASH / SIGNATURE
```

Ini persis alur investigation yang kamu minta.

---

# 4. SYSMON EVENT YANG PALING PENTING UNTUK KITA

Hari ini kita belum menghafal banyak Event ID.

Kita fokus dulu pada:

### Event ID 1 — Process Creation

Ini sangat penting.

Secara konsep:

```text
Event ID 1
    ↓
Process Create
```

Artinya Sysmon mencatat process baru dibuat.

Dan biasanya telemetry process creation sangat berguna untuk melihat:

```text
Image
CommandLine
ParentImage
ParentCommandLine
User
Hashes
```

Ini jauh lebih dekat dengan investigation yang sedang kita latih.

---

# 5. CHECK APAKAH SYSMON SUDAH ADA

Jalankan PowerShell:

```powershell
Get-Command sysmon64 -ErrorAction SilentlyContinue
```

Lalu:

```powershell
Get-Service Sysmon* -ErrorAction SilentlyContinue
```

Tujuannya sederhana:

> Apakah Sysmon sudah terpasang dan berjalan?

---

# 6. KALAU SYSMON TERPASANG

Cari log Sysmon:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

Kemudian:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

---

# 7. KALAU BELUM TERPASANG

Jangan mengarang hasil.

Tulis saja:

```text
Sysmon belum terpasang.
```

Untuk Day 12 kita tetap bisa memahami alurnya, tetapi agar praktik Sysmon benar-benar berjalan, nanti kita pasang Sysmon dari **Microsoft Sysinternals** dan membuat konfigurasi lab yang aman.

Kita tidak perlu melakukan attack untuk menghasilkan telemetry.

Kita bisa menghasilkan event dari aktivitas normal.

---

# 8. MENGAPA PROCESS CREATION SANGAT PENTING?

Misalnya ada:

```text
powershell.exe
```

Kita sebelumnya harus menjalankan query process.

Tetapi dengan telemetry process creation, kita bisa mendapatkan historical evidence:

```text
Time
Process
Parent
CommandLine
User
Hash
```

Jadi saat process sudah mati:

```text
Process ended
```

kita masih bisa memiliki:

```text
Historical process creation event
```

Inilah solusi terhadap masalah yang kamu temui pada Day 11.

---

# 9. CURRENT STATE VS TELEMETRY

Sekarang perbedaan kita semakin jelas.

### Current State

```powershell
Get-Process
```

Menjawab:

> **What is alive now?**

### Historical Windows Event

```powershell
Get-WinEvent
```

Menjawab:

> **What was recorded?**

### Sysmon

Memberikan:

> **Richer endpoint telemetry about what happened.**

Mental model:

```text
NOW
↓
Get-Process

PAST
↓
Windows Event Logs

PAST + RICHER ENDPOINT TELEMETRY
↓
Sysmon
```

---

# 10. PRAKTIK 1 — CEK SYSMON

Jalankan:

```powershell
Get-Command sysmon64 -ErrorAction SilentlyContinue
```

dan:

```powershell
Get-Service Sysmon* -ErrorAction SilentlyContinue
```

Kirim hasilnya.

---

# 11. PRAKTIK 2 — CARI SYSMON EVENT

Kalau Sysmon sudah tersedia:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

Cari apakah ada:

```text
Id : 1
```

---

# 12. PRAKTIK 3 — FILTER EVENT ID 1

Gunakan:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

Cari satu process creation event.

---

# 13. JANGAN LANGSUNG MEMBACA SEMUA FIELD

Kalau menemukan event panjang, gunakan investigator mindset:

### STEP 1 — WHEN

```text
TimeCreated
```

### STEP 2 — WHAT

```text
Process
```

### STEP 3 — WHO

```text
User
```

### STEP 4 — WHO STARTED IT

```text
Parent Process
```

### STEP 5 — HOW

```text
CommandLine
```

### STEP 6 — WHERE

```text
Image / Executable
```

### STEP 7 — FINGERPRINT

```text
Hash
```

Jadi:

```text
WHEN
 ↓
WHAT
 ↓
WHO
 ↓
PARENT
 ↓
HOW
 ↓
WHERE
 ↓
HASH
```

🔥 Ini yang ingin aku tanamkan di otakmu.

---

# 14. PROCESS INVESTIGATION WORKFLOW

Misalnya kamu menemukan:

```text
Event ID 1
Process:
powershell.exe
```

Jangan langsung buka hash.

Urutannya:

```text
1. Kapan?
2. Process apa?
3. User siapa?
4. Parent siapa?
5. CommandLine apa?
6. ExecutablePath/image di mana?
7. File apa?
8. Hash?
9. Signature?
10. Network?
11. Correlate dengan event lain
12. Assessment
```

Kenapa tidak langsung hash?

Karena **hash hanya menjawab identity/fingerprint file**.

Kita masih belum tahu:

> Kenapa file tersebut berjalan?

---

# 15. CONTOH

Misalnya:

```text
Time:
20:15:01

Process:
powershell.exe

User:
Buya

Parent:
explorer.exe

CommandLine:
powershell.exe -File C:\Temp\a.ps1
```

Sekarang kita punya context.

Baru kemudian:

```text
a.ps1
 ↓
metadata
 ↓
signature
 ↓
hash
```

Kemudian:

```text
network
 ↓
correlation
```

---

# 16. OBSERVATION → HYPOTHESIS

Contoh:

```text
PowerShell
+
Temp path
+
script execution
```

### Observation

> PowerShell menjalankan script dari Temp.

### Hypothesis

> Aktivitas ini memerlukan investigation lebih lanjut.

### Bukan langsung:

> Ini malware.

Karena kita masih memerlukan evidence.

---

# 17. PRAKTIK 4 — BUAT PROCESS NORMAL

Kita membutuhkan aktivitas yang mudah diamati.

Jalankan:

```cmd
notepad.exe
```

Kemudian jangan langsung ditutup.

Cari process:

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Catat:

```text
PID
PPID
ExecutablePath
CommandLine
```

Kemudian tutup Notepad.

Ini membantu kamu memahami:

```text
PROCESS EXISTS
      ↓
INVESTIGATE
      ↓
PROCESS EXITS
```

---

# 18. PRAKTIK 5 — HUBUNGKAN DENGAN USER

Saat Notepad masih berjalan:

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Sekarang kamu memiliki:

```text
Process
PID
PPID
Path
CommandLine
User
```

---

# 19. PRAKTIK 6 — FILE INVESTIGATION

Ambil `ExecutablePath` Notepad.

Misalnya:

```text
C:\Windows\System32\notepad.exe
```

Gunakan:

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

Kemudian:

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

dan:

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

Sekarang kamu sudah melakukan:

```text
PROCESS
 ↓
USER
 ↓
PARENT
 ↓
PATH
 ↓
FILE
 ↓
METADATA
 ↓
SIGNATURE
 ↓
HASH
```

---

# 20. MINI INVESTIGATION DAY 12 🔥

Kita buat latihan yang jauh lebih sesuai dengan cara belajar kamu.

Jangan langsung mencari kesimpulan.

Ikuti urutan.

## CASE

Kita akan investigate:

```text
notepad.exe
```

### STEP 1 — PROCESS

Cari:

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### STEP 2 — USER

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

### STEP 3 — PARENT

Ambil PPID.

Kemudian:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### STEP 4 — FILE

Ambil executable path.

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### STEP 5 — SIGNATURE

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

### STEP 6 — HASH

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

---

# 21. SEKARANG JANGAN LANGSUNG MENILAI

Buat evidence table:

```text
=== DAY 12 EVIDENCE ===

PROCESS
Name:
PID:
PPID:

USER
User:

PARENT
Name:
PID:

EXECUTABLE
Path:

COMMAND
CommandLine:

METADATA
Name:
Length:
CreationTime:
LastWriteTime:
LastAccessTime:

SIGNATURE
Status:
Signer:

HASH
SHA-256:
```

Baru setelah semua data terkumpul:

```text
ANALYSIS

1. Apa yang observed?

2. Apa yang normal/expected?

3. Apakah ada anomaly?

4. Evidence apa yang mendukung anomaly?

5. Evidence apa yang belum ada?

6. Assessment:
   Benign-looking /
   Needs Investigation /
   Insufficient Evidence
```

---

# 22. 🧠 INI YANG AKAN TERUS KITA LATIH

Kamu sebelumnya mengatakan ingin tahu:

> **“yang mana dulu harus dilihat, mana dulu harus dicari, mana yang harus diperiksa, lalu mana yang dieksekusi sampai kesimpulan.”**

Maka mulai Day 12 kita gunakan pola:

```text
                 ALERT
                   ↓
             IDENTIFY OBJECT
                   ↓
                 TIME
                   ↓
              WHAT HAPPENED?
                   ↓
              WHO INVOLVED?
                   ↓
             PROCESS / USER
                   ↓
              PARENT PROCESS
                   ↓
              COMMAND LINE
                   ↓
             EXECUTABLE PATH
                   ↓
                  FILE
                   ↓
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    METADATA   SIGNATURE    HASH
        └──────────┼──────────┘
                   ↓
              NETWORK
                   ↓
          CORRELATE EVENTS
                   ↓
            EVIDENCE GAP
                   ↓
              HYPOTHESIS
                   ↓
             MORE EVIDENCE
                   ↓
              ASSESSMENT
                   ↓
              CONCLUSION
```

**Tidak semua case membutuhkan semua cabang.**

Investigator harus belajar memilih **evidence yang paling relevan terhadap pertanyaan yang sedang dicari**.

Itu yang membedakan:

> **investigation**

dengan:

> **sekadar menjalankan command.**

---

# 23. ATTACK METHOD

Hari ini kita hanya mengenal konsepnya:

### Process Injection / Suspicious Process Execution

Attacker bisa menyalahgunakan legitimate process atau menjalankan tool legitimate untuk mencapai tujuan mereka.

Karena itu:

```text
powershell.exe
cmd.exe
svchost.exe
```

tidak boleh dinilai hanya berdasarkan nama.

SOC harus melihat:

```text
Parent
CommandLine
User
Path
File
Hash
Network
Timeline
```

---

# 24. DEFENSE METHOD

Defensive strategy:

```text
Endpoint Telemetry
       ↓
Process Creation
       ↓
Parent/Child Relationship
       ↓
CommandLine
       ↓
File Identity
       ↓
Correlation
```

Sysmon membantu menyediakan telemetry yang lebih kaya untuk proses seperti ini.

---

# 💻 DAY 12 — COMMANDS

### Sysmon check

```powershell
Get-Command sysmon64 -ErrorAction SilentlyContinue
```

```powershell
Get-Service Sysmon* -ErrorAction SilentlyContinue
```

### Sysmon events

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

### Sysmon fields

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

### Sysmon Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

### Notepad process

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### Notepad + User

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

### Parent Process

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### File Metadata

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### Digital Signature

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

### SHA-256

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

---

# 🧠 DAY 12 COMMAND MAP

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

# 🔄 ACTIVE RECALL DAY 12

Sesuai aturan kita, aku **tidak bombardir semua materi lama**.

Hari ini hanya ada **3 reminder yang masih perlu diperkuat**:

### Q1

Apa perbedaan **Service dan Process**?

### Q2

Apa itu **Security Context**?

### Q3

Apa arti `TimeCreated` pada Event Log?

---

# 🎯 TARGET DAY 12

Kali ini jangan cuma mengejar:

> "Command apa yang harus dipakai?"

Tapi mulai hafalkan **alur berpikirnya**:

```text
ADA ALERT
   ↓
APA OBJECT-NYA?
   ↓
APA YANG TERJADI?
   ↓
KAPAN?
   ↓
SIAPA?
   ↓
PROCESS APA?
   ↓
PARENT SIAPA?
   ↓
DIJALANKAN DENGAN APA?
   ↓
FILE-NYA DIMANA?
   ↓
IDENTITAS FILE?
   ↓
HASH / SIGNATURE
   ↓
CORRELATE
   ↓
APA YANG MASIH KURANG?
   ↓
ASSESSMENT
   ↓
KESIMPULAN
```

**Kerjakan Praktik 1–6 + Mini Investigation + Q1–Q3.**

Dan untuk pertama kalinya, kita akan mencoba **mengikuti satu case dari awal sampai akhir secara benar**, bukan sekadar mengumpulkan command.
