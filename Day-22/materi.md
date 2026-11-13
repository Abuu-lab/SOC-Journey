# Day 22 — Materi: Subnetting, Routing, DNS, Firewall dan TCP/UDP

**Status IN PROGRESS; skor sementara 72/100.** Penjelasan utama Bahasa Indonesia.

## Subnetting
PrefixLength adalah konfigurasi **per interface**, tidak dijumlahkan. /24 memiliki mask 255.255.255.0. /26 memiliki mask 255.255.255.192 dan membentuk blok 64 alamat: 0–63, 64–127, 128–191, 192–255 pada oktet terakhir untuk contoh 192.168.10.x.
Contoh challenge: 192.168.10.50/26 mempunyai network address **192.168.10.0/26**; 192.168.10.60/26 masih dalam subnet itu; 192.168.10.70/26 berada pada **192.168.10.64/26**. Membandingkan oktet ketiga saja salah untuk /26.

## Route dan default gateway
Route on-link mempunyai NextHop 0.0.0.0 pada output Windows, misalnya prefix jaringan lokal Ethernet. Default route 0.0.0.0/0 memiliki next hop router. Windows memilih prefix route paling spesifik yang cocok sebelum mempertimbangkan metric untuk alternatif berkecocokan setara. NAT merupakan fungsi penerjemahan IP/port, bukan sinonim default gateway.

## DNS dan konektivitas
DNS resolver mengembalikan jawaban rekaman domain. DNS success tidak menjamin TCP success. TCP success tidak menjamin TLS sertifikat diverifikasi atau permintaan HTTP berhasil. DNS server dapat berupa router yang juga bertindak sebagai gateway, tetapi fungsi berbeda.

## Loopback, TCP dan UDP
127.0.0.1 adalah IPv4 loopback; ::1 adalah IPv6 loopback. TCP Established mempunyai koneksi berpasangan local/remote endpoints. UDP connectionless tidak memakai state TCP Established; Get-NetUDPEndpoint menunjukkan UDP endpoint lokal saja dan bukan seluruh tujuan komunikasi UDP.

## Windows Firewall
Get-NetFirewallProfile dapat menampilkan Domain/Private/Public, Enabled dan kebijakan. DefaultInboundAction Block untuk traffic yang tidak match rule tidak berarti semua inbound tanpa pengecualian terblokir; rule Allow efektif dapat diterapkan. Output NotConfigured tidak boleh langsung dibaca sebagai Block ataupun Allow. Lihat docs Microsoft https://learn.microsoft.com/en-us/powershell/module/netsecurity/set-netfirewallprofile

## SOC Reporting
Observation = fakta dari output. Evidence Gap = hal yang belum diuji. Assessment = kesimpulan sementara dan alasannya. Next Actions = langkah konkret untuk memeriksa hipotesis. Pada error TCP, pertimbangkan firewall/middlebox, route, host destination, dan port layanan; tidak semua failure adalah insiden keamanan.

## Latihan penguatan
Remedial pendek berisi empat pertanyaan dengan data baru tentang /26, on-link vs default, DNS vs TCP, serta firewall policy. Penulisan mini-report dari output lama masih diperlukan, tanpa mengulang seluruh command.