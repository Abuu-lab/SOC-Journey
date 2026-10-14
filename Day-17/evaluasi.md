# Day 17 — Evaluasi, Jawaban, Koreksi, Mini SOC Report

**Status: PASS (80/100, penilaian formatif mentor).** Ambang PASS 76/100. Nilai mencerminkan pemahaman konsep dan keberhasilan praktik dengan bimbingan, **bukan penguasaan scripting mandiri**. Tidak semua jawaban awal benar; remediasi bertahap tercatat di bawah.

**Sumber:** jawaban pengguna dan PowerShell output yang dikirim dalam sesi Day 17. Contoh challenge adalah SIMULASI, dibedakan dari data endpoint asli. Identitas user/host dan raw GUID disamarkan.

## 1. Active Recall Q1–Q9

| Q | Jawaban pengguna | Koreksi / ideal |
|---|---|---|
| Q1 | “Processguid itu proses instance yang mempunyai corelation dengan sysom event 3” | Benar sebagian. PID adalah ID proses dan bisa dipakai ulang; ProcessGuid mengidentifikasi satu process instance untuk correlation Sysmon ID 1/3/22 dan lainnya, bukan hanya ID 3. |
| Q2 | “process : updater.exe parent:explorer.exe” | Tepat untuk contoh: Image updater.exe, ParentImage explorer.exe. |
| Q3 | “evidence available : berapakah evidence yg tersedia; evidence gap : evidence apa saja yg belum tersedia; assessment : tidak tau” | Evidence Available = bukti yang ada; Gap = bukti belum ada/terverifikasi; Assessment = penilaian sementara yang beralasan, bukan daftar tindakan. |
| Q4 | “belum pasti tercatat bisa jadi network log nya belum aktif atau belum tercatat” | Benar inti. Event 3 bisa tidak tercatat karena rules/filter, konfigurasi logging, atau retention. |
| Q5 | “tidak, bisa jadi port 443 digunakan malware untuk menyerang” | Benar: port 443 bukan proof legitimacy; dapat digunakan benign/malicious. |
| Q6 | “Get-NetTCPConnection mungkin bisa memperlihatkan, belum bisa menjelaskan” | Get-NetTCPConnection menampilkan current TCP connection view, bukan historical evidence dari koneksi yang telah berakhir. |
| Q7 | “commandline/executablepath/image ... SHA256,digitalsignature,metadata” | Benar sebagian: powershell.exe bisa menjadi interpreter script maintenance.ps1. Periksa user, parent, arguments, script, hash/signature, timestamp dan network context sebelum verdict. |
| Q8 | “memiliki PID sama tapi prosesguid berbeda, belum bisa menjelaskan” | ProcessGuid berbeda biasanya menunjukkan process instances berbeda; PID bisa reused setelah instance lama berakhir. |
| Q9 | “tidak boleh menyimpulkan ... id 3 tidak ada, get-nettcpconnection established kemungkinan id 3 belum tercatat” | Benar prinsip: current established connection dan historical Event 3 punya cakupan berbeda. |

## 2. Praktik 1 — Real DNS Event ID 22

Pengguna menjalankan Get-WinEvent Event ID 22; awalnya tidak ada output karena variable assignment, lalu berhasil menampilkan dengan Format-List. Mentor menjelaskan simpan variabel vs tampilkan objek.

Observed (identitas host, user, GUID disamarkan):
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
| P9 | “evidence yg belum ada yaitu event 1 dan 3” | Tepat sebagai evidence gap untuk sampel yang diperiksa. |

**Perbaikan metode:** pengguna tidak memahami foreach, [xml], dan hashtable saat diminta melakukan langkah itu. Mentor mengakui latihan melompat terlalu cepat dari kemampuan PowerShell dasar; script berhasil **dengan panduan, belum mandiri**.

## 4. Challenge Case #017 — SIMULASI, bukan kejadian nyata

Simulasi: 14:22:10 Event1 updater.exe dari explorer.exe pada Temp; 14:22:12 ID22 domain updates.example.com resolves 203.0.113.10; 14:22:14 ID3 connects 203.0.113.10:443; ProcessGuid sama; hash, signature, metadata belum dicek.

