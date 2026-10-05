# 🛡️ DAY 13 — SYSMON PROCESS INVESTIGATION

### ProcessGuid • ParentImage • ParentCommandLine • IntegrityLevel • Full Process Chain

Hari ini kita **tidak lompat jauh dari Day 12**.

Day 12 kamu sudah berhasil mendapatkan Sysmon Event ID 1:

```text
Process Create
        ↓
ProcessId
Image
CommandLine
User
Hash
ParentProcessId
ParentImage
ParentCommandLine
```

Bahkan dari laptopmu sendiri, Sysmon mencatat `MicrosoftEdgeUpdate.exe` berjalan sebagai `SYSTEM`, memiliki parent `svchost.exe`, command line scheduler, SHA-256, dan `ParentCommandLine`. 

Hari ini kita belajar **membaca telemetry tersebut sebagai investigator**, bukan sekadar melihat output.

---

# 🎯 TARGET DAY 13

Setelah hari ini, ketika mendapatkan:

```text
Sysmon Event ID 1
```

kamu mulai tahu **mana yang harus diperiksa dulu, mana yang menyusul, dan kapan harus lanjut ke file investigation**.

Alur yang kita latih:

```text
EVENT
 ↓
TIME
 ↓
PROCESS
 ↓
USER
 ↓
INTEGRITY LEVEL
 ↓
PARENT
 ↓
PARENT COMMAND
 ↓
COMMANDLINE
 ↓
EXECUTABLE
 ↓
HASH
 ↓
FILE
 ↓
SIGNATURE
 ↓
CORRELATION
 ↓
ASSESSMENT
```

---

# 1. SYSMON EVENT ID 1

`Event ID 1` = **Process Create**.

Artinya Sysmon merekam bahwa sebuah process dibuat.

Contoh dari sistemmu:

```text
Image:
C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe

ProcessId:
22968

CommandLine:
"...MicrosoftEdgeUpdate.exe" /ua /installsource scheduler

User:
NT AUTHORITY\SYSTEM

ParentImage:
C:\Windows\System32\svchost.exe

ParentProcessId:
2128
```



---

# 2. JANGAN BACA FIELD SECARA ACAK

Ini penting untuk cara berpikirmu.

Ketika mendapat alert process, jangan langsung:

> "Hash-nya apa?"

Gunakan urutan:

### STEP 1 — WHEN

```text
UtcTime / TimeCreated
```

Pertanyaan:

> Kapan process dibuat?

---

### STEP 2 — WHAT

```text
Image
```

Pertanyaan:

> Process apa?

---

### STEP 3 — WHO

```text
User
```

Pertanyaan:

> Siapa security context yang menjalankan process?

---

### STEP 4 — PRIVILEGE

```text
IntegrityLevel
```

Pertanyaan:

> Process berjalan pada security integrity level apa?

---

### STEP 5 — WHO STARTED IT?

```text
ParentImage
ParentProcessId
```

Pertanyaan:

> Siapa parent process?

---

### STEP 6 — HOW?

```text
CommandLine
ParentCommandLine
```

Pertanyaan:

> Bagaimana process dijalankan?

> Dengan argument apa?

> Parent menjalankan dirinya dengan command apa?

---

### STEP 7 — WHERE?

```text
Image
```

Pertanyaan:

> Executable berada di mana?

---

### STEP 8 — FINGERPRINT

```text
Hashes
```

Pertanyaan:

> Fingerprint file-nya apa?

---

# 3. PROCESSGUID

Sekarang konsep baru.

Di Sysmon kamu melihat:

```text
ProcessGuid:
{f1261a38-9ecc-6ac3-7eb0-000000001d00}
```



**ProcessGuid** adalah identifier yang membantu membedakan instance process secara lebih unik daripada hanya mengandalkan PID.

Kenapa ini penting?

Karena:

```text
Process A
PID 5000
↓
exit

Process B
PID 5000
↓
running later
```

PID bisa digunakan kembali.

Maka:

```text
PID
```

tidak selalu cukup untuk membedakan dua process instance dari waktu berbeda.

Sysmon menyediakan:

