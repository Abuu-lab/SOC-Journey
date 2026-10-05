# 🟢 EVALUASI DAY 12 — PASSED

Aku sudah cek seluruh hasil Day 12. **PASS** sesuai aturan kita: 76–100% langsung lanjut.

Dan menurutku Day 12 ini **cukup penting**, karena untuk pertama kalinya kamu benar-benar mendapatkan telemetry yang jauh lebih kaya dari Sysmon, bukan cuma `Get-Process`.

---

# 1. PRAKTIK SYSMON ✅✅

Kamu menemukan:

```text
Sysmon64.exe
Version 15.22.0.0
```

dan servicenya:

```text
Running
Sysmon64
```

Jadi Sysmon memang sudah terpasang dan aktif di laptopmu. 

✅ Bagus.

---

# 2. SYSMON EVENT ID 1 ✅✅

Kamu mendapatkan:

```text
Event ID: 1
Process Create
```

dengan informasi yang sangat kaya:

```text
ProcessId: 22968
Image:
C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe

CommandLine:
...MicrosoftEdgeUpdate.exe /ua /installsource scheduler

User:
NT AUTHORITY\SYSTEM

IntegrityLevel:
System

SHA256:
5497DD...
ParentProcessId:
2128

ParentImage:
C:\Windows\System32\svchost.exe
```



🔥 Ini contoh nyata kenapa Sysmon berguna.

Sebelumnya kita mungkin harus melakukan beberapa query:

```text
Process
+
User
+
Parent
+
CommandLine
+
Hash
```

Sekarang satu historical event bisa memberikan banyak informasi tersebut.

---

# 3. HAL PENTING YANG HARUS KAMU SADARI

Perhatikan Sysmon mencatat:

```text id="zjd0f7"
ProcessGuid
ProcessId
Image
CommandLine
User
Hash
ParentProcessId
ParentImage
ParentCommandLine
```

Artinya investigation kita mulai berubah dari:

> **“Apa yang sedang berjalan sekarang?”**

menjadi:

> **“Apa yang pernah dijalankan, kapan, oleh siapa, dengan command apa, dan siapa parent-nya?”**

Ini **persis masalah yang kita temui di Day 11** ketika PID `24416` sudah hilang.

Sysmon memberi kita historical telemetry yang lebih kaya.

---

# 4. PRAKTIK NOTEPAD ✅

Kamu menjalankan Notepad dan menemukan:

```text
Notepad.exe
PID 12388
PPID 12196
```

dengan:

```text
C:\Program Files\WindowsApps\...\Notepad.exe
```

dan CommandLine yang menunjuk ke executable tersebut. 

User context:

```text
DESKTOP-C7BHMKL\Buya
```



✅ Benar.

---

# 5. FILE INVESTIGATION ✅✅

Kamu kemudian mengikuti alurnya:

```text
Process
↓
User
↓
File
↓
Metadata
↓
Signature
↓
Hash
```

Metadata:

```text
Length: 3305272
CreationTime: 9/1/2026
LastWriteTime: 9/1/2026
LastAccessTime: 10/5/2026
```



Signature:

```text
Valid
```



SHA-256:

```text
03745E9E21684C0E6D9FDC50D91CAA0935F4BAF50F626C11987AB7D6991D58D1
```



✅ Kamu berhasil melakukan **full file investigation**.

---

# 6. MINI INVESTIGATION DISCORD 🔥

Ini bagian yang paling aku suka.

Kamu menemukan:

```text
Discord.exe
PID 15896
PPID 21128
User Buya
```



Lalu kamu mencari parent:

```text
Discord.exe
PID 21128
PPID 13636
```

dengan CommandLine:

```text
Discord.exe --start-inactive
```



Kemudian kamu lanjut ke file:

```text
Metadata
Signature
Hash
```

dan mendapatkan signature `Valid` serta SHA-256:

```text
DD3D7B9A55893153084BC75FC57436C792394705BB30CC7D5B80AC08CBE79B51
```



🔥 Ini sudah sesuai alur investigation yang kita inginkan.

---

# 7. ANALYSIS-MU 🟢

Kamu menyimpulkan:

> sejauh ini tidak

lalu ingin melihat:

> Threat Intelligence

Ini cukup baik.

Tetapi aku ingin satu perubahan dalam cara berpikirmu:

Jangan langsung menulis:

> "aman."

Gunakan:

> **"No obvious anomaly identified from the evidence currently collected."**

Karena evidence yang kamu punya belum mencakup semuanya.

Kamu sendiri sudah sadar masih ada:

```text
Threat Intelligence
Network
```

Bagus. 

---