| Soal | Jawaban pengguna | Koreksi / jawaban ideal |
|---|---|---|
| C1 | “process dibuat 14.22.10 ... DNS Query 14.22.12 ... terhubung 14.22.14” | Tepat: tiga observations berurutan, parent explorer.exe. |
| C2 | “ProcessGuid” | Tepat untuk mengaitkan event dari process instance sama. |
| C3 | “4 detik” | Tepat untuk 14:22:10 sampai 14:22:14. |
| C4 | “kyknya ngga, soalnya status nya 0” | Jawaban tidak pasti/otomatis berkaitan: status DNS 0 berarti successful DNS query, bukan kegagalan. Same resolved IP dan DestinationIp mendukung dugaan hubungan, belum membuktikan sebab-akibat. |
| C5 | “commandline, image, destinationip” | Ini field yang perlu ditinjau. Faktor pemeriksaan khusus: executable dari Temp, generic updater name, background argument, context dan publisher belum tervalidasi; IP dan port saja bukan bukti jahat. |
| C6 | “legitimate processguid menunjukan corelation yg normal; malicious updater.exe dengan argument background” | ProcessGuid hanya identity/correlation, **tidak menilai legitimacy**; benign update yang sah vs malware masquerading sebagai updater adalah dua hypotheses, keduanya belum terbukti. |
| C7 | “evidence tersedia Sysmon 1,3,22; evidence belum tersedia sha256,digitalsignature” | Benar. Bisa ditambah metadata/verification, reputation/context dan behavior evidence. |
| C8 | “belum tentu aman, port 443 bisa untuk jalan malware” | Tepat. |
| C9 | “melihat event id/log” | Perlu lebih spesifik: historical Sysmon Event ID 1/3/22 dan ProcessGuid, bukan current PID saja bila proses telah mati; timeline dan existing log retention juga penting. |
| C10 | “terlihat process explorer sedang melakukan update dan meminta koneksi ...” | Overclaim: updater.exe melakukan connection, explorer.exe parent; belum terbukti aktivitas update sah. Ideal: Needs Investigation, evidence untuk file legitimacy belum cukup. |

## 5. Remediasi bertahap — tidak memakai command baru

**Tahap B (Repeated Events):**
- Q1: “melakukan dns query ke 163.com secara berulang dalam 45 mnit” → tepat pola; angka diperbaiki menjadi 46 menit 42 detik.
- Q2: “belum tentu, bisa saja aktivitas background dari aplikasi” → tepat; repeated DNS != automatically malicious.
- Q3: “belum tentu, bisa saja log belum aktif atau event ID 3 belum tercatat” → benar konsep gap. Pada endpoint ini Day16 pernah mencatat ID3, sehingga jangan menyatakan jenis logging itu tidak aktif secara keseluruhan tanpa bukti.

**Tahap C (Observation, Assessment):**
- C1: repeated query 46 menit; ID1 dan ID3 belum ditemukan → benar jika diberi pembatas hasil 1.000 event.
- C2: “kita perlu mencari evidence lain” → itu Next Action, **bukan** Assessment.
- Latihan chrome.exe simulasi: pengguna membaca “35 query dalam 30 menit” sebagai “35x dalam 1 menit” → perbaiki attention to detail terhadap frekuensi.
- Assessment pengguna berikutnya: “Needs Investigation” → tepat; reason masih keliru menyebut ID2 sebagai gap, harusnya ID3.

**Mini Report — Evidence Available/Gap:**
- Available: ID22, MuMuNxMain.exe, QueryName www.163.com, repeating frequency/duration → benar. “alamat tujuan 163.com” dipertegas sebagai **domain DNS queried, bukan confirmed network destination**.
- Gap: pengguna menulis Event ID 1 & 2 → koreksi ID 1 & **3**.

