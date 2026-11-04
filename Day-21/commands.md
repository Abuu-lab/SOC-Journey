# Day 21 — Commands (Praktik)

Status: **IN PROGRESS**. Tema: IP, subnet, DNS, routing, TCP dan investigasi jaringan. Nilai akhir belum ditetapkan.

## Perintah yang sudah dijalankan pengguna

### 1. Konfigurasi interface
```powershell
Get-NetIPConfiguration
```
Hasil: Ethernet aktif dengan IPv4 privat [REDACTED_PRIVATE_IP], default gateway [REDACTED_GATEWAY], DNS server IPv4 router dan resolver IPv6 link-local. InterfaceIndex belum dipahami.

### 2. IPv4 Prefix
```powershell
Get-NetIPAddress -AddressFamily IPv4 |
Select-Object InterfaceAlias, IPAddress, PrefixLength, AddressState
```
Hasil: Ethernet **PrefixLength /24**, AddressState Preferred. Loopback 127.0.0.1/8 Preferred. Beberapa interface lain menggunakan alamat IPv4 link-local 169.254.0.0/16 dengan state Tentative. Nilai PrefixLength tidak dijumlahkan untuk menentukan subnet Ethernet.

### 3. Default route
```powershell
Get-NetRoute -DestinationPrefix '0.0.0.0/0' |
Select-Object InterfaceAlias, DestinationPrefix, NextHop, RouteMetric
```
Hasil: satu default route untuk Ethernet, NextHop [REDACTED_GATEWAY], RouteMetric 0. Gateway merupakan next hop, bukan sinonim NAT.

### 4. DNS resolution
```powershell
Resolve-DnsName -Name example.com -Type A -DnsOnly |
Select-Object Name, Type, IPAddress
```
Hasil: berhasil, dua record A: 104.20.23.154 dan 172.66.147.243. DNS sukses sendiri tidak membuktikan TCP berhasil.

### 5. TCP connection test
```powershell
Test-NetConnection -ComputerName example.com -Port 443 -InformationLevel Detailed
```
Hasil: TcpTestSucceeded True, RemoteAddress 172.66.147.243 dan RemotePort 443. NameResolutionResults memuat dua IP hasil DNS. SourceAddress adalah alamat lokal privat (direduksi). NetRoute NextHop = router lokal (direduksi). Ini tes TCP, bukan pemeriksaan isi HTTPS/TLS.

### 6. Current connections
Pelajar melaporkan sepuluh entri yang ditampilkan, tetapi **tidak mengirim output lengkap** dari command berikut:
```powershell
Get-NetTCPConnection -State Established |
Select-Object -First 10 LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```
OwningProcess konkret dan nama process belum diverifikasi.

## Command lanjutan untuk mini-project (DIBERIKAN, BELUM DIJALANKAN/OUTPUT BELUM DIKIRIM)
```powershell
Get-NetTCPConnection -State Established |
Select-Object -First 5 LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```
Pilih satu PID aktual dan cari identitasnya:
```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 1234" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath
```
Ganti 1234 dengan OwningProcess yang ditemukan. Jangan mengklaim koneksi historis memiliki instance sama hanya karena PID cocok.

## Privasi
Username, hostname, alamat IP lokal, serta informasi identitas perangkat tidak dicantumkan pada repository publik.
