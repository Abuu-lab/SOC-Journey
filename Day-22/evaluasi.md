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