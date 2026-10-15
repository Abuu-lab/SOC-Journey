# Day 17 — Evaluasi, Jawaban, Koreksi, Mini SOC Report

**Status: PASS (80/100, penilaian formatif mentor).** Ambang PASS 76/100. Nilai mencerminkan pemahaman konsep dan keberhasilan praktik dengan bimbingan, **bukan penguasaan scripting mandiri**. Tidak semua jawaban awal benar; remediasi bertahap tercatat di bawah.

**Sumber:** jawaban pengguna dan PowerShell output yang dikirim dalam sesi Day 17. Contoh challenge adalah SIMULASI, dibedakan dari data endpoint asli. Identitas user/host dan raw GUID disamarkan.

## 1. Active Recall Q1–Q9

| Q | Jawaban pengguna | Koreksi / ideal |
|---|---|---|
| Q1 | “Processguid itu proses instance yang mempunyai corelation dengan sysom event 3” | Benar sebagian. PID adalah ID proses dan bisa dipakai ulang; ProcessGuid mengidentifikasi satu process instance untuk correlation Sysmon ID 1/3/22 dan lainnya, bukan hanya ID 3. |
| Q2 | “process : updater.exe parent:explorer.exe” | Tepat untuk contoh: Image updater.exe, ParentImage explorer.exe. |
| Q3 | “evidence available : berapakah evidence yg tersedia; evidence gap : evidence apa saja yg belum tersedia; assessment : tidak tau” | `Evidence Available` berarti bukti yang ada; `Evidence Gap` berarti bukti yang belum tersedia/terverifikasi; `Assessment` berarti penilaian sementara yang disertai alasan, bukan daftar tindakan. |
| Q4 | “belum pasti tercatat bisa jadi network log nya belum aktif atau belum tercatat” | Benar inti. Event 3 bisa tidak tercatat karena rules/filter, konfigurasi logging, atau retention. |
| Q5 | “tidak, bisa jadi port 443 digunakan malware untuk menyerang” | Benar: port 443 bukan proof legitimacy; dapat digunakan benign/malicious. |
| Q6 | “Get-NetTCPConnection mungkin bisa memperlihatkan, belum bisa menjelaskan” | `Get-NetTCPConnection` menampilkan koneksi TCP yang sedang aktif, bukan bukti historis dari koneksi yang telah berakhir. |
| Q7 | “commandline/executablepath/image ... SHA256,digitalsignature,metadata” | Benar sebagian: powershell.exe bisa menjadi interpreter script maintenance.ps1. Periksa user, parent, arguments, script, hash/signature, timestamp dan network context sebelum verdict. |
| Q8 | “memiliki PID sama tapi prosesguid berbeda, belum bisa menjelaskan” | ProcessGuid berbeda biasanya menunjukkan process instances berbeda; PID bisa reused setelah instance lama berakhir. |
| Q9 | “tidak boleh menyimpulkan ... id 3 tidak ada, get-nettcpconnection established kemungkinan id 3 belum tercatat” | Benar prinsip: current established connection dan historical Event 3 punya cakupan berbeda. |

## 2. Praktik 1 — Real DNS Event ID 22

Pengguna menjalankan Get-WinEvent Event ID 22; awalnya tidak ada output karena variable assignment, lalu berhasil menampilkan dengan Format-List. Mentor menjelaskan simpan variabel vs tampilkan objek.

Hasil pengamatan (identitas host, user, dan GUID disamarkan):
- TimeCreated: 2026-10-13 16:29:32 lokal untuk event yang mula-mula dibaca.
- Image basename: MuMuNxMain.exe; historical PID: 7904.
- ProcessGuid: [REDACTED_PROCESSGUID] (tersedia secara lokal).
- QueryName: www.163.com; QueryStatus: 0.
- QueryResults: DNS aliases tipe CNAME dan IPv4-mapped IPv6 addresses.

