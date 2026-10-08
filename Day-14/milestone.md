# Day 14 — Milestone Hari Ini

**Tema:** Security Context, IntegrityLevel & Svchost Investigation  
**Status:** PASS — 78/100 (estimasi pembelajaran, bukan ujian standar)

## 📌 Catatan Penting / Milestone Hari Ini
- Menggunakan `Get-Process -IncludeUserName` untuk **current process + username**.
- Menemukan Sysmon **Event ID 1** PowerShell historis dengan ProcessGuid, PID, User, IntegrityLevel, CommandLine dan Parent.
- Mempraktikkan perbedaan **Current State vs Historical Evidence** ketika PID lama tidak tersedia.
- Mempraktikkan `Test-Path` dan memaknai hasil **True**: file ditemukan di lokasi yang diperiksa.
- Mengidentifikasi `PowerShell` yang berjalan dengan **IntegrityLevel High**; belajar bahwa High bukan bukti malicious dan berbeda dari privileges.
- Memperkenalkan ulang **ProcessGuid** untuk process instance correlation ketika PID berubah/reused (masih perlu active recall).
- Memahami hubungan **services.exe → svchost.exe → hosted Windows service**, serta fungsi `-k <group>` (masih perlu reinforcement).
- Melatih korelasi **Win32_Service.ProcessId** dengan PID svchost; praktik live PID lama tidak berhasil karena proses tidak ada.
- Mempelajari perbedaan **Sysmon User** dan **Win32_Service StartName** dalam mini case.
- Melatih **investigation workflow**: alert → PID/parent → user/commandline → file evidence → network/other telemetry → assessment.
- Menggunakan **Get-Item**, **Get-AuthenticodeSignature**, dan **Get-FileHash -Algorithm SHA256** dalam jawaban skenario.
- Memahami pentingnya membedakan **PowerShell executable** dari **script `update.ps1`** yang dijalankan.
- Mempelajari **risk-based triage**, evidence gap, dan *unusual ≠ malicious*.

## Masih Lemah — untuk Active Recall Day 15
- Mengingat ProcessGuid vs PID dan PID reuse tanpa melihat materi.
- IntegrityLevel Medium/High/System vs User, Groups, Privileges.
- Syntax `Win32_Service` dan proses-host-service relationship.
- Identifikasi Process User vs service StartName (NETWORK SERVICE vs SYSTEM).
- Query historical Event ID 1 dengan time window dan field ProcessId tepat, bukan hanya `-MaxEvents 50`.
- Command `Test-Path`, perbedaan service group dan bukti akses kamera.

> Milestone adalah materi yang dipelajari atau dilatih hari ini, **bukan klaim bahwa semua poin sudah dikuasai**.
