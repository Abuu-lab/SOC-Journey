# 📌 Day 16 — Milestone Hari Ini

**Status: PASS (77/100 formative assessment)**

**Tema:** Security Telemetry Troubleshooting & Evidence Validation.

## Dibuktikan oleh praktik aktual hari ini
- Berhasil menghitung distribusi Sysmon event IDs menggunakan Get-WinEvent, Group-Object Id, Select-Object, Sort-Object.
- Mendeteksi **71 Event ID 1 Process Create**, **108 Event ID 3 Network Connection**, dan **352 Event ID 22 DNS Query** pada log yang tersedia saat pemeriksaan.
- Menemukan Event ID tambahan 2 (29), 4 (1), 5 (3), 11 (4), 13 (5), 16 (1).
- Memvalidasi Sysmon service: **Running**, **Automatic**, dan Win32_Service state/startmode/ProcessId/PathName.
- Mengetahui Event ID 3 memang telah tercatat pada sistem praktik, berbeda dari dugaan sebelumnya.
- Mengambil Sysmon Event ID 1 lengkap dan mengenali EpicGamesUpdater.exe (melalui koreksi), parent EpicGamesLauncher.exe, medium integrity, command line queryinstallation.
- Berlatih correlation dengan ProcessGuid dan memahami bahwa current process bisa sudah hilang walaupun historical process event masih ada.
- Melakukan simulated incident triage ketika current PID tidak ditemukan dan menghindari kesimpulan false positive yang tidak didukung.
- Berlatih mengevaluasi telemetry gaps dan perbedaan 'no event' vs 'no activity', meski masih perlu memperkuat penalaran ini.
- Berlatih membedakan Observation, Hypothesis, Test, Expected Result, Evidence Gap, Assessment, Next Action, namun mini case belum dikerjakan dengan sempurna.

## Belum dikuasai / Active Recall Day 17
- **PID vs ProcessGuid**: ProcessGuid = instance correlation identifier, bukan gabungan PID + koneksi.
- **Network Event ID 3**: Running Sysmon tidak memastikan logging semua Event ID. Histories can be filtered, incomplete or absent.
- **Image → Process Name**: basename Image adalah executable name, mis. EpicGamesUpdater.exe.
- **Evidence Available vs Evidence Gap**: event yang sudah diberikan tidak boleh dicatat sebagai missing evidence.
- **Historical vs Live Evidence**: perbedaan cakupan dan keterbatasan.
- **Assessment**: harus berupa penilaian berdasarkan evidence, bukan sekadar tindakan berikutnya.
- **Case separation**: hasil nyata (ID 3 sebanyak 108) tidak boleh dicampur dengan kasus simulasi (ID 3 tidak ditemukan).

## Privasi portfolio
Hostname, local username, user-directory path dan machine-specific GUID tidak dimasukkan ke file GitHub. Gunakan [REDACTED_HOST] / [REDACTED_USER] bila perlu.

> Milestone mencatat praktik dan konsep yang benar-benar muncul hari ini, tanpa menyatakan semua poin sudah dikuasai.