**Hypothesis:**
- H1: “aktifitas background ... aplikasi sedang melakukan Update” → possible legitimate hypothesis yang masuk akal, belum terbukti.
- H2: “bisa jadi pengiriman data pribadi ke 163.com” → possible concern, tapi exfiltration tidak dibuktikan dengan DNS event saja; possible periodic beaconing sebagai hypothesis alternatif, tetap butuh evidence.

**Next Action — jawaban akhir pengguna:**
- “aku tidak hafal command nya tapi aku akan cari event ID 1 dan 3 ... gk hafal command nya” → **pilihan investigation tepat**. Menghafal command bukan keharusan pada Day 17; gunakan catatan/help. Langkah yang benar adalah mencari historical Event ID 1 dan 3 relevan, matching ProcessGuid/time window, lalu membedakan temuan dari missing data.
- Command sederhana Get-WinEvent ID 1,3 -MaxEvents 10 dibagikan sebagai referensi, **belum dieksekusi**.

## 6. Mini SOC Report — dirangkai bersama mentor dari bukti aktual

**Case:** Repeated DNS Query / Process-DNS-Network correlation (real endpoint; tidak diklaim insiden).
**Source:** Microsoft-Windows-Sysmon/Operational.
**Process:** MuMuNxMain.exe.
**ProcessGuid:** [REDACTED_PROCESSGUID].
**Time range:** 2026-10-13 16:03:11–16:49:53 local (46m42s), 47 matching DNS events.

**Observation:** MuMuNxMain.exe berulang kali melakukan DNS query www.163.com sekitar sekali per menit.
**DNS:** Event ID22, QueryStatus 0, QueryResults berisi CNAME dan beberapa resolved IPs.
**Network:** DestinationIp / DestinationPort: **not established by matching Event ID 3** dalam queried dataset.
**Event correlation:** 47 records ID22 dengan ProcessGuid yang sama. ID1/ID3 tidak ditemukan dalam 1.000-event search.
**Hypothesis 1 (benign):** normal background/update check.
**Hypothesis 2 (suspicious):** periodic malicious communication mungkin, namun tidak terverifikasi; no exfiltration evidence.
**Evidence Available:** actual ID22 records, process name, PID at event, historical timestamps, QueryName/Status/Results, ProcessGuid matching.
**Evidence Gap:** matching Event ID1/ID3 belum ditemukan di window/sampel; real network connection, tujuan/legitimacy aktivitas belum diketahui.
**Test Performed:** Get-WinEvent ID22 + ProcessGuid filter atas ID1/3/22 1.000 events; sorting event by time.
**Actual Result:** 47 matching ID22 records, 0 matching ID1/3 in sampled events.
**Assessment:** **Needs Investigation / Insufficient Evidence for malicious verdict.** Pattern may be legitimate; correlation cannot establish network connection.
**Next Action:** search relevant historical Sysmon ID1 and ID3 by ProcessGuid and targeted time window, check retained logs/filters if not found. Executable file validation can follow if appropriate.

**Authorship note:** report akhir ini merupakan ringkasan **mentor-assisted** dari jawaban dan evidence pengguna; pengguna belum menulis seluruh report secara mandiri.

## 7. Evaluasi akhir

**Score 80/100 (formative) — PASS.** Alasan: berhasil mengambil DNS event asli, membuat timeline, membedakan query dari connection melalui remediasi, menghasilkan dua hypotheses, memilih Assessment dan Next Action tepat. Perlu bantuan besar pada PowerShell syntax, membaca field dan membuat assessment mandiri. PASS tidak berarti seluruh materi dikuasai.

**Carry-forward Active Recall Day18:**
1. Event ID 1 vs 2 vs 3 vs 22 (terutama ID 3).
2. QueryName vs QueryResults vs DestinationIp; DNS ≠ connection ≠ exfiltration.
3. Observation vs Assessment vs Next Action.
4. Baca angka dengan teliti: 35 queries/30 min tidak sama dengan 35 queries/1 min.
5. ProcessGuid ID correlation; PID dapat reused.
6. PowerShell foundational commands: variabel, Get-WinEvent, Format-List, sorting output sebelum loops/XML.
7. Tidak mengklaim missing events sebagai no activity; sebutkan batas pencarian.
