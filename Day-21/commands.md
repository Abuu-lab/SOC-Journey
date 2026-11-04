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


## Mini-project follow-up — koneksi TCP dan OwningProcess (SUDAH DIJALANKAN)

User mengirim output satu current TCP Established connection:
- LocalAddress: [REDACTED_PRIVATE_IP]; LocalPort 65514.
- RemoteAddress: 104.89.118.72; RemotePort 443.
- OwningProcess: 10512.

User lalu menjalankan:
```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 10512" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Hasil: Name **msedgewebview2.exe**, PID 10512, ParentProcessId 4100, executable di lokasi instalasi Microsoft Edge WebView2 versi 156.0.4314.58. Argumen menyebut `--type=utility`, `--utility-sub-type=network.mojom.NetworkService`, dan `--webview-exe-name=Widgets.exe`, serta data directory Windows Client WebExperience. Ini **konsisten dengan kemungkinan Windows Widgets/WebView2**; belum merupakan verdict legitimate.

**Evidence gap:** identitas parent PID 4100 dan verifikasi signature/hash file msedgewebview2.exe belum dijalankan. Pemilik connection adalah hasil snapshot, tidak otomatis berkaitan dengan event historis.

Next command diberikan (belum ada output):
```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 4100" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

## Mini-project — Parent PID 4100 (SUDAH DIJALANKAN)

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 4100" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

Hasil nyata: Name `msedgewebview2.exe`, ProcessId `4100`, ParentProcessId `6200`. ExecutablePath merujuk ke instalasi EdgeWebView versi `156.0.4314.58`. CommandLine memiliki argumen `--embedded-browser-webview=1`, `--webview-exe-name=Widgets.exe`, `--edge-webview-is-background`, serta konfigurasi WebView2. Username dan data path pengguna disamarkan. Parent PID6200 BELUM diperiksa. Signature dan hash BELUM dijalankan pada sesi ini.

Rangkaian snapshot: TCP Established OwningProcess10512 → WebView2 PID10512 ParentPID4100 → WebView2 PID4100 ParentPID6200 (belum terverifikasi prosesnya).
