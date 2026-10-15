# 📌 Day 18 — Extended Hands-on Milestone

## Status
Praktik tambahan pada bagian 4–8, tantangan simulasi, dan sebagian laporan telah dikerjakan/dikirim. Status keseluruhan Day 18 tetap PASS berdasarkan nilai formatif sebelumnya, 90/100; latihan tambahan menunjukkan beberapa hal yang perlu diperkuat pada Day 19. Belum ada nilai angka baru untuk praktik tambahan ini.

## Praktik yang benar-benar dilakukan
- Membaca timestamp dari lima catatan historis Sysmon ID3.
- Membaca pesan lengkap dari tiga event jaringan: semuanya milik Spotify.exe dan menggunakan UDP; dua event memiliki ProcessGuid yang sama, sedangkan satu berasal dari instance process lain.
- Membedakan port sumber lokal dari port tujuan serta memahami `Initiated true`.
- Memeriksa PID historis 8808 terhadap daftar process saat ini; PID tersebut tidak ditemukan.
- Mencatat enam process Discord.exe yang sedang berjalan.
- Memeriksa endpoint UDP saat ini dan memastikan tidak ada yang cocok dengan PID historis 8808.
- Memeriksa keberadaan file executable Discord dengan `Test-Path`.
- Mengambil metadata file: ukuran 220542392 byte dan timestamp.
- Memeriksa Authenticode: status `Valid`, penanda tangan Discord Inc.
- Menghitung sidik jari SHA256 dengan `Get-FileHash`.
- Menjawab pertanyaan investigasi simulasi dan mencoba menyusun laporan investigasi SOC terstruktur.

## Pembelajaran dan hal yang masih perlu diperkuat
- Sudah dipahami dengan benar: port tujuan saja tidak membuktikan malware; kondisi saat ini berbeda dari bukti historis; aplikasi dapat memiliki beberapa nama/instance process; signature `Valid` merupakan bukti, tetapi tidak menjamin keamanan.
- Perlu diperbaiki: SHA256 tidak memindai virus; port 443 tidak membuktikan aktivitas sah; ID3 bukan DNS Query; kecocokan ProcessGuid harus diamati, bukan diasumsikan.
- Laporan akhir pengguna keliru menyatakan metadata/signature/SHA256 belum diperiksa, padahal ketiganya sudah diuji.
- Timeline P17–P19 tidak dilengkapi keluaran aktual Command 11/12, dan keterangan tentang PID 8808 tidak konsisten. Jawaban tersebut tidak memverifikasi event historis Discord tambahan.
- Parent dari PID historis Discord 8808 belum diamati; informasi parent process Discord lain yang sedang berjalan bukan bukti parent historis PID tersebut.
- Perkuat pemahaman bahwa penilaian = klasifikasi disertai alasan yang didukung bukti, sedangkan tindakan berikutnya = pengujian tertentu. Dalam laporan akhir, hubungkan timestamp event dan IP tujuan dengan event yang benar-benar diperiksa; jangan mencampuradukkan observasi.
- Ulangi definisi false positive, fungsi `Format-List`, dan arti field UDP dibandingkan TCP.

## Privasi
Hanya contoh yang sudah disamarkan yang boleh dimasukkan ke GitHub publik: samarkan hostname, username, alamat sumber LAN, jalur profil lokal lengkap, dan ProcessGuid khusus host.