```text
ProcessGuid
```

untuk membantu correlation.

Hari ini kamu belum perlu menghafal struktur GUID-nya.

Cukup:

> **PID = process ID yang dapat berubah/reuse.**

> **ProcessGuid = identifier yang lebih cocok untuk correlation terhadap instance process di Sysmon.**

---

# 4. PARENT PROCESS

Day 7 kita sudah belajar:

```text
PID
PPID
Parent Process
```

Sekarang Sysmon memberi:

```text
ParentProcessId
ParentImage
ParentCommandLine
ParentUser
```

Contoh dari hasilmu:

```text
ParentProcessId:
2128

ParentImage:
C:\Windows\System32\svchost.exe

ParentCommandLine:
C:\WINDOWS\system32\svchost.exe -k netsvcs -p -s Schedule
```



Sekarang kita tidak hanya tahu:

> parent-nya `svchost.exe`.

Kita juga tahu:

> **svchost.exe dijalankan dengan command apa.**

Ini sangat berguna untuk investigation.

---

# 5. PARENT ≠ AUTOMATICALLY SAFE

Misalnya:

```text
svchost.exe
   ↓
suspicious.exe
```

atau:

```text
explorer.exe
   ↓
powershell.exe
```

Relationship tersebut **tidak otomatis malicious**.

Parent-child relationship hanyalah:

> **contextual evidence**

Baru kita tanya:

```text
Apakah relationship-nya expected?
CommandLine-nya normal?
User-nya masuk akal?
Path-nya expected?
File legitimate?
Network behavior bagaimana?
```

---

# 6. INTEGRITY LEVEL

Day 12 kamu menemukan:

```text
IntegrityLevel:
System
```



**Integrity Level** menunjukkan level mandatory integrity dari process/security token.

Untuk mental model dasar:

```text
Low
Medium
High
System
```

Semakin tinggi levelnya, secara umum process mempunyai **security context dengan kemampuan akses yang lebih tinggi** dibanding level rendah.

Tetapi:

> **System ≠ malicious**

Karena banyak Windows service memang berjalan sebagai SYSTEM.

---

# 7. USER VS INTEGRITY LEVEL

Jangan dicampur.

Misalnya:

```text
User:
NT AUTHORITY\SYSTEM

IntegrityLevel:
System
```

Kita memiliki dua informasi:

```text
WHO?
→ User / account

WHAT PRIVILEGE CONTEXT?
→ Integrity Level
```

---

# 8. COMMAND LINE VS PARENT COMMAND LINE

Contoh:

### Child

```text
CommandLine:
something.exe -update
```

### Parent

```text
ParentCommandLine:
powershell.exe -File C:\Temp\a.ps1
```

Sekarang kita bisa bertanya:

> Bagaimana parent membuat child tersebut?

Ini jauh lebih berguna dibanding hanya melihat:

```text
something.exe
```

---

# 9. PRAKTIK 1 — AMBIL SYSMON PROCESS CREATION

Jalankan:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message
```

Pilih **satu event yang menarik**.

Jangan pilih berdasarkan nama yang terlihat "seram".

Pilih yang informasinya menarik untuk dianalisis.

---

# 10. PRAKTIK 2 — IDENTIFY FIELD

Dari event yang kamu pilih, catat:

```text
=== PROCESS IDENTITY ===

Time:
ProcessGuid:
ProcessId:
Image:
User:
IntegrityLevel:
CommandLine:

ParentProcessGuid:
ParentProcessId:
ParentImage:
ParentCommandLine:
ParentUser:

Hashes:
```

Kalau field tertentu tidak muncul, tulis:

```text
Not present / unavailable
```

Jangan mengarang.

---

# 11. PRAKTIK 3 — PROCESS CHAIN

Misalnya hasil:

```text
ParentImage:
C:\Windows\System32\cmd.exe

ParentCommandLine:
cmd.exe /c notepad.exe

Image:
C:\Windows\System32\notepad.exe
```

Kamu buat:

```text
cmd.exe
   ↓
notepad.exe
```

Kemudian jawab:

> **Why was notepad.exe started?**

Jawab berdasarkan evidence, bukan tebakan.

---

# 12. PRAKTIK 4 — BUAT PROCESS CHAIN SENDIRI

Sekarang kita sengaja membuat process chain **benign**.

Jalankan:

```cmd
cmd.exe /c notepad.exe
```

Notepad akan terbuka.

**Jangan langsung ditutup.**

Sekarang cari Sysmon Event ID 1 terbaru:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message
```

Cari event `notepad.exe` yang baru dibuat.

Kemudian coba identifikasi:

```text
Process:
Parent:
ParentCommandLine:
User:
CommandLine:
```

Tujuannya supaya kamu melihat sendiri:

```text
cmd.exe
   ↓
notepad.exe
```

bukan hanya belajar dari contoh.

---

# 13. PRAKTIK 5 — CARI PROCESS GUID

Dari event tersebut, cari:

```text
ProcessGuid
ProcessId
ParentProcessGuid
ParentProcessId
```

Kemudian jawab:

> Kenapa Sysmon memberikan `ProcessGuid` selain `ProcessId`?

Gunakan konsep Day 13.

---

# 14. PRAKTIK 6 — FILE INVESTIGATION

Setelah menemukan:

```text
Image:
C:\Windows\System32\notepad.exe
```

ikuti alur:

### Metadata

```powershell
Get-Item "<Image>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### Signature

```powershell
Get-AuthenticodeSignature "<Image>"
```

### Hash

```powershell
Get-FileHash "<Image>" -Algorithm SHA256
```

Jangan berhenti di hash.

Tanyakan:

```text
Is the path expected?
Is the signature valid?
Is the hash known?
Does the command line make sense?
Is the parent expected?
Is the user expected?
```

---

# 15. PRAKTIK 7 — CORRELATION DENGAN PROCESS STATE

Setelah selesai investigation Notepad, jalankan:

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Bandingkan hasilnya dengan Sysmon.

Sekarang kamu mempunyai dua sumber:

```text
LIVE
↓
Get-CimInstance

HISTORICAL
↓
Sysmon
```

Apakah informasinya sama?

Apakah PID-nya sama?

Apakah parent-nya sama?

Ini latihan **correlation**.

---

# 16. ⚠️ PERHATIKAN PID

Misalnya Sysmon mencatat:

```text
ProcessId:
12388
```

Kemudian kamu query beberapa menit kemudian dan:

```text
notepad.exe
PID:
12388
```

Masih sama.

Bagus.

Tetapi kalau process mati dan dijalankan lagi:

```text
notepad.exe
PID:
15000
```

itu normal.

Jangan berpikir:

> "Kok PID berubah? Berarti malware."

Tidak.

PID dapat berubah antar process instance.

---

# 17. FULL INVESTIGATION WORKFLOW DAY 13

Ini bagian yang paling penting.

Saat mendapatkan Sysmon Process Creation Alert:

```text
              ALERT
                ↓
         EVENT ID 1
                ↓
              TIME
                ↓
            PROCESS
                ↓
              USER
                ↓
        INTEGRITY LEVEL
                ↓
             PARENT
                ↓
      PARENT COMMAND LINE
                ↓
          COMMAND LINE
                ↓
         EXECUTABLE PATH
                ↓
          FILE EXISTS?
                ↓
            METADATA
                ↓
        DIGITAL SIGNATURE
                ↓
             HASH
                ↓
          NETWORK
                ↓
       CORRELATE EVENTS
                ↓
         EVIDENCE GAP
                ↓
           ASSESSMENT
                ↓
           CONCLUSION
```

### Jangan terbalik.

Misalnya jangan:

```text
Hash
↓
Threat Intel
↓
baru cari process
```

Karena kita bahkan belum tahu **context process-nya**.

---

# 18. INVESTIGATOR DECISION TREE

Sekarang aku mau kamu mulai belajar **memutuskan langkah berikutnya**.

Misalnya:

### Case A

```text
Process:
notepad.exe

Path:
C:\Windows\System32\notepad.exe

Parent:
explorer.exe

User:
Buya

