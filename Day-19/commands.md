# Day 19 — Commands

**Status:** Hands-on 1–5 sudah dijalankan; Hands-on 6 berupa penalaran tanpa keluaran korelasi tambahan; mini-project belum dikumpulkan.

## Hands-on 1: Daftar process Discord saat ini
~~~powershell
Get-CimInstance Win32_Process -Filter "Name = 'Discord.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
~~~
**Hasil aktual:** enam instance process Discord.exe dengan enam PID berbeda: 2724, 7856, 22476, 9028, 18508, dan 12432. Lima process anak memiliki `ParentProcessId 2724`, sedangkan PID 2724 memiliki parent PID 4632. Jalur dan akun lokal disamarkan. Command line menunjukkan peran Electron: crashpad-handler, gpu-process, network utility, renderer, dan audio utility. Peran tersebut sendiri tidak membuktikan bahwa file aman atau berbahaya.

## Hands-on 2: Memeriksa PID yang dipilih
~~~powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 2724" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
~~~
**Hasil aktual:** process yang dicari adalah Discord.exe dengan PID 2724 dan **ParentProcessId 4632**. Identitas process yang memiliki PID 4632 belum diperiksa. Jawaban awal P4 keliru menyebut 4632 sebagai nama parent; angka tersebut adalah PPID, bukan hasil query nama parent.

## Hands-on 3: Event ID 1 Sysmon terbaru
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 1 | Format-List TimeCreated, Message
~~~
**Hasil aktual:** 2026-10-15 pukul 17:12:32 waktu lokal, `opushutil.exe` dari Microsoft Office (PID 3424), command line `/pushregistration`, `IntegrityLevel Medium`, parent `svchost.exe` (PID 2144), dan `ParentCommandLine` memuat `-k netsvcs -p -s Schedule`. GUID user/perangkat dan akun privat disamarkan. Jangan menyimpulkan sah atau malicious hanya berdasarkan basename.

## Hands-on 4: Memeriksa file executable Discord saat ini
~~~powershell
$file = Join-Path $env:LOCALAPPDATA 'Discord\app-1.0.9261\Discord.exe'
Test-Path -LiteralPath $file
Get-Item -LiteralPath $file |
Select-Object Name, Length, CreationTime, LastWriteTime
Get-AuthenticodeSignature -LiteralPath $file |
Select-Object Status, StatusMessage
Get-FileHash -LiteralPath $file -Algorithm SHA256
~~~
**Hasil aktual:** `Test-Path True`; Discord.exe berukuran 220542392 byte; `CreationTime 2026-10-08 16:02:01`, `LastWriteTime 16:02:04` (waktu lokal). Status Authenticode `Valid`, signature terverifikasi. SHA256 berhasil dihitung; repo publik tidak mencantumkan sidik jari lengkap. SHA256 adalah sidik jari isi file, **bukan** pemindaian virus. Pemeriksaan file ini menggambarkan kondisi saat ini dan tidak membuktikan byte file yang sama digunakan pada setiap event process historis.

## Hands-on 5: Tiga Event Jaringan Sysmon
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 3 | Format-List TimeCreated, Message
~~~
**Hasil aktual:** tiga catatan Spotify.exe ID 3 pada 2026-10-15 waktu lokal:
- 17:24:40: TCP, PID 22300, ProcessGuid [REDACTED_GUID_A], koneksi keluar menuju [REDACTED_EXTERNAL_IP]:443.
- 17:24:25: UDP, PID 6680, ProcessGuid [REDACTED_GUID_B], Initiated false, source multicast 239.255.255.250:1900 toward local UDP endpoint [REDACTED_PRIVATE_IP]:56104.
- 17:24:24: UDP, PID 6680 dan ProcessGuid [REDACTED_GUID_B] yang sama, `Initiated true`, dari endpoint lokal menuju multicast 239.255.255.250:1900.

Port 1900 diberi label `ssdp` pada log. Lalu lintas multicast terkait SSDP berbeda dari lalu lintas internet pada port 443. Sampel ini memuat **dua instance process** (dua GUID berbeda), meskipun terdapat tiga event.

## Hands-on 6: Linimasa
Tidak ada keluaran perintah tambahan yang dikirim. Kamu mencoba mengurutkan Event ID 1 → 3 → 22, tetapi catatan aktual yang diberikan adalah ID 1 dari opushutil.exe dan tiga ID 3 dari Spotify.exe. **Tidak ada Event ID 22 yang disertakan dalam lab ini.** Urutan kronologis catatan yang diamati:
1. 17:12:32 ID 1 opushutil.exe.
2. 17:24:24 ID 3 Spotify.exe (UDP).
3. 17:24:25 ID 3 Spotify.exe (UDP).
4. 17:24:40 ID 3 Spotify.exe (TCP).
Jangan menyatakan bahwa event Office menyebabkan atau berkaitan dengan event Spotify; GUID-nya berbeda.

## Perintah pelaporan lanjutan — SUDAH DIAJARKAN, BELUM DIJALANKAN UNTUK MINI-PROJECT
Pilihan perintah hanya-baca yang sudah dipelajari untuk menindaklanjuti satu kasus:
~~~powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 6680" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
    StartTime = (Get-Date '2026-10-15 17:24:00')
    EndTime = (Get-Date '2026-10-15 17:25:00')
} | Format-List TimeCreated, Message
~~~
Jangan menganggap PID historis 6680 masih digunakan. Hingga laporan ini dibuat, kedua perintah di atas belum dijalankan untuk mini-project.

## Privasi
Semua username lokal, hostname mentah, GUID Sysmon unik, jalur profil pengguna yang dapat mengungkap identitas, serta alamat IP privat disamarkan dari portofolio ini.


## Tindak lanjut akhir: korelasi parent Spotify historis

Perintah tambahan yang benar-benar dijalankan setelah catatan awal:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 6680" | Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```
Hasil: tidak ada process yang cocok saat ini.

Pencarian pada 200 pesan Sysmon ID 1 terbaru tidak menemukan kandidat yang cocok dalam sampel terbatas tersebut. Pencarian berdasarkan rentang waktu pada 15 Oktober 2026 kemudian menemukan lima catatan pembuatan process anak Spotify (PID anak 7160, 6308, 7208, 18628, 13636), masing-masing memiliki `ParentProcessId 6680`. `ParentProcessGuid`-nya cocok dengan ProcessGuid event UDP Spotify untuk PID historis 6680. Catatan pembuatan parent itu sendiri tidak termasuk dalam lima hasil tersebut.

**Status akhir Day 19: PASS 80/100**, bersifat formatif. Ringkasan kasus lengkap pada evaluasi.md disusun dengan bantuan mentor; field lokal yang sensitif telah dihilangkan.
