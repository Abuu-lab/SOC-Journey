# Day 17 - Milestone Hari Ini

**Status: PASS, 80/100 (penilaian formatif mentor)**

## Yang dikerjakan dan dipelajari

- Mengambil historical Sysmon DNS Query Event ID 22 dan menampilkan hasilnya.
- Mengenali field ProcessName, ProcessId, ProcessGuid, QueryName, QueryStatus, dan QueryResults.
- Mengamati 47 Event ID 22 yang cocok dengan satu ProcessGuid, domain www.163.com, rentang 16:03:11 sampai 16:49:53 pada 13 Oktober 2026.
- Menjelaskan bahwa repeated DNS Query tidak otomatis malicious, dan belum tentu berarti network connection.
- Mengidentifikasi Evidence Available dan Evidence Gap dengan koreksi.
- Berlatih membuat Observation, dua Hypotheses, Assessment Needs Investigation, dan Next Action.
- Memilih pencarian Sysmon Event ID 1 dan 3 sebagai langkah berikutnya tanpa harus menghafal command.
- Menyelesaikan laporan investigasi secara terbimbing; belum mandiri sepenuhnya.

## Konsep yang masih perlu diperkuat

- Event ID 1 = Process Create, ID 2 = File Creation Time Changed, ID 3 = Network Connection, ID 22 = DNS Query.
- QueryName dan QueryResults berbeda dari DestinationIp; DNS bukan bukti exfiltration.
- Bedakan Assessment dari Next Action; jelaskan assessment berdasarkan bukti.
- Baca frekuensi dan durasi secara teliti.
- ProcessGuid dipakai untuk correlation, bukan jaminan legitimate.
- PowerShell variable assignment, output dan dasar Get-WinEvent sebelum mempelajari foreach, hashtable, atau XML parsing.

## Metode pembelajaran berikutnya

Satu konsep, satu command sederhana, pahami hasil, lalu latihan analisis. Tidak menuntut hafalan script. Identitas endpoint tidak disertakan.
