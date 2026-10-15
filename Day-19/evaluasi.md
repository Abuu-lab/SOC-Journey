# Day 19 — Evaluasi, Jawaban Pengguna & Koreksi

**Final status: PASS — 80/100 (mentor formative).** Initial snapshot at the beginning of Day19: Active Recall + Hands-on 1–5 submitted. Hands-on 6 reasoning answered without separate correlated event evidence. Simulated Challenge questions C1–C9 answered (C10 unanswered). Mini SOC Project **not yet submitted**: learner asked to be reminded of the commands. Do not fabricate final grade/PASS.

**Data protection:** Raw host/user/profile paths, local device IP and unique ProcessGuid suppressed. Simulated evidence clearly separated from real endpoint results.

## 1. Active Recall (verbatim short answers + correction)

| Q | Jawaban pengguna | Koreksi dan alasan |
|---|---|---|
| Q1 SHA-256/virus | "fungsi sha 256 untuk mengetahui nilai hash sebuah file, bisa jadi tapi bisa juga tidak" | Fungsi hash benar; **SHA-256 tidak mendeteksi virus**. It produces a fingerprint to compare, not verdict. |
| Q2 Current vs historical | "klo current kan proses yang sedang berlangsung, kalau historical evidence event yang telah terjadi sebelumnya" | Benar inti: Get-CimInstance Win32_Process current process snapshot, Sysmon ID1 historical Process Create. |
| Q3 False positive | "pernah dicurigai, tapi tidak ada bukti kuat" | Belum tepat: inconclusive != false positive. False positive is alert incorrectly indicating the targeted threat after sufficient verification. |
| Q4 Signature | "ya karena bisa jadi attacker juga memiliki digital signature yg valid juga" | Benar prinsip bahwa Valid != safe. Signed legitimate executable may be abused; signatures themselves do not validate all behavior. |
| Q5 UDP443 | "msih bingung memahami kalimat soalnya" | If Protocol=udp, CANNOT infer HTTPS **via TCP**. UDP443 may be QUIC but Event3 alone not verification. |
| Q6 ID 1/3/22 | "event 1 berisi historical event proses, event 3 berisi historical event network, event 22 berisi historical event DNS Query" | Correct; specifically process creation, network connection, DNS Query. |
| Q7 PID reuse | "agak susah memahami soalnya, tolong berikan contoh" | Illustrative: Process A uses PID5000 (GUID-A) at 9AM and terminates. Later Process B uses PID5000 (GUID-B). Same PID, different process instance. |
| Q8 Not checked signature | "salah, karena sebagai seorang SOC tidak boleh langsung menyimpulkan bahwa file tersebut malware, masih banyak evidence yg belum di check" | Correct; Not checked does NOT equal Invalid. Confirmed Malware verdict unsupported. |

## 2. Hands-on 1 & 2: Discord Process/Parent

- P1 "6": correct. Six different PIDs.
- P2 "tidak, ada 1 PID yg beda": **incorrect**. All six PIDs are different; five child processes share ParentProcessId 2724.
- P3 "tidak ada 1 yg beda, kenapa ya kalau semua PPID yg sama?": five processes have **same PPID2724**, suggesting same parent (the Discord.exe instance PID2724). PID2724 itself has ParentProcessId 4632. Shared PPID is not shared PID.
- Actual query for ProcessId2724 returned **Name: Discord.exe, PID:2724, ParentProcessId:4632**.
- P4 "4632": **incorrect if question asks parent process name**. 4632 is PPID of queried Discord.exe, not name of parent process. Must query ProcessId4632 for parent identity if still active, or find historical log.
- P5 "tidak, semua sama": valid *for current sample* where all Image basenames Discord.exe. In general parent/child **can** have same or different names.
- P6 "tidak, karena bisa saja proses tersebut sudah tidak berjalan, kita bisa melihat dari historical event": correct.

## 3. Hands-on 3: Sysmon Process Create

- Actual: 2026-10-15 17:12:32 local, opushutil.exe under Office installation, commandline /pushregistration, parent svchost.exe, `-k netsvcs -p -s Schedule`, Medium Integrity. GUID and local user redacted.
- P7 "opushutil.exe": correct.
- P8 "svchost.exe": correct.
- P9 "karena ... Historical Event ... process hilang pada current, tapi tidak hilang pada historical event": correct as long as event remains retained in log. Sysmon Event ID1 != live process view.

## 4. Hands-on 4: File Investigation

- Actual: Discord.exe currently exists; size 220542392 bytes; created 2026-10-08 16:02:01, last write 16:02:04. Authenticode Valid, signature verified. SHA256 successfully computed.
- P10 "sha256 untuk memeriksa nilai hash ... digitalsignature ... publisher/valid ... metadata ... pembuatan, kapan di check, kapan diubah": mostly correct. Metadata output displays CreationTime and LastWriteTime, not a field called "last checked" in that command. SHA256 is a fingerprint.
- P11 "belum tentu, bisa saja user mengubah ... atau malware ...": correct no automatic malware verdict. Legitimate software update/reinstall is another plausible explanation; changing signed contents generally invalidates the original signature unless newly signed.
- P12 "nggk boleh ... investigasi lebih dalam": correct principle. If commandline unusual, test behavior and context; signature alone is insufficient.

