# Day 23 — Commands Lab Network Web Connectivity

**STATUS:** daftar perintah dan panduan; **belum ada bukti perintah dijalankan pada Day23**. Jangan mencatat contoh sebagai output peserta.

## Tahap pertama — jalankan satu-satu di PowerShell

### 1. Cek program curl
```powershell
curl.exe --version
```
Fungsi: memastikan program command-line curl tersedia. Tidak mengakses jaringan dengan sendiri.

### 2. HTTPS HEAD request
```powershell
curl.exe --head --max-time 15 https://example.com
```
Fungsi: meminta response headers HTTP lewat HTTPS dengan durasi maksimum 15 detik. `curl.exe` dipakai agar tidak tertukar dengan alias `curl` pada beberapa versi PowerShell. Perintah ini mengirim request jaringan keluar ke example.com. Tidak mengubah pengaturan Windows.

**Interpretasi:** `HTTP/2 200` atau `HTTP/1.1 200 OK` menandakan response 200 terhadap request yang diuji. 301/302 = redirect, 403 = forbidden, 404 = not found. Bisa juga error timeout/proxy/TLS. Catat hasil yang benar-benar keluar, bukan yang diharapkan.

### 3. Diagnosis detail jika relevan
```powershell
curl.exe --verbose --head --max-time 15 https://example.com
```
Fungsi: menunjukkan proses komunikasi lebih rinci. Output bisa berubah sesuai backend TLS, proxy, OS. Jangan unggah informasi rahasia dari header/token.

## Tahap lanjut — setelah hasil pertama dievaluasi

### 4. DNS A records
```powershell
Resolve-DnsName -Name example.com -Type A -DnsOnly |
Select-Object Name, Type, IPAddress
```
DNS query berfungsi mengamati jawaban IPv4; tidak membuktikan TCP/TLS/HTTP.

### 5. Uji TCP 443
```powershell
Test-NetConnection -ComputerName example.com -Port 443 -InformationLevel Detailed
```
Perhatikan TcpTestSucceeded dan RemoteAddress. TCP True tidak membuktikan HTTP status ataupun validasi sertifikat.

### 6. Cari owner PID koneksi aktif
```powershell
Get-NetTCPConnection -State Established |
Select-Object -First 10 LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```
Jika memilih PID tertentu, gunakan:
```powershell
Get-CimInstance Win32_Process -Filter 'ProcessId = 1234' |
Select-Object ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine
```
Ganti PID 1234 dengan PID nyata yang aktif. CommandLine berpotensi mengandung token/parameter privat sehingga redaksi sebelum publikasi. Tidak semua koneksi aktif di port 443; koneksi bisa langsung tertutup.

## Prinsip keselamatan lab
- Perintah-perintah di atas tidak mengubah konfigurasi firewall atau routing.
- Hanya target domain latihan example.com; jangan melakukan pemindaian atau percobaan akses ke server tak diotorisasi.
- Jangan menambahkan `-k` atau `--insecure` untuk menutupi error TLS.
- Redaksi hostname, private IP, username, token, cookies dan informasi sensitif sebelum menyimpan ke GitHub.