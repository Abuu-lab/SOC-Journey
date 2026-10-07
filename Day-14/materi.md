# Day 14 — Materi

**Security Context, Integrity Level & Svchost**

## 🛡️ DAY 14 — SECURITY CONTEXT, INTEGRITY LEVEL & SVCHOST INVESTIGATION

**Target hari ini:** kamu mampu menjawab bukan hanya *“process ini apa?”*, tetapi juga:

> **Siapa yang menjalankan? Dengan security level apa? Siapa parent-nya? Service apa yang berada di balik `svchost.exe`? Dan apakah evidence yang kita lihat current atau historical?**

Hari ini tidak mengejar banyak command. Kita memperkuat **cara berpikir investigator**.

---

## 🧠 BAGIAN 1 — ACTIVE RECALL DAY 13

Jangan buka catatan dulu.

Jawab dengan pemahamanmu sendiri.

#### Q1

Apa perbedaan:

```text
PID
ProcessGuid
```

---

#### Q2

Kenapa SOC tidak boleh hanya mengandalkan PID untuk correlation?

---

#### Q3

Apa arti:

```text
User: DESKTOP-C7BHMKL\Buya
IntegrityLevel: Medium
```

Jelaskan **User** dan **IntegrityLevel** secara terpisah.

---

#### Q4

Apakah:

```text
IntegrityLevel: High
```

berarti process tersebut malicious?

Kenapa?

---

#### Q5

Apa arti command line:

```text
svchost.exe -k CameraMonitor
```

Apa kira-kira fungsi `-k` di sini?

Jangan menebak bahwa `CameraMonitor` pasti berarti process sedang menggunakan kamera.

---

#### Q6

Jelaskan hubungan ini:

```text
services.exe
      ↓
svchost.exe
      ↓
Windows Service
```

---

#### Q7

Bagaimana cara mengetahui **service apa yang menggunakan PID tertentu**, misalnya PID `6920`?

Tulis command PowerShell yang akan kamu gunakan.

---

#### Q8

Apa arti:

```text
File Exists
```

dan command apa yang digunakan untuk mengeceknya?

---

#### Q9

Apa perbedaan:

```text
Historical Evidence
Current State
```

Berikan masing-masing satu contoh.

---

#### Q10

Command apa yang kamu gunakan untuk mencari **Sysmon Event ID 1**?

---

#### Q11

Command apa yang kamu gunakan untuk melihat **process yang sedang berjalan sekarang** berdasarkan PID?

---

#### Q12

Lengkapi:

```text
Security Context = User + ______ + ______
```

---

## 📚 BAGIAN 2 — SECURITY CONTEXT

Sekarang kita masuk konsep utama hari ini.

Saat SOC melihat sebuah process, jangan hanya bertanya:

> “Ini process apa?”

Tanyakan:

> **“Process ini berjalan sebagai siapa dan dengan security context seperti apa?”**

Secara sederhana:

```text
PROCESS
   │
   ├── User
   ├── Groups
   └── Privileges
```

Itulah bagian dari **Security Context** yang perlu kamu pahami.

Contoh:

```text
Process: powershell.exe
User: Buya
Groups: ...
Privileges: ...
```

Dua process bisa sama-sama bernama:

```text
powershell.exe
```

tetapi context-nya bisa berbeda.

Misalnya:

```text
powershell.exe
User = Buya
Integrity = Medium
```

vs

```text
powershell.exe
User = Buya
Integrity = High
```

Process name sama.

Tetapi **security context berbeda**.

Inilah yang membuat User dan IntegrityLevel menjadi evidence penting.

---

## 🔐 BAGIAN 3 — INTEGRITY LEVEL

Kamu tadi masih bingung dengan IntegrityLevel.

Kita sederhanakan.

**Integrity Level** memberi gambaran tentang tingkat kepercayaan/security level Windows terhadap process dalam security model-nya.

Contoh level yang mungkin kamu lihat:

```text
Low
Medium
High
System
```

Gambaran kasar:

```text
Low
 ↓
Medium
 ↓
High
 ↓
System
```

Semakin tinggi bukan berarti semakin jahat.

Contoh normal:

```text
Windows system process
→ System
```

Itu sangat normal.

Begitu juga aplikasi yang dijalankan **Run as administrator** dapat berjalan dengan level lebih tinggi.

Jadi SOC harus berpikir:

```text
High
   ↓
Interesting
   ↓
Investigate
```

bukan:

```text
High
   ↓
Malware
```

---

## 🔎 BAGIAN 4 — USER vs INTEGRITY LEVEL

Ini wajib kamu bisa bedakan.

#### User

Menjawab:

> **WHO?**

Contoh:

```text
User: DESKTOP-C7BHMKL\Buya
```

Artinya process berjalan dalam user context Buya.

#### IntegrityLevel

Menjawab:

> **WHAT SECURITY LEVEL?**

Contoh:

```text
IntegrityLevel: High
```

Berarti process berjalan pada security integrity level High.

Jadi:

