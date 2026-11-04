# Day 21 — Materi: Network Fundamentals I

Status: **IN PROGRESS** — belum ada nilai akhir. Materi Indonesia dan praktik PowerShell read-only.

## IP, subnet dan prefix
Alamat host IPv4 memiliki network prefix dan host portion. Untuk subnet /24, subnet mask 255.255.255.0; alamat 192.168.50.10/24 dan 192.168.50.20/24 berada dalam network 192.168.50.0/24, sedangkan 192.168.60.30/24 berbeda. Perbandingan tiga oktet pertama hanya penyederhanaan untuk /24. Pada /25, 192.168.1.10/25 dan 192.168.1.200/25 justru berbeda subnet. PrefixLength harus dibaca per interface.

## IP khusus
127.0.0.1 adalah loopback, untuk host sendiri. 169.254.0.0/16 adalah IPv4 link-local, tidak untuk routing internet. AddressState Tentative berarti alamat sedang diuji keunikannya sebelum siap digunakan. Banyak interface dimungkinkan karena Ethernet, Wi-Fi, virtual adapters, VPN, serta loopback.

## Gateway dan DNS
Default route 0.0.0.0/0 mengarahkan trafik yang tidak cocok dengan route lebih spesifik ke next hop. Gateway dan DNS server dapat berada pada router yang sama tetapi tugasnya berbeda. DNS menerjemahkan nama domain; DNS berhasil tidak membuktikan koneksi TCP ke server berhasil.

## TCP dan UDP
TCP connection-oriented; handshake SYN/SYN-ACK/ACK dapat menghasilkan state Established. UDP connectionless pada transport layer, tanpa TCP state Established. UDP dapat dipakai protokol aplikasi yang mempunyai reliability sendiri. Port 443 tidak membuktikan protokol TCP atau isi HTTPS; selalu baca Protocol dan bukti aplikasi.

## NAT, HTTP, HTTPS/TLS, firewall, proxy, VPN
NAT menerjemahkan alamat/port (umum pada router), bukan otomatis membuat endpoint aman. HTTP merupakan protokol aplikasi web; HTTPS menggunakan TLS untuk perlindungan koneksi, bukan bukti website jujur. Firewall menerapkan aturan allow/block. Proxy meneruskan permintaan sebagai perantara. VPN membuat jaringan/tunnel virtual.

## Kualitas evidence
Assessment merupakan penilaian status dan alasan berbasis bukti, bukan daftar kegiatan mengumpulkan data. Next Action merupakan tindakan pemeriksaan lanjutan yang spesifik. File hash berbeda berarti isi byte tidak identik, tetapi perubahan tidak otomatis malicious. Proses sama dibuktikan dengan ProcessGuid dan konteks, bukan waktu yang dekat saja.

## Mini-project
Local Network Investigation Report: konfigurasi interface/prefix, DNS response, TCP test, satu contoh TCP Established connection dan nama process owner, Evidence Available/Gap, Assessment, Next Action. Komponen koneksi aktual dan owner masih perlu dibuktikan dari output. Laporan dapat disusun secara terbimbing.

## Rujukan
- [Microsoft Get-NetIPAddress](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-netipaddress)
- [Microsoft Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection)
- [RFC 3927 IPv4 Link-Local](https://www.rfc-editor.org/rfc/rfc3927.html)
