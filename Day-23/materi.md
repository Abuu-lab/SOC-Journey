# Day 23 — Network Fundamentals III: DNS → TCP → TLS → HTTP/HTTPS

**Status: MULAI / IN PROGRESS.** Belum ada jawaban Active Recall, output Day23, skor, atau penilaian kelulusan. Seluruh konten di bawah adalah materi dan tugas yang diberikan mentor, bukan bukti bahwa peserta telah mengerjakannya.

## Fokus pembelajaran
Menentukan tahap kegagalan konektivitas web sebelum membuat Assessment SOC. Materi tidak menuntut subnet biner kompleks. Penguatan Day22: IP/prefix dan pemilihan route paling spesifik dilakukan singkat dalam kesempatan berikutnya.

## 1. Model diagnosis empat tahap

1. **DNS:** nama domain dicari menjadi records, misalnya A (IPv4) dan AAAA (IPv6). Resolusi sukses tidak menjamin jaringan ke tujuan dapat tersambung.
2. **TCP:** untuk contoh HTTPS melalui TCP443, handshake berhasil membuktikan sambungan TCP ke IP:port uji, bukan keberhasilan TLS atau permintaan HTTP.
3. **TLS:** klien dan server melakukan handshake kriptografis; identitas/sertifikat server diperiksa oleh klien. TLS yang gagal dapat membuat web tidak terbuka walaupun DNS dan TCP sukses.
4. **HTTP:** aplikasi menerima request dan memberikan response status. HTTP 200 bukan bukti situs aman; 301/302 redirect; 401 memerlukan autentikasi; 403 akses dilarang; 404 resource tidak ditemukan; 500 error server.

**Catatan:** HTTP/3 dapat memakai QUIC pada UDP443, sehingga diagram DNS→TCP→TLS→HTTP adalah model umum untuk HTTPS lewat TCP, bukan aturan universal semua koneksi HTTPS.

## 2. Batas bukti

| Evidence | Yang bisa dinyatakan | Yang belum bisa dinyatakan |
|---|---|---|
| Jawaban DNS ada | Nama domain memiliki jawaban IP | Koneksi TCP sukses; server aman |
| Test-NetConnection TcpTestSucceeded=True | TCP ke IP:port yang dipilih berhasil pada saat tes | TLS berhasil; HTTP sukses; malware tidak ada |
| curl.exe -I sukses dan memberi status HTTP | Server/aplikasi memberikan respons terhadap HEAD | Semua fitur aplikasi berjalan; situs aman |
| Curl TLS validation error | Permintaan gagal pada/berkaitan validasi TLS | Pasti malicious interception; pasti sertifikat palsu |
| HTTP 403 | Request sampai ke layanan HTTP dan ditolak sesuai kondisi/aturan | Otomatis serangan |

## 3. Penting untuk SOC

Jangan menyamakan `port443` dengan keamanan. Tinjau proses pemilik koneksi dan process tree, destination, konteks aplikasi, waktu, log, serta bukti lain bila ada alert. Perbedaan `Observation` (apa yang terlihat) dan `Assessment` (kesimpulan terbatas bukti) wajib dipertahankan.

**Privacy:** Sebelum publikasi, hapus hostname perangkat, user profile, IP privat, token/cookie/header Authorization dan identitas perusahaan. Jangan mengunggah output curl verbose mentah jika memuat rahasia.

## 4. Active Recall (belum dijawab)

- AR1: DNS berhasil, apakah TCP selalu berhasil? Mengapa?
- AR2: `TcpTestSucceeded=True` apakah cukup membuktikan website HTTPS berfungsi penuh?
- AR3: Apa beda `DestinationPrefix 0.0.0.0/0` dan `NextHop 0.0.0.0` dalam tabel routing?
- AR4: Jika HTTP 403, apakah ini otomatis bukti serangan?
- AR5 (opsional): Untuk `10.20.0.0/16`, apakah tujuan `10.20.5.30` termasuk network itu? Apakah `8.8.8.8` termasuk?

## 5. Hands-on bertahap

**H1 — Memastikan curl tersedia:** `curl.exe --version`. Pelajari bahwa `curl.exe` adalah program curl sungguhan, bukan kemungkinan alias PowerShell `curl`.

**H2 — HTTP response melalui TLS:** `curl.exe --head --max-time 15 https://example.com`. Output mungkin 200, redirect, kesalahan sertifikat, proxy, timeout, dll. **Jangan mengarang status.** `--head` mengirim HEAD dan hanya meminta response headers. `--max-time 15` memberi batas durasi.

**H3 — Jika perlu detail:** `curl.exe --verbose --head --max-time 15 https://example.com`. Interpretasi output bergantung backend TLS dan proxy; tidak selalu menampilkan string yang sama. Hindari opsi `-k/--insecure` karena akan menonaktifkan pemeriksaan sertifikat.

**H4 — Penguatan DNS dan TCP (gunakan evidence Day22 historis atau lakukan tes baru bila diperlukan):** `Resolve-DnsName example.com -Type A -DnsOnly` dan `Test-NetConnection example.com -Port 443 -InformationLevel Detailed`. Bandingkan IP dan status; A records dapat berubah.

**H5 — Proses pemilik koneksi (opsional dan tergantung koneksi aktif):** `Get-NetTCPConnection -State Established | Select-Object -First 10 LocalAddress,LocalPort,RemoteAddress,RemotePort,State,OwningProcess`. PID yang dipilih diperiksa dengan `Get-CimInstance Win32_Process -Filter 'ProcessId = <PID>' | Select ProcessId,ParentProcessId,Name,ExecutablePath,CommandLine`, mengganti placeholder PID nyata; koneksi bisa hilang sebelum diperiksa.

## 6. SOC Challenge #023 — simulasi, bukan kejadian pada PC peserta

Kasus: pengguna tidak bisa membuka `portal.lab.example` (nama simulasi). DNS success dengan IPv4 203.0.113.44 (alamat dokumentasi). TCP ke 203.0.113.44:443 True. `curl.exe --head https://portal.lab.example` gagal dengan `curl: (60) SSL certificate problem: certificate verify failed`. Tidak ada output HTTP status, certificate chain maupun pengujian browser tambahan.

Jawab C1: Apa yang sudah terbukti di DNS? C2: Apa arti TCP True? C3: tahap mana yang tampak bermasalah? C4: apakah ini bukti malware/serangan? C5: dua hipotesis wajar? C6: dua Evidence Gaps? C7: dua Next Actions yang aman dan spesifik? Tidak disarankan mematikan validasi TLS untuk menganggap masalah selesai.

## 7. Mini-project — Connectivity Layer Triage Report

Susun berdasarkan output **yang benar-benar ada**: [Observation DNS], [Observation TCP], [Observation HTTPS curl], [Assessment], [Evidence Gap 1–2], [Next Action 1–2]. Bila DNS/TCP adalah evidence dari Day22, nyatakan bahwa datanya historis; jangan mengklaim pengujian baru. Status belum dikerjakan.

## Sumber teknis
- Microsoft Test-NetConnection: https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection
- Microsoft Resolve-DnsName: https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname
- Microsoft Get-NetTCPConnection: https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-nettcpconnection
- Curl command reference: https://curl.se/docs/manpage.html