# 🛡️ DAY 11 — WINDOWS AUTHENTICATION INVESTIGATION

### 4624 • 4625 • Logon Type • Account • Caller Process • Correlation

Day 10 kita sudah belajar:

> **Event Log → Time → Timeline → Correlation**

Sekarang kita fokus pada satu jenis event yang sangat penting di SOC:

> **Windows Authentication / Logon Events**

Kita akan belajar bukan cuma arti `4624` dan `4625`, tetapi **menginvestigasi satu authentication event sampai menemukan process yang terlibat**.

Ini mulai masuk ke pola yang kamu minta:

```text
ALERT
 ↓
IDENTIFY
 ↓
EXTRACT EVIDENCE
 ↓
CORRELATE
 ↓
INVESTIGATE PROCESS
 ↓
INVESTIGATE FILE
 ↓
ASSESSMENT
```

---

# 1. REMINDER DAY 10

Sebelum masuk materi baru, kita selesaikan sedikit yang masih salah.

### Reminder 1 — Service ≠ Process

```text
Service
   ≠
Process
```

Service adalah background component yang dikelola **Service Control Manager**.

Process adalah instance program yang sedang berjalan.

---

### Reminder 2 — Security Context

Secara sederhana:

> **Security Context = identity + groups + privileges yang digunakan saat action/process berjalan.**

Contoh:

```text
User: Buya
```

berbeda context dengan:

```text
User: NT AUTHORITY\SYSTEM
```

---

### Reminder 3 — Current vs Historical

```text
Get-Process
↓
What is happening now?
```

```text
Get-WinEvent
↓
What happened?
```

---

### Reminder 4 — `TimeCreated`

`TimeCreated` pada event berarti:

> **kapan event tersebut terjadi/tercatat**

Bukan otomatis:

> kapan file dibuat.

---

### Reminder 5 — Timeline

Cara paling dasar:

```text
Event
 ↓
ambil TimeCreated
 ↓
urutkan berdasarkan waktu
 ↓
earliest → latest
```

Command:

```powershell
Sort-Object TimeCreated
```

---

### Reminder 6 — Causality

Kalau:

```text
4688
↓
7045
```

terjadi berdekatan, kita **tidak boleh otomatis menyatakan**:

> 4688 menyebabkan 7045.

Berhubungan secara waktu:

> **Temporal correlation**

bukan otomatis:

> **Causation**

Ini penting.

---

# 2. APA ITU AUTHENTICATION?

**Authentication** secara sederhana adalah proses untuk memverifikasi identity.

Contoh:

```text
User
 ↓
masukkan credential
 ↓
Windows memproses authentication
 ↓
berhasil / gagal
```

Kalau berhasil:

```text
4624
```

Kalau gagal:

```text
4625
```

---

# 3. EVENT 4624

`4624` menunjukkan:

> **An account was successfully logged on.**

Jadi:

```text
4624 = Successful Logon
```

Tetapi ingat:

> `4624` sendiri bukan berarti user adalah legitimate atau attacker tidak ada.

Kita harus melihat context.

---

# 4. EVENT 4625

`4625` menunjukkan:

> **An account failed to log on.**

Jadi:

```text
4625 = Failed Logon
```

Tapi:

```text
4625
≠
Brute Force
≠
Attacker
```

Contoh sederhana:

```text
Buya salah password
↓
4625
```

Itu bisa normal.

---

# 5. LOGON TYPE

Ini adalah bagian penting Day 11.

Pada event 4624/4625, kita dapat melihat:

```text
Logon Type
```

Contoh umum yang akan kita gunakan:

### Logon Type 2

> **Interactive logon**

Secara sederhana: logon secara langsung pada komputer.

### Logon Type 3

> **Network logon**

Secara sederhana: akses melalui network.

Kita belum perlu menghafal semua jenis logon.

Hari ini fokus:

```text
2 = Interactive
3 = Network
```

---

# 6. KENAPA LOGON TYPE PENTING?

Misalnya:

```text
4625
Account: Buya
Logon Type: 2
```

dan:

```text
4625
Account: Buya
Logon Type: 3
```

Keduanya sama-sama failed logon.

Tetapi context-nya berbeda.

Maka investigator bertanya:

```text
What kind of logon?
```

bukan hanya:

```text
Was it failed?
```

---

# 7. EVENT 4625 YANG KAMU TEMUKAN

Pada Day 9/10 kamu pernah menemukan event:

```text
Event ID: 4625
Account: Buya
Logon Type: 2
Failure Reason:
Unknown user name or bad password
```

dan juga:

```text
Caller Process:
msedgewebview2.exe
```

Event tersebut memang muncul dalam data yang kamu berikan sebelumnya. 

