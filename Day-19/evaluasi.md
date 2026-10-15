# Day 19 — Evaluasi, Jawaban Pengguna & Koreksi

**Status akhir: PASS — 80/100 (penilaian formatif mentor).** Pada awal Day 19, Active Recall dan Hands-on 1–5 telah dikirim. Penalaran Hands-on 6 dijawab tanpa bukti event terpisah yang telah dikorelasikan. Pertanyaan tantangan simulasi C1–C9 dijawab; C10 belum dijawab. Mini SOC Project **belum dikumpulkan**: pengguna meminta diingatkan tentang perintah yang diperlukan. Jangan mengarang nilai/status akhir.

**Perlindungan data:** hostname/user/jalur profil mentah, IP perangkat lokal, dan ProcessGuid unik tidak dicantumkan. Bukti simulasi dipisahkan dengan jelas dari hasil endpoint aktual.

## 1. Active Recall (jawaban singkat asli dan koreksi)

| Q | Jawaban pengguna | Koreksi dan alasan |
|---|---|---|
| Q1 SHA-256/virus | "fungsi sha 256 untuk mengetahui nilai hash sebuah file, bisa jadi tapi bisa juga tidak" | Fungsi hash dipahami dengan benar; **SHA-256 tidak mendeteksi virus**. SHA-256 menghasilkan sidik jari file untuk dibandingkan, bukan keputusan tentang aman atau tidak. |
| Q2 Current vs historical | "klo current kan proses yang sedang berlangsung, kalau historical evidence event yang telah terjadi sebelumnya" | Benar inti: Get-CimInstance Win32_Process current process snapshot, Sysmon ID1 historical Process Create. |
| Q3 False positive | "pernah dicurigai, tapi tidak ada bukti kuat" | Belum tepat: `inconclusive` (belum dapat dipastikan) tidak sama dengan false positive. False positive adalah alert yang setelah diverifikasi ternyata keliru menandai ancaman tertentu. |
| Q4 Signature | "ya karena bisa jadi attacker juga memiliki digital signature yg valid juga" | Prinsipnya benar: `Valid` tidak sama dengan aman. Executable sah yang ditandatangani pun dapat disalahgunakan; signature tidak memverifikasi seluruh perilaku file. |
| Q5 UDP443 | "msih bingung memahami kalimat soalnya" | Jika `Protocol=udp`, kita **tidak dapat** menyimpulkan bahwa koneksi menggunakan HTTPS melalui TCP. UDP port 443 mungkin digunakan QUIC, tetapi Event 3 saja tidak cukup untuk memverifikasinya. |
| Q6 ID 1/3/22 | "event 1 berisi historical event proses, event 3 berisi historical event network, event 22 berisi historical event DNS Query" | Benar; secara spesifik, Event ID 1 mencatat pembuatan process, ID 3 koneksi jaringan, dan ID 22 DNS Query. |
| Q7 Penggunaan ulang PID | "agak susah memahami soalnya, tolong berikan contoh" | Contoh: Process A menggunakan PID 5000 (GUID-A) pada pukul 09.00 lalu berhenti. Kemudian Process B menggunakan PID 5000 (GUID-B). PID-nya sama, tetapi instance process-nya berbeda. |
| Q8 Not checked signature | "salah, karena sebagai seorang SOC tidak boleh langsung menyimpulkan bahwa file tersebut malware, masih banyak evidence yg belum di check" | Benar: “belum diperiksa” **tidak** sama dengan `Invalid`. Kesimpulan bahwa file merupakan malware yang terkonfirmasi juga belum didukung bukti. |

## 2. Hands-on 1 dan 2: Process/Parent Discord