## 5. Hands-on 5: Sysmon Network Events

- Actual 3 Spotify.exe Sysmon ID3: 17:24:40 PID22300 TCP outbound external:443; 17:24:25 PID6680 UDP Initiated=false from multicast SSDP endpoint; 17:24:24 PID6680 UDP Initiated=true to multicast SSDP endpoint. Two distinct ProcessGuids/instances, all images Spotify.exe.
- P13 "3": correct.
- P14 "Spotify.exe 3 event": correct.
- P15 "1 process menggunakan tcp 2 process menggunakan udp": correct if meant **one event TCP and two events UDP**; inaccurate if meant three distinct process instances. There were two PIDs and GUIDs across 3 events.
- P16 "tidak cukup ... bukti lain": correct. Port 443 isn't a safe/unsafe verdict.

## 6. Hands-on 6: Timeline

- P17 "event 1 > event 3 > event 22": **incorrect for supplied data**. Actual available entries: ID1 17:12:32 opushutil.exe; ID3 17:24:24 UDP Spotify; ID3 17:24:25 UDP Spotify; ID3 17:24:40 TCP Spotify. No ID22 record in sample.
- P18 "ProcessGuid": correct type of evidence, but **observed GUIDs differ** among Office and Spotify, plus two Spotify process instances. Do not correlate same host+time into same process.
- P19 "kita tidak mendapatkan bukti yg lebih luas lagi ... waktu berdekatan": correct. Causality requires stronger correlation than temporal proximity.

## 7. Simulated SOC Case #019

**Simulated data only:** updater.exe in Public, signed Valid, `--silent`, parent explorer.exe, GUID LAB-019-A; ID1 at 10:15:21 and matching ID3 TCP to documentation IP:443 at 10:15:25; ID22 Not checked.

| C | Jawaban pengguna | Correction |
|---|---|---|
| C1 | "menemukan ProcessGuid ... commandline dan PID" | Partial. State process creation: updater.exe from C:\Users\Public at 10:15:21, parent explorer.exe, commandline updater.exe --silent, PID6200 and ProcessGuid. |
| C2 | "DestinationIP, Destinationport, protocol" | Partial. Explicit values: updater.exe initiates TCP toward 203.0.113.50:443 at 10:15:25. These are fictitious documentation IPs. |
| C3 | "iya benar, memang processguid berfungsi seperti itu" | Correct: GUID LAB-019-A appears in both ID1 and ID3, supporting same process instance. |
| C4 | "belum tentu, kita harus melihat evidence lain" | Correct: Public path is not automatic malware proof. |
| C5 | "tidak boleh menyimpulkan hanya dengan 1 evidence" | Correct: valid signature by itself cannot close alert as false positive. |
| C6 | "aku tidak tau apa arti dari argument silent" | Teach: --silent generally runs without dialogs or user interaction, common in normal installers/updaters too. |
| C7 | "belum di periksa, kita bisa liat dari ProcessGuid nya" | Correct Not checked vs no event. GUID useful only after retrieving relevant actual DNS events. |
| C8 | "benign: ... executablepath ... public ... anomaly; malicious: ... silent ... suspicious" | Does not give a benign hypothesis. Public path and --silent alone are not malicious indicators. Ideal benign: authorized signed updater running in quiet mode; malicious: renamed/abused binary needing verification. |
| C9 | "masih perlu investigasi ... mencari historical event 22" | Needs Investigation is reasonable, though ID22 is just one optional corroboration and should not be assumed present. |
| C10 | no answer submitted | Not completed. Ideal order: validate historical process/user/parent and commandline, verify executable signature/hash and publisher context, then investigate related network/DNS activity for that GUID/time window. |

## 8. Mini-project

User message: **"ini command nya apa aja ya? lupa untuk dapat semua informasi itu"**. This is a request for guidance, **NOT a completed mini-project**. Do not invent Assessment or finalized mini report.

Suggested grounded real case for next step: one Spotify.exe ID3 process instance (PID6680 for UDP multicast or PID22300 for TCP) and its actual event record, NOT the unrelated opushutil.exe ID1. Use only existing verified evidence, mark unperformed file/parent/DNS checks as NOT CHECKED.

## 9. Current progress & learning goals

Strengths: many real read-only PowerShell commands executed, current vs historical understanding, no port=malware inference, use of file metadata/signature/hash, basic Sysmon field reading and evidence caution.

Active recall next: all PID unique vs same PPID; parent name vs PPID; false positive vs inconclusive; SHA256 vs virus scan; UDP vs HTTPS/TCP; actual timeline and differing ProcessGuids; hypothesis benign vs malicious; exact values and source separation. Day 19 final grade only after mini report and correction of missing steps.

## Penutupan Day 19 — Penilaian Akhir (menggantikan status sementara di atas)

