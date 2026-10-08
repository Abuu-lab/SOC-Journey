# Day 14 — Evaluasi

**Materi:** Security Context, Integrity Level & Svchost Investigation  
**Status:** PASS (penilaian pembelajaran terestimasi 78/100, ambang 76%).  
**Catatan:** Kemampuan investigation workflow berkembang; recall konsep Windows security dan service correlation masih perlu diperkuat. Nilai bukan skor tes standar.

## 1. Active Recall (Q1–Q12) — jawaban pengguna dan koreksi

| Q | Jawaban pengguna (ringkasan) | Evaluasi / jawaban yang tepat |
|---|---|---|
| 1 | PID ID process; lupa ProcessGuid | **Partial.** PID = nomor proses; **ProcessGuid** = identifier instance proses dari Sysmon untuk korelasi. |
| 2 | PID hanya untuk identifikasi | **Partial.** PID dapat digunakan kembali setelah proses berhenti; PID saja tidak cukup memastikan identitas proses historis. |
| 3 | User akun; lupa IntegrityLevel | **Partial.** User = identitas akun, IntegrityLevel = level integritas Windows (mis. Medium/High/System), bukan daftar privileges. |
| 4 | High belum tentu malicious | **Benar sebagian.** High mengindikasikan elevated integrity, umum pada proses administrator; bukan bukti serangan. |
| 5 | Belum tahu arti CameraMonitor | **Perlu belajar.** `-k CameraMonitor` menentukan nama svchost service host group. Nama dapat terkait fungsi kamera tetapi **bukan bukti kamera sedang digunakan**. |
| 6 | Service menghasilkan process svchost | **Perlu perbaikan.** `services.exe` adalah Service Control Manager, mengelola services dan dapat meluncurkan `svchost.exe` yang meng-host service berbasis DLL. Tidak semua service berjalan lewat svchost. |
| 7 | `Get-Service -Filter " ProcessId = 6920"` | **Belum tepat.** Gunakan `Get-CimInstance Win32_Service | Where-Object {$_.ProcessId -eq 6920}`. |
| 8 | File sudah tidak ada; lupa command | **Belum tepat.** File Exists adalah cek keberadaan file sekarang: `Test-Path "C:\path\file.exe"`; `True` masih ada, `False` tidak ditemukan di path itu. |
| 9 | Historical log; current belum dijawab | **Partial.** Historical = catatan kejadian sebelumnya (Sysmon); Current = state sekarang (`Get-CimInstance`). |
| 10 | Query Get-WinEvent, syntax kurang benar | **Partial.** Perlu separator: `@{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1}` dan `-MaxEvents 10`. |
| 11 | `Get-CimInstance Win32-Process` | **Partial.** Class benar `Win32_Process` (underscore), dengan filter ProcessId dan Select-Object. |
| 12 | User + group + command | **Belum tepat.** Model sederhana security context = User + Groups + Privileges; command line adalah process evidence lain. |

## 2. Practice — hasil dan koreksi

### Practice 1 — PowerShell Sysmon Event ID 1
**Output pengguna:** `powershell.exe`, PID 23808, ProcessGuid `{f1261a38-47b0-6ac6-88c3-000000001d00}`, User `DESKTOP-C7BHMKL\Buya`, IntegrityLevel `High`, Parent `explorer.exe` (PID 8368), command `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"`.  
**Jawaban pengguna:** User = Buya; IntegrityLevel mengharuskan privilege High; High bukan malware; periksa metadata.  
**Koreksi:** User benar. High adalah elevated integrity level, **bukan kewajiban memiliki suatu privilege bernama High**. Metadata layak diperiksa; korelasikan process dengan parent, alasan elevasi, aktivitas, dan event terkait. `Get-Process -IncludeUserName` melihat proses live, Sysmon Event ID 1 memberi historical process creation telemetry. **Status: selesai dengan koreksi.**

### Practice 2 — PID lama
**Jawaban:** PID 18824 dari hari sebelumnya hilang.  
**Koreksi:** Normal bila proses sudah mati, mesin reboot, atau PID telah berganti. Gunakan historical event dengan waktu dan ProcessGuid; jangan mengasumsikan PID lama masih menunjuk proses yang sama. **Status: live check tidak dapat dilaksanakan.**

### Practice 3 — Test-Path
**Jawaban:** `True` = file masih ada, `False` tidak otomatis malware.  
**Koreksi:** Benar. Ini hanya menguji keberadaan pada path saat itu, bukan keamanan file. **Status: benar.**

### Practice 4 — Historical versus current
**Jawaban:** Current process sudah hilang dan historical query tidak memberi hasil.  
**Koreksi:** `-MaxEvents 50` membatasi pencarian ke 50 event terbaru; PID 18824 dari hari sebelumnya bisa berada di luar batas tersebut. Mencari angka dalam `Message` juga bisa salah mengidentifikasi ParentProcessId. Gunakan `StartTime/EndTime`, parse XML `EventData.Data` dengan Name `ProcessId`, cek ProcessGuid. Hasil kosong bukan bukti proses tidak pernah berjalan. **Status: perlu pengulangan teknik query pada Day 15.**

## 3. Practice 5 — Challenge investigator (Q1–Q10)