- P1 “6”: benar. Ada enam PID yang berbeda.
- P2 “tidak, ada 1 PID yg beda”: **keliru**. Keenam PID berbeda; lima process anak memiliki `ParentProcessId 2724` yang sama.
- P3 “tidak ada 1 yg beda, kenapa ya kalau semua PPID yg sama?”: lima process memiliki **PPID 2724 yang sama**, yang menunjukkan kemungkinan parent-nya sama (instance Discord.exe dengan PID 2724). PID 2724 sendiri memiliki `ParentProcessId 4632`. PPID yang sama bukan berarti PID-nya sama.
- Query aktual untuk `ProcessId 2724` menghasilkan **Name: Discord.exe, PID: 2724, ParentProcessId: 4632**.
- P4 “4632”: **keliru jika pertanyaannya meminta nama parent process**. 4632 adalah PPID dari Discord.exe yang diperiksa, bukan nama parent-nya. Periksa `ProcessId 4632` untuk mengetahui identitas parent jika masih berjalan, atau cari catatan historisnya.
- P5 “tidak, semua sama”: benar *untuk sampel saat ini*, karena basename `Image` semuanya Discord.exe. Secara umum, nama parent dan child **bisa** sama atau berbeda.
- P6 "tidak, karena bisa saja proses tersebut sudah tidak berjalan, kita bisa melihat dari historical event": benar.

## 3. Hands-on 3: Sysmon Process Create

- Hasil aktual: 2026-10-15 pukul 17:12:32 waktu lokal, `opushutil.exe` dari instalasi Office, command line `/pushregistration`, parent `svchost.exe`, argumen `-k netsvcs -p -s Schedule`, dan `Medium Integrity`. GUID serta user lokal disamarkan.
- P7 “opushutil.exe”: benar.
- P8 “svchost.exe”: benar.
- P9 "karena ... Historical Event ... process hilang pada current, tapi tidak hilang pada historical event": benar selama event tersebut masih tersimpan dalam log. Sysmon Event ID 1 adalah catatan historis, bukan tampilan process yang sedang berjalan.

## 4. Hands-on 4: Investigasi File

- Hasil aktual: Discord.exe tersedia saat ini; ukuran 220542392 byte; `CreationTime` 2026-10-08 16:02:01 dan `LastWriteTime` 16:02:04. Authenticode berstatus `Valid` dan signature terverifikasi. SHA256 berhasil dihitung.
- P10 “sha256 untuk memeriksa nilai hash ... digitalsignature ... publisher/valid ... metadata ... pembuatan, kapan di check, kapan diubah”: sebagian besar benar. Keluaran metadata menampilkan `CreationTime` dan `LastWriteTime`, bukan field “terakhir diperiksa”. SHA256 adalah sidik jari file.
- P11 “belum tentu, bisa saja user mengubah ... atau malware ...”: benar bahwa perubahan tersebut tidak otomatis berarti malware. Pembaruan atau pemasangan ulang perangkat lunak yang sah juga mungkin menjadi penyebabnya; perubahan pada isi file yang ditandatangani biasanya membatalkan signature lama kecuali file ditandatangani ulang.
- P12 “nggk boleh ... investigasi lebih dalam”: prinsipnya benar. Jika command line tidak biasa, periksa perilaku dan konteksnya; signature saja belum cukup.

## 5. Hands-on 5: Event Jaringan Sysmon

- Tiga Sysmon ID3 aktual dari Spotify.exe: pukul 17:24:40, PID 22300, TCP keluar ke alamat eksternal pada port 443; pukul 17:24:25, PID 6680, UDP `Initiated=false` dari endpoint multicast SSDP; pukul 17:24:24, PID 6680, UDP `Initiated=true` menuju endpoint multicast SSDP. Ada dua ProcessGuid/instance yang berbeda, meskipun semua `Image` bernama Spotify.exe.
- P13 "3": benar.
- P14 “Spotify.exe 3 event”: benar.
- P15 “1 process menggunakan tcp 2 process menggunakan udp”: benar jika maksudnya **satu event TCP dan dua event UDP**; keliru jika maksudnya ada tiga instance process berbeda. Tiga event tersebut mencakup dua PID dan dua GUID.
- P16 “tidak cukup ... bukti lain”: benar. Port 443 bukan penentu bahwa aktivitas aman atau berbahaya.

