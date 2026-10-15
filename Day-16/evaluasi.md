# Day 16 — Evaluasi dan Koreksi Lengkap

**Status: PASS | Nilai: 77/100 (perkiraan formatif mentor).**
Ambang kelulusan adalah 76. Kesalahan kecil dibawa ke Active Recall Day 17 tanpa mengulang seluruh materi hari itu.
**Privasi bukti:** dokumen ini tidak memuat hostname mentah, username lokal, jalur profil, atau ProcessGuid khusus host.

## 1. Active Recall Q1–Q8

| Soal | Jawaban pengguna (ringkas, setia pada maksud) | Koreksi / jawaban ideal |
|---|---|---|
| Q1 | update.exe; parent explorer.exe | Benar: update.exe pemilik connection, explorer.exe parent. |
| Q2 | serangan dapat masuk lewat port '433' | Port yang dimaksud 443. Port 443 adalah nomor layanan umum untuk HTTPS; tidak membuktikan protocol atau legitimate behavior. Malware pun bisa menggunakan HTTPS. |
| Q3 | PID adalah ID proses; ProcessGuid mengandung PID dan correlation koneksi | PID bisa dipakai ulang; ProcessGuid mengidentifikasi instance proses untuk correlation antar Sysmon events. Tidak tepat menyatakan ProcessGuid = PID + koneksi. |
| Q4 | Event 1 historical process; Event 3 historical connection | Benar, tepatnya Process Create dan Network Connection. |
| Q5 | ::1 IPv6 loopback dua endpoint, Established sudah terhubung | Benar: loopback connection dalam host sendiri. |
| Q6 | channel belum tahu; ID tidak ada mungkin belum terjadi | Channel unavailable = log yang diminta tidak ada/terdaftar pada host yang diperiksa; event tidak ada = ID tidak ditemukan di channel saat query (logging disabled/filtered/rolled over atau memang tidak terjadi). |
| Q7 | High bukan semua privileges, lupa alasan | High = elevated integrity level, bukan pemberian setiap privilege. Privileges adalah hak khusus dalam access token. |
| Q8 | process menggunakan group CameraMonitor | -k CameraMonitor menunjuk service host group, bukan bukti perangkat kamera dipakai. |

## 2. Praktik 1: Sebaran Event ID Sysmon

Command pengguna:
~~~powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' |
Group-Object Id | Select-Object Name, Count | Sort-Object Name
~~~

Hasil asli (jumlah catatan per ID):
- ID 1: 71
- ID 2: 29
- ID 3: 108
- ID 4: 1
- ID 5: 3
- ID 11: 4
- ID 13: 5
- ID 16: 1
- ID 22: 352

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P1 | IDs 1,2,3,4,5,11,13,16,22 | Benar. |
| P2 | Memberikan hitungan sembilan ID lengkap | Benar. |
| P3 | Event ID 3 mungkin belum tercatat | Benar untuk skenario hipotesis; pencatatan juga bisa dinonaktifkan, tersaring, atau catatan lama tertimpa. Pada data aktual, ID 3 ADA (108 catatan). |
| P4 | 'agar tidak terkesan menyalahkan' | Tujuan memeriksa kondisi telemetry adalah agar kita tidak menyamakan “tidak ada bukti” dengan “tidak ada aktivitas”, sehingga tidak menghasilkan false negative (ancaman terlewat). |

## 3. Praktik 2: Kondisi Layanan Sysmon

Hasil: Sysmon berstatus `Running`, jenis startup `Automatic`; Win32_Service menunjukkan `State Running`, `StartMode Auto`, `ProcessId 26508`, dan `PathName C:\Windows\Sysmon.exe`.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P5 | Sysmon | Benar. |
| P6 | Running dan Auto | Benar. |
| P7 | State kondisi sekarang; StartMode otomatis/manual | Benar. `State` menunjukkan status layanan saat diperiksa; `StartMode` menunjukkan pengaturan startup (auto/manual/disabled). |
| P8 | 'ya sudah berjalan' | Salah. Layanan berstatus `Running` saja tidak membuktikan Event ID 3 dicatat; dalam kasus ini, **108 catatan Event ID 3** membuktikan bahwa event tersebut memang tercatat. |

## 4. Praktik 3: Catatan Historis Process Create

Contoh Event ID 1: `TimeCreated` 2026-10-10 13:31:44 waktu lokal; `ProcessId` 21144; akhir `Image` adalah EpicGamesUpdater.exe; akhir `ParentImage` adalah EpicGamesLauncher.exe (PPID 15924); `IntegrityLevel Medium`; command line `EpicGamesUpdater.exe queryinstallation`. User, ProcessGuid, dan host disamarkan.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P9 | 'gk ada nama' | Nama proses **EpicGamesUpdater.exe**, diambil dari basename field Image. |
| P10 | EpicGamesLauncher.exe | Benar, ParentImage. |
| P11 | menggunakan ProcessGuid | Benar, cocokkan ProcessGuid antar events untuk instance sama. |
| P12 | tetap menggunakan historical event | Benar; current process gone tidak berarti investigasi berhenti. |

## 5. Praktik 4: Validasi Bukti (SIMULASI)