Jawaban praktik:
| P | Jawaban pengguna | Koreksi |
|---|---|---|
| P1 | “MMuNxMain.exe” | Mengenali process name dengan benar. |
| P2 | “belum tau” | QueryName = domain ditanya, QueryResults = DNS response/addresses. |
| P3 | “seharusnya iya, aku melihat di query result seperti ipv6” | DNS reply bukan bukti network connection; prefix ::ffff adalah IPv4-mapped IPv6, bukan bukti traffic menuju IP. |
| P4 | “untuk melihat corelation hubungan” | ProcessGuid membantu correlation event dalam process instance sama; perlu membedakan field domain dan destination. |

## 3. Praktik 2 — Real ProcessGuid Correlation

Pengguna menjalankan script mentor: Get-WinEvent untuk IDs 1,3,22 sebanyak 1.000 event terbaru, parse field, cocokkan ProcessGuid, urutkan timestamp. Hasil:
- 47 Event ID 22 dari MuMuNxMain.exe, QueryName www.163.com.
- Rentang: 16:03:11 hingga 16:49:53 (46 menit 42 detik); frekuensi sekitar satu per menit.
- Tidak ditemukan ID 1 atau ID 3 dengan ProcessGuid target **dalam data 1.000 event tersebut**.

| P | Jawaban pengguna | Koreksi |
|---|---|---|
| P5 | “event Id 22” | Tepat, event hasil filter yang muncul hanya ID 22. |
| P6 | “mungkin sama, gkada hasil process guid” | Semua baris lolos filter GUID sama, tetapi kolom ProcessGuid tidak dicetak; keterbatasan tampilan script mentor. |
| P7 | “sudah urut diatas” | Tepat; hasil diurutkan berdasarkan timestamp. |
| P8 | “tidak boleh” | Benar tidak boleh menyimpulkan no network activity hanya karena ID 3 tidak muncul di pencarian terbatas. |
| P9 | “evidence yg belum ada yaitu event 1 dan 3” | Tepat sebagai kesenjangan bukti pada sampel yang diperiksa. |

**Perbaikan metode:** pengguna tidak memahami foreach, [xml], dan hashtable saat diminta melakukan langkah itu. Mentor mengakui latihan melompat terlalu cepat dari kemampuan PowerShell dasar; script berhasil **dengan panduan, belum mandiri**.

## 4. Challenge Case #017 — SIMULASI, bukan kejadian nyata

Simulasi: 14:22:10 Event1 updater.exe dari explorer.exe pada Temp; 14:22:12 ID22 domain updates.example.com resolves 203.0.113.10; 14:22:14 ID3 connects 203.0.113.10:443; ProcessGuid sama; hash, signature, metadata belum dicek.

| Soal | Jawaban pengguna | Koreksi / jawaban ideal |
|---|---|---|
| C1 | “process dibuat 14.22.10 ... DNS Query 14.22.12 ... terhubung 14.22.14” | Tepat: tiga observations berurutan, parent explorer.exe. |
| C2 | “ProcessGuid” | Tepat untuk mengaitkan event dari process instance sama. |
| C3 | “4 detik” | Tepat untuk 14:22:10 sampai 14:22:14. |
| C4 | “kyknya ngga, soalnya status nya 0” | Jawaban tersebut belum pasti dan tidak otomatis berkaitan. Status DNS 0 berarti query DNS berhasil, bukan gagal. IP hasil resolusi yang sama dengan `DestinationIp` mendukung dugaan keterkaitan, tetapi belum membuktikan hubungan sebab-akibat. |
| C5 | “commandline, image, destinationip” | Ini field yang perlu ditinjau. Faktor pemeriksaan khusus: executable dari Temp, generic updater name, background argument, context dan publisher belum tervalidasi; IP dan port saja bukan bukti jahat. |
| C6 | “legitimate processguid menunjukan corelation yg normal; malicious updater.exe dengan argument background” | ProcessGuid hanya identity/correlation, **tidak menilai legitimacy**; benign update yang sah vs malware masquerading sebagai updater adalah dua hypotheses, keduanya belum terbukti. |
| C7 | “evidence tersedia Sysmon 1,3,22; evidence belum tersedia sha256,digitalsignature” | Benar. Dapat ditambahkan pemeriksaan metadata, verifikasi, reputasi/konteks, dan bukti perilaku. |
| C8 | “belum tentu aman, port 443 bisa untuk jalan malware” | Tepat. |
| C9 | “melihat event id/log” | Perlu lebih spesifik: historical Sysmon Event ID 1/3/22 dan ProcessGuid, bukan current PID saja bila proses telah mati; timeline dan existing log retention juga penting. |
| C10 | “terlihat process explorer sedang melakukan update dan meminta koneksi ...” | Kesimpulan terlalu jauh: `updater.exe` yang membuat koneksi, sedangkan `explorer.exe` adalah parent; belum terbukti bahwa aktivitas pembaruan itu sah. Penilaian yang sesuai: Perlu Investigasi (Needs Investigation), karena bukti keabsahan file belum cukup. |