## 6. Hands-on 6: Linimasa

- P17 “event 1 > event 3 > event 22”: **keliru untuk data yang diberikan**. Catatan aktual yang tersedia: ID1 pukul 17:12:32 dari opushutil.exe; ID3 pukul 17:24:24 dan 17:24:25 menggunakan UDP dari Spotify; ID3 pukul 17:24:40 menggunakan TCP dari Spotify. Tidak ada catatan ID22 dalam sampel.
- P18 “ProcessGuid”: jenis bukti yang tepat, tetapi **GUID yang diamati berbeda** antara Office dan Spotify, dan ada dua instance process Spotify. Jangan mengorelasikan process hanya karena berasal dari host yang sama dan waktunya berdekatan.
- P19 “kita tidak mendapatkan bukti yg lebih luas lagi ... waktu berdekatan”: benar. Hubungan sebab-akibat membutuhkan korelasi yang lebih kuat daripada sekadar waktu yang berdekatan.

## 7. Kasus SOC Simulasi #019

**Data simulasi saja:** updater.exe berada di Public, signature berstatus Valid, menggunakan `--silent`, parent `explorer.exe`, GUID LAB-019-A; ID1 tercatat pukul 10:15:21 dan ID3 TCP yang cocok menuju IP dokumentasi:443 pukul 10:15:25; ID22 belum diperiksa.

| C | Jawaban pengguna | Koreksi |
|---|---|---|
| C1 | "menemukan ProcessGuid ... commandline dan PID" | Sebagian benar. Jelaskan pembuatan process: updater.exe dari C:\Users\Public pada pukul 10:15:21, parent explorer.exe, command line `updater.exe --silent`, PID 6200, dan ProcessGuid. |
| C2 | "DestinationIP, Destinationport, protocol" | Sebagian benar. Nilai lengkapnya: updater.exe memulai koneksi TCP ke 203.0.113.50:443 pada pukul 10:15:25. Ini alamat IP fiktif untuk dokumentasi. |
| C3 | "iya benar, memang processguid berfungsi seperti itu" | Benar: GUID LAB-019-A muncul pada ID1 dan ID3, mendukung bahwa keduanya berasal dari instance process yang sama. |
| C4 | "belum tentu, kita harus melihat evidence lain" | Benar: file yang berada di jalur Public tidak otomatis terbukti sebagai malware. |
| C5 | "tidak boleh menyimpulkan hanya dengan 1 evidence" | Benar: signature `Valid` saja tidak cukup untuk menutup alert sebagai false positive. |
| C6 | "aku tidak tau apa arti dari argument silent" | Penjelasan: `--silent` biasanya menjalankan program tanpa kotak dialog atau interaksi pengguna; opsi ini juga lazim pada installer/updater yang normal. |
| C7 | "belum di periksa, kita bisa liat dari ProcessGuid nya" | Benar dalam membedakan “belum diperiksa” dari “tidak ada event”. GUID baru berguna untuk korelasi setelah event DNS aktual yang relevan ditemukan. |
| C8 | "benign: ... executablepath ... public ... anomaly; malicious: ... silent ... suspicious" | Belum memberikan hipotesis wajar. Jalur Public dan opsi `--silent` saja bukan indikator malicious. Contoh hipotesis wajar: updater bertanda tangan yang diizinkan sedang berjalan tanpa dialog. Contoh hipotesis malicious: binary yang diganti nama atau disalahgunakan dan perlu diverifikasi. |
| C9 | "masih perlu investigasi ... mencari historical event 22" | Penilaian Perlu Investigasi masuk akal, tetapi ID22 hanyalah salah satu bukti pendukung opsional dan tidak boleh diasumsikan tersedia. |
| C10 | belum ada jawaban | Belum diselesaikan. Urutan yang disarankan: validasi process/user/parent historis dan command line; verifikasi signature/hash executable serta konteks penerbitnya; lalu periksa aktivitas jaringan/DNS terkait berdasarkan GUID dan rentang waktu. |

