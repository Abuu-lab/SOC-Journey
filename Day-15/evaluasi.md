# Day 15 — Evaluasi, Jawaban Pengguna & Koreksi

**Status:** Belum PASS. **Nilai sementara:** 69/100 (estimasi pembelajaran, bukan tes standar). Threshold PASS = 76/100. Tidak perlu mengulang seluruh Day; cukup remedial terarah.

**Aturan evaluasi:** selalu tampilkan **jawaban pengguna → jawaban tepat → alasan**. Catat data aktual dan simulasi secara terpisah.

## 1. Active Recall Day 14 (Q1–Q9)

| Soal | Jawaban pengguna (sebagaimana ditulis) | Jawaban tepat dan alasan |
|---|---|---|
| Q1 | "PID itu id milik proses / Prosesguid lupa" | PID = process ID, bisa digunakan kembali; ProcessGuid = identitas process instance Sysmon untuk korelasi berbagai event. |
| Q2 | "ya" | IntegrityLevel High = elevated integrity, tetapi **bukan** berarti memiliki seluruh privileges Windows. |
| Q3 | "user itu account yg dipakai / group itu golongan user / privileges itu hak Batasan user" | User = account identity; Groups = security-group membership; Privileges = kemampuan khusus pada security token (bukan sekadar pembatasan). |
| Q4 | "tidak, aku belum bisa mengartikan" | `-k CameraMonitor` = nama grup service host, bukan bukti kamera sedang merekam. |
| Q5 | "service.exe menghasilkan svchost.exe lalu windows service" | Nama yang benar `services.exe` (SCM). SCM mengelola service dan dapat menjalankan svchost sebagai proses host service. |
| Q6 | `Get-CimInstance Win32_Process -Filter "ProcessId = 6920" | Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine` | Ini query *process*, bukan service. Benar: `Get-CimInstance Win32_Service | Where-Object {$_.ProcessId -eq 6920} | Select-Object Name,DisplayName,State,StartName,ProcessId`. |
| Q7 | `Test-Path "C:\path\file.exe"` | Benar. `False` berarti file tidak ditemukan di path itu saat pengecekan, tidak otomatis malicious. |
| Q8 | "perbedaannya hanya Sysmon Event ID 1 lebih kaya informasi" | Tambahan esensial: CIM = *current state*; Sysmon Event ID 1 = *historical* Process Create. |
| Q9 | "harusnya sama" | Tidak. Sysmon User = user context proses yang tercatat; Win32_Service StartName = akun service terkonfigurasi. Jangan samakan otomatis. |

## 2. Praktik 1 — Real Host Data

User:
```text
LocalAddress: 192.168.1.7
LocalPort: 64894
RemoteAddress: 4.213.25.240
RemotePort: 443
State: Established
OwningProcess: 4320
```

| Soal | Jawaban pengguna | Jawaban tepat |
|---|---|---|
| Q1 | `192.168.1.7` | Benar, LocalAddress. |
| Q2 | `4.213.25.240` | Benar, RemoteAddress. |
| Q3 | "belum tau" | OwningProcess = PID dari proses pemilik koneksi, di sini PID 4320. |
| Q4 | "belum tentu" | Benar. RemotePort 443 lazim untuk HTTPS, tetapi tidak otomatis secure/legitimate. |

**Data gap:** belum ada identitas proses untuk PID 4320. Tidak boleh menyamakannya dengan EpicGamesLauncher.exe yang ternyata mempunyai PID lain.

## 3. Praktik 2 — Real Host Data

User melakukan:
```powershell
$conn = Get-NetTCPConnection -State Established | Select-Object -First 1
$conn | Format-List LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
$targetPid = $conn.OwningProcess
Get-CimInstance Win32_Process -Filter "ProcessId = $targetPid" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
Get-Process -Id $targetPid -IncludeUserName |
Select-Object ProcessName, Id, UserName
```

Hasil:
```text
LocalAddress: ::1
LocalPort: 51455
RemoteAddress: ::1
RemotePort: 35783
State: Established
OwningProcess: 22704

Name: EpicGamesLauncher.exe
ProcessId: 22704
ParentProcessId: 11220
ExecutablePath: C:\Program Files\Epic Games\Launcher\Portal\Binaries\Win64\EpicGamesLauncher.exe
CommandLine: "C:\Program Files\Epic Games\Launcher\Portal\Binaries\Win64\EpicGamesLauncher.exe"
             "C:\Program Files\Epic Games\Launcher\Portal\Binaries\Win64\EpicGamesLauncher.exe"
             -silent -launchcontext=boot -AllowSoftwareRendering -SaveToUserDir -Messaging
             -enablehighdpi -EOSBootstrappedFlag -ForcedRestart
UserName: [REDACTED_HOST]\[REDACTED_USER]
```

**Assessment koreksi:** loopback IPv6 internal `::1 → ::1` milik EpicGamesLauncher, bukan bukti komunikasi internet atau malicious activity. Parent PID diketahui (11220), nama parent belum diuji. Evidensi yang sudah ada cukup untuk *initial benign-looking context*, belum membuktikan publisher, file integrity, atau alasan spesifik local IPC.

## 4. Praktik 3 — Real Tool Error