## 5. Remediasi bertahap — tidak memakai command baru

**Tahap B (Repeated Events):**
- Q1: “melakukan dns query ke 163.com secara berulang dalam 45 mnit” → tepat pola; angka diperbaiki menjadi 46 menit 42 detik.
- Q2: “belum tentu, bisa saja aktivitas background dari aplikasi” → tepat; repeated DNS != automatically malicious.
- Q3: “belum tentu, bisa saja log belum aktif atau event ID 3 belum tercatat” → benar konsep gap. Pada endpoint ini Day16 pernah mencatat ID3, sehingga jangan menyatakan jenis logging itu tidak aktif secara keseluruhan tanpa bukti.

**Tahap C (Observasi dan Penilaian):**
- C1: repeated query 46 menit; ID1 dan ID3 belum ditemukan → benar jika diberi pembatas hasil 1.000 event.
- C2: “kita perlu mencari evidence lain” → itu adalah Tindakan Berikutnya, **bukan** Penilaian.
- Latihan simulasi chrome.exe: kamu membaca “35 query dalam 30 menit” sebagai “35 kali dalam 1 menit” → tingkatkan ketelitian saat membaca frekuensi.
- Penilaian pengguna berikutnya: “Needs Investigation” → tepat; alasannya masih keliru menyebut ID2 sebagai kesenjangan bukti, padahal yang dimaksud ID3.

**Laporan Mini — Bukti Tersedia/Kesenjangan Bukti:**
- Bukti tersedia: ID22, MuMuNxMain.exe, QueryName www.163.com, frekuensi/durasi query berulang → benar. Frasa “alamat tujuan 163.com” perlu diperjelas menjadi **domain yang ditanyakan melalui DNS, bukan tujuan jaringan yang sudah dipastikan**.
- Kesenjangan bukti: pengguna menulis Event ID 1 dan 2 → yang benar ID 1 dan **3**.

**Hipotesis:**
- H1: “aktifitas background ... aplikasi sedang melakukan Update” → hipotesis wajar yang masuk akal, tetapi belum terbukti.
- H2: “bisa jadi pengiriman data pribadi ke 163.com” → kekhawatiran yang mungkin, tetapi exfiltration tidak dapat dibuktikan hanya dari event DNS. Beaconing berkala dapat menjadi hipotesis alternatif, tetapi tetap membutuhkan bukti.

**Tindakan Berikutnya — jawaban akhir pengguna:**
- “aku tidak hafal command nya tapi aku akan cari event ID 1 dan 3 ... gk hafal command nya” → **pilihan investigasi tepat**. Menghafal perintah bukan keharusan pada Day 17; gunakan catatan atau bantuan perintah. Langkah yang tepat adalah mencari Event ID 1 dan 3 historis yang relevan, mencocokkan ProcessGuid/rentang waktu, lalu membedakan temuan dari data yang belum tersedia.
- Contoh sederhana `Get-WinEvent` untuk ID 1 dan 3 dengan `-MaxEvents 10` dibagikan sebagai referensi, **belum dijalankan**.

## 6. Laporan SOC Mini — dirangkai bersama mentor dari bukti aktual