## 8. Mini-project

Pesan pengguna: **"ini command nya apa aja ya? lupa untuk dapat semua informasi itu"**. Ini adalah permintaan panduan, **bukan mini-project yang sudah selesai**. Jangan mengarang penilaian atau laporan mini final.

Saran kasus aktual untuk langkah berikutnya: pilih satu instance process Spotify.exe dari ID3 (PID 6680 untuk UDP multicast atau PID 22300 untuk TCP) beserta catatan event aslinya, bukan ID1 opushutil.exe yang tidak terkait. Gunakan bukti terverifikasi yang sudah tersedia dan tandai pemeriksaan file/parent/DNS yang belum dilakukan sebagai BELUM DIPERIKSA.

## 9. Kemajuan saat ini dan sasaran belajar

Hal yang sudah dikuasai: menjalankan banyak perintah PowerShell hanya-baca; memahami perbedaan kondisi saat ini dan bukti historis; tidak menyimpulkan port=malware; menggunakan metadata/signature/hash file; membaca field Sysmon dasar; serta berhati-hati dalam menilai bukti.

Materi Active Recall berikutnya: perbedaan PID dan PPID; nama parent dan PPID; false positive dan hasil yang belum pasti; SHA256 dan pemindaian virus; UDP dan HTTPS/TCP; linimasa aktual dan ProcessGuid yang berbeda; hipotesis wajar dan malicious; nilai yang tepat serta pemisahan sumber bukti. Nilai akhir Day 19 diberikan setelah laporan mini dan langkah yang kurang diperbaiki.

## Penutupan Day 19 — Penilaian Akhir (menggantikan status sementara di atas)

**Status final: PASS, 80/100 — estimasi formatif mentor.** Ambang PASS 76/100. Nilai bukan hasil tes kompetensi standar dan bukan klaim pengguna telah menulis mini report mandiri.

### Bukti baru setelah evaluasi sementara

- Kamu menjalankan query untuk PID 6680 saat ini; hasilnya kosong.
- Kamu mencari PID 6680 dalam **200 pesan Sysmon ID 1 terbaru**; hasilnya kosong. Pencarian dengan batas tersebut tidak membuktikan bahwa event tidak pernah ada.
- Kamu memperluas pencarian dengan rentang waktu 15 Oktober 2026 dan menemukan **lima catatan historis Sysmon ID 1**, semuanya process anak Spotify.exe dengan `ParentProcessId 6680`.
- Kelima catatan itu memiliki `ParentProcessGuid` yang **sama**, dan GUID tersebut **cocok dengan ProcessGuid event UDP Spotify Sysmon ID 3** untuk PID 6680 yang diamati sebelumnya.
- Setiap process anak memiliki ProcessGuid berbeda. Ini menunjukkan hubungan parent–child dan aktivitas jaringan pada satu **instance process parent**.
- Event ID 1 yang secara langsung mencatat **pembuatan parent PID 6680** tidak ditemukan dalam lima hasil tersebut.
- C10 belum dijawab secara mandiri; bagian berikut adalah contoh jawaban mentor, bukan jawaban pengguna.

### C10 — Contoh jawaban mentor, bukan jawaban pengguna

Urutannya: (1) Periksa konteks Sysmon ID 1, termasuk parent, user, dan command line. (2) Verifikasi file executable, metadata, tanda tangan digital, dan hash. (3) Korelasikan event jaringan/DNS yang benar-benar tersedia berdasarkan ProcessGuid dan waktu.

### Laporan Investigasi SOC Mini — Disusun mentor dari data pengguna