- **Q1:** User mengidentifikasi `-ExecutionPolicy Bypass`. **Tambahan koreksi:** lokasi script `AppData\Local\Temp\update.ps1` juga indikator untuk ditelusuri, bukan bukti malware.
- **Q2:** User menyebut ExecutablePath. **Koreksi:** path PowerShell di System32, user Buya, parent explorer.exe dan penggunaan `-File` dapat legitimate; butuh baseline dan konteks.
- **Q3:** Hipotesis berdasar Bypass. **Koreksi ideal:** PowerShell sedang menjalankan script Temp dengan Bypass; bisa administrasi legitimate atau penyalahgunaan; masih uncertain.
- **Q4:** Metadata, signature, hash, network. **Benar**, ditambah pemeriksaan isi script dan historical related events.
- **Q5:** `Get-Item "ExecutablePath" | Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime`. **Syntax bagus**, tetapi ganti placeholder dengan path nyata; investigasi kedua objek: PowerShell executable dan `update.ps1`.
- **Q6:** `Get-AuthenticodeSignature "ExecutablePath"`. **Benar secara struktur**, juga cek script jika relevan; unsigned script tidak otomatis malicious.
- **Q7:** `Get-FileHash "ExecutablePath" -Algorithm SHA256`. **Benar secara struktur**, hash script perlu dihitung juga.
- **Q8:** Setelah hash belum selesai. **Benar.**
- **Q9:** Network analysis. **Benar**, dapat ditambah child process, file changes, network, threat intel.
- **Q10:** User menyusun alur CPU high → Task Manager → PID/PPID → explorer → PowerShell → commandline Bypass → metadata/signature/hash. **Bagus secara metode**, tetapi CPU high bukan evidence yang disediakan di skenario. Flow ideal mulai dari alert, verifikasi event, user, IntegrityLevel, parent, commandline, script, correlated telemetry, assessment.

## 4. Mini SOC Case (Q1–Q9)

**Scenario:** Sysmon `svchost.exe` PID 7420, parent `services.exe` PID 1424, **Process User = NT AUTHORITY\SYSTEM**, IntegrityLevel System, command `-k LocalServiceNetworkRestricted`. Win32_Service: service **Dnscache / DNS Client**, ProcessId 7420, StartName `NT AUTHORITY\NETWORK SERVICE`, State Running, StartMode Auto.

| Q | Jawaban pengguna | Koreksi tepat |
|---|---|---|
| 1 | DNS Client Running Auto, akun SYSTEM | Sebagian benar; **beda User process Sysmon vs StartName service**. |
| 2 | `Service.exe` | **`services.exe`**, PID 1424. |
| 3 | `NETWORK SERVICE` | Pertanyaan Process User: **NT AUTHORITY\SYSTEM** dari Sysmon. NETWORK SERVICE adalah service StartName dari Win32_Service. |
| 4 | System | **Benar**, IntegrityLevel System. |
| 5 | PID 1424 / service.exe | **Salah.** Service adalah **Dnscache (DNS Client)** pada PID 7420. |
| 6 | Belum cukup untuk malicious | **Benar.** Nama grup dan path tidak menentukan niat. |
| 7 | ExecutablePath looks normal | **Sebagian benar.** Parent `services.exe`, path System32, serta korelasi PID ke Dnscache menambah dukungan. |
| 8 | Ingin cek hash, signature, metadata, network, intel bila cukup waktu | **Perlu prioritas risk-based.** Tidak harus semua check untuk setiap proses; fokus bukti yang menjawab hipotesis dan risiko. |
| 9 | Assessment mengarah normal namun belum confirmed, evidence list + gap | **Sebagian benar.** Perlu menyebut perbedaan User Sysmon dan StartName. Jangan klaim timestamp belum ada karena UtcTime diberikan. |

**Assessment mentor:** **Likely Legitimate (low-to-moderate confidence, based on case data only).**  
**Reason:** Path, parent, svchost service association dan status service mengarah ke Windows service normal.  
**Evidence gap:** Perbedaan Process User `SYSTEM` dan service StartName `NETWORK SERVICE` perlu pemeriksaan/konteks; signature/hash dan aktivitas terkait belum dicek. Ini contoh skenario pembelajaran, bukan verifikasi endpoint aktual.  

## 5. Follow-up Active Recall Day 15

1. Jelaskan ProcessGuid vs PID serta PID reuse.
2. IntegrityLevel High vs privileges; contoh akun sama berbeda IL.
3. Perbedaan Sysmon User dan Win32_Service StartName.
4. `services.exe` vs `svchost.exe` vs service Dnscache.
5. `-k CameraMonitor`: service host group vs bukti camera access.
6. Command `Win32_Service` dengan filter PID.
7. `Test-Path` dan arti True/False.
8. Historical query: mengapa `-MaxEvents 50` bisa melewatkan event lama; korelasi ProcessGuid dan exact fields.
9. Di PowerShell `-File update.ps1`, bedakan executable dengan script.
10. Prioritas alert: risk-based triage dan evidence gap.

**Hasil akhir: PASS (78/100 estimasi mentor).** Belajar Day 15 tanpa mengulang semua Day 14.
