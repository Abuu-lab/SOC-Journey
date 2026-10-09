# 📌 Day 15 — Catatan Penting / Milestone Hari Ini

**Tema:** Windows Process-to-Network Investigation
**Status:** Belum PASS — nilai sementara 69/100; ambang PASS 76/100.

## Benar-benar dikerjakan / dipelajari hari ini
- Mengambil TCP connection menggunakan \`Get-NetTCPConnection -State Established\`.
- Mengidentifikasi LocalAddress, LocalPort, RemoteAddress, RemotePort, State, dan OwningProcess.
- Mengetahui bahwa **OwningProcess merupakan field berisi PID** koneksi TCP.
- Memeriksa koneksi publik \`192.168.1.7:64894 → 4.213.25.240:443\` dengan PID 4320; proses pemilik PID 4320 belum diidentifikasi.
- Memeriksa koneksi IPv6 **loopback** \`::1:51455 → ::1:35783\` dengan PID 22704.
- Berhasil menghubungkan \`OwningProcess 22704\` dengan \`EpicGamesLauncher.exe\`, PPID 11220 dan user \`DESKTOP-6MVPCMR\elvin\`, menggunakan \`Win32_Process\` dan \`Get-Process -IncludeUserName\`.
- Memahami bahwa loopback berarti komunikasi internal host, bukan bukti koneksi internet.
- Mencoba membaca Sysmon Event ID 3 tetapi mendapat **NoMatchingLogsFound** (channel Sysmon tidak ditemukan pada host yang diperiksa), bukan hanya tidak ada Event ID 3.
- Berlatih correlation Event ID 1/3 dengan ProcessGuid dan jendela waktu 4 detik pada scenario simulasi; Q2/Q3 benar.
- Mengidentifikasi pentingnya menguji path file, hash, signature, network context sebelum assessment.
- Mengembangkan *evidence-first reasoning*, tetapi masih sering menganggap port 443 otomatis legitimate.
- Mulai membedakan **parent process** vs **network connection owner**; challenge C1 masih salah.
- Mini SOC Report belum dikirim oleh pengguna; mentor dapat merekonstruksi contoh dari bukti asli tetapi tidak boleh mengklaim pengguna sudah menyelesaikannya.

## Prioritas perbaikan / Active Recall berikutnya
- ProcessGuid vs PID; IntegrityLevel vs Privileges; User/Groups/Privileges.
- \`services.exe\` → \`svchost.exe\` → Service; service group \`-k CameraMonitor\`.
- \`Win32_Service\` vs \`Win32_Process\`; user pada Sysmon vs service StartName.
- Mengapa 443 tidak otomatis HTTPS/legitimate.
- Perbedaan suspicious indicator dengan evidence gap.
- Event ID 1 (process creation) vs Event ID 3 (network), current connection vs historical evidence.
- Ketelitian host context: output Day 14 \`DESKTOP-C7BHMKL\Buya\`, Day 15 \`DESKTOP-6MVPCMR\elvin\`.
- Mengumpulkan observation, hypothesis, test, result, evidence gap dan final assessment dalam mini SOC report.

> Milestone ini membedakan kemampuan yang **telah dipraktikkan** dari konsep yang **baru diperkenalkan/belum dikuasai**. Tidak mengklaim PASS sebelum mencapai threshold.