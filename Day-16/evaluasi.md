# Day 16 — Evaluasi dan Koreksi Lengkap

**Status: PASS | Score: 77/100 (mentor's formative estimate).**
Passing threshold 76. Minor mistakes carried into Day 17 Active Recall rather than requiring repetition of the whole Day.
**Evidence privacy:** No raw hostname, local username, profile path, or host-specific ProcessGuid in this document.

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

## 2. Praktik 1: Sysmon Event ID Distribution

Command pengguna:
~~~powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' |
Group-Object Id | Select-Object Name, Count | Sort-Object Name
~~~

Output asli (count per ID):
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
| P3 | Event ID 3 mungkin belum tercatat | Benar untuk skenario hipotesis; selain itu logging bisa disabled, filtered atau log rolled over. Pada data nyata ID 3 ADA (108 events). |
| P4 | 'agar tidak terkesan menyalahkan' | Tujuan memeriksa telemetry health adalah agar tidak menganggap 'no evidence' sebagai 'no activity', dan tidak menghasilkan false negative. |

## 3. Praktik 2: Sysmon Service Health

Output: Sysmon Running, Automatic; Win32_Service menunjukkan State Running, StartMode Auto, ProcessId 26508 dan PathName C:\Windows\Sysmon.exe.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P5 | Sysmon | Benar. |
| P6 | Running dan Auto | Benar. |
| P7 | State kondisi sekarang; StartMode otomatis/manual | Benar, State status service saat query dan StartMode startup configuration (auto/manual/disabled). |
| P8 | 'ya sudah berjalan' | Salah. Service Running sendiri tidak membuktikan logging Event ID 3; di sini **108 recorded Event 3** yang membuktikan event itu memang tercatat. |

## 4. Praktik 3: Historical Process Create

Event ID 1 sample: TimeCreated 2026-10-10 13:31:44 local; ProcessId 21144; Image ends in EpicGamesUpdater.exe; ParentImage ends in EpicGamesLauncher.exe (PPID 15924); IntegrityLevel Medium; command line EpicGamesUpdater.exe queryinstallation. User/ProcessGuid/host redacted.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P9 | 'gk ada nama' | Nama proses **EpicGamesUpdater.exe**, diambil dari basename field Image. |
| P10 | EpicGamesLauncher.exe | Benar, ParentImage. |
| P11 | menggunakan ProcessGuid | Benar, cocokkan ProcessGuid antar events untuk instance sama. |
| P12 | tetap menggunakan historical event | Benar; current process gone tidak berarti investigasi berhenti. |

## 5. Praktik 4: Evidence Validation (SIMULATION)

Scenario: Event 1 shows powershell.exe PID 8200, ProcessGuid LAB-016-A, parent explorer.exe, running script maintenance.ps1, later CIM PID 8200 not found.

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| P13 | tidak tepat, cari historical evidence | Benar; PID sudah hilang bukan false positive otomatis. |
| P14 | process name, PID, ProcessGuid, CommandLine, ParentImage | Benar: fields already provided by event. |
| P15 | historical Event ID 1/3 | Partially wrong: Event ID 1 was GIVEN; missing script contents, file metadata, hash/signature if appropriate, behavior, network telemetry. Event 3 not shown. |
| P16 | correlate ProcessGuid, check hash/metadata/signature, ID1/ID3 | Reasonable next steps; prioritize script and relevant historical events rather than assuming all must exist. |
| P17 | script perlu diperiksa meskipun namanya maintenance | Correct conclusion; because powershell.exe is an interpreter and the script provides its instructions. |

## 6. Mini SOC Case (SIMULATION; ID 3 assumed absent)

**User answer (unmodified meaning):**
- Observation: Sysmon running, '43 Event ID 1' (incorrect).
- Hypotheses: Event ID 3 has not been recorded; Get-NetTCPConnection shows Established connections.
- Expected Result: Sysmon Operational is True.
- Evidence: Sysmon ID 1.
- Gap: Sysmon ID 3.
- Assessment: 'cari menggunakan historical event dan get tcp connection' (this is a next step, not an assessment).
- Next Action: search Sysmon ID 3.

**Mentor's ideal answer:**
- Observation: service running, channel enabled, 43 *total records* in hypothetical snapshot, historical ID1 available, ID3 not found, Established live connections.
- Hypothesis A: Event ID3 not enabled or excluded by filters.
- Hypothesis B: ID3 records unavailable in retained time window.
- Test: inspect event distribution and Sysmon configuration, time window; compare to live TCP.
- Expected Result: determine whether ID3 collection is configured and any matching events exist.
- Evidence Available: Sysmon service/channel status, ID1, Get-NetTCPConnection live data.
- Evidence Gap: historical network telemetry relevant to alert.
- Assessment: insufficient historical network visibility; needs investigation.
- Next Action: inspect logging configuration and alternative evidence; never invent actual test result.

**Important:** Actual practice snapshot shows 108 Event ID3; do not merge the simulation with the actual host results.

## 7. Challenge C1–C5

| Q | Jawaban pengguna | Koreksi |
|---|---|---|
| C1 | Established connections should be logged in ID3, unless not recorded | Not necessarily: disabled/filtered/log rollover, and live vs historical views differ. |
| C2 | belum dapat bayangan | Yes, Get-NetTCPConnection + Win32_Process allow live network investigation even without ID3. |
| C3 | live information may not yet be recorded | Main limitation is transient: process/connection might be gone when inspected; historical evidence can preserve prior activity. |
| C4 | belum dapat bayangan | Same PID but different ProcessGuid may mean different process instances or PID reuse; investigate timestamps and images. |
| C5 | B | Correct: missing ID3 doesn't justify 'no network activity'. |

## 8. Mentor Assessment

**Strong:** command execution, event aggregation, service status, extracting parent from Process Create, resisting 'PID gone = false positive'.

**Needs reinforcement on Day 17:** PID vs ProcessGuid; Event 3 completeness; identifying basename from Image; evidence available vs gaps; writing a real assessment and alternative hypotheses; live vs historical limitations.

**Result:** PASS 77/100 based on substantive practical progress. This is a formative teaching score, not a certified/exam metric. Weak answers are documented rather than ignored.
