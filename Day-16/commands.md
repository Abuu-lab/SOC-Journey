# Day 16 — Commands

**Topik:** Kesehatan Sysmon, sebaran event, analisis `Process Create`, dan kesenjangan telemetry.

## Perintah yang benar-benar dijalankan

### 1. Menghitung catatan Sysmon berdasarkan Event ID
~~~powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' |
Group-Object Id |
Select-Object Name, Count |
Sort-Object Name
~~~

Hasil yang diamati (username/hostname tidak dicantumkan):

| Event ID | Count |
|---|---:|
| 1 | 71 |
| 2 | 29 |
| 3 | 108 |
| 4 | 1 |
| 5 | 3 |
| 11 | 4 |
| 13 | 5 |
| 16 | 1 |
| 22 | 352 |

**Makna:** terdapat 108 catatan historis Event ID 3. Ini membuktikan bahwa Event ID 3 pernah dicatat, tetapi tidak menjamin semua koneksi jaringan tertangkap.

### 2. Memeriksa status layanan Sysmon
~~~powershell
Get-Service -Name Sysmon,Sysmon64 -ErrorAction SilentlyContinue |
Select-Object Name, Status, StartType
~~~

Hasil: Sysmon berstatus `Running`, jenis startup `Automatic`.

### 3. Memeriksa detail Win32_Service
~~~powershell
Get-CimInstance Win32_Service |
Where-Object {$_.Name -in @('Sysmon', 'Sysmon64')} |
Select-Object Name, State, StartMode, ProcessId, PathName
~~~

Hasil: Sysmon; `Running`; `Auto`; PID 26508 (cuplikan historis); `PathName` C:\Windows\Sysmon.exe.

### 4. Mengambil catatan historis Process Create terbaru
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 1 |
Format-List TimeCreated, Id, Message
~~~

Hasil: `EpicGamesUpdater.exe` (PID 21144), parent `EpicGamesLauncher.exe` (PPID 15924), `IntegrityLevel Medium`, dan `CommandLine EpicGamesUpdater.exe queryinstallation`. Akun pengguna dan GUID unik tidak dicantumkan.

## Langkah lanjutan — sudah diajarkan, tetapi BELUM dijalankan pada Day 16

~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 1 |
Format-List TimeCreated, Message
~~~

Jika tersedia, baca event `Network Connection`. Cocokkan `ProcessGuid`-nya dengan Event ID 1; jangan hanya mengandalkan PID.

## Keamanan dan privasi
Jangan pernah menyimpan hostname, username lokal, jalur direktori profil pengguna, atau log mentah yang memuat informasi tersebut. Jika diperlukan, ganti dengan [REDACTED_HOST] dan [REDACTED_USER].