CommandLine:
notepad.exe
```

Investigator:

> Looks normal → verify file → close/continue monitoring.

---

### Case B

```text
Process:
unknown.exe

Path:
C:\Users\...\Temp\unknown.exe

Parent:
powershell.exe

CommandLine:
unknown.exe -update

User:
Buya
```

Investigator:

> Needs investigation.

Lanjut:

```text
Metadata
↓
Signature
↓
Hash
↓
Network
↓
Threat Intelligence
```

Jadi tools digunakan **berdasarkan kebutuhan evidence**.

---

# 19. OBSERVATION → HYPOTHESIS → TEST

Mulai hari ini gunakan pola:

### Observation

```text
unknown.exe
Temp path
PowerShell parent
```

### Hypothesis

> Process mungkin suspicious.

### Test

```text
Check metadata
Check signature
Check hash
Check network
```

### Result

```text
Evidence mendukung?
Evidence menyangkal?
Evidence masih kurang?
```

### Assessment

```text
Benign-looking
Needs Investigation
Insufficient Evidence
```

🔥 Inilah pola investigator.

---

# 20. CHALLENGE DAY 13 🧠

Kamu menemukan Sysmon Event ID 1:

```text
Process:
powershell.exe

ProcessId:
9000

ProcessGuid:
{AAA}

Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

User:
DESKTOP\Buya

IntegrityLevel:
High

CommandLine:
powershell.exe -File C:\Users\Buya\AppData\Local\Temp\update.ps1

ParentProcessId:
8000

ParentImage:
C:\Windows\explorer.exe

ParentCommandLine:
C:\Windows\explorer.exe

ParentUser:
DESKTOP\Buya

SHA256:
ABC123...
```

### Q1

Apa object yang sedang kita investigasi?

### Q2

Apa yang harus kamu lihat **pertama kali**?

### Q3

Siapa user yang menjalankan process?

### Q4

Apa arti `IntegrityLevel = High`?

### Q5

Siapa parent process?

### Q6

Apa yang membuat command line ini perlu diperiksa lebih lanjut?

### Q7

Apakah `High IntegrityLevel` berarti malicious?

### Q8

Apakah `explorer.exe → powershell.exe` otomatis malicious?

### Q9

Setelah melihat command line dan parent, **apa langkah investigation berikutnya?**

Pilih urutannya sendiri.

### Q10

File apa yang sekarang menjadi object investigation?

### Q11

Sebutkan pemeriksaan file yang akan kamu lakukan.

### Q12

Kalau process sudah mati saat kamu mencoba `Get-Process -Id 9000`, apakah investigation selesai?

### Q13

Bagaimana Sysmon tetap dapat membantu?

### Q14

Apa perbedaan:

```text
PID
ProcessGuid
```

### Q15

Buat investigation chain lengkap dari alert sampai assessment.

---

# 21. MINI SOC INVESTIGATION — DAY 13 🔥

Sekarang kita lakukan **real practical investigation**, bukan simulasi.

Gunakan process chain benign yang tadi kita buat:

```text
cmd.exe
   ↓
notepad.exe
```

Buat report:

```text
=== DAY 13 SYSMON PROCESS INVESTIGATION ===

ALERT
Event ID:
Time:


PROCESS
Name:
PID:
ProcessGuid:
User:
IntegrityLevel:


PARENT
Parent PID:
Parent Process:
ParentProcessGuid:
Parent User:
Parent CommandLine:


CURRENT STATE
Process masih berjalan?
PID sekarang:


COMMAND
CommandLine:


FILE
ExecutablePath:
File Exists:


METADATA
Name:
Length:
CreationTime:
LastWriteTime:
LastAccessTime:


DIGITAL SIGNATURE
Status:
Signer:


SHA-256
Hash:


CORRELATION

Sysmon mengatakan:
...

Get-CimInstance mengatakan:
...


ANALYSIS

Observation:
...

Hypothesis:
...

Evidence yang mendukung:
...

Evidence yang belum tersedia:
...


ASSESSMENT

Benign-looking /
Needs Investigation /
Insufficient Evidence

