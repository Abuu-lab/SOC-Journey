# Day 03 — Evaluasi

Catatan ini mempertahankan jawaban, koreksi, dan penilaian yang tercatat. Sebagian jawaban hanya tersedia sebagai ringkasan mentor; pertanyaan atau jawaban lengkap yang tidak tercatat tidak direkonstruksi. Skenario latihan tidak dianggap sebagai incident pada endpoint aktual.

## Challenge

### 7. CHALLENGE — Di sinilah kamu perlu diperbaiki

Kasus:

```text
unknown.exe
PID 4216
CPU 92%
Memory 180 MB

PPID 5500
Executable:
C:\Users\Buya\AppData\Local\Temp\unknown.exe

CommandLine:
unknown.exe -update

PID 5500:
firefox.exe
```

Kamu mengatakan:

> “70% itu malicious.”

Nah, **ini yang tidak boleh kita lakukan** dalam investigation.

Bukan karena kecurigaanmu tidak masuk akal.

Kecurigaan awalnya **masuk akal**.

Yang bermasalah adalah memberikan angka:

> **70% malicious**

padahal kita belum mempunyai evidence yang cukup untuk menghitung probabilitas tersebut.

Gunakan:

> **Suspicious / warrants further investigation**

daripada:

> **70% malicious**

---
### 9. Bagian paling penting dari Challenge kamu

Kamu berkata:

> “kalau PPID-nya Firefox berarti harusnya PID-nya Firefox juga”

Nah, justru di sini kamu perlu memperbaiki mental model.

Yang benar:

```text
Firefox
PID 5500
    │
    └── unknown.exe
        PID 4216
        PPID 5500
```

Tidak ada masalah dengan:

```text
Parent = Firefox
Child = unknown.exe
```

Itu memang arti dari parent-child relationship.

Yang perlu diselidiki adalah:

> **Mengapa Firefox membuat unknown.exe?**

Itulah pertanyaan SOC.

---
#### Challenge dan jawaban

CHALLENGE

Sekarang saya beri kamu kasus.

Kamu sedang melakukan investigation terhadap:

unknown.exe
PID: 4216
CPU: 92%
Memory: 180 MB

Kamu menjalankan PowerShell dan mendapatkan:

Name: unknown.exe
ProcessId: 4216
ParentProcessId: 5500

ExecutablePath:
C:\Users\Buya\AppData\Local\Temp\unknown.exe

CommandLine:
unknown.exe -update

Kemudian:

PID 5500
Name: firefox.exe
Pertanyaan:

1. Apa yang kamu ketahui dari data tersebut?
kemungkinan berbahaya, ini hanya kecurigaan awal. soalnya kalau PPID nya firefox harusnya PIDnya firefox juga , tapi ini unknown.exe. okelah anggap kita positif think. tapi yg jadi masalah itu executablenya kok di "temp" yang notabene temp itu kan folder sampah yang umumnya ke hidden dan jarang dibuka user karena emg isinya sampah dan bawaan dasarnya hidden. ini jadi muncul pertanyaan kenapa berada disana
2. Apa yang masih belum kamu ketahui?
mungkin aku belum tau evidence lain yang bisa menjadi bukti untuk meyakinkanku bahwa itu file berbahaya atau bukan. mungkin bisa aku coba check melalui resmon untuk melihat traffic data
3. Apakah kamu boleh langsung menyimpulkan unknown.exe malicious? Jelaskan.
kalau dalam pembelajaranku di day 2, karena belum meluas cara pandangku terhadap banyaknya evidence. aku menyimpulkan 70% itu malicious, karena terlihat dari executable pathnya yg gk masuk akal berada di "temp" sedangkan parent id nya aja firefox
4. Apa hubungan antara Firefox dan unknown.exe berdasarkan data tersebut?
kyknya unknown.exe itu file diluar firefox executable firefox umumnya bukan di "temp" temp itu tempat sampah tapi firefox punya PID unknown.exe jadi timbul pertanyaan. 
5. Evidence apa yang ingin kamu cari berikutnya?
Kemungkinan sya mau liat traffic data menggunakan Resmon, trs CommandLine, atau kyk license ya? privasi policy? intinya kyk ngecheck keasliannya. gk tau nama metodenya apa belum smpai sana, bisa lihat commandline juga kita bisa tau perintah apa saja yg dijalankan si unknown.exe itu
Minimal 3 evidence tambahan.

