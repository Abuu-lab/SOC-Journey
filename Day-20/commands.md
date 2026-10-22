# Day 20 — Commands (praktik nyata)

**Status:** PASS 80/100 (penilaian formatif). Semua perintah bersifat read-only. Contoh output pribadi sudah disanitasi.

## 1. Sysmon Process Terminated: tiga event terbaru
```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 5
} -MaxEvents 3 | Format-List TimeCreated, Message
```
Hasil aktual pada 22 Oktober 2026 (waktu lokal):
| Waktu | Process | PID |
|---|---|---:|
| 04:37:53 | Discord.exe | 29948 |
| 04:34:37 | Discord.exe | 9520 |
| 04:31:17 | opera_autoupdate.exe | 18688 |

Setiap record mempunyai ProcessGuid berbeda, tetapi identitas GUID mentah tidak dipublikasikan. Event ID 5 membuktikan terminasi tercatat, **bukan penyebabnya**.

## 2. Historical Spotify Event ID 5 pada tanggal sebelumnya
```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 5
    StartTime = (Get-Date '2026-10-15 15:00:00')
    EndTime = (Get-Date '2026-10-15 23:59:59')
} -MaxEvents 100 |
Where-Object { $_.Message -like '*Spotify.exe*' } |
Select-Object -First 3 |
Format-List TimeCreated, Message
```
Hasil aktual: **NoMatchingEventsFound**. Ini bukan bukti process tidak pernah berhenti. Get-WinEvent tidak menemukan Event ID 5 pada filter waktunya; batas 100 dan keberadaan log historis harus diperhatikan.

## 3. Multi-Event Query
```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1,3,5,22
    StartTime = (Get-Date '2026-10-15 17:20:00')
    EndTime = (Get-Date '2026-10-15 17:30:00')
} -MaxEvents 15 | Select-Object TimeCreated, Id, RecordId
```
Hasil aktual: **NoMatchingEventsFound** pada filter tanggal/waktu tersebut.

## 4. Targeted Spotify Event ID 3
User mengirim ulang perintah dengan interval 2026-10-15 17:24:00–17:25:00 tetapi **tidak menyertakan output**. Jangan menyatakan berhasil/gagal tanpa output.

## 5. Current Spotify File Investigation
```powershell
$file = Join-Path $env:APPDATA 'Spotify\Spotify.exe'
Test-Path -LiteralPath $file
Get-Item -LiteralPath $file | Select-Object Name, Length, CreationTime, LastWriteTime
Get-AuthenticodeSignature -LiteralPath $file | Select-Object Status, StatusMessage
Get-FileHash -LiteralPath $file -Algorithm SHA256
```
Hasil aktual:
- Test-Path: True.
- Name: Spotify.exe; length 4001152 bytes.
- CreationTime: 2026-06-24 19:14:40 lokal; LastWriteTime: 2026-10-22 04:27:45 lokal.
- Authenticode: Valid, Signature verified.
- Current SHA-256 calculated (fingerprint starts `638F9266...`). Historical Spotify Event ID 1 on 2026-10-15 recorded SHA-256 beginning `06DB5B88...`: **different hashes**. Difference supports that file bytes are not established as identical between historical and current examinations; may be due to software update. No malware verdict follows from this difference alone.

## 6. Historical Spotify DNS Event ID 22
```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 22
    StartTime = (Get-Date '2026-10-15 17:20:00')
    EndTime = (Get-Date '2026-10-15 17:30:00')
} -MaxEvents 100 |
Where-Object { $_.Message -like '*Spotify.exe*' } |
Select-Object -First 3 | Format-List TimeCreated, Message
```
Hasil aktual: **NoMatchingEventsFound**. Tidak berarti DNS Query tidak pernah terjadi.

## Command baru untuk Mini Project — diberi mentor, BELUM dijalankan
Mulai dengan menentukan umur log paling lama yang masih ada:
```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -Oldest -MaxEvents 1 |
Select-Object TimeCreated, Id, RecordId
```
Jika record paling tua setelah 15 Oktober 2026, log tanggal itu tidak lagi tersedia melalui channel tersebut. Gunakan evidence yang masih tersimpan, misalnya 22 Oktober 2026. Jangan mengarang data hilang.

**Privacy:** username/hostname/IP privat, path profil akun, dan GUID khusus host tidak dimasukkan ke dokumentasi.
