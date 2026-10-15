# Day 18 — Evaluasi Praktik Tambahan

**Status:** Day 18 PASS (latihan inti sebelumnya mendapat 90/100 secara formatif); lab tambahan telah diselesaikan dengan beberapa koreksi dan masih ada bukti yang belum dipastikan. Tidak ada hasil baru yang direka.

## Bagian 4: Dasar Jaringan
- N1 jawaban pengguna: “sourceport itu port local dr pc kita ... port tujuan.” Benar. Istilah yang digunakan adalah `SourcePort` dan `DestinationPort`.
- N2: kamu belum mengetahui port 4444, tetapi tidak langsung menganggapnya sebagai malware. Benar: nomor port saja tidak membuktikan maksud atau tujuan aktivitas.
- N3: “inisiasi local.” Benar untuk `Initiated true`.

## Bagian 5: Event Jaringan Historis
Perintah 1: `Get-WinEvent FilterHashtable Id3 -MaxEvent 5 | Select-Object TimeCreated,Id`. Kamu berhasil menjalankannya; lima catatan sekitar pukul 20:23 pada 14 Oktober ditampilkan. PowerShell menerima bentuk singkat `-MaxEvent`; sebaiknya gunakan `-MaxEvents` secara lengkap.
Perintah 2: `-MaxEvents 3 | Format-List TimeCreated,Message`.
Jawaban pengguna: “format-list itu untuk memunculkan semua informasi ya?” Koreksi: perintah ini menata properti yang dipilih dalam bentuk daftar. Properti `Message` memuat rincian field; `Format-List *` menampilkan semua properti objek yang tersedia.
P4 jawaban pengguna “Spotify.exe semua; ada 2 PID yg sama 1 berbeda” benar; dua catatan dengan PID 21452 memiliki ProcessGuid yang sama, sedangkan PID 21148 merupakan instance terpisah.
P5 jawaban “udp semua” benar.
P6: jawaban bahwa event dengan PID yang sama memiliki IP/port sama, sedangkan event dengan PID berbeda memiliki tujuan berbeda, sudah benar. Catatan berulang tidak membuktikan bahwa koneksinya terpisah; event `Initiated false` memuat alamat lokal yang menyerupai alamat broadcast, bukan otomatis lalu lintas malicious dari luar.

## Bagian 6: Process saat ini dan bukti historis
P7: kamu melaporkan PID 8808 tidak ditemukan saat ini. Hasil yang dilaporkan sudah benar.
P8: keluaran menampilkan enam process Discord.exe saat ini (16220, 17708, 19312, 9144, 20744, 21412). Jawaban benar.
P9: jawaban “aku tidak mengerti false positive, tapi ... jangan memberi kesimpulan terburu2” menunjukkan kehati-hatian yang baik, tetapi definisinya belum dipahami. False positive adalah alert/deteksi yang menyatakan adanya perilaku tidak diinginkan, tetapi setelah diverifikasi ternyata tidak didukung bukti. Event historis tidak otomatis false positive hanya karena PID asalnya sudah tidak ada. Nilai `ParentProcessId` dari process Discord yang berjalan sekarang tidak membuktikan siapa parent historis PID 8808.

## Bagian 7: Endpoint UDP
P10: `OwningProcess` menyimpan PID; benar.
P11: `Get-NetUDPEndpoint -OwningProcess 8808` menghasilkan `ObjectNotFound`; tidak ada endpoint UDP SAAT INI yang cocok. Benar.
P12: tampilan endpoint UDP saat ini berbeda dari catatan historis Sysmon ID3. Benar; hal ini tidak membatalkan event jaringan sebelumnya.

## Bagian 8: Investigasi file
P13: `Test-Path True` — benar untuk file yang diperiksa.
P14: metadata file mencatat Discord.exe berukuran 220542392 byte, `CreationTime 2026-10-08 16:02:01`, dan `LastWriteTime 16:02:04` — benar.
P15: status signature `Valid` dan terverifikasi; subjek sertifikatnya Discord Inc. — benar, tetapi ini bukan bukti mutlak bahwa file aman.
P16 jawaban pengguna: “SHA256 bisa sebagai evidence untuk memeriksa apakah file mengandung konten virus.” Kurang tepat: SHA256 tidak dapat memindai atau mendeteksi malware dengan sendirinya. Sidik jari file dapat dibandingkan dengan hash yang sudah diketahui atau sampel file lain, lalu ditinjau bersama bukti konteks. Kamu berhasil menghitung SHA256; repo tidak mencantumkan hash lengkap karena tidak diperlukan dan bersifat unik untuk file tersebut.