## Mini-project / Investigasi

#### Hasil mini-project

Process Investigation

##### Process 1
Name            : Spotify.exe
ProcessId       : 9472
ParentProcessId : 19740
ExecutablePath  : C:\Users\Buya\AppData\Roaming\Spotify\Spotify.exe
CommandLine     : "C:\Users\Buya\AppData\Roaming\Spotify\Spotify.exe" --type=utility
                  --utility-sub-type=media.mojom.CdmServiceBroker --lang=en-US --service-sandbox-type=cdm
                  --video-capture-use-gpu-memory-buffer --no-pre-read-main-dll
                  --user-data-dir="C:\Users\Buya\AppData\Local\Spotify" --log-severity=disable
                  --user-agent-product="Chrome/146.0.7680.179 Spotify/1.3.0.277"
                  --metrics-shmem-handle=7312,i,7574247829178312417,11017353352888449004,524288
                  --field-trial-handle=2564,i,9697757951994868595,17242469109847768957,262144 --disable-features=AutofillActorMod
                  e,BackForwardCache,FullscreenBubbleShowOrigin,GlicActorUi,HardwareMediaKeyHandling,LensOverlay,PartitionAllocDa
                  nglingPtr,PartitionAllocUnretainedDanglingPtr,StorageNotificationService --variations-seed-version
                  --pseudonymization-salt-handle=2560,i,11956326233228186941,9705302225072860105,4
                  --trace-process-track-uuid=3190708992871164437 --enable-logging=handle --log-file=7552
                  --mojo-platform-channel-handle=8348 /prefetch:14

##### Process 2
Name            : Discord.exe
ProcessId       : 16464
ParentProcessId : 15004
ExecutablePath  : C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe
CommandLine     : "C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe" --type=utility
                  --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none
                  --user-data-dir="C:\Users\Buya\AppData\Roaming\discord" --standard-schemes=disclip
                  --secure-schemes=disclip,sentry-ipc --bypasscsp-schemes=sentry-ipc --cors-schemes=disclip,sentry-ipc
                  --fetch-schemes=disclip,sentry-ipc --streaming-schemes=disclip
                  --field-trial-handle=1852,i,13112856409038008341,10999823638448049431,262144
                  --enable-features=DocumentPolicyIncludeJSCallStacksInCrashReports --disable-features=AllowAggressiveThrottlingW
                  ithWebSocket,DropInputEventsWhilePaintHolding,HardwareMediaKeyHandling,IntensiveWakeUpThrottling,LocalNetworkAc
                  cessChecks,MediaSessionService,NetworkServiceSandbox,ScreenAIOCREnabled,SpareRendererForSitePerProcess,TraceSit
                  eInstanceGetProcessCreation,UseEcoQoSForBackgroundProcess,WinRetrieveSuggestionsOnlyOnDemand
                  --variations-seed-version --pseudonymization-salt-handle=1856,i,12559488081512397146,713315348952602427,4
                  --trace-process-track-uuid=3190708989122997041 --mojo-platform-channel-handle=2056 /prefetch:11