Nah, sekarang kita akan **investigasi event seperti itu secara berurutan**.

---

# 8. AUTHENTICATION EVENT STRUCTURE

Saat menemukan 4625, mulai ambil:

```text
TimeCreated
↓
Account
↓
Logon Type
↓
Failure Reason
↓
Caller Process
↓
Caller PID
↓
Source / Network information
```

Mental model:

```text
WHO?
↓
WHAT?
↓
WHEN?
↓
HOW?
↓
FROM WHERE?
```

---

# 9. SUBJECT VS ACCOUNT FOR WHICH LOGON FAILED

Ini mulai agak penting.

Dalam event 4625 kita bisa melihat:

```text
Subject
```

dan:

```text
Account For Which Logon Failed
```

Jangan langsung anggap keduanya sama.

Contoh dari event-mu:

```text
Subject:
Account Name: Buya
```

Kemudian:

```text
Account For Which Logon Failed:
Account Name: Buya
```

Kadang mereka berbeda tergantung jenis activity.

Untuk sekarang fokus pada:

> **Account For Which Logon Failed**

karena itu menjawab:

> account mana yang gagal login.

---

# 10. FAILURE REASON

Misalnya:

```text
Unknown user name or bad password.
```

Artinya authentication request gagal karena username tidak dikenal atau password salah.

Dalam SOC:

```text
One failed logon
```

belum cukup untuk membuat kesimpulan attack.

Kita perlu melihat:

```text
Frequency
Timing
Account
Source
Process
Pattern
```

---

# 11. CALLER PROCESS

Ini bagian yang menghubungkan Day 9 dengan Day 7.

Misalnya event menunjukkan:

```text
Caller Process:
C:\Windows\System32\svchost.exe
```

atau:

```text
Caller Process:
C:\Program Files\...\msedgewebview2.exe
```

Sekarang pertanyaannya:

> **Process apa yang melakukan/request authentication tersebut?**

Kita bisa mengambil:

```text
Caller Process
Caller Process ID
```

Lalu masuk ke Process Investigation.

---

# 12. HEX PID

Kadang Event Log tidak memberikan PID dalam decimal.

Contoh:

```text
Caller Process ID:
0x5f60
```

Itu adalah **hexadecimal representation**.

Kita perlu mengubahnya menjadi decimal agar bisa digunakan pada command seperti:

```powershell
Get-Process -Id <PID>
```

Gunakan:

```powershell
[Convert]::ToInt32("5f60",16)
```

Contoh:

```text
0x5f60
   ↓
5f60
   ↓
decimal PID
```

Tidak perlu takut dengan hexadecimal. Kita akan belajar perlahan.

---

# 13. PRAKTIK 1 — CARI EVENT 4625

Jalankan:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

Pilih **satu event**.

Lalu cari:

```text
TimeCreated:
Account:
Logon Type:
Failure Reason:
Caller Process ID:
Caller Process Name:
Source Network Address:
```

---

# 14. PRAKTIK 2 — KONVERSI CALLER PID

Misalnya event memberikan:

```text
Caller Process ID:
0x5f60
```

Jalankan:

```powershell
[Convert]::ToInt32("5f60",16)
```

Catat hasil decimal-nya.

---

# 15. PRAKTIK 3 — CARI PROCESS BERDASARKAN PID

Setelah mendapatkan decimal PID:

```powershell
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Contoh:

```powershell
Get-Process -Id 24416 -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Kalau process sudah tidak ada:

> **Itu bukan error investigation.**

Bisa saja process sudah berhenti.

Ingat:

```text
Historical Event
      ↓
Process mungkin sudah exit
```

---

# 16. PRAKTIK 4 — INVESTIGASI PROCESS TERSEBUT

Kalau process masih ada:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Sekarang kita memiliki:

```text
Event
 ↓
Caller PID
 ↓
Process
 ↓
PPID
 ↓
ExecutablePath
 ↓
CommandLine
```

Ini adalah **correlation**.

---

# 17. PRAKTIK 5 — USER CONTEXT

Kemudian:

```powershell
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Sekarang:

```text
Event
 ↓
Caller Process
 ↓
PID
 ↓
User
```

---

# 18. PRAKTIK 6 — FILE INVESTIGATION

Kalau executable path berhasil ditemukan:

```text
C:\Program Files\Example\example.exe
```

baru kita gunakan ilmu Day 6:

### Metadata

```powershell
Get-Item "C:\Program Files\Example\example.exe" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### Digital Signature

```powershell
Get-AuthenticodeSignature "C:\Program Files\Example\example.exe"
```

### SHA-256

