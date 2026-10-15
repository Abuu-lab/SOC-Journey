# Day 19 — Verifikasi Process

**Hasil: PASS 80/100 (penilaian formatif mentor).** Laporan akhir disusun dengan bantuan mentor.

Materi yang dilatih: process saat ini dibandingkan event Sysmon historis; PID dan parent PID; identitas instance process; metadata file, signature Authenticode dan SHA-256; linimasa event Windows dan penilaian insiden.

Praktik aktual: enam process Discord.exe yang sedang berjalan diamati. Satu event pembuatan process Office yang terpisah juga diperiksa. Tiga event jaringan Spotify diperiksa: dua catatan UDP untuk PID historis 6680 dan satu catatan TCP dari instance process lain. Pencarian berdasarkan rentang waktu kemudian menemukan lima event pembuatan process anak Spotify dengan parent PID 6680. Pengenal instance parent yang sama cocok dengan catatan UDP historis tersebut.

Keterbatasan: event yang mencatat pembuatan process parent itu sendiri tidak ditemukan di antara lima catatan process anak. File Spotify dan catatan DNS yang berkaitan belum diverifikasi. Belum ada kesimpulan bahwa aktivitas tersebut malicious ataupun false positive.

Ulangi pada Day 20: perbedaan PID dan PPID; pengenal instance child dan parent; sidik jari SHA-256; false positive dibandingkan hasil yang belum dapat dipastikan; serta korelasi berdasarkan waktu yang akurat.