Reason:
...
```

### Jangan memaksakan Threat Intelligence.

Untuk Notepad, kita **belum perlu browsing hash ke internet** pada Day 13.

Tujuan utamanya adalah belajar:

> **bagaimana investigator bergerak dari satu evidence ke evidence berikutnya.**

---

# 22. ATTACK METHOD — PROCESS CHAIN ABUSE

Attacker dapat mencoba menjalankan process melalui parent yang legitimate.

Misalnya secara konsep:

```text
Office
  ↓
PowerShell
  ↓
Script
```

atau:

```text
Browser
  ↓
Process
```

Yang membuat investigation menarik bukan nama process semata, tetapi:

```text
Parent
+
CommandLine
+
User
+
Path
+
File
+
Timeline
```

---

# 23. DEFENSE METHOD

Sysmon Process Creation membantu defender mendapatkan:

```text
Process
User
Parent
CommandLine
Image
Hash
Time
```

Kemudian SOC dapat membuat detection berdasarkan kombinasi kondisi.

Contoh konsep:

```text
PowerShell
+
suspicious parent
+
Temp path
+
suspicious command line
```

lebih informatif daripada:

```text
PowerShell exists
```

---

# 💻 DAY 13 — COMMANDS

### 1. Sysmon Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message
```

### 2. Membuat process chain benign

```cmd
cmd.exe /c notepad.exe
```

### 3. Live Notepad investigation

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'notepad.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### 4. User context

```powershell
Get-Process -Name notepad -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

### 5. Parent process

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### 6. File metadata

```powershell
Get-Item "<Image>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### 7. Digital Signature

```powershell
Get-AuthenticodeSignature "<Image>"
```

### 8. SHA-256

```powershell
Get-FileHash "<Image>" -Algorithm SHA256
```

---

# 🧠 COMMAND MAP DAY 13

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

# 🔄 ACTIVE RECALL DAY 13

Sesuai aturan kita, aku hanya mengambil **yang masih kurang dari Day 12**, lalu satu konsep baru.

### Q1

Apa itu **Security Context**?

### Q2

`TimeCreated` pada Event Log menunjukkan apa?

### Q3

Apa perbedaan:

```text
Current State
Historical Evidence
```

### Q4

Kenapa process yang sudah mati **tidak selalu berarti investigation selesai**?

### Q5

Apa fungsi utama `ProcessGuid` pada Sysmon?

---

# 📌 MILESTONE 1 TAHUN — DAY 13

### DAY 13 — Sysmon Process Investigation

* Sysmon Event ID 1 — Process Creation
* ProcessGuid
* ProcessId
* ParentProcessId
* ParentProcessGuid
* ParentImage
* ParentCommandLine
* ParentUser
* IntegrityLevel
* Image / Executable
* Process CommandLine
* Historical process telemetry
* Live process correlation
* Process chain
* Process → User → Parent → CommandLine → File
* Observation → Hypothesis → Test → Assessment
* PID dapat berubah/reuse
* ProcessGuid membantu correlation process instance
* **Investigation tidak berhenti hanya karena process sudah mati**

---

# 🎯 TARGET DAY 13

Aku ingin kamu mulai punya refleks seperti ini ketika melihat alert:

```text
             ALERT
                ↓
       "Apa sebenarnya ini?"
                ↓
              TIME
                ↓
            PROCESS
                ↓
              USER
                ↓
           INTEGRITY
                ↓
             PARENT
                ↓
        PARENT COMMAND
                ↓
          COMMANDLINE
                ↓
             PATH
                ↓
              FILE
                ↓
       SIGNATURE / HASH
                ↓
            NETWORK
                ↓
          CORRELATION
                ↓
          EVIDENCE GAP
                ↓
           ASSESSMENT
```

**Kerjakan Praktik 1–7 + Challenge + Mini SOC Investigation + Active Recall.**

Dan kali ini, jangan hanya kirim hasil command. **Tulis juga langkah investigasimu satu per satu:**
`“Aku melihat X → karena itu aku mencari Y → hasilnya Z → karena hasil Z aku lanjut mencari A.”`

Itu bagian terpenting Day 13 karena kita sedang melatih **cara berpikir investigator**, bukan sekadar hafalan command.
