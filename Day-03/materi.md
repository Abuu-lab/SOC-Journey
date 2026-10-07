# Day 03 — Materi

**Process Investigation**

### Konsep utama

- **ProcessId (PID)** mengidentifikasi process; **ParentProcessId (PPID)** adalah PID parent process.
- Untuk mencari parent, query process dengan PID yang sama dengan PPID child.
- **ExecutablePath** menunjukkan lokasi executable; **CommandLine** menunjukkan command beserta arguments.
- Parent dan child dapat menjalankan executable yang berbeda dan memiliki path berbeda.
- Parent browser, lokasi Temp, dan CPU tinggi memerlukan investigasi lanjutan sebelum verdict.

### Praktik

Gunakan Win32_Process untuk mencatat Name, PID, PPID, ExecutablePath, dan CommandLine pada Firefox, Spotify, dan Discord. Hasil hands-on, mini-project, challenge, self-test, dan koreksi tersedia di [Evaluasi](evaluasi.md).

### Alur investigasi

Process → parent process → command line → executable path → file evidence → network/context → assessment.