**Status final: PASS, 80/100 — estimasi formatif mentor.** Ambang PASS 76/100. Nilai bukan hasil tes kompetensi standar dan bukan klaim pengguna telah menulis mini report mandiri.

### Bukti baru setelah evaluasi sementara

- User mengeksekusi query current PID **6680** → hasil kosong.
- User mencari Sysmon ID 1 dalam **200 terbaru** berdasarkan teks PID 6680 → kosong; pencarian terbatas tidak membuktikan event tidak pernah ada.
- User memperluas pencarian dengan time window pada 15 Oktober 2026 → menemukan **5 historical Sysmon ID 1**, semuanya child Spotify.exe dengan ParentProcessId 6680.
- Lima record tersebut mempunyai ParentProcessGuid **sama**, dan GUID tersebut **cocok dengan ProcessGuid Spotify UDP Sysmon ID 3** milik PID 6680 yang diamati lebih awal.
- Masing-masing child memiliki ProcessGuid yang berbeda. Ini menunjukkan keterkaitan parent–child dan network activity pada satu **parent process instance**.
- Tidak menemukan Event ID 1 yang secara langsung mendokumentasikan **penciptaan parent PID 6680** dalam lima output itu.
- C10 belum dijawab mandiri; berikut contoh jawaban dari mentor, bukan jawaban pengguna.

### C10 — Contoh jawaban mentor, bukan jawaban pengguna

Urutan: (1) Periksa konteks Sysmon ID 1 termasuk parent, user, command line. (2) Verifikasi file executable, metadata, digital signature, dan hash. (3) Korelasikan network/DNS events yang benar-benar ada berdasarkan ProcessGuid dan waktu.

### Mini SOC Investigation Report — Disusun mentor dari data pengguna

**CASE:** Historical Spotify Parent-Child and UDP Activity

**SOURCE:** Microsoft-Windows-Sysmon/Operational

**OBSERVATION:** Pada 15 Oktober 2026, lima child Spotify.exe dicatat dibuat pukul 15:12:20–15:14:19 waktu lokal. Child PIDs: 7160, 6308, 7208, 18628, 13636. Semuanya mempunyai ParentProcessId 6680 dan ParentProcessGuid sama. Dua Sysmon Event ID 3 pada 17:24:24 dan 17:24:25 mencatat aktivitas UDP milik Spotify.exe PID 6680 dengan ProcessGuid yang cocok; ada aktivitas ke/dari alamat multicast 239.255.255.250 pada port 1900.

**PROCESS INVESTIGATION:** PID 6680 tidak ditemukan ketika current process list diperiksa. Parent dari PID 6680 sendiri belum diketahui dari output yang ada. Pencarian hanya 200 ID1 terbaru menghasilkan no match, lalu pencarian time window mengungkap lima child ID1; keduanya konsisten dengan batasan masing-masing query.

**FILE INVESTIGATION:** Metadata, signature dan SHA-256 **file Spotify** belum diuji pada kasus ini. Signature Valid dan hash yang diuji di bagian lain pada Day 19 adalah **file Discord**, bukan Spotify.

**EVIDENCE AVAILABLE:** Child ID1 with same ParentProcessGuid, two matching parent UDP ID3 records, historical timestamps, roles of child process. Spotify TCP record pada PID 22300 mempunyai ProcessGuid lain dan **tidak digabungkan** sebagai satu process instance.

**EVIDENCE GAP:** Process Create Event ID 1 milik PID 6680 sendiri, parent process/commandline awalnya, signature/hash Spotify, purpose/application-layer detail, relevant DNS events (not verified).

**BENIGN HYPOTHESIS:** Spotify mungkin melakukan aktivitas local-device discovery lewat SSDP/multicast sebagai fungsi normal aplikasi.

**POTENTIALLY MALICIOUS HYPOTHESIS:** Penyalahgunaan proses atau aktivitas jaringan yang menyimpang belum dapat dikesampingkan; UDP sendiri **bukan** indikator malicious.

**ASSESSMENT:** **Insufficient Evidence / Needs Investigation.** Bukti menunjukkan parent/child relationship dan network activity; belum ada cukup indikator untuk confirmed malware, namun belum ada verifikasi lengkap untuk menyatakan false positive.

**NEXT ACTION:** Jika dilanjutkan, cari Event ID 1 parent PID 6680 berdasarkan identity/time; periksa file Spotify yang tepat; korelasikan jaringan/DNS pada ProcessGuid yang sama.

### Alasan nilai 80/100

Kredit: hands-on read-only, historical/current differentiation, pengamatan field Sysmon, file checks terhadap Discord, keberhasilan menemukan **cross-event parent ProcessGuid correlation** pada Spotify, dan berhati-hati menghindari verdict dini. Koreksi: PID vs PPID, false positive vs inconclusive, SHA256, timeline, hypothesis, dan laporan final masih disusun dengan bantuan mentor.

**PASS:** kelemahan dibawa ke Active Recall Day 20, bukan menahan progres. Aturan remedial: nilai 76–100 PASS; nilai di bawah 76 → materi singkat + 2–5 soal/praktik yang fokus sampai memenuhi ambang.
