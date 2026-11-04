# 📌 Day 21 — Milestone Sementara

**Status final: PASS 78/100 (formatif); catatan di bawah merekam perkembangan hingga penutupan.** Tahap 2 roadmap SOC: Network Fundamentals I.

Kemampuan yang sudah dipraktikkan:
- Menggunakan Get-NetIPConfiguration untuk melihat interface Ethernet, IP/gateway/DNS.
- Membaca Get-NetIPAddress: Ethernet /24 Preferred, loopback, dan beberapa alamat IPv4 link-local.
- Mengambil Get-NetRoute default IPv4 route dan next hop.
- Melakukan Resolve-DnsName example.com: mendapatkan dua record A.
- Menggunakan Test-NetConnection example.com:443: TcpTestSucceeded True.
- Mengerjakan active recall dan skenario SOC #021 secara parsial.
- Melaporkan contoh current Established TCP connections, tetapi output lengkap/owner belum dikirim.

Kelemahan konsep yang ditemukan: PrefixLength bukan penjumlahan antarinterface, loopback, DNS tidak menjamin TCP, default gateway bukan NAT, correlation butuh lebih dari ProcessGuid saja, Assessment harus berbasis bukti, serta dua hipotesis alternatif masih perlu latihan.

Mini-project Local Network Investigation Report belum selesai dan nilai final belum ditetapkan. Materi tambahan serta contoh model laporan disediakan oleh mentor, bukan hasil kerja mandiri.

Pengamanan data: username/hostname, alamat privat dan identitas host tidak dipublikasikan.


## Tambahan milestone — koneksi TCP ke pemilik process

- Berhasil mengambil contoh current Established TCP: local port65514 menuju remote port443 dengan OwningProcess10512.
- Berhasil menghubungkan PID10512 melalui Win32_Process ke **msedgewebview2.exe**, parent PID4100 dan command-line network service WebView2 terkait Widgets.
- Berlatih membedakan fakta observasi vs dugaan bahwa proses berkaitan dengan fitur Windows Widgets.
- Belum memverifikasi parent PID4100, digital signature/hash executable maupun status remote endpoint. Mini-project dan penilaian final Day21 masih berjalan.

## Progress terbaru — Parent process berikutnya

- Menggunakan query current Win32_Process untuk PID4100 dan menemukan WebView2 parent process lain.
- Bukti aktual: msedgewebview2.exe PID4100 memiliki ParentProcessId6200 dan commandline embedded browser terkait Widgets.exe.
- Berlatih menelusuri chain koneksi TCP hingga dua level proses dan membedakan evidence dari dugaan proses yang lebih tua.
- Parent PID6200 dan file signature/hash belum dikonfirmasi. Nilai akhir belum ditetapkan.

## 📌 Final Day 21 milestone

**PASS 78/100 — formative mentor evaluation.** Advance to Day22.

- Successfully followed actual Established TCP owner PID10512 through the Win32_Process parent chain: WebView2 PID10512 → WebView2 PID4100 → Widgets.exe PID6200.
- Recognized Widgets.exe command-line context in Windows Client WebExperience package.
- Executed `Get-AuthenticodeSignature` for WebView2; output Status Valid, message Signature verified. Certificate subject was truncated; `Get-FileHash` was invoked but output not supplied, therefore SHA256 **not recorded as verified**.
- Combined subnet /24, DNS resolution, TCP connectivity, current network connections and parent-process findings into a mentor-assisted case assessment.
- No categorical malware or clean verdict; remote service and certain process identity/file checks remain unverified.
- Independence: **Terpandu** for final report, with growing hands-on ability. Carry prefix/gateway/DNS and assessment writing into Day22 active recall.
