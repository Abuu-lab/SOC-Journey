# Day 13 — Evaluasi

Catatan ini mempertahankan jawaban, koreksi, dan penilaian yang tercatat. Sebagian jawaban hanya tersedia sebagai ringkasan mentor; pertanyaan atau jawaban lengkap yang tidak tercatat tidak direkonstruksi. Skenario latihan tidak dianggap sebagai incident pada endpoint aktual.

## Challenge

Catatan terpisah untuk bagian ini belum tersedia dalam evaluasi hari ini. Soal dalam Materi merupakan latihan yang diberikan; penyelesaiannya tidak diasumsikan.

## Mini-project / Investigasi

Catatan terpisah untuk bagian ini belum tersedia dalam evaluasi hari ini. Soal dalam Materi merupakan latihan yang diberikan; penyelesaiannya tidak diasumsikan.

## Active Recall / Self-test

#### Active Recall Day 14

Aku sudah masukkan beberapa kelemahan Day 13 sebagai bahan recall berikutnya.

Yang akan muncul kembali terutama:

```text
1. Apa fungsi ProcessGuid?
2. Apa beda PID dan ProcessGuid?
3. Apa itu IntegrityLevel?
4. Apa beda User dan IntegrityLevel?
5. Apa arti svchost.exe -k <group>?
6. Apa hubungan services.exe → svchost.exe → Service?
7. Apa itu File Exists?
8. Mana historical evidence dan mana current state?
9. Command apa untuk historical Sysmon?
10. Command apa untuk current process?
11. Security Context terdiri dari apa saja?
```

##### DAY 13 MILESTONE

* Sysmon Event ID 1 — Process Create
* `ProcessGuid`
* `ProcessId`
* `ParentProcessGuid`
* `ParentProcessId`
* `ParentImage`
* `ParentCommandLine`
* `ParentUser`
* `IntegrityLevel`
* Historical Process Telemetry
* Current State vs Historical Evidence
* Process correlation
* `services.exe → svchost.exe`
* `svchost.exe -k <service group>`
* File metadata
* Digital Signature
* SHA-256
* Evidence gap
* Investigation workflow
* Observation → Hypothesis → Evidence/Test → Assessment → Conclusion
* **PID dapat digunakan kembali; ProcessGuid membantu correlation**
* **Process mati ≠ investigation selesai**

**Kesimpulan Day 13:** kamu **PASS**. Bukan karena semua jawaban sempurna, tetapi karena kamu sudah bisa melakukan investigation flow dan tahu kapan evidence belum cukup untuk mengambil kesimpulan.

## Penilaian dan Hasil Praktik

**DAY 13: PASS ✅**

Secara keseluruhan, kamu sudah menangkap alur investigasi Sysmon. Aku nilai sekitar **80–85%**, jadi sesuai rule kita: **PASS, lanjut ke Day 14**. Yang belum kuat tidak mengulang Day 13; kita bawa sebagai **active recall Day 14**.

##### 1. Kenapa `svchost.exe -k CameraMonitor`?

Ini memang bagian yang perlu kita luruskan.

Kamu melihat:

```text
ParentImage: C:\Windows\System32\services.exe
CommandLine: C:\WINDOWS\System32\svchost.exe -k CameraMonitor
```

Pertama, nama yang benar adalah:

```text
services.exe
```

bukan `service.exe`.

Hubungannya secara sederhana:

```text
Windows Service Control Manager
        ↓
services.exe
        ↓
svchost.exe
        ↓
service yang di-host
```

`svchost.exe` adalah **Host Process for Windows Services**. Jadi dia bukan biasanya aplikasi yang dijalankan user secara langsung, melainkan proses yang menjadi wadah untuk menjalankan service tertentu.

Sedangkan:

```text
-k CameraMonitor
```

**bukan berarti `svchost.exe` sedang "mengakses kamera".**

`-k` adalah parameter untuk menentukan **service host group** bernama `CameraMonitor`.

Jadi secara kasar:

```text
svchost.exe -k CameraMonitor
             ↑
        service group
```

Nah, untuk mengetahui service apa yang menggunakan PID tersebut, ini yang lebih penting:

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 6920} |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName
```

Untuk PID `18824`:

```powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 18824} |
Select-Object Name, DisplayName, State, StartMode, StartName, ProcessId, PathName
```

Jadi **jangan menebak arti `CameraMonitor` dari namanya**. Dalam SOC, kita cari evidence yang menghubungkannya ke service sebenarnya.

---

##### 2. `ProcessGuid` yang tadi kamu lupa

Ini penting.

Kamu sudah bisa membaca:

```text
ProcessId: 6920
ParentProcessId: 1424

ProcessGuid: {....}
ParentProcessGuid: {....}
```

Yang perlu kamu ingat:

**PID = identifier proses saat ini.**

Tetapi PID bisa dipakai kembali setelah suatu process mati.

Contoh:

```text
notepad.exe → PID 5000 → mati

beberapa waktu kemudian

