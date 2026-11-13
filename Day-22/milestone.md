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


## Tambahan sesi penguatan routing

Pelajar berhasil sebelumnya pada satu latihan subnetting `/26` (3/3), tetapi ketika memilih route untuk tujuan `10.10.5.80`, salah menganggap network `10.10.5.0/24` hanya mencakup 0–63. Mentor menjelaskan bahwa rentang 0–63 adalah contoh blok `/26`, sedangkan `/24` mencakup 0–255. Route lebih spesifik `10.10.5.0/24` mengungguli default route `0.0.0.0/0` untuk tujuan tersebut. **Pemahaman routing belum dikonfirmasi lewat jawaban baru; status dan skor tidak berubah.**

## 📌 Tambahan perkembangan routing
Peserta mengidentifikasi bahwa destination `192.168.40.90` masuk blok di atas 63, tetapi masih memilih on-link route `192.168.40.0/26`, yang hanya mencakup 0–63. Mentor membimbing: block yang benar adalah 64–127 dengan network `192.168.40.64/26`; route itu dan route /25 tidak tercantum di table sehingga default route yang dipakai. Konsep penting: menghitung network address tidak sama dengan memilih route yang tersedia; gunakan longest-prefix match dari routing table yang nyata. Status Day22 masih latihan, skor sementara 72/100.
