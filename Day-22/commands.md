# Day 22 — Commands dan hasil praktik

**Status: IN PROGRESS. Skor sementara 72/100, belum PASS.** Perintah berikut benar-benar dijalankan; output identitas perangkat disamarkan.

## 1. Ethernet IPv4
```powershell
Get-NetIPAddress -InterfaceAlias 'Ethernet' -AddressFamily IPv4 |
Select-Object InterfaceAlias, IPAddress, PrefixLength, AddressState
```
Hasil: Ethernet mempunyai IPv4 privat [REDACTED], PrefixLength 24, AddressState Preferred. Untuk /24, subnet mask adalah 255.255.255.0.

## 2. Routing table IPv4
```powershell
Get-NetRoute -AddressFamily IPv4 |
Select-Object InterfaceAlias, DestinationPrefix, NextHop, RouteMetric
```
Hasil: Ethernet memiliki route subnet lokal [REDACTED_LOCAL_SUBNET]/24 dengan NextHop 0.0.0.0 (on-link), dan default route 0.0.0.0/0 dengan NextHop [REDACTED_GATEWAY]. Ada pula routes multicast, loopback, dan interface virtual. Jawaban pelajar pada P6 salah menunjuk LAC*, bukan Ethernet on-link.

## 3. DNS
```powershell
Resolve-DnsName -Name example.com -Type A -DnsOnly |
Select-Object Name, Type, IPAddress

Get-DnsClientServerAddress -AddressFamily IPv4 |
Select-Object InterfaceAlias, ServerAddresses
```
Hasil: dua record A untuk example.com: 104.20.23.154 dan 172.66.147.243. DNS server Ethernet sama dengan alamat router lokal [REDACTED_GATEWAY] pada saat uji. Kesamaan alamat tidak menjadikan fungsi DNS dan gateway identik.

## 4. TCP test
```powershell
Test-NetConnection -ComputerName example.com -Port 443 -InformationLevel Detailed
```
Hasil: TcpTestSucceeded True, RemoteAddress 172.66.147.243, RemotePort 443, interface Ethernet, source IP privat [REDACTED]. RemoteAddress cocok dengan salah satu DNS A records. Koneksi TCP berhasil tidak memverifikasi sertifikat TLS atau respons HTTP.

## 5. Firewall
```powershell
Get-NetFirewallProfile |
Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```
Hasil: Domain, Private, Public Enabled True; kedua Default Action bernilai NotConfigured pada output yang dikirim. Jangan menyatakan kebijakan efektif tanpa pemeriksaan lanjutan.

## 6. TCP dan UDP endpoints
```powershell
Get-NetTCPConnection -State Established |
Select-Object -First 5 LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess

Get-NetUDPEndpoint |
Select-Object -First 5 LocalAddress, LocalPort, OwningProcess
```
Hasil: empat koneksi TCP Established menggunakan IPv6 loopback ::1 dengan PID 23160/17768. Satu koneksi TCP IPv4 menuju remote 104.18.32.47:443 dimiliki PID 3220; nama executable PID3220 belum diperiksa. Lima UDP endpoint mempunyai LocalAddress/LocalPort/OwningProcess tetapi tidak mempunyai TCP State Established.

## Belum dilakukan
Mini-project report lengkap yang ditulis mandiri, serta remedial subnet, DNS/TCP, route dan firewall. Tidak ada perintah konfigurasi firewall maupun perubahan sistem yang dijalankan.

**Privasi:** raw IP privat, hostname, username dan alamat profil tidak dipublikasikan.