Name            : Discord.exe
ProcessId       : 20204
ParentProcessId : 15004
ExecutablePath  : C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe
CommandLine     : "C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe" --type=renderer
                  --user-data-dir="C:\Users\Buya\AppData\Roaming\discord" --standard-schemes=disclip
                  --secure-schemes=disclip,sentry-ipc --bypasscsp-schemes=sentry-ipc --cors-schemes=disclip,sentry-ipc
                  --fetch-schemes=disclip,sentry-ipc --streaming-schemes=disclip
                  --app-user-model-id=com.squirrel.Discord.Discord
                  --app-path="C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\resources\app.asar" --no-sandbox --no-zygote
                  --enable-blink-features=EnumerateDevices,AudioOutputDevices --autoplay-policy=no-user-gesture-required
                  --disable-background-timer-throttling --enable-h264-mf --enable-h264-mf-zero-copy
                  --video-capture-use-gpu-memory-buffer --lang=en-US --device-scale-factor=1.5 --num-raster-threads=4
                  --enable-main-frame-before-activation --renderer-client-id=6 --time-ticks-at-unix-epoch=-1789482264973760
                  --launch-time-ticks=144144040876 --field-trial-handle=1852,i,13112856409038008341,10999823638448049431,262144
                  --enable-features=DocumentPolicyIncludeJSCallStacksInCrashReports --disable-features=AllowAggressiveThrottlingW
                  ithWebSocket,DropInputEventsWhilePaintHolding,HardwareMediaKeyHandling,IntensiveWakeUpThrottling,LocalNetworkAc
                  cessChecks,MediaSessionService,NetworkServiceSandbox,ScreenAIOCREnabled,SpareRendererForSitePerProcess,TraceSit
                  eInstanceGetProcessCreation,UseEcoQoSForBackgroundProcess,WinRetrieveSuggestionsOnlyOnDemand
                  --variations-seed-version --pseudonymization-salt-handle=1856,i,12559488081512397146,713315348952602427,4
                  --trace-process-track-uuid=3190708991934122588 --mojo-platform-channel-handle=4232
                  --enable-node-leakage-in-renderers /prefetch:1

Name            : Discord.exe
ProcessId       : 484
ParentProcessId : 15004
ExecutablePath  : C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe
CommandLine     : "C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Discord.exe" --type=utility
                  --utility-sub-type=audio.mojom.AudioService --lang=en-US --service-sandbox-type=audio
                  --video-capture-use-gpu-memory-buffer --user-data-dir="C:\Users\Buya\AppData\Roaming\discord"
                  --standard-schemes=disclip --secure-schemes=disclip,sentry-ipc --bypasscsp-schemes=sentry-ipc
                  --cors-schemes=disclip,sentry-ipc --fetch-schemes=disclip,sentry-ipc --streaming-schemes=disclip
                  --field-trial-handle=1852,i,13112856409038008341,10999823638448049431,262144
                  --enable-features=DocumentPolicyIncludeJSCallStacksInCrashReports --disable-features=AllowAggressiveThrottlingW
                  ithWebSocket,DropInputEventsWhilePaintHolding,HardwareMediaKeyHandling,IntensiveWakeUpThrottling,LocalNetworkAc
                  cessChecks,MediaSessionService,NetworkServiceSandbox,ScreenAIOCREnabled,SpareRendererForSitePerProcess,TraceSit
                  eInstanceGetProcessCreation,UseEcoQoSForBackgroundProcess,WinRetrieveSuggestionsOnlyOnDemand
                  --variations-seed-version --pseudonymization-salt-handle=1856,i,12559488081512397146,713315348952602427,4
                  --trace-process-track-uuid=3190708992871164437 --mojo-platform-channel-handle=4636 /prefetch:12

##### Process 3
Name            : Cloudflare WARP.exe
ProcessId       : 17864
ParentProcessId : 9568
ExecutablePath  : C:\Program Files\Cloudflare\Cloudflare WARP\Cloudflare WARP.exe
CommandLine     : "C:\Program Files\Cloudflare\Cloudflare WARP\Cloudflare WARP.exe"

##### Observation

Apa yang saya temukan?
Saya menemukan hamper keseluruhan info, dari pid hingga parent, lalu saya bisa melihat executablepath dan commandline semua menjadi terfilter dan lebih mudah membacanya , karena kita mencari hanya di 1 tujuan