```powershell
Get-FileHash "C:\Program Files\Example\example.exe" -Algorithm SHA256
```

Perhatikan:

> Kita **tidak selalu langsung melakukan SHA-256**.

Kita melakukan berdasarkan **object yang sedang kita investigasi**.

---

# 19. FULL INVESTIGATION CHAIN DAY 11

Ini pola penting hari ini:

```text
4625
Failed Logon
   ↓
Account
   ↓
Logon Type
   ↓
Failure Reason
   ↓
Caller Process
   ↓
Caller PID
   ↓
Convert HEX → Decimal
   ↓
Process Investigation
   ↓
User Context
   ↓
Parent Process
   ↓
ExecutablePath
   ↓
CommandLine
   ↓
File Investigation
   ↓
Metadata
   ↓
Digital Signature
   ↓
SHA-256
```

🔥 Nah, ini sudah mulai menjadi **SOC investigation workflow nyata**.

---

# 20. PRAKTIK END-TO-END DAY 11

Sekarang kita buat **satu investigation lengkap**, bukan lagi praktik terpisah.

Ambil satu `4625` dari laptopmu.

### STEP 1 — Alert

```text
Event ID:
4625
```

---

### STEP 2 — Identify

Catat:

```text
Time:
Account:
Logon Type:
Failure Reason:
Caller Process:
Caller PID:
Source:
```

---

### STEP 3 — Caller PID

Kalau PID berbentuk:

```text
0x....
```

konversi:

```powershell
[Convert]::ToInt32("<HEX>",16)
```

---

### STEP 4 — Process

Cari:

```powershell
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

---

### STEP 5 — Process Context

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

---

### STEP 6 — Parent

Kalau mendapatkan:

```text
PPID = 5000
```

cari:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 5000" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

---

### STEP 7 — File

Kalau executable path tersedia:

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

---

### STEP 8 — Signature

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

---

### STEP 9 — Hash

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

---

# 21. ANALYSIS

Sekarang kamu punya banyak evidence.

Jangan langsung menulis:

> malicious.

Pisahkan:

### Observation

Apa yang benar-benar terlihat?

Contoh:

```text
4625 occurred.
Account Buya.
Logon Type 2.
Caller Process X.
```

### Correlation

Contoh:

```text
4625
 ↓
Caller Process X
 ↓
Process started by Y
```

### Hypothesis

Contoh:

> Authentication failure may have been generated by an application running under the user context.

### Evidence Gap

Apa yang belum kita ketahui?

---

# 22. ASSESSMENT

Untuk level kita sekarang, gunakan:

```text
Benign-looking
Needs Investigation
Insufficient Evidence
```

Jangan memaksakan:

```text
MALWARE
```

hanya karena ada satu suspicious indicator.

---

# 23. CHALLENGE DAY 11 🧠

Kasus:

```text
Time:
20:15:32

Event ID:
4625

Account:
Buya

Logon Type:
2

Failure Reason:
Unknown user name or bad password.

Caller Process ID:
0x3000

Caller Process:
C:\Program Files\Example\App.exe

Source:
127.0.0.1
```

### Q1

Apa yang terjadi?

### Q2

Account siapa yang mengalami failed logon?

### Q3

Apa arti Logon Type `2`?

### Q4

Apa arti:

```text
Caller Process ID:
0x3000
```

### Q5

Bagaimana mengubah `0x3000` menjadi decimal?

### Q6

Setelah mendapatkan decimal PID, command apa yang digunakan untuk mencari process + user?

### Q7

Command apa yang digunakan untuk mencari:

```text
PID
PPID
ExecutablePath
CommandLine
```

?

### Q8

Kalau process tersebut sudah berhenti, apakah kita gagal melakukan investigation?

### Q9

Kalau executable path ditemukan, sebutkan tiga investigation Day 6 yang bisa dilakukan terhadap file.

### Q10

Apakah satu Event 4625 cukup untuk mengatakan brute-force?

### Q11

Mengapa `Source = 127.0.0.1` menarik untuk dicatat?

Jangan langsung menyimpulkan malicious.

### Q12

Buat investigation chain dari event tersebut:

```text
4625
↓
?
↓
?
↓
?
```

sampai file investigation.

---

# 24. MINI SOC CASE — END-TO-END 🔥

Sekarang ini bagian yang paling sesuai dengan permintaanmu sebelumnya.

Gunakan **satu Event 4625 nyata dari laptopmu**.

Buat:

```text
=== DAY 11 AUTHENTICATION INVESTIGATION ===

ALERT
Event ID:
Time:

1. AUTHENTICATION

Account:
Logon Type:
Failure Reason:

Caller Process:
Caller Process ID:
Source Network Address:


2. PID CORRELATION