chrome.exe → PID 5000
```

Kalau SOC hanya melihat PID, kita bisa salah mengira kedua proses itu adalah proses yang sama.

`ProcessGuid` membantu mengidentifikasi **instance/process creation tertentu** dan sangat berguna untuk correlation antar Sysmon events.

Jadi hafalan sederhananya:

```text
PID        = nomor proses
ProcessGuid = identitas instance proses untuk correlation
```

Dan:

```text
ParentProcessId
ParentProcessGuid
```

memberi kita konteks siapa parent dari process instance tersebut.

---

##### 3. `File Exists` itu apa?

Ini sederhana tetapi penting.

**File Exists = apakah file executable tersebut masih ada di disk sekarang?**

Command:

```powershell
Test-Path "C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SoftLandingTask\SoftLandingTask.exe"
```

Output:

```text
True
```

berarti file masih ada.

```text
False
```

berarti file sudah tidak ada di lokasi tersebut.

Kenapa berguna?

Misalnya Sysmon mencatat:

```text
Process Create
Image: C:\Temp\update.ps1.exe
```

Kemudian saat kita investigasi:

```text
Test-Path ...
False
```

Berarti executable tersebut **sudah tidak ada sekarang**.

Itu tidak otomatis malicious, tetapi menjadi **evidence gap** yang perlu kita pahami.

---

##### 4. Jawaban Challenge-mu yang perlu diperbaiki

Beberapa sudah bagus.

**Q6**

Kamu menjawab:

> karena dia mempunyai argument `-file`, sepertinya process tersebut sedang mengakses file

Ini belum tepat.

Kalau command line PowerShell misalnya:

```text
powershell.exe -File update.ps1
```

`-File` lebih tepat dibaca sebagai:

> PowerShell diperintahkan untuk menjalankan script file tersebut.

Jadi:

```text
-File update.ps1
```

lebih dekat ke:

```text
PowerShell → execute script update.ps1
```

bukan otomatis:

```text
PowerShell → sedang mengakses file
```

---

**Q7 — IntegrityLevel**

Kamu bilang belum paham. Ini memang jadi salah satu materi yang harus kita bawa ke Day 14.

Sederhananya:

```text
User
↓
WHO is running?

IntegrityLevel
↓
WHAT security level/context is the process running under?
```

Contoh:

```text
User: Buya
IntegrityLevel: Medium
```

berarti process berjalan atas user Buya dengan **Medium Integrity Level**.

Contoh:

```text
User: Buya
IntegrityLevel: High
```

berarti process Buya berjalan dalam context yang lebih elevated.

Contoh lain:

```text
User: SYSTEM
IntegrityLevel: System
```

sangat tinggi.

Tetapi:

**High/System ≠ malicious.**

Process Windows yang normal juga bisa menggunakan level tinggi.

---

**Q14**

Kamu menjawab:

> PID ID milik proses, kalau process guid lupa

Sekarang koreksinya:

```text
PID
= Process Identifier

ProcessGuid
= Identifier untuk process instance yang membantu correlation
```

Yang paling penting bukan sekadar hafal namanya, tetapi **kenapa SOC membutuhkan keduanya**.

---

##### 5. Bagian mini-project: ada satu koreksi penting

Kamu menulis:

```text
Process Name: Microsoft-Windows-Sysmon
PID: 3240
```

`Microsoft-Windows-Sysmon` itu **ProviderName**, bukan nama process yang sedang diinvestigasi.

Process yang sebenarnya dari data kamu adalah:

```text
SoftLandingTask.exe
```

Karena:

```text
ExecutablePath:
C:\Windows\SystemApps\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\SoftLandingTask\SoftLandingTask.exe
```

Sedangkan:

```text
Microsoft-Windows-Sysmon
```

adalah provider yang menghasilkan event.

Ini justru contoh bagus kenapa **ketelitian sangat penting dalam SOC**. Satu field tertukar bisa mengubah pemahaman investigasi.

---

##### 6. Analysis mini-project-mu sebenarnya sudah mengarah benar

Kamu menulis:

> digital signaturenya valid
> hashnya sudah nampak
> commandline dan argument sudah terlihat
> sejauh ini kita perlu evidence tambahan

Ini **benar**.

Bahkan evidence yang sudah kamu punya lebih banyak:

```text
Time
Process
PID
ProcessGuid
User
IntegrityLevel
Parent PID
Parent Process
Parent CommandLine
ExecutablePath
CommandLine
Metadata
Digital Signature
SHA256
```

Yang belum ada/masih bisa diperiksa:

```text
Network activity
Threat Intelligence / hash reputation
Service correlation
Additional related Sysmon events
```

Jadi kalimatmu:

> “Evidence yang belum tersedia: kita akan cari nilai hash nya di internet”

perlu sedikit diperbaiki.

**Hash-nya sudah tersedia.**

Yang belum tersedia adalah:

> **hasil reputation/threat-intelligence dari hash tersebut.**

---

##### 7. Ini command correlation yang tadi kamu tanyakan

Untuk **historical Sysmon evidence**, command yang kamu pakai adalah:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 10
```

Kemudian kita bisa melihat event Process Create.

Untuk **current state**, kita bisa gunakan:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 3240" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Jadi konsepnya:

```text
Sysmon / Get-WinEvent
        ↓
Historical evidence
        ↓
Apa yang terjadi?

Get-CimInstance
        ↓
Current state
        ↓
Apa yang sedang terjadi sekarang?
```

Dan kalau current process sudah mati:

```text
Get-CimInstance
      ↓
tidak ditemukan

Get-WinEvent
      ↓
masih bisa ditemukan
```

**Itulah salah satu kekuatan Sysmon dalam investigation.**

---

##### 8. Jawaban Q15-mu bagus

Flow yang kamu tulis:

```text
Alert
↓
Get Process
↓
kalau tidak ada → historical event
↓
process create time
↓
PID
↓
user
↓
parent
↓
ExecutablePath
↓
CommandLine
↓
Metadata
↓
SHA256
↓
Digital Signature
↓
evidence tambahan
```

Ini sudah mulai menjadi **investigation workflow**, bukan sekadar hafalan command.

Nanti kita akan terus melatih kamu membuat reasoning seperti:

```text
Saya melihat X
↓
Karena itu saya memeriksa Y
↓
Hasilnya Z
↓
Maka saya menguji A
↓
Hasil A...
↓
Assessment
```

Itu yang saya ingin semakin kuat pada kamu.

---