## Bagian 9: Linimasa
P17 jawaban pengguna “ya masih ada PID 8808” belum terverifikasi: keluaran Command 11/12 untuk pencarian historis berdasarkan rentang waktu tidak diberikan. Jawaban ini tidak konsisten dengan P7 yang menyebut PID tersebut tidak ditemukan saat ini; adanya event historis tidak membuktikan process masih berjalan.
P18 jawaban “tidak ada, semua PID 8808” belum dapat diverifikasi tanpa keluaran event linimasa yang sesuai.
P19 jawaban “ya benar 1 processguid” belum terbukti; PID atau nama `Image` yang sama saja tidak membuktikan ProcessGuid cocok. Periksa GUID pada setiap event historis yang relevan.

## Bagian 10: Tantangan Simulasi #018
Skenario: `audiohelper.exe` berasal dari Public Downloads; Sysmon ID3 menunjukkan UDP yang dimulai secara lokal menuju 198.51.100.44:443; parent/signature/hash belum diketahui. Tidak ada event DNS yang diberikan.
C1 jawaban “Audiohelper.exe” benar.
C2 jawaban “UDP” benar.
C3: konsep IP sumber dan tujuan sudah benar, walaupun alamat sumber pada skenario berasal dari rentang dokumentasi, bukan jaringan privatmu.
C4: file di folder Downloads boleh dianggap perlu diperiksa, tetapi itu bukan kesimpulan bahwa file tersebut malware.
C5: nama executable dapat menyesatkan; jawaban benar.
C6: tidak menyebut signature `Valid` sebelum memeriksanya menunjukkan disiplin yang benar dalam menggunakan bukti.
C7: kesimpulan “sah karena port 443” keliru. Port 443 tidak membuktikan perilaku sah. Kemungkinan wajar: aplikasi yang diizinkan sedang bekerja. Kemungkinan malicious: executable yang menyamar sebagai aplikasi lain.
C8: menyebut “DNS Query” ke alamat IP kurang tepat; bukti simulasi hanya memuat Event ID 3 jaringan. Penilaian yang sesuai: Perlu Investigasi (Needs Investigation), karena identitas parent/file dan konteks jaringan belum cukup diverifikasi.
C9 jawaban “check Sysmon ID1 dan 22 lalu SHA256, DigitalSignature” arahnya tepat, tetapi belum menyusun urutan investigasi tiga langkah yang jelas. Saran urutan: ambil ID1 untuk memeriksa parent/command line; periksa jalur file/signature/hash; lalu tinjau konteks jaringan/DNS yang berkaitan.