Caller PID HEX:
Caller PID DECIMAL:


3. PROCESS INVESTIGATION

Process:
PID:
User:

PPID:
Parent Process:

ExecutablePath:

CommandLine:


4. FILE INVESTIGATION

Name:
Length:
CreationTime:
LastWriteTime:
LastAccessTime:


5. DIGITAL SIGNATURE

Status:
Signer:


6. SHA-256

Hash:


7. TIMELINE

Time:
Event:

Time:
Event:

Time:
Event:


8. OBSERVATION

Apa yang benar-benar terlihat?


9. CORRELATION

Apa saja evidence yang kemungkinan berkaitan?


10. EVIDENCE GAP

Apa yang masih belum diketahui?


11. ASSESSMENT

Benign-looking /
Needs Investigation /
Insufficient Evidence

Reason:
```

🔥 **Ini bukan simulasi abstrak lagi.**

Kita akan mengambil satu event asli dari Windows kamu dan menelusurinya **sampai sejauh evidence memungkinkan**.

Kalau process-nya sudah tidak ada, kita tetap lanjut menggunakan **historical evidence** yang tersedia. Jangan membuat data yang tidak ada.

---

# 25. ATTACK METHOD — FAILED LOGON PATTERN

Konsep attack yang mulai kita kenal:

### Brute Force

Secara sederhana:

```text
Repeated authentication attempts
        ↓
many failures
        ↓
possible credential attack
```

Tetapi SOC tidak hanya melihat:

```text
4625
```

SOC mencari pattern:

```text
4625
4625
4625
4625
4625
...
```

Kemudian:

```text
Same account?
Same source?
Same time window?
Same process?
```

Baru kita mulai mempunyai dasar untuk hypothesis.

---

# 26. DEFENSE METHOD

Defensive monitoring dapat menggunakan:

```text
Authentication Logs
+
Process Logs
+
User Context
+
Network Information
```

Misalnya:

```text
4625
 ↓
Account: Buya
 ↓
Caller: App.exe
 ↓
Process
 ↓
Executable
```

Jadi authentication event tidak berdiri sendiri.

---

# 27. COMMANDS DAY 11

### 1. Cari 4625

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
```

### 2. Konversi HEX PID → Decimal

```powershell
[Convert]::ToInt32("5f60",16)
```

Ganti `"5f60"` dengan hexadecimal PID yang kamu dapat.

### 3. Cari Process + User

```powershell
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

### 4. Cari Full Process Context

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### 5. Cari Parent Process

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

### 6. File Metadata

```powershell
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
```

### 7. Digital Signature

```powershell
Get-AuthenticodeSignature "<ExecutablePath>"
```

### 8. SHA-256

```powershell
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
```

---

# 🧠 DAY 11 COMMAND MAP

```text
Get-WinEvent
      ↓
Find Authentication Event
      ↓
4625
      ↓
Caller Process ID
      ↓
HEX → DECIMAL
      ↓
Get-Process
      ↓
User
      ↓
Get-CimInstance Win32_Process
      ↓
PID / PPID / Path / CommandLine
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

# 🎯 TARGET DAY 11

Hari ini targetmu **bukan menghafal semua Event ID**.

Targetnya adalah bisa melakukan:

```text
AUTHENTICATION EVENT
        ↓
WHO?
        ↓
WHEN?
        ↓
WHAT?
        ↓
CALLER PROCESS
        ↓
PID
        ↓
USER
        ↓
PARENT
        ↓
EXECUTABLE
        ↓
FILE
        ↓
SIGNATURE
        ↓
HASH
        ↓
ASSESSMENT
```

Ini sudah mulai menjadi **investigation chain** yang sebenarnya.

---

## 📌 SATU HAL PENTING

Dari hasil Day 10 kamu, kita sebenarnya punya contoh nyata `4625`:

```text
2026-09-28 19:15:33
Account: Buya
Logon Type: 2
Failure: Unknown user name or bad password
Caller:
msedgewebview2.exe
```



Jadi kamu **tidak perlu menunggu muncul kasus baru**. Untuk Mini SOC Day 11, kamu bisa mulai dari event tersebut dan mencoba mengikuti chain:

```text
4625
↓
Caller PID
↓
PID decimal
↓
Process
↓
User
↓
Parent
↓
ExecutablePath
↓
Metadata
↓
Signature
↓
SHA-256
↓
Assessment
```

**Jawab Challenge + kerjakan Mini SOC Case end-to-end.** Untuk Active Recall, aku sengaja hanya memasukkan materi yang sebelumnya masih salah atau lupa, sesuai aturanmu; yang sudah konsisten benar tidak akan kita bombardir setiap hari.