User:
```text
Get-WinEvent: There is not an event log on the localhost computer that matches
"Microsoft-Windows-Sysmon/Operational".
FullyQualifiedErrorId: NoMatchingLogsFound,Microsoft.PowerShell.Commands.GetWinEventCommand
```

**Koreksi:** Sysmon *channel/log* tidak ditemukan, **bukan** Event ID 3 kosong. Sebelum konfigurasi ulang, cek `hostname`, `whoami`, `Get-Service -Name Sysmon64,Sysmon -ErrorAction SilentlyContinue`, dan `Get-WinEvent -ListLog '*Sysmon*'`.

## 5. Challenge — SOC Alert #015 (SIMULASI)

| Soal | Jawaban pengguna | Jawaban tepat |
|---|---|---|
| C1 | "Explorer.exe" | **update.exe** melakukan koneksi; `explorer.exe` parent. |
| C2 | "ProcessGuid" | Benar. Dua event sama-sama `{LAB-015-A}`. |
| C3 | "4 detik" | Benar, creation 09:14:20 dan network 09:14:24. |
| C4 | "soure IP adalah identitas IP kita / source port adalah port yg digunakan untuk lalulintas network / Destination IP luar adalah IP yang berkoneksi dengan kita / Destination Port adalah Port luar untuk jalur masuk" | SourceIp/Port = endpoint asal; DestinationIp/Port = endpoint tujuan. Port bukan khusus "jalur masuk"; destination bisa private/public/loopback. |
| C5 | "legitimate karena HTTPS" | Tidak. Port 443 saja tidak membuktikan HTTPS ataupun bahwa aktivitas legitimate. |
| C6 | "SHA256 dan DigitalSignature" | Itu evidence *belum diperiksa*, bukan suspicious indicators saat ini. Uji path Temp, executable yang mengaku updater, dan koneksi segera setelah launch. |
| C7 | "port 443 karena HTTPS dan protocol tcp" | Bukan legitimate explanations. Contohnya updater resmi mengecek update atau installer melakukan license/version check. |
| C8 | "belum tau" | Cek path executable vs recorded Image, `Test-Path`, metadata, SHA256, digital signature, dan bila tersedia historical hashes/correlated telemetry; current file bisa berubah sejak event. |
| C9 | "kurang tau" | Ya, kedua historical event tetap bisa dikorelasikan dengan ProcessGuid sama meski proses sudah mati, selama log tersimpan. |
| C10 | "saya melihat process 6840 lalu saya memeriksa user yg menjalankan lalu mengecheck commandline nya mempunyai integrity level medium lalu melihat Sysmon network menghasilkan IP mana yg berhubungan dengan computer [REDACTED_USER] evidence gap sha 256, digital signature" | Flow awal benar, tetapi perlu Observation → Hypothesis → Test → Result → Evidence Gap → Assessment. Jangan mengklaim hasil tes yang belum dilakukan. |

**Contoh C10 jawaban ideal:**
- **Observation:** Event 1 membuat update.exe di Temp; 4 detik kemudian Event 3 menghubungkan PID 6840 / ProcessGuid `{LAB-015-A}` dengan koneksi ke IP dokumentasi port 443.
- **Hypothesis:** updater sah *atau* program menyamar sebagai updater.
- **Test:** pastikan path, cek hash/signature/metadata dan konteks peluncuran, cari log aktivitas jaringan tambahan.
- **Result:** belum tersedia dari simulasi.
- **Evidence Gap:** publisher, file identity, reputation, konteks vendor/instalasi.
- **Assessment:** Needs Investigation.

## 6. Mini SOC Report

**Jawaban pengguna:** "mini soc / not available".

**Koreksi:** Report formal belum dikirim, tetapi sebagian evidence tersedia dari Praktik 2. Berikut **rekonstruksi mentor dari output pengguna, bukan laporan yang sudah ditulis pengguna**:
```text
Source: Get-NetTCPConnection -State Established
Local: ::1:51455
Remote: ::1:35783
State: Established
OwningProcess/PID: 22704
Process: EpicGamesLauncher.exe
Parent PID: 11220
User: [REDACTED_HOST]\[REDACTED_USER]
ExecutablePath: C:\Program Files\Epic Games\Launcher\Portal\Binaries\Win64\EpicGamesLauncher.exe
Sysmon ID 3: unavailable; channel not found
Assessment: observed local IPv6 loopback; no malicious activity demonstrated
Evidence Gap: parent process name, historical Sysmon telemetry, process purpose for loopback peer
```

## 7. Evaluation & Follow-up

**Kekuatan:** Praktik 1 dan 2 berhasil mengumpulkan evidence langsung dari PowerShell, dan pemilik koneksi loopback berhasil diidentifikasi.

**Perlu latihan pendek sebelum PASS:**
1. Perbedaan Process vs Parent vs OwningProcess, pilih yang melakukan koneksi.
2. Mengapa 443 ≠ legitimate; sebutkan indicators vs missing evidence.
3. Makna Sysmon channel tidak ada; bedakan historical Event ID 1/3 vs live TCP.
4. Buat 1 mini SOC report lengkap dari hasil EpicGamesLauncher yang sudah ada.

**Status:** 69/100 provisional; jangan tandai PASS dan jangan catat Day 16 mulai sebelum remedial dinilai.