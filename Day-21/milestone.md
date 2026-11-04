# 📌 Day 21 — Milestone Sementara

**IN PROGRESS — belum ada nilai akhir.** Tahap 2 roadmap SOC: Network Fundamentals I.

Kemampuan yang sudah dipraktikkan:
- Menggunakan Get-NetIPConfiguration untuk melihat interface Ethernet, IP/gateway/DNS.
- Membaca Get-NetIPAddress: Ethernet /24 Preferred, loopback, dan beberapa alamat IPv4 link-local.
- Mengambil Get-NetRoute default IPv4 route dan next hop.
- Melakukan Resolve-DnsName example.com: mendapatkan dua record A.
- Menggunakan Test-NetConnection example.com:443: TcpTestSucceeded True.
- Mengerjakan active recall dan skenario SOC #021 secara parsial.
- Melaporkan contoh current Established TCP connections, tetapi output lengkap/owner belum dikirim.

Kelemahan konsep yang ditemukan: PrefixLength bukan penjumlahan antarinterface, loopback, DNS tidak menjamin TCP, default gateway bukan NAT, correlation butuh lebih dari ProcessGuid saja, Assessment harus berbasis bukti, serta dua hipotesis alternatif masih perlu latihan.

Mini-project Local Network Investigation Report belum selesai dan nilai final belum ditetapkan. Materi tambahan serta contoh model laporan disediakan oleh mentor, bukan hasil kerja mandiri.

Pengamanan data: username/hostname, alamat privat dan identitas host tidak dipublikasikan.