```text
User
= siapa

IntegrityLevel
= security level process
```

Jangan dicampur.

---

## 🧬 BAGIAN 5 — PROCESS GUID

Kita perkuat lagi.

Misalnya:

```text
ProcessId: 6920

ProcessGuid:
{f1261a38-f420-6ac4-91ba-000000001d00}
```

PID adalah angka identifier process.

Tetapi:

```text
PID 6920
```

tidak selamanya dimiliki oleh process yang sama.

Process mati → PID dapat digunakan lagi.

Karena itu Sysmon menggunakan:

```text
ProcessGuid
```

untuk membantu tracking **specific process instance**.

Investigator akan lebih mudah melakukan:

```text
Process Create
     ↓
ProcessGuid
     ↓
correlate dengan event lain
```

Jadi mental model:

```text
PID
= nomor identitas process

ProcessGuid
= identitas instance process untuk correlation
```

---

## 🏢 BAGIAN 6 — SVCHOST.EXE

Ini bagian yang tadi membuatmu bertanya:

> “kenapa command-nya argument CameraMonitor?”

Bagus kamu mempertanyakannya.

Karena jangan sekadar menghafal command line.

---

### Apa itu `svchost.exe`?

`svchost.exe` adalah:

> **Host Process for Windows Services**

Artinya satu process dapat menjadi tempat beberapa Windows services berjalan.

Secara sederhana:

```text
services.exe
     ↓
svchost.exe
     ↓
service group
     ↓
service
```

Misalnya:

```text
svchost.exe -k GPSvcGroup
```

atau:

```text
svchost.exe -k CameraMonitor
```

Parameter:

```text
-k
```

digunakan untuk menentukan **service host group**.

Jadi jangan membaca:

```text
-k CameraMonitor
```

sebagai:

> “svchost sedang membuka kamera.”

Itu kesimpulan tanpa evidence.

Sebagai investigator:

```text
Observation:
svchost.exe -k CameraMonitor

↓
Hypothesis:
process ini menjalankan service group tertentu

↓
Test:
cari service berdasarkan PID

↓
Evidence
```

Itulah cara berpikir SOC.

---

## 🔧 BAGIAN 7 — CORRELATE SVCHOST DENGAN SERVICE

Sekarang kita praktik.

Kamu punya:

```text
PID = 6920
```

Gunakan:

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 6920} |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName
```

Perhatikan:

```text
Name
DisplayName
State
StartMode
StartName
ProcessId
PathName
```

Kita sedang membuat correlation:

```text
Sysmon
  ↓
PID 6920
  ↓
svchost.exe
  ↓
Win32_Service
  ↓
service sebenarnya
```

Ini jauh lebih kuat daripada hanya membaca `CameraMonitor`.

---

## 🧪 PRACTICE 1 — INTEGRITY LEVEL

Jalankan:

```powershell
Get-Process -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Cari:

```text
powershell.exe
explorer.exe
svchost.exe
```

Kemudian pilih **satu process** yang menarik dan cari Sysmon Event ID 1-nya.

Gunakan:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 50 |
Where-Object {$_.Message -match "powershell.exe"} |
Select-Object -First 1 -ExpandProperty Message
```

Cari:

```text
User:
IntegrityLevel:
ProcessId:
ProcessGuid:
CommandLine:
ParentImage:
```

#### Tugas

Jawab:

```text
Process:
User:
IntegrityLevel:
PID:
ProcessGuid:
Parent:
CommandLine:

1. User menjelaskan apa?
2. IntegrityLevel menjelaskan apa?
3. Apakah IntegrityLevel tersebut otomatis berarti malicious?
4. Apa evidence tambahan yang akan kamu cari?
```

---

## 🧪 PRACTICE 2 — SVCHOST INVESTIGATION

Sekarang ambil PID `18824` dari hasil Day 13:

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 18824} |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName
```

Kemudian lakukan:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 18824" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Sekarang bandingkan.

Buat correlation:

```text
PID:
Process:
CommandLine:
Parent:
Service Name:
Service DisplayName:
Service State:
StartMode:
StartName:
```

#### Pertanyaan

1. Apa hubungan `services.exe` dengan `svchost.exe`?
2. Apa hubungan `svchost.exe` dengan service yang kamu temukan?
3. Mengapa `-k GPSvcGroup` sendiri belum cukup untuk menyimpulkan malicious?
4. Evidence apa yang menurutmu paling penting berikutnya?

---

## 🧪 PRACTICE 3 — FILE EXISTS

Gunakan executable yang kamu investigasi kemarin:

```powershell
Test-Path "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SoftLandingTask\SoftLandingTask.exe"
```

Kemudian jawab:

```text
Result:
True / False

Apa arti hasil tersebut?

Kalau hasilnya False, apakah otomatis malicious?
Kenapa?
```

---

## 🧪 PRACTICE 4 — CURRENT vs HISTORICAL

Pakai PID yang kamu temukan pada Sysmon.

#### A. Current State

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 18824" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

#### B. Historical Evidence

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 50 |
Where-Object {$_.Message -match "18824"} |
Select-Object TimeCreated, Message
```

#### Pertanyaan

Misalnya:

```text
Get-CimInstance
→ tidak menemukan PID 18824
```

tetapi:

```text
Get-WinEvent
→ menemukan Process Create
```

Apakah investigation harus berhenti?

Jelaskan **kenapa**.

---

## 🕵️ PRACTICE 5 — INVESTIGATOR THINKING

Jangan langsung mencari apakah process malicious.

Gunakan format:

```text
Observation:
Hypothesis:
Test:
Result:
Assessment:
Evidence Gap:
```

Investigator scenario:

```text
ALERT

Process:
powershell.exe

PID:
15740

User:
DESKTOP-C7BHMKL\Buya

IntegrityLevel:
High

CommandLine:
powershell.exe -ExecutionPolicy Bypass -File C:\Users\Buya\AppData\Local\Temp\update.ps1

Parent:
explorer.exe

ParentUser:
DESKTOP-C7BHMKL\Buya

File:
C:\Users\Buya\AppData\Local\Temp\update.ps1

Digital Signature:
Belum diperiksa

SHA256:
Belum diperiksa

Network:
Belum diperiksa
```

#### Jangan langsung jawab “malware”.

Jawab:

**Q1.** Apa saja yang terlihat suspicious?

**Q2.** Apa yang masih normal/possible legitimate?

**Q3.** Apa hypothesis awalmu?

**Q4.** Evidence apa yang kamu periksa berikutnya?

**Q5.** Apa command untuk memeriksa metadata file?

**Q6.** Apa command untuk memeriksa digital signature?

**Q7.** Apa command untuk mendapatkan SHA-256?

**Q8.** Setelah mendapat hash, apakah investigation otomatis selesai?

**Q9.** Evidence apa yang masih ingin kamu cari?

**Q10.** Buat investigation flow dari alert sampai conclusion.

Gunakan pola:

```text
Saya melihat X
→ karena itu saya memeriksa Y
→ hasilnya Z
→ karena hasil Z saya memeriksa A
→ ...
```

---

## 🎯 DAY 14 MINI SOC CASE

Sekarang bagian terpenting.

Kamu mendapatkan Sysmon Event ID 1:

```text
UtcTime:
2026-10-06 14:35:21

Image:
C:\Windows\System32\svchost.exe

ProcessId:
7420

ProcessGuid:
{ABC-123}

User:
NT AUTHORITY\SYSTEM

IntegrityLevel:
System

CommandLine:
C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted

ParentProcessId:
1424

ParentImage:
C:\Windows\System32\services.exe

ParentCommandLine:
C:\Windows\System32\services.exe
```

Kemudian:

```text
Get-CimInstance Win32_Service
```

menghasilkan:

```text
Name:
Dnscache

DisplayName:
DNS Client

State:
Running

StartMode:
Auto

StartName:
NT AUTHORITY\NETWORK SERVICE

ProcessId:
7420

PathName:
C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
```

#### Tugas investigator

Jawab:

**1.** Apa yang kamu observasi?

**2.** Siapa parent process?

**3.** Apa User dari process?

**4.** Apa IntegrityLevel-nya?

**5.** Service apa yang berkaitan dengan PID `7420`?

**6.** Apakah `svchost.exe -k LocalServiceNetworkRestricted` sendiri cukup untuk mengatakan malicious?

**7.** Evidence apa yang mendukung bahwa process ini kemungkinan legitimate?

**8.** Apakah kamu masih akan melakukan additional investigation? Jelaskan.

**9.** Buat final assessment:

```text
Assessment:
Reason:
Confidence:
Evidence:
Evidence Gap:
```

Jangan menggunakan angka probability seperti `70% malicious`. Gunakan bahasa analyst:

```text
Likely Legitimate
Needs Investigation
Suspicious
Likely Malicious
```

dan **jelaskan evidence-nya**.

---

## ✅ PASS RULE DAY 14

**76–100% → PASS → langsung Day 15.**

Kesalahan kecil **tidak mengulang Day 14**. Yang masih lemah akan masuk ke active recall Day 15.

Target hari ini bukan menjadi hafal semua istilah.

Targetnya:

> **Melihat sebuah process sebagai sebuah security context + process relationship + service relationship + historical evidence.**

---

### 📝 FORMAT JAWABAN

Kirim:

```text
ACTIVE RECALL
Q1:
Q2:
...
Q12:

PRACTICE 1
...

PRACTICE 2
...

PRACTICE 3
...

PRACTICE 4
...

PRACTICE 5
Q1:
...
Q10:

MINI SOC CASE
1:
...
9:
```

Aku akan mencatat **hanya materi yang benar-benar kamu pelajari hari ini**, termasuk bagian yang masih menjadi kelemahan, supaya saat sudah Day 100–200 kamu tetap bisa melihat kembali perkembanganmu dari Day kecil.
