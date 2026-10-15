# Day 18 — Commands (Extended Hands-on Lab)

## Perintah yang benar-benar dijalankan beserta hasilnya

### 1. Membaca lima event jaringan Sysmon historis

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 5 | Select-Object TimeCreated, Id
```

Kamu mengetik `-MaxEvent 5`; PowerShell menerima bentuk singkat parameter tersebut. Sebaiknya gunakan bentuk lengkap `-MaxEvents` agar lebih mudah dibaca. Lima catatan event ditampilkan pada 2026-10-14 sekitar pukul 20:23 waktu lokal.

### 2. Memeriksa tiga event jaringan historis

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 3 | Format-List TimeCreated, Message
```

Tiga event yang ditampilkan semuanya milik Spotify.exe dan menggunakan UDP. Satu event dengan PID historis 21148 memiliki `Initiated false` dan alamat sumber yang berakhiran .255 (alamat broadcast LAN), dengan tujuan lokal [REDACTED_PRIVATE_IP] dan port tujuan 57621. Dua catatan event menggunakan PID historis 21452, ProcessGuid [REDACTED_PROCESSGUID], tujuan 35.186.224.24:443, dan `Initiated true`. PID dan ProcessGuid yang sama menunjukkan bahwa kedua event berasal dari instance process yang sama, tetapi tidak menjelaskan penyebab pencatatan berulang.

`Format-List TimeCreated, Message` menampilkan dua properti tingkat atas yang dipilih dalam bentuk daftar; perintah ini tidak otomatis menampilkan semua properti. Properti `Message` sendiri memuat banyak field Sysmon. `Format-List *` dapat menampilkan semua properti objek.

### 3. Membandingkan PID historis Discord dengan daftar process saat ini

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 8808" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Kamu melaporkan tidak ada hasil untuk PID historis 8808. Hal ini tidak membatalkan catatan historis Sysmon ID 3.

### 4. Mencatat instance Discord.exe yang sedang berjalan

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'Discord.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath
```

Ditemukan enam process Discord.exe yang sedang berjalan, dengan PID 16220, 17708, 19312, 9144, 20744, dan 21412. Lima di antaranya memiliki `ParentProcessId` 16220; process dengan PID 16220 memiliki parent PID 15444. PID historis 8808 berbeda dari keenam PID tersebut. Parent process yang ada sekarang **tidak** membuktikan siapa parent historis PID 8808.

### 5. Menampilkan sepuluh catatan endpoint UDP saat ini

```powershell
Get-NetUDPEndpoint |
Select-Object -First 10 LocalAddress, LocalPort, OwningProcess
```

Cuplikan endpoint berhasil diperoleh. `OwningProcess` berisi PID process saat ini. Alamat IP lokal yang sebenarnya tidak dicantumkan.

### 6. Memeriksa apakah PID historis 8808 memiliki endpoint UDP saat ini

```powershell
Get-NetUDPEndpoint -OwningProcess 8808 |
Select-Object LocalAddress, LocalPort, OwningProcess
```

Hasilnya `No MSFT_NetUDPEndpoint objects found ... OwningProcess equal to 8808`. Artinya tidak ada endpoint UDP SAAT INI yang cocok, bukan berarti tidak pernah ada aktivitas sebelumnya.

### 7. Memeriksa jalur file executable

```powershell
$file = Join-Path $env:LOCALAPPDATA 'Discord\app-1.0.9261\Discord.exe'
Test-Path -LiteralPath $file
```

Hasil: `True` untuk file saat ini.

### 8. Membaca metadata file

```powershell
Get-Item -LiteralPath $file |
Select-Object Name, Length, CreationTime, LastWriteTime
```

Hasil pengamatan: Discord.exe, `Length 220542392 bytes`, `CreationTime 2026-10-08 16:02:01`, dan `LastWriteTime 2026-10-08 16:02:04` (waktu lokal). Ini adalah timestamp sistem file, bukan waktu pertama file dipastikan dijalankan.

### 9. Memeriksa tanda tangan digital Authenticode

```powershell
Get-AuthenticodeSignature -LiteralPath $file |
Format-List Status, StatusMessage, SignerCertificate
```

Hasil pengamatan: `Status: Valid`; `StatusMessage: Signature verified.`; subjek penanda tangan adalah Discord Inc. Verifikasi ini berlaku untuk file yang diperiksa saat pengujian dan tidak menjamin bahwa perilaku jaringannya pasti wajar.

### 10. Menghitung SHA256

```powershell
Get-FileHash -LiteralPath $file -Algorithm SHA256
```

Hasil pengamatan: hash berhasil dihitung; SHA256 diawali `08F12BDED9410C45...`. Hash adalah sidik jari file, bukan hasil pemindaian antivirus. Jangan menganggap byte file historis identik dengan file saat ini tanpa verifikasi.

## Perintah yang ditugaskan dalam lab, tetapi hasilnya BELUM diberikan

Perintah `Get-WinEvent` dengan `StartTime`, `EndTime`, dan `Where-Object` sudah dibahas dalam materi. Kamu menjawab P17–P19, tetapi belum memberikan hasil aktual dari kedua perintah tersebut; karena itu, tidak ada catatan historis Discord tambahan yang dapat diverifikasi.

## Privasi

Semua nama lokal pribadi, hostname, ProcessGuid khusus host, alamat IP lokal/privat, dan jalur profil yang dapat mengungkap identitas telah disamarkan dalam materi repo.