# 8. COMMANDLINE DISCORD

CommandLine Discord-mu sangat panjang dan berisi:

```text
--type=utility
--utility-sub-type=network.mojom.NetworkService
...
--user-data-dir=...
```



Kamu mengatakan:

> "harusnya sih bisa dilihat dari commandline, tpi kyknya aman"

🟢 Cara berpikirmu sudah benar arahnya, tetapi **jangan mengatakan aman hanya dari CommandLine**.

Yang bisa kita katakan:

> CommandLine memberikan context yang konsisten dengan proses utility/network-service Discord.

Kemudian kita validasi dengan:

```text
User
Parent
Path
Metadata
Signature
Hash
Network
```

Ini adalah perbedaan penting antara:

**Observation** dan **Conclusion**.

---

# 9. ACTIVE RECALL DAY 12

### Q1 — Service vs Process ✅

Jawabanmu:

> service adalah background component yang dijalankan oleh Windows Service Manager, process adalah program yang berjalan

🟢 **BENAR.**

Akhirnya ini sudah jauh lebih presisi.

---

### Q2 — Security Context 🟡

Kamu:

> user + group

Benar sebagai bagian utama, tetapi masih kurang.

Untuk level kita:

```text
Security Context
=
Identity/User
+
Groups
+
Privileges
```

Jadi kita **belum menganggap ini selesai sepenuhnya**.

Tidak masalah karena tetap PASS; nanti kita bawa reminder.

---

### Q3 — TimeCreated ✅

> waktu yang dibuat saat event terjadi

🟢 Maksudmu sudah benar.

Lebih tepat:

> **Timestamp yang menunjukkan kapan event terjadi/tercatat.**

---

# 📊 DAY 12 SCORE

| Materi                     | Status |
| -------------------------- | ------ |
| Sysmon installation check  | 🟢     |
| Sysmon Event ID 1          | 🟢     |
| Process Creation telemetry | 🟢     |
| Process + User             | 🟢     |
| Parent Process             | 🟢     |
| CommandLine                | 🟢     |
| File investigation         | 🟢     |
| Metadata                   | 🟢     |
| Digital Signature          | 🟢     |
| SHA-256                    | 🟢     |
| Evidence correlation       | 🟢     |
| Investigation flow         | 🟢     |
| Security Context           | 🟡     |

# 🟢 **DAY 12 — PASSED**

Dan menurutku ada kemajuan nyata:

Di Day 11 kamu sempat berhenti ketika:

```text
PID → Process tidak ditemukan
```

Sekarang kamu mulai memahami:

```text
Historical Telemetry
↓
ambil evidence yang masih tersedia
↓
lanjut investigation
```

Itu **cara berpikir investigator** yang memang ingin kita bangun.

---

# 📌 MILESTONE 1 TAHUN — DAY 12

Catatan kecil yang akan kita simpan untuk milestone-mu:

### DAY 12 — Sysmon & Endpoint Telemetry

* Sysmon
* Sysmon Service
* Sysmon Event ID 1 — Process Creation
* Historical Process Telemetry
* ProcessGuid
* ProcessId
* Image
* CommandLine
* User
* IntegrityLevel
* ParentProcessId
* ParentImage
* ParentCommandLine
* Sysmon Hash telemetry
* Endpoint visibility
* Historical process investigation
* Correlation: Process → User → Parent → CommandLine → File → Hash
* **Process yang sudah mati tidak selalu mengakhiri investigation**

Data praktismu sendiri menunjukkan Sysmon Event ID 1 dapat menyediakan `ProcessId`, `Image`, `CommandLine`, `User`, hash, `ParentProcessId`, `ParentImage`, dan `ParentCommandLine` dalam satu event. 

---

# 🧠 REMINDER UNTUK DAY 13

Hanya satu yang masih perlu diperkuat:

> **Security Context = User + Groups + Privileges**

Yang lainnya **tidak perlu aku ulang besok** kecuali memang muncul lagi secara natural dalam investigation.

Dan mulai Day 13, kita akan semakin sering memakai pola:

```text
ALERT
 ↓
IDENTIFY
 ↓
WHAT HAPPENED?
 ↓
WHEN?
 ↓
WHO?
 ↓
PROCESS
 ↓
PARENT
 ↓
COMMANDLINE
 ↓
FILE
 ↓
HASH / SIGNATURE
 ↓
NETWORK
 ↓
CORRELATE
 ↓
EVIDENCE GAP
 ↓
ASSESSMENT
 ↓
CONCLUSION
```

Jadi teori tetap ada, tetapi **tanganmu akan semakin sering menjalankan investigation tersebut sendiri**.
