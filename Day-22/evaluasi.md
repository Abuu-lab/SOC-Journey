# Day 22 — Evaluasi Sementara

**IN PROGRESS, skor sementara 72/100, belum PASS (ambang 76).** Kiriman pengguna dua kali berisi jawaban sama; hanya dinilai sekali. Semua hands-on sudah dikerjakan. Mini-project terstruktur belum diserahkan.

## Rubrik sementara
| Aspek | Maks | Skor |
|---|---:|---:|
| Pemahaman konsep | 20 | 13 |
| Pengumpulan dan pembacaan bukti | 25 | 24 |
| Analisis dan penalaran | 25 | 16 |
| Tindak lanjut dan eskalasi | 15 | 9 |
| Dokumentasi dan komunikasi | 15 | 10 |
| **Total** | **100** | **72** |

## Active Recall — Jawaban dan koreksi
- Q1: 'belum paham' -> prefix Ethernet tidak dijumlahkan dengan prefix loopback.
- Q2: 'Default Gateway ada IP router; NAT mengubah IP private menjadi Publik' -> inti benar; NAT translation dan default route merupakan fungsi berbeda.
- Q3: 'benar terhubung' setelah DNS berhasil -> salah; DNS tidak menjamin TCP berhasil.
- Q4: '127.0.0.1 ... IP untuk alamat' -> belum tepat; loopback ke host sendiri.
- Q5: 'local alamat IP PC; remote alamat IP tujuan' -> benar inti, dapat juga loopback.
- Q6: 'belum tentu aman' untuk TCP443 -> benar.
- Q7: 'laporan sementara berdasarkan bukti-bukti' -> benar inti sebagai Assessment.
- Q8: 'ya ... angka 168 sama dan /24' -> jawaban benar, alasan salah; untuk /24 perlu tiga oktet pertama sama sesuai prefix.

## Hands-on — Hasil nyata
- Ethernet PrefixLength 24 Preferred. P1 benar; P2 jawab 0–32 bukan subnet mask 255.255.255.0; P3 inti benar.
- Default route ditemukan, next hop router benar. P6 salah menunjuk virtual LAC, padahal route lokal Ethernet prefix [REDACTED_SUBNET]/24 dengan NextHop 0.0.0.0.
- DNS example.com mengembalikan dua IPv4, DNS server Ethernet pada router. P7 dan P8 benar, P9 belum memahami DNS resolver.
- Test-NetConnection TCP443 True, RemoteAddress salah satu DNS answer. P10 benar; P11 belum bisa membandingkan; P12 benar TCP bukan pembuktian TLS/HTTP.
- Firewall Domain/Private/Public Enabled True dengan DefaultActions NotConfigured. P13–P14 benar; P15 salah, default Block tidak berarti semua inbound diblokir tanpa pengecualian.
- Empat TCP Established IPv6 ::1 plus satu IPv4 TCP remote443. Lima UDP endpoints. P16 benar UDP tak punya Established, P17 belum memahami perbedaan jenis tampilan, P18 benar owning PID dapat dipakai mencari executable.

## Simulated SOC Challenge #022
- C1 menjawab 192.168.10.1 -> salah; network 192.168.10.0/26.
- C2 'sama karena oktet ketiga 10' -> keputusan benar, alasan belum benar; 50 dan 60 masih blok 0–63 pada /26.
- C3 'masih sama karena oktet ketiga' -> salah; 70 berada pada network 192.168.10.64/26.
- C4 'gateway jembatan dari lokal ke luar' -> inti benar.
- C5 DNS berhasil -> benar.
- C6 TCP False -> benar.
- C7 firewall mungkin penyebab -> kemungkinan wajar, bukan terbukti pasti.
- C8 default outbound Allow tidak menjamin konektivitas -> benar.
- C9 menyebut kemungkinan firewall dan faktor lain tanpa dua hipotesis spesifik; fokus next action diarahkan ke file signature/hash tanpa membedakan gangguan konektivitas dari dugaan malware.
- C10 belum disusun terpisah sebagai Assessment, dua Evidence Gaps dan dua Next Actions.

## Mini-project
Pertanyaan pengguna: 'bagian 5 sepertinya sudah kutulis ya?' Observasi dan output sudah dikirim, tetapi **belum berupa laporan mandiri dengan struktur yang diminta**. Mentor akan menyiapkan tabel observation berbasis hasil dan meminta pengguna melengkapi Analysis/Assessment/Next Action. Tidak perlu menjalankan ulang command.

## Target penyelesaian
Jawab empat remedial dengan data berbeda dan susun paragraf singkat penilaian kasus nyata. Setelah itu skor final dapat ditentukan. Jika >=76, lanjut Day23 dan kesalahan kecil masuk Active Recall.
## Pembelajaran terarah setelah penilaian sementara

