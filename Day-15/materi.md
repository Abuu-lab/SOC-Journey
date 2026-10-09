# Day 15 — Materi
## Process-to-Network Investigation | Windows & Sysmon

**Learning goal:** menghubungkan network connection → OwningProcess/PID → process → user/parent/commandline → assessment. Dibangun di atas Day 14: User, Groups, Privileges, IntegrityLevel, ProcessGuid, services.exe vs svchost.exe.

### Istilah utama
- \`LocalAddress\` / \`LocalPort\`: endpoint lokal.
- \`RemoteAddress\` / \`RemotePort\`: endpoint lawan komunikasi.
- \`State: Established\`: koneksi TCP sudah terbentuk.
- \`OwningProcess\`: field berisi PID dari proses yang memiliki TCP connection.
- PID bisa digunakan kembali setelah proses mati.
- \`ProcessGuid\`: identitas instance proses Sysmon untuk mengorelasikan event (contohnya Event ID 1 dan Event ID 3).
- \`::1\`: IPv6 loopback, koneksi internal pada host itu sendiri.
- Remote port 443 umum untuk HTTPS tetapi bukan jaminan HTTPS, keamanan, atau legitimasi.

### Investigative reasoning
**Observation → Hypothesis → Test → Result → Evidence Gap → Assessment.**
Satu koneksi keluar bukan otomatis exfiltration; loopback bukan koneksi ke internet; High Integrity bukan otomatis malicious; file di Temp bukan otomatis malware.

### Perbedaan telemetry
- \`Get-NetTCPConnection\`: current TCP connections; berubah sesuai waktu.
- \`Get-CimInstance Win32_Process\`: current process context.
- \`Get-Process -IncludeUserName\`: current process + user.
- Sysmon Event ID 1: historical process creation.
- Sysmon Event ID 3: historical network connections, **bila telemetry diaktifkan**.
- Channel Sysmon *tidak ditemukan* berbeda dari channel ada tetapi tidak terdapat Event ID 3.
- Sysmon Event ID 3 dinonaktifkan secara default: lihat Microsoft Learn, https://learn.microsoft.com/en-au/sysinternals/downloads/sysmon.

### Data praktik asli, jangan digabung
**Praktik 1 (koneksi publik, process belum diketahui):**
\`192.168.1.7:64894 → 4.213.25.240:443\`, Established, OwningProcess 4320. TIDAK ada output nama executable untuk PID 4320.

**Praktik 2 (loopback):**
\`::1:51455 → ::1:35783\`, Established, OwningProcess 22704.
Proses: \`EpicGamesLauncher.exe\`, PPID 11220,
\`C:\Program Files\Epic Games\Launcher\Portal\Binaries\Win64\EpicGamesLauncher.exe\`,
User: \`DESKTOP-6MVPCMR\elvin\`.
Tidak ada bukti bahwa EpicGamesLauncher pemilik koneksi \`4.213.25.240\`.

**Praktik 3:** query Sysmon Operational gagal: \`NoMatchingLogsFound\`. Sysmon channel tidak ditemukan pada host yang sedang diperiksa. Context host berbeda dari Day 14: dulu \`DESKTOP-C7BHMKL\Buya\`, kini \`DESKTOP-6MVPCMR\elvin\`; perlu verifikasi \`hostname\` dan \`whoami\`.

### Mini lesson—Active recall yang belum kuat
- User = account; Groups = membership; Privileges = kemampuan spesifik dalam access token.
- IntegrityLevel: Mandatory Integrity Control level; High ≠ semua privilege.
- services.exe = Service Control Manager, svchost.exe = process host untuk service tertentu.
- \`svchost.exe -k CameraMonitor\`: nama grup host layanan; bukan bukti kamera merekam.
- \`Get-CimInstance Win32_Service\` untuk service; \`Win32_Process\` untuk process.
- Sysmon \`User\` dan Win32_Service \`StartName\` bukan field identik.

### Referensi teknis
- https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-nettcpconnection
- https://learn.microsoft.com/en-au/sysinternals/downloads/sysmon
- https://learn.microsoft.com/en-us/windows/security/operating-system-security/sysmon/sysmon-events
