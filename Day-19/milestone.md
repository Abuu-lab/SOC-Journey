# Day 19 — Milestone Hari Ini

**PASS — 80/100 (penilaian formatif mentor).**

Yang telah dilakukan: mencatat process Windows yang sedang berjalan; memeriksa catatan historis pembuatan process Sysmon; memeriksa metadata, signature, dan hash file Discord; serta mengamati tiga event jaringan Spotify.

Hasil utama: pencarian berdasarkan rentang waktu menemukan lima event pembuatan process anak Spotify yang memiliki parent PID 6680. Pengenal instance parent yang sama cocok dengan event jaringan UDP yang sebelumnya dikumpulkan untuk PID 6680. Ini mendukung korelasi antara hubungan parent-child dan bukti jaringan.

Kesenjangan bukti: catatan pembuatan parent itu sendiri, verifikasi file Spotify, dan telemetry DNS terkait belum dikonfirmasi. Laporan akhir dan jawaban tantangan simulasi yang belum terjawab disusun dengan bantuan mentor, bukan diselesaikan secara mandiri.

Topik Active Recall Day 20: PID/PPID; identitas instance process; perbedaan fungsi SHA-256 dan pemindaian malware; false positive dibandingkan bukti yang belum cukup; serta penyusunan linimasa yang cermat berdasarkan bukti.

Aturan penilaian: nilai 76–100 berarti PASS dan lanjut. Jika di bawah 76, berikan penjelasan singkat yang berfokus pada kelemahan, lalu 2–5 soal/praktik remedial dan nilai kembali tanpa mengulang seluruh materi hari itu.

Privasi: pengenal perangkat/akun tidak dipublikasikan.