Apakah process tersebut mempunyai parent process?
ya 3 process tersebut mempunyai parent process

Apakah executable path terlihat masuk akal?
ya saya lihat jalur atau tempat aplikasi berada dalam folder yg masuk akal

Apakah command line terlihat normal?
saya belum begitu paham cara membaca commandline

Apa yang masih belum saya ketahui?
sepertinya hanya commandline, ini menurut pribadiku. karena kan aku belum punya pandangan, mungkin ada yg belum kuketahui tapi km mengetahuinya. jadi aku belum bisa menyebutkan karena emg benar2 belum tahu

## Active Recall / Self-test

### 12. Self-Test kamu

##### 1. Application vs Process — ✅

Sudah cukup benar.

Koreksi kecil:

> Process bukan “informasi/data program yang berjalan”.

Lebih tepat:

> **Process adalah instance dari program yang sedang dieksekusi oleh operating system.**

---

##### 2. PID — ✅

Pemahaman dasarnya benar.

---

##### 3. PID menentukan malicious? — ✅

Benar.

---

##### 4. Executable — ✅

Benar.

---

##### 5. Executable Path — ✅

Benar.

---

##### 6. Parent Process — ⚠️

Kamu menulis:

> “proses utama”

Jangan gunakan definisi itu.

Parent Process bukan harus:

> “main process”.

Lebih tepat:

> **Process yang menjadi parent bagi process lain dalam process relationship/tree.**

Contoh:

```text
explorer.exe
   ↓
notepad.exe
```

`explorer.exe` adalah parent dari `notepad.exe`.

---

##### 7. PID vs PPID — ❌

Ini yang paling perlu diperbaiki.

Ingat:

```text
PID  = ID process ini
PPID = ID parent process
```

---

##### 8. Command Line — ⚠️

Konsep bahwa command line memberi tambahan information **benar**.

Tetapi jangan menganggapnya sebagai daftar seluruh aktivitas internal application.

---

##### 9. PID 5500 — ✅/⚠️

Kamu ingin mencari executable path dan command line parent.

Bagus.

Tambahkan:

> **Kita harus mencari process dengan PID 5500 dan melihat siapa sebenarnya parent tersebut.**

---

##### 10. Firefox → unknown.exe — ✅/⚠️

Kamu sudah memahami:

> “patut dicurigai, tapi tetap harus check.”

Bagus.

Hanya jangan langsung memberikan angka probabilitas.

---
### CORRECTION DRILL — Sebelum Day 4

Jawab **5 pertanyaan ini saja**, tidak perlu menjalankan lab lagi:

##### 1.

```text
firefox.exe
PID 5500

   ↓

unknown.exe
PID 4216
PPID 5500
```

Siapa PID **4216**?

##### 2.

Siapa PID **5500**?

##### 3.

Apa arti:

```text
PPID 5500
```

pada `unknown.exe`?

##### 4.

Apakah ini normal secara konsep?

```text
C:\Program Files\Mozilla Firefox\firefox.exe
        ↓
C:\Users\Buya\AppData\Local\Temp\unknown.exe
```

Kenapa?

##### 5.

Kalau menemukan:

```text
unknown.exe
CPU 92%
Temp folder
Parent = firefox.exe
```

tulis **5 evidence berikutnya** yang ingin kamu cari **tanpa memberikan kesimpulan malicious/benign terlebih dahulu**.

Setelah 5 ini benar, **Day 3 saya nyatakan PASS**, dan baru kita lanjut ke Day 4.
#### Self-test dan jawaban

SELF TEST

Jawab tanpa melihat materi di atas sebisa mungkin.

1. Apa perbedaan Application dan Process?
aplikasi adalah program itu sendiri, process adalah informasi data program yang berjalan

2. Apa fungsi PID? untuk memudahkan kita mengidentifikasi, karena contoh firefox memiliki childproses yg bnyak itulah fungsi PID karena angkanya berbeda dan memudahkan kita mengidentifikasi