Skenario: Event 1 menunjukkan `powershell.exe` PID 8200, ProcessGuid LAB-016-A, parent `explorer.exe`, dan script `maintenance.ps1` yang dijalankan; pemeriksaan CIM sesudahnya tidak menemukan PID 8200.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P13 | tidak tepat, cari historical evidence | Benar; PID sudah hilang bukan false positive otomatis. |
| P14 | process name, PID, ProcessGuid, CommandLine, ParentImage | Benar: semua field tersebut sudah tercantum dalam event. |
| P15 | historical Event ID 1/3 | Sebagian keliru: Event ID 1 SUDAH DIBERIKAN; yang belum tersedia ialah isi script, metadata file, hash/signature jika relevan, perilaku, dan telemetry jaringan. Event 3 tidak ditampilkan. |
| P16 | correlate ProcessGuid, check hash/metadata/signature, ID1/ID3 | Langkah berikutnya masuk akal. Prioritaskan pemeriksaan script dan event historis yang relevan; jangan menganggap semua event pasti tersedia. |
| P17 | script perlu diperiksa meskipun namanya maintenance | Kesimpulannya benar karena `powershell.exe` adalah interpreter dan script berisi instruksi yang dijalankannya. |

## 6. Mini SOC Case (SIMULASI; diasumsikan ID 3 tidak ditemukan)

**Makna jawaban pengguna dipertahankan:**
- Observasi: Sysmon berjalan, “43 Event ID 1” (keliru).
- Hipotesis: Event ID 3 belum tercatat; `Get-NetTCPConnection` menunjukkan koneksi berstatus `Established`.
- Hasil yang Diharapkan: channel Sysmon Operational tersedia.
- Bukti: Sysmon ID 1.
- Kesenjangan Bukti: Sysmon ID 3.
- Penilaian: “cari menggunakan historical event dan get tcp connection” (ini merupakan tindakan berikutnya, bukan penilaian).
- Tindakan Berikutnya: cari Sysmon ID 3.

**Contoh jawaban ideal mentor:**
- Observasi: layanan berjalan, channel aktif, terdapat 43 *catatan keseluruhan* pada cuplikan hipotesis, ID1 historis tersedia, ID3 tidak ditemukan, dan koneksi saat ini berstatus `Established`.
- Hipotesis A: pencatatan Event ID 3 tidak diaktifkan atau dikecualikan oleh filter.
- Hipotesis B: catatan ID3 tidak tersedia dalam rentang waktu yang tersimpan.
- Pengujian: periksa sebaran event, konfigurasi Sysmon, dan rentang waktu; bandingkan dengan koneksi TCP saat ini.
- Hasil yang Diharapkan: mengetahui apakah pengumpulan ID3 dikonfigurasi dan apakah ada event yang cocok.
- Bukti yang Tersedia: status layanan/channel Sysmon, ID1, dan data langsung dari `Get-NetTCPConnection`.
- Kesenjangan Bukti: telemetry jaringan historis yang berkaitan dengan alert.
- Penilaian: visibilitas jaringan historis belum memadai; perlu investigasi lanjutan.
- Tindakan Berikutnya: periksa konfigurasi pencatatan dan sumber bukti alternatif; jangan mengarang hasil pengujian.

**Penting:** cuplikan praktik aktual menunjukkan 108 catatan Event ID 3; jangan mencampuradukkan skenario simulasi dengan hasil host yang sebenarnya.

## 7. Challenge C1–C5

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| C1 | Established connections should be logged in ID3, unless not recorded | Belum tentu. Pencatatan mungkin dinonaktifkan, tersaring, atau catatannya sudah tertimpa; tampilan saat ini juga berbeda dari bukti historis. |
| C2 | belum dapat bayangan | Ya. `Get-NetTCPConnection` dan `Win32_Process` memungkinkan pemeriksaan jaringan saat ini meskipun ID3 tidak tersedia. |
| C3 | live information may not yet be recorded | Keterbatasan utamanya adalah sifat data yang sementara: process atau koneksi mungkin sudah berhenti saat diperiksa, sedangkan bukti historis dapat menyimpan aktivitas sebelumnya. |
| C4 | belum dapat bayangan | PID yang sama dengan ProcessGuid berbeda dapat menunjukkan instance process yang berbeda atau penggunaan ulang PID; periksa timestamp dan `Image`. |
| C5 | B | Benar: ID3 yang tidak ditemukan tidak membenarkan kesimpulan “tidak ada aktivitas jaringan”. |

## 8. Penilaian Mentor

**Hal yang sudah kuat:** menjalankan perintah, mengelompokkan event, memeriksa status layanan, mengambil parent dari `Process Create`, dan tidak langsung menyamakan “PID sudah hilang” dengan false positive.

**Perlu diperkuat pada Day 17:** perbedaan PID dan ProcessGuid; kelengkapan Event 3; cara mengenali nama file dari `Image`; bukti yang tersedia dan kesenjangannya; menyusun penilaian serta hipotesis alternatif; dan keterbatasan data langsung dibandingkan bukti historis.

**Hasil:** PASS 77/100 berdasarkan kemajuan praktik yang bermakna. Ini adalah nilai formatif pembelajaran, bukan hasil sertifikasi atau ujian standar. Jawaban yang masih lemah dicatat untuk diperkuat.