Pengguna meminta mentor menjelaskan dasar konsep sebelum remedial saat menjawab 'belum paham' atau keliru total. Mentor sudah mendemonstrasikan subnetting /26 (blok 64), route paling spesifik /24 dibanding default /0, pemisahan DNS versus TCP, serta pengecualian rule Allow ketika firewall default Block. Pada penguatan R3, pengguna menjawab DNS tidak selalu bermasalah dan firewall bisa jadi penyebab: **benar secara inti**. Pada R4, pengguna menjawab rule Allow TCP8080 dapat menjadi pengecualian Block: **benar secara inti** dengan syarat rule efektif.

**Latihan satu soal berbeda — sudah dijawab pengguna:** PC A `192.168.20.75/26`, PC B `192.168.20.110/26`.
- Jawaban asli pengguna: "Network PC A: 192.168.20.64/26; Network PC B: 192.168.20.64/26; Subnet sama? Ya".
- Hasil: **benar 3/3**; keduanya berada pada blok 64–127 dari subnet 192.168.20.64/26.
- Ini membuktikan peningkatan pemahaman untuk satu contoh /26, bukan berarti seluruh skill routing dan laporan SOC sudah dinilai mandiri.

**Langkah berikutnya:** satu latihan pemilihan route dengan data baru lalu mini-project assessment ringkas; skor akhir Day22 belum diputuskan. Skor 72/100 adalah hasil evaluasi awal dan belum dihitung ulang setelah penguatan.


## Latihan routing setelah remedial subnetting

**Jawaban asli peserta:**
- Route pilihan: `0.0.0.0/0 (Gateway)`.
- Alasan: `karena jawaban 10.10.5.0/24 berada di subnet 0-63`.

**Evaluasi: belum tepat.** Peserta menerapkan blok 0–63 yang berlaku untuk subnet `/26` pada jaringan `10.10.5.0/24`. Prefix `/24` memakai mask `255.255.255.0` dan mencakup alamat `10.10.5.0` sampai `10.10.5.255`. Destination `10.10.5.80` cocok dengan route `10.10.5.0/24`, yang lebih spesifik daripada `0.0.0.0/0`; NextHop `0.0.0.0` berarti on-link sehingga route `10.10.5.0/24` yang dipilih.

**Metode koreksi:** mentor memberi penjelasan /24 versus /26, membedakan jumlah alamat per blok, dan memperagakan pemilihan route paling spesifik. Bukan langsung memberi remedial baru tanpa pengajaran. Satu pertanyaan analogi diberikan untuk mengecek apakah kesalahan konsep telah diperbaiki, belum ada jawaban dari pengguna saat catatan ini ditulis.

**Status Day22:** masih 72/100 sementara, belum PASS; jangan klaim remedial routing sudah benar.

## Latihan routing lanjutan

**Soal:** route 192.168.40.0/26 on-link dan default 0.0.0.0/0 via 192.168.40.1; destination 192.168.40.90.

**Jawaban asli pengguna:** Route 192.168.40.0/26 (On-link). Alasan: angka 90 termasuk 64-128, tapi harusnya /25.

**Hasil:** Pemilihan route belum tepat. Blok /26 yang memuat angka 90 adalah 64-127, dengan network address 192.168.40.64/26. Route itu tidak ada dalam tabel; route 192.168.40.0/26 hanya mencakup 0-63. Default route 0.0.0.0/0 via 192.168.40.1 merupakan satu-satunya route tabel yang cocok. Jaringan 192.168.40.0/25 secara matematis juga mencakup 90 tetapi tidak ada di tabel, jadi jangan mengarang route.

**Proses belajar:** Mentor menjelaskan perbedaan perhitungan network address dan pemilihan route dari routing table yang tersedia sebelum memberi latihan lanjutan. Skor sementara tetap 72/100 dan belum PASS.

## Cek pemahaman routing — hasil terbaru

**Soal penguatan:** destination 192.168.40.90; tersedia 192.168.40.0/26 (on-link) dan 0.0.0.0/0 (gateway).

**Jawaban asli peserta:**
- Network /26 yang memuat .90: 192.168.40.64/26.
- Route yang tersedia dan dipilih: 0.0.0.0/0 (Gateway).

**Evaluasi: benar 2/2.** Network yang memuat IP tidak otomatis merupakan route yang tersedia. Karena route 192.168.40.0/26 hanya mencakup alamat .0–.63, untuk .90 hanya default 0.0.0.0/0 yang cocok.

Riwayat pembelajaran dijaga: awalnya salah memilih route /26 on-link; setelah contoh dan penjelasan berhasil menjawab kasus yang sama secara tepat. Latihan subnet /26 sebelumnya juga benar 3/3. Langkah selanjutnya penulisan Assessment mini-project dari DNS example.com berhasil, TCP port443 berhasil, dan batas uji. **Belum ada laporan akhir tertulis peserta** dan nilai final belum ditetapkan.