**KASUS:** Hubungan Parent-Child Historis Spotify dan Aktivitas UDP

**SUMBER:** Microsoft-Windows-Sysmon/Operational

**OBSERVASI:** Pada 15 Oktober 2026, lima process anak Spotify.exe tercatat dibuat pukul 15:12:20–15:14:19 waktu lokal. PID anak: 7160, 6308, 7208, 18628, dan 13636. Semuanya memiliki `ParentProcessId 6680` dan `ParentProcessGuid` yang sama. Dua Sysmon Event ID 3 pada pukul 17:24:24 dan 17:24:25 mencatat aktivitas UDP Spotify.exe PID 6680 dengan ProcessGuid yang cocok; terdapat aktivitas ke/dari alamat multicast 239.255.255.250 pada port 1900.

**INVESTIGASI PROCESS:** PID 6680 tidak ditemukan saat daftar process saat ini diperiksa. Parent dari PID 6680 belum diketahui berdasarkan keluaran yang tersedia. Pencarian terhadap 200 ID1 terbaru tidak menemukan kecocokan; pencarian berdasarkan rentang waktu kemudian menemukan lima ID1 process anak. Hasil tersebut sesuai dengan batas masing-masing query.

**INVESTIGASI FILE:** Metadata, signature, dan SHA-256 **file Spotify** belum diperiksa dalam kasus ini. Signature `Valid` dan hash yang diperiksa di bagian lain Day 19 adalah milik **file Discord**, bukan Spotify.

**BUKTI YANG TERSEDIA:** Catatan ID1 process anak dengan ParentProcessGuid yang sama, dua catatan ID3 UDP parent yang cocok, timestamp historis, dan peran process anak. Catatan TCP Spotify pada PID 22300 memiliki ProcessGuid berbeda dan **tidak digabungkan** sebagai satu instance process.

**KESENJANGAN BUKTI:** Catatan `Process Create` Event ID 1 milik PID 6680, parent process/command line awalnya, signature/hash Spotify, tujuan/detail lapisan aplikasi, dan event DNS yang relevan (belum diverifikasi).

**HIPOTESIS WAJAR:** Spotify mungkin melakukan pencarian perangkat lokal melalui SSDP/multicast sebagai fungsi normal aplikasi.

**HIPOTESIS MALICIOUS YANG MUNGKIN:** Penyalahgunaan process atau aktivitas jaringan yang menyimpang belum dapat dikesampingkan; UDP sendiri **bukan** indikator malicious.

**PENILAIAN:** **Bukti Belum Cukup / Perlu Investigasi.** Bukti menunjukkan hubungan parent/child dan aktivitas jaringan; belum ada cukup indikator untuk memastikan malware, tetapi verifikasi juga belum lengkap untuk menyebutnya false positive.

**TINDAKAN BERIKUTNYA:** Jika investigasi dilanjutkan, cari Event ID 1 parent PID 6680 berdasarkan identitas/waktu; periksa file Spotify yang tepat; lalu korelasikan event jaringan/DNS dengan ProcessGuid yang sama.

### Alasan nilai 80/100

Nilai diberikan atas praktik hanya-baca, pembedaan kondisi saat ini dan bukti historis, pembacaan field Sysmon, pemeriksaan file Discord, keberhasilan menemukan **korelasi ParentProcessGuid lintas event** pada Spotify, serta kehati-hatian dalam menghindari kesimpulan dini. Hal yang perlu diperbaiki: PID dan PPID, false positive dan hasil yang belum pasti, SHA256, linimasa, hipotesis, serta laporan akhir yang masih disusun dengan bantuan mentor.

**PASS:** hal yang masih lemah dibawa ke Active Recall Day 20 dan tidak menahan progres. Aturan remedial: nilai 76–100 berarti PASS; nilai di bawah 76 mendapat materi singkat dan 2–5 soal/praktik terarah hingga mencapai ambang.