**Kasus:** DNS Query berulang / korelasi Process-DNS-Network (endpoint aktual; tidak dinyatakan sebagai insiden).
**Sumber:** Microsoft-Windows-Sysmon/Operational.
**Process:** MuMuNxMain.exe.
**ProcessGuid:** [REDACTED_PROCESSGUID].
**Rentang waktu:** 2026-10-13 pukul 16:03:11–16:49:53 waktu lokal (46 menit 42 detik), 47 event DNS yang cocok.

**Observasi:** MuMuNxMain.exe berulang kali melakukan DNS query ke www.163.com sekitar sekali per menit.
**DNS:** Event ID 22, `QueryStatus 0`, `QueryResults` berisi CNAME dan beberapa alamat IP hasil resolusi.
**Jaringan:** `DestinationIp` / `DestinationPort` **belum dapat dipastikan dari Event ID 3 yang cocok** dalam kumpulan data yang diperiksa.
**Korelasi event:** 47 catatan ID22 memiliki ProcessGuid yang sama. ID1/ID3 tidak ditemukan dalam pencarian 1.000 event.
**Hipotesis 1 (wajar):** pemeriksaan latar belakang atau pembaruan yang normal.
**Hipotesis 2 (mencurigakan):** mungkin terjadi komunikasi malicious berkala, tetapi belum terverifikasi; tidak ada bukti exfiltration.
**Bukti yang Tersedia:** catatan ID22 aktual, nama process, PID pada event, timestamp historis, QueryName/Status/Results, dan kecocokan ProcessGuid.
**Kesenjangan Bukti:** Event ID 1/3 yang cocok belum ditemukan dalam rentang/sampel; koneksi jaringan yang sebenarnya, tujuan, dan keabsahan aktivitas belum diketahui.
**Pengujian yang Dilakukan:** menggunakan `Get-WinEvent` untuk ID22 dan filter ProcessGuid pada 1.000 event ID1/3/22; hasil diurutkan berdasarkan waktu.
**Hasil Aktual:** 47 catatan ID22 yang cocok; tidak ada ID1/3 yang cocok dalam event yang dijadikan sampel.
**Penilaian:** **Perlu Investigasi / Bukti Belum Cukup untuk menyimpulkan malicious.** Pola tersebut mungkin wajar; korelasi yang ada belum membuktikan terjadinya koneksi jaringan.
**Tindakan Berikutnya:** cari Sysmon ID1 dan ID3 historis yang relevan berdasarkan ProcessGuid dan rentang waktu tertentu; jika tidak ditemukan, periksa retensi log dan filter. Pemeriksaan file executable dapat dilakukan setelahnya jika relevan.

**Catatan penyusunan:** laporan akhir ini merupakan ringkasan yang disusun **dengan bantuan mentor** berdasarkan jawaban dan bukti pengguna; pengguna belum menulis seluruh laporan secara mandiri.

## 7. Evaluasi akhir

**Nilai 80/100 (formatif) — PASS.** Alasannya: berhasil mengambil event DNS aktual, menyusun linimasa, membedakan query dari koneksi melalui remediasi, membuat dua hipotesis, serta memilih Penilaian dan Tindakan Berikutnya dengan tepat. Kamu masih memerlukan banyak bantuan untuk sintaks PowerShell, membaca field, dan menyusun penilaian secara mandiri. PASS tidak berarti seluruh materi sudah dikuasai.

**Carry-forward Active Recall Day18:**
1. Event ID 1 vs 2 vs 3 vs 22 (terutama ID 3).
2. QueryName vs QueryResults vs DestinationIp; DNS ≠ connection ≠ exfiltration.
3. Perbedaan Observasi, Penilaian, dan Tindakan Berikutnya.
4. Baca angka dengan teliti: 35 queries/30 min tidak sama dengan 35 queries/1 min.
5. ProcessGuid ID correlation; PID dapat reused.
6. PowerShell foundational commands: variabel, Get-WinEvent, Format-List, sorting output sebelum loops/XML.
7. Tidak mengklaim missing events sebagai no activity; sebutkan batas pencarian.
