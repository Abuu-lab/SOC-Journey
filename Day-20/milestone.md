# 📌 Day 20 — Milestone Hari Ini

**PASS — 80/100 (formatif).**

Praktik selesai: membaca tiga Sysmon Process Terminated ID5, membedakan historical dan current evidence, mencari event Spotify dari periode lama, serta memverifikasi keberadaan, metadata, signature dan SHA256 file Spotify terbaru.

Hasil pencarian event Spotify lama: NoMatchingEventsFound pada cakupan yang diperiksa; retensi log belum diperiksa. Hands-on4 tidak disertai output. Hasil file terbaru: signature Valid, tetapi hash berbeda dari hash yang dicatat Sysmon di masa lalu. Tidak ada malware verdict otomatis.

Challenge menunjukkan kemampuan membaca event IDs dan timestamp; perlu penguatan ProcessGuid correlation, arti UDP vs port443, perubahan file, dan pemisahan Assessment dari Next Action.

Laporan kasus contoh disusun mentor, bukan laporan yang ditulis mandiri. Dengan nilai PASS, peserta bisa langsung melanjutkan Day21 dan mendapat latihan penguatan terarah.

Semua detail identitas endpoint disamarkan.


## Tambahan setelah PASS — Verifikasi cakupan historical logs

- Menjawab latihan penguatan R1 dengan benar: protocol UDP tetap UDP meski port tujuan 443.
- Pada R2, menolak tuduhan malware hanya karena hash berbeda; perlu ditegaskan bahwa dua nilai SHA-256 berbeda berarti file tersebut tidak mempunyai isi byte identik.
- Berhasil menjalankan `Get-WinEvent -Oldest -MaxEvents 1`; record tertua Sysmon tercatat 20 Oktober 2026 pukul 12:28:45 lokal, Event ID2, RecordId 203160.
- **Penemuan:** log Sysmon 15 Oktober sudah tidak tersedia pada channel saat pemeriksaan. Penyebab hilangnya log belum dipastikan.
- Status tetap PASS 80/100, lanjut ke Day21. Penguatan konsep hash dibawa ke Active Recall.