## Bagian 11: Laporan mini pengguna dan koreksi rinci
Laporan pengguna mencantumkan pengamatan Discord pada pukul 15:59:50 dan tujuan 162.159.128.232. Event Discord yang sebelumnya terdokumentasi terjadi pukul 15:55:07 dengan tujuan 162.159.130.233. Waktu/IP yang berbeda mungkin merujuk pada event tambahan, tetapi keluaran asli yang cocok dengan pengamatan berikutnya BELUM diberikan; jangan menganggapnya sebagai event yang sama. Kaitkan laporan dengan event pukul 15:55:07 yang sudah diverifikasi atau sertakan log event yang sesuai.
Laporan bahwa PID historis 8808 tidak ditemukan sesuai dengan query sebelumnya.
Kolom ParentProcess kosong: parent historis belum diverifikasi.
Jalur profil pada `ExecutablePath` bersifat sensitif; jalur disamarkan menjadi `C:\Users\[REDACTED_USER]\AppData\Local\Discord\app-1.0.9261\Discord.exe`.
File ditemukan (`True`), metadata diperiksa, signature berstatus `Valid`, dan SHA256 dihitung; laporan menyebut tiga hal tersebut belum diperiksa. Ini bertentangan dengan bukti yang dikumpulkan. Perbaiki laporan dengan memasukkan hasil yang benar-benar diperoleh.
Sysmon ID3 historis tersedia. Endpoint UDP saat ini untuk PID 8808 tidak ditemukan. Tiga event Spotify historis lainnya telah diperiksa, tetapi tidak dikorelasikan dengan event Discord.
Kesenjangan bukti **tidak boleh** mencantumkan metadata/signature/hash sebagai “belum diperiksa” setelah pemeriksaan dilakukan. ID1/22 mungkin tidak ditemukan atau belum dicari berdasarkan ProcessGuid Discord; tandai “BELUM DIPERIKSA”, jangan tulis seolah-olah event tersebut tidak ada.
Hipotesis bahwa “jalurnya tampak normal” mendukung sebuah kemungkinan, tetapi tidak membuktikan bahwa file itu sah.
Hipotesis malicious yang hanya berbunyi “karena UDP” tidak valid; UDP sendiri bukan indikator malicious.
Penilaian sebaiknya menyatakan “Bukti Belum Cukup / Perlu Investigasi” disertai alasan dan konteks, bukan hanya menulis “perlu investigasi lebih lanjut”.
Tindakan Berikutnya harus menyebut pemeriksaan pembuatan process historis, parent, dan korelasi jaringan Discord berdasarkan ProcessGuid/waktu.

## Contoh laporan koreksi yang disusun dengan bantuan mentor
Kasus: event jaringan UDP historis Sysmon dari Discord.exe.
Observasi: 2026-10-14 pukul 15:55:07 waktu lokal, Sysmon Event ID 3, PID historis 8808, UDP `Initiated true`, tujuan 162.159.130.233:443.
Process: PID 8808 tidak ditemukan saat diperiksa; terdapat enam process Discord.exe lain pada saat pengecekan; parent historis belum diverifikasi.
Executable: file Discord.exe saat ini ditemukan pada jalur profil pengguna yang telah disamarkan; ukuran 220542392 byte; timestamp metadata 8 Oktober 2026; Authenticode `Valid`, ditandatangani Discord Inc.; SHA256 berhasil dihitung.
Jaringan: event ID3 historis tersedia; tidak ada endpoint UDP saat ini yang dimiliki PID 8808; Event ID1/22 belum dikorelasikan dengan instance historis yang sama.
Kesenjangan bukti: parent/command line process historis, protokol lapisan aplikasi, korelasi event pendukung, dan penilaian perilaku yang meyakinkan.
Hipotesis wajar: perilaku jaringan Discord yang biasa. Hipotesis malicious: penyalahgunaan/penyamaran process atau aktivitas anomali perlu diuji; UDP saja bukan alasan untuk menyimpulkan malicious.
Penilaian: bukti belum cukup untuk menyimpulkan malicious. Bukti mendukung kemungkinan bahwa executable tersebut sah, tetapi belum cukup untuk menilai seluruh aktivitasnya.
Tindakan Berikutnya: periksa `Process Create` historis dengan ProcessGuid yang sama dan event jaringan Sysmon di sekitar waktu tersebut; pastikan identitas event/waktu dan cari log DNS yang relevan sebelum menyatakan adanya hubungan sebab-akibat.

## Milestone dan materi yang dibawa ke Day 19
Praktik yang selesai: mencari catatan historis Sysmon; membaca `Image`/PID/GUID/IP sumber dan tujuan/protokol; memeriksa process dan endpoint UDP saat ini; memeriksa keberadaan file, metadata, Authenticode, dan SHA256; serta menyusun penalaran investigasi awal.
Materi Active Recall: fungsi SHA256; arti false positive; perbedaan `Format-List` dan `Select-Object`; kondisi saat ini dan bukti historis; penggunaan ulang PID/identitas ProcessGuid; Event ID 3 bukan DNS; port 443 tidak membuktikan keamanan; perbedaan observasi, hipotesis, dan penilaian; serta konsistensi bukti dalam laporan.