3. Apakah PID bisa menentukan sebuah process malicious atau tidak? tidak, itu hanya untuk memudahkan kita mengidentifikasi

4. Apa yang dimaksud Executable? file yang bisa dijalankan

5. Mengapa Executable Path penting dalam investigation? untuk mengetahui apakah aplikasi itu masuk akal jika disimpan disana .

6. Apa yang dimaksud Parent Process? proses Utama dan dia memmiliki childproses

7. Apa hubungan PID dan PPID? PPID adalah mainproses PID adalah anak proses

8. Mengapa Command Line dapat lebih berguna daripada hanya melihat Process Name? ya karena kita bisa melihat aktivitas didalam aplikasi tersebut atau aktivitas PID yang kita periksa. apakah normal atau tidak

9. Jika:

unknown.exe
PID 4216
PPID 5500

apa yang ingin kamu cari dari PID 5500?
Executable path apakah berhubungan dengan PID 4216 atau tidak biasanya Parentproses dan childproses executablepathnya hamper sama
lalu Commandline 

10. Jika menemukan:

firefox.exe
   ↓
unknown.exe

apakah otomatis malicious? Mengapa?
patut dicurigai, tapi tetap kita harus check 
dari sisi commanline,executable path
agar lebih meyakinkan

## Penilaian dan Hasil Praktik

EVALUATION

**Status: DEVELOPING → hampir READY, tetapi perlu correction drill.**

Bagian bagusnya: kamu benar-benar menjalankan command dan mendapatkan data nyata dari Windows. Bahkan kamu menemukan sesuatu yang penting:

```text
Discord.exe
PID 15004
    ├── Discord.exe PID 16464
    ├── Discord.exe PID 20204
    └── Discord.exe PID 484
```

Ini bukti nyata bahwa **satu application dapat mempunyai beberapa process**. Itu observasi yang bagus.

---
### 1. HANDS-ON — ✅ PASS

Task 1 sampai Task 4 berhasil.

Kamu berhasil mendapatkan:

```text
Name
PID
PPID
ExecutablePath
CommandLine
```

Dan ini jauh lebih penting daripada sekadar menghafal command.

Kamu juga menggunakan `-Filter`:

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'Spotify.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Itu sudah mulai menunjukkan kemampuan **mempersempit investigation ke process tertentu**.

---
### 2. Observasimu tentang Discord — bagus

Kamu menemukan:

```text
Discord.exe PID 15004
    ↓
Discord.exe PID 16464
Discord.exe PID 20204
Discord.exe PID 484
```

Ini sangat bagus untuk memahami:

> **Application ≠ satu process saja.**

Discord memang dapat menggunakan beberapa process dengan fungsi berbeda, misalnya:

```text
--type=renderer
--type=utility
--utility-sub-type=network...
--utility-sub-type=audio...
```

Jadi command line yang berbeda dapat memberikan **context mengenai role/function process tersebut**.

Ini mulai mendekati cara analyst membaca telemetry.

---
### 3. MINOR CORRECTION — Executable Path ✅

Kamu menulis:

> jalur atau tempat aplikasi berada dalam folder yang masuk akal

Benar.

Tetapi jangan ubah menjadi:

> Path masuk akal = file aman.

Karena:

```text
C:\Program Files\...
```

bisa meningkatkan confidence bahwa file berada di lokasi yang expected, tetapi **belum membuktikan file legitimate**.

Begitu juga:

```text
C:\Temp\...
```

tidak otomatis malware.

Ini penting untuk Challenge kamu.

---
### 4. KOREKSI BESAR #1 — PPID

Kamu beberapa kali menulis:

> “PPID adalah parent proses, PID adalah child proses”

Ini **salah konsep**.

##### PID

**PID = ID milik process itu sendiri.**

##### PPID

**PPID = PID milik parent process.**

Contoh:

```text
firefox.exe
PID: 5500

    ↓ creates/starts

unknown.exe
PID: 4216
PPID: 5500
```

