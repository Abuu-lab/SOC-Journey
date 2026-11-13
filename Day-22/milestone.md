# 📌 Day 22 — Catatan Penting / Milestone Sementara

**Status IN PROGRESS — 72/100 sementara, belum PASS.** Nilai mentor formatif.

## Yang telah benar-benar dilakukan
- Membaca Ethernet IPv4 Preferred /24 menggunakan Get-NetIPAddress.
- Membaca tabel route IPv4 dan menemukan default route serta on-link route lokal, walaupun interpretasi on-link masih perlu penguatan.
- Melakukan DNS query example.com yang berhasil mengembalikan dua IPv4, membaca DNS server router.
- Melakukan TCP port443 test, hasil True pada salah satu IP hasil DNS.
- Melihat tiga profil firewall aktif dengan Default Inbound dan Outbound NotConfigured.
- Mengamati empat TCP Established via IPv6 loopback ::1 serta satu TCP IPv4 remote port443; membandingkan dengan daftar UDP endpoints tanpa State.
- Menjawab Active Recall dan SOC Challenge; mengenali bahwa tidak semua trafik pada port443 aman serta kegagalan TCP tidak otomatis malware.

## Yang belum dikuasai/selesai
- Menentukan network address /26 dan membedakan subnet 0–63 vs 64–127.
- Subnet mask /24, loopback dan pemisahan DNS resolution vs TCP connection.
- Route Ethernet on-link dibanding interface virtual, dan pengecualian firewall default Block.
- Dua hipotesis penyebab TCP failure, Assessment + Evidence Gap + Next Actions, serta report akhir yang ditulis sendiri.

## Remedial
Empat pertanyaan singkat dengan skenario berbeda dan laporan pendek menggunakan hasil yang sudah diperoleh. Jangan mengulang seluruh Day; jika nilai mencapai 76, lanjut Day23. Penilaian kemandirian masih Terpandu/Bantuan Terbatas.

Privasi: IP privat, hostname, username dan host-specific identifiers dihilangkan dari repository.
## 📌 Milestone tambahan — subnet /26 terbukti melalui latihan baru

Pelajar menjawab mandiri contoh baru PC A 192.168.20.75/26 dan PC B 192.168.20.110/26: kedua network address **192.168.20.64/26**, sama subnet, dinilai **3/3 benar**. Ini progress yang benar-benar diuji setelah materi demonstrasi. Pemahaman routing dan laporan SOC masih perlu diuji secara bertahap. Prinsip mentoring diperbarui: **jelaskan konsep dan contoh dahulu jika belum paham, lalu satu soal penguatan, bukan langsung banyak remedial**.
