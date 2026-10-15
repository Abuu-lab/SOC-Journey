# Day 18 — Materi Extended Lab

## Alur pembelajaran
Satu konsep, satu perintah yang dijelaskan, periksa hasilnya, lakukan investigasi, lalu buat penilaian. Sesi tambahan ini menindaklanjuti masukan bahwa materi Day 18 sebelumnya terlalu singkat.

## Sysmon dan jaringan
ID 3 mencatat event jaringan. ID 22 mencatat DNS Query; resolusi DNS bukan bukti bahwa koneksi jaringan benar-benar terjadi. ID 1 adalah `Process Create`; ID 2 adalah `File Creation Time Changed`. ProcessGuid membantu mengenali instance process, sedangkan PID dapat digunakan kembali. `SourceIp/SourcePort` dan `DestinationIp/DestinationPort` menunjukkan endpoint. `Initiated true` berarti koneksi dimulai dari perangkat lokal; nilai `false` perlu ditafsirkan dengan hati-hati. UDP tidak menggunakan status TCP `Established`. Port 443, termasuk label layanan `https`, tidak membuktikan bahwa koneksi memakai HTTPS melalui TCP, aman, atau digunakan untuk eksfiltrasi.

## Bukti historis dan kondisi saat ini
`Get-WinEvent` mengambil bukti historis yang tercatat. `Get-CimInstance Win32_Process` dan `Get-NetUDPEndpoint` menampilkan kondisi saat ini. PID historis yang tidak ditemukan dalam daftar kondisi saat ini tidak membatalkan log Sysmon. Process mungkin sudah berhenti atau PID-nya digunakan kembali. Satu aplikasi dapat memiliki beberapa catatan process; nama yang sama tidak membuktikan bahwa instance-nya sama.

## Dasar PowerShell
`Get-WinEvent` dengan `FilterHashtable` dan `MaxEvents` mengambil event yang sesuai. `Select-Object` memilih properti objek. `Format-List` menata keluaran dalam bentuk daftar; `Format-List TimeCreated,Message` tidak berarti semua properti ditampilkan. `Format-List *` dapat menampilkan semua properti objek yang tersedia. `Where-Object` digunakan untuk menyaring hasil. Pahami perintah sederhana terlebih dahulu sebelum mempelajari perulangan XML yang lebih rumit.

## Pemeriksaan file
`Test-Path` memeriksa jalur yang dipilih. `Get-Item` menampilkan metadata file. `Get-AuthenticodeSignature` memeriksa signature Authenticode dan penanda tangan sertifikat. `Get-FileHash -Algorithm SHA256` menghitung sidik jari isi file, **bukan** mendeteksi virus. Signature berstatus `Valid` tidak menjamin perilaku file aman. Timestamp metadata file tidak membuktikan kapan file pertama kali dijalankan.

## Hasil pengamatan lab aktual
Event jaringan historis `Discord.exe` pada 14 Oktober 2026 menunjukkan aktivitas UDP yang dimulai secara lokal menuju 162.159.130.233:443, dengan PID historis 8808. Pemeriksaan `Get-CimInstance` berikutnya menemukan enam process Discord.exe yang sedang berjalan, tetapi tidak menemukan PID 8808. `Get-NetUDPEndpoint -OwningProcess 8808` tidak menemukan endpoint UDP saat ini. `Test-Path` untuk Discord.exe menghasilkan `True`. Metadata menunjukkan `Length 220542392`, `CreationTime 2026-10-08 16:02:01`, dan `LastWriteTime 16:02:04`. Signature Authenticode berstatus `Valid`, dengan penanda tangan Discord Inc. Hash SHA256 berhasil dihitung. Hasil tersebut berkaitan dengan file yang diperiksa belakangan; belum dibuktikan bahwa byte file itu sama dengan yang digunakan saat event historis terjadi.

Query lain mengambil tiga event jaringan Spotify.exe yang semuanya menggunakan UDP. Dua event memiliki PID 21452, ProcessGuid yang sama, dan tujuan 35.186.224.24:443. Satu event menggunakan PID 21148, `Initiated false`, serta endpoint jaringan lokal privat yang menyerupai alamat broadcast. Nama `Image` yang sama tidak berarti instance process-nya sama. Duplikasi event tidak dapat ditentukan hanya dari tiga field tersebut.

## Alur investigasi SOC
Observasi → Bukti yang Tersedia → Kesenjangan Bukti → Hipotesis → Pengujian dan Hasil → Penilaian → Tindakan Berikutnya.
Penilaian dan Tindakan Berikutnya merupakan dua hal berbeda. Tidak terlihatnya ID1/22 dalam keluaran yang diberikan bukan bukti bahwa event tersebut tidak ada. Jangan menyebut adanya DNS Query jika kasus hanya memuat ID3. Jangan menyatakan aktivitas malicious hanya berdasarkan jalur unduhan atau nomor port.

## Hal yang perlu diperkuat
Kamu sudah memahami banyak field observasi, tetapi perlu lebih teliti saat menyusun penilaian, memahami fungsi hash file, membedakan timestamp dan PID, serta menjaga konsistensi laporan. Script panjang tanpa penjelasan kurang membantu; sesi tetap dapat mendalam, dengan langkah pendek yang dijelaskan satu per satu.