Perhatikan:

```text
unknown.exe
PID 4216
PPID 5500
```

Artinya:

> `unknown.exe` sendiri memiliki PID **4216**.

Sedangkan:

> Parent-nya memiliki PID **5500**.

Kemudian kita cari PID 5500:

```text
PID 5500 = firefox.exe
```

Jadi:

```text
Parent:
firefox.exe
PID 5500

       ↓

Child:
unknown.exe
PID 4216
PPID 5500
```

##### Ingat kalimat ini:

> **PID = “Saya siapa?”**

> **PPID = “Siapa parent saya?”**

Bukan:

> PID = child
> PPID = parent.

---
### 5. KOREKSI BESAR #2 — Parent dan Child tidak harus punya Executable Path yang sama

Kamu menjawab:

> “biasanya Parentproses dan childproses executablepathnya hampir sama”

Tidak.

Itu **tidak menjadi aturan**.

Contoh:

```text
firefox.exe
    ↓
unknown.exe
```

Bisa saja:

```text
Parent Path:
C:\Program Files\Mozilla Firefox\firefox.exe

Child Path:
C:\Users\Buya\AppData\Local\Temp\unknown.exe
```

Justru **path yang berbeda sangat mungkin** karena parent dan child memang dua executable yang berbeda.

Contoh normal lain:

```text
explorer.exe
    ↓
notepad.exe
```

Path-nya jelas berbeda.

Yang perlu kita tanyakan adalah:

> **Apakah relationship tersebut masuk akal?**

Bukan:

> “Apakah path mereka hampir sama?”

---
### 6. KOREKSI BESAR #3 — Command Line

Kamu menulis:

> “membantu investigasi apa saja instruksi didalam aplikasi tersebut.”

Ini kurang tepat.

**Command Line tidak menunjukkan seluruh instruksi internal application.**

Command line menunjukkan **bagaimana executable tersebut dipanggil/dijalankan, termasuk arguments/parameters**.

Contoh milikmu:

```text
firefox.exe
-contentproc
-isForBrowser
-parentPid 10284
-crashHelperPid 0
...
```

Kita dapat belajar:

```text
process ini adalah content process
parent PID-nya 10284
mempunyai parameter tertentu
```

Bukan berarti kita membaca:

> “Firefox sedang menjalankan instruksi A, B, C di dalam kode program.”

Untuk melihat aktivitas internal yang lebih dalam, nanti kita menggunakan evidence lain:

* Event Logs
* Sysmon
* Network telemetry
* File activity
* EDR telemetry
* memory analysis
* dan sebagainya.

---
### 8. Apakah `Temp` mencurigakan?

**Bisa menjadi clue, tetapi tidak otomatis malicious.**

Kamu menyebut Temp sebagai:

> “folder sampah”

Untuk troubleshooting sehari-hari, analogi itu lumayan.

Tetapi dari perspektif SOC, jangan berpikir:

```text
Temp = malware
```

Karena legitimate software juga dapat menggunakan temporary directories untuk:

* installer
* updater
* temporary extraction
* cache
* runtime files
* update mechanism

Jadi:

```text
Temp
+
unknown.exe
```

adalah:

**interesting / suspicious indicator**

bukan:

**proof of malware.**

---
### 10. Evidence berikutnya yang kamu pilih — ✅

Kamu menyebut:

* Network traffic
* Command Line
* keaslian/license/signature

Bagus.

Untuk kasus ini, saya ingin kamu mulai mengenal urutan investigation:

```text
1. Process
2. Parent Process
3. Executable Path
4. Command Line
5. User
6. Network Connection
7. File Signature
8. Hash
9. File creation/modification
10. Event Logs
```

Tidak semuanya harus dipakai setiap kali.

Kita memilih evidence berdasarkan kasus.

---
### 11. Tentang “license / keaslian”

Yang kamu cari sebenarnya kemungkinan adalah:

##### Digital Signature

