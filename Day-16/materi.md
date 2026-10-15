# Day 16 — Materi
## Sysmon Telemetry Troubleshooting & Evidence Validation

**Tujuan:** Memastikan apakah pencatatan Sysmon tersedia, jenis event apa yang ada, membedakan bukti saat ini dan historis, serta mencatat observasi, hipotesis, pengujian, hasil, kesenjangan bukti, dan penilaian.

## Istilah dan model berpikir

- Sysmon adalah layanan sistem Windows dan penyedia event. Status layanan `Running` **tidak** membuktikan bahwa setiap Event ID Sysmon dicatat.
- Event ID 1: `Process Create`; mencatat `Image`, `ProcessId`, `ProcessGuid`, informasi parent, command line, user context, dan hash (bergantung pada konfigurasi).
- Event ID 3: `Network Connection`; mencatat aktivitas jaringan TCP/UDP yang terkait dengan process jika fitur ini aktif dan sesuai dengan aturan.
- Event ID 22: `DNS Query`; dapat melengkapi bukti koneksi jaringan.
- Event ID 2: `File Creation Time Change`; 4: `Sysmon Service State Change`; 5: `Process Terminate`; 11: `File Create`; 13: `Registry Value Set`; 16: `Sysmon Configuration Change`.
- Windows dapat menggunakan kembali `ProcessId` (PID) setelah suatu process berhenti.
- `ProcessGuid` mengidentifikasi satu instance process dalam Sysmon dan membantu mengorelasikan Event ID 1 dengan jenis event lain; nilainya **bukan** sekadar PID yang ditambah informasi jaringan.
- Event ID 1 = catatan historis pembuatan process; Event ID 3 = catatan historis koneksi jaringan; `Get-NetTCPConnection` = tampilan koneksi TCP saat ini; `Get-CimInstance Win32_Process` = informasi process saat ini.
- Jika kedua endpoint bernilai `::1`, koneksinya adalah IPv6 loopback di host yang sama, bukan tujuan di internet.
- `RemotePort 443` saja tidak membuktikan bahwa koneksinya menggunakan HTTPS atau bahwa aktivitasnya sah.
- `IntegrityLevel High` menunjukkan tingkat integritas yang lebih tinggi, tetapi tidak otomatis memberikan semua hak istimewa Windows.
- `services.exe` adalah process Service Control Manager; `svchost.exe` menampung layanan Windows terkait; `-k CameraMonitor` menunjukkan nama grup service host, bukan bukti bahwa kamera sedang merekam.

## Tiga kondisi telemetry

1. Log channel tidak tersedia: tool tidak menemukan channel yang diminta pada sistem yang diperiksa. Periksa host, layanan, dan channel yang terdaftar; jangan langsung memasang ulang.
2. Channel tersedia, tetapi Event ID yang diminta tidak ditemukan: event mungkin memang belum terjadi, pencatatan mungkin dinonaktifkan atau tersaring, atau catatan lama mungkin sudah tertimpa.
3. Event ID ditemukan: ada bukti bahwa catatan dengan jenis tersebut dikumpulkan, tetapi perilaku tertentu dan cakupan pencatatan masih perlu diverifikasi.

## Hasil Day 16 yang sudah dikonfirmasi

**Praktik aktual:** layanan Sysmon berstatus `Running/Auto` dan channel operasional berhasil digunakan; ditemukan 108 catatan Event ID 3 dan 352 catatan Event ID 22. Contoh Event ID 1 terbaru menunjukkan `EpicGamesUpdater.exe` dijalankan dengan argumen `queryinstallation`, dengan parent `EpicGamesLauncher.exe`. Jalur process tampak sesuai dengan instalasi Epic Games, tetapi signature, perilaku, dan reputasi lengkapnya belum diverifikasi. Jangan menyatakan bahwa process ini pasti aman.

**Perbedaan penting:** contoh sebelumnya yang menyebut 43 catatan adalah *jumlah seluruh catatan pada pemeriksaan saat itu*, bukan “43 Event ID 1”. Cuplikan Day 16 memiliki 71 catatan Event ID 1 dan 108 catatan Event ID 3. Mini SOC case merupakan skenario hipotesis yang menganggap Event ID 3 tidak ditemukan; jangan mencampurkannya dengan data host aktual.

## Templat laporan investigasi

~~~text
Observasi: Event atau field apa yang benar-benar diamati?
Hipotesis 1 / 2: Kemungkinan penjelasan lain.
Pengujian: Perintah atau pemeriksaan yang direncanakan/dilakukan.
Hasil yang Diharapkan: Hal apa yang akan diperjelas oleh pengujian tersebut?
Hasil Aktual: Diisi hanya jika pengujian benar-benar dilakukan.
Bukti yang Tersedia: Sumber yang sudah diperiksa (jangan mengarang).
Kesenjangan Bukti: Apa yang belum tersedia atau belum diperiksa?
Penilaian: Kesimpulan sementara yang didasarkan pada bukti.
Tindakan Berikutnya: Langkah lanjutan yang paling tepat.
~~~

Hasil kosong dari `Get-CimInstance` untuk PID historis tidak membuktikan bahwa deteksi tersebut false positive. Jika `ProcessId` sama tetapi `ProcessGuid` berbeda, periksa apakah keduanya merupakan instance process yang berbeda atau terjadi penggunaan ulang PID. Koneksi yang sedang aktif juga belum tentu memiliki Event ID 3 yang cocok karena pencatatan bisa dinonaktifkan, tersaring, atau tidak lengkap.

## Technical reference (outside lesson)
- Microsoft Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Sysmon events: https://learn.microsoft.com/en-us/windows/security/operating-system-security/sysmon/sysmon-events

## Privasi dokumentasi
Cuplikan log dan pengenal akun/host yang spesifik pada perangkat telah disamarkan. Jangan menyimpan username, hostname, atau jalur profil pribadi dalam repo.