Misalnya file memiliki signature dari publisher tertentu.

Ini berbeda dengan:

##### Hash

Hash menjawab:

> “Fingerprint file ini apa?”

Digital Signature lebih dekat dengan:

> “Apakah file memiliki tanda tangan digital dari publisher yang dapat diverifikasi?”

Nanti kita pelajari keduanya.

Jadi tebakanmu sudah mengarah ke konsep yang benar, hanya **nama metode** yang belum kamu tahu.

---
### 13. Jadi, apakah Day 3 lulus?

Saya beri:
### **DAY 3 — DEVELOPING ⚠️**

Bukan karena hands-on kamu gagal.

Justru **hands-on kamu berhasil**.

Yang menahan kamu dari PASS penuh adalah:

```text
❌ PID vs PPID masih tertukar
❌ Parent Process dianggap “main process”
❌ Mengira child/parent executable path biasanya sama
⚠️ Command Line masih dianggap sebagai isi instruksi internal
⚠️ Masih memberi probabilitas "70% malicious" tanpa basis probabilitas
```

Ini konsep fundamental. Lebih baik kita rapikan sekarang daripada kesalahan ini terbawa sampai Wazuh/SIEM.

---
#### Hasil hands-on

HANDS-ON
Task 1 — Find 3 processes

Buka beberapa application:

Firefox
Spotify
Discord

Kemudian jalankan:

Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId

Cari tiga process tersebut.

Catat:

Process: Discord.exe
PID: 15004
PPID: 17936

Process: Spotify.exe
PID: 19740
PPID: 9568

Process: firefox.exe
PID: 10284
PPID: 11068


Task 2 — Find the Executable Path

Sekarang gunakan:

Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath

Cari:

firefox.exe
spotify.exe
discord.exe

Catat:

Process: firefox.exe
PID: 10284
Executable Path: C:\Program Files\Mozilla Firefox\firefox.exe

Process: spotify.exe
PID: 19740
Executable Path: C:\Users\Buya\AppData\Roaming\Spotify\Spotify.exe

Process: discord.exe
PID: 15004
Executable Path: C:\Users\Buya\AppData\Local\Discord\app-1.0.9257\Disco...

Jangan kaget kalau beberapa process tidak menampilkan path. Windows dapat membatasi informasi tertentu untuk process tertentu.


Task 3 — Find Command Line

Sekarang:

Get-CimInstance Win32_Process |
Select-Object Name, ProcessId, ParentProcessId, CommandLine

Cari salah satu application milikmu.

Catat:

Process: Cloudflare WARP.exe
PID: 17864
Command Line: "C:\Program Files\Cloudflare\Cloudflare WARP\Cloudflare WARP.exe"

Process: OneDrive.exe
PID: 13384
Command Line : "C:\Program Files\Microsoft OneDrive\OneDrive.exe" /background

Perhatikan bahwa command line bisa sangat panjang.

Itu normal.


Task 4 — Investigation satu process

Sekarang pilih SATU process, misalnya:

Spotify.exe

Cari seluruh informasi:

Get-CimInstance Win32_Process -Filter "Name = 'Spotify.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine

Hasilnya akan memberikan:

Name: firefox.exe
ProcessId: 6872
ParentProcessId: 10284
ExecutablePath: C:\Program Files\Mozilla Firefox\firefox.exe
CommandLine: "C:\Program Files\Mozilla Firefox\firefox.exe" -contentproc -isForBrowser -prefsHandle 23700:52635 -prefMapHandle 15656:306489 -jsInitHandle 13188:162620 -parentBuildID 20260909172920 -ipcHandle 17396 -initialChannelId {84cfb72a-57cd-4ec4-888b-6e2ba3268351} -parentPid 10284 -crashHelperPid 0 -crashHelper 22556 -crashReporter "\\.\pipe\gecko-crash-server-pipe.10284" -win32kLockedDown -appDir "C:\Program Files\Mozilla Firefox\browser" - 86 tab

