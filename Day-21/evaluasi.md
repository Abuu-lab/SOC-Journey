# Day 21 — Evaluasi Sementara

**Status final: PASS 78/100 (formatif); lihat penilaian akhir di bagian bawah.** Active Recall, Hands-on 1–6 dan Challenge telah dijawab sebagian besar; mini-report mandiri belum diserahkan. Koreksi mengutamakan salah konsep penting, bukan kekurangan redaksi.

## Active Recall
- Q1: 'udp tidak bisa menjadi tcp' — benar.
- Q2: 'ProcessGuid berbeda' — benar.
- Q3: 'mungkin mengalami update, bukan otomatis malware' — sebagian benar; hash berbeda menunjukkan byte tidak identik.
- Q4: 'assessment mengumpulkan semua bukti' — salah bagian inti; Assessment adalah penilaian berdasar bukti. Next Action adalah langkah berikutnya.
- Q5: 'ID5 terminate tapi tidak menjelaskan sebab' — benar.
- Q6: 'event log mungkin' — belum lengkap; periksa record tertua, filter query dan retensi log.

## Hands-on 1–3
- Interface Ethernet; IPv4 lokal privat, satu default gateway dan DNS lokal serta resolver IPv6 link-local.
- InterfaceIndex belum dipahami: Windows menggunakan angka itu untuk merujuk interface.
- Ethernet PrefixLength **24** (bukan total penjumlahan 80). Ada loopback 127.0.0.1, fungsi komunikasi lokal host. 169.254 IPv4 link-local pada beberapa interface lainnya.
- Default route 0.0.0.0/0 di Ethernet, satu route teramati; perbedaan PC IP vs router IP dijawab benar.

## Hands-on 4–6
- Resolve-DnsName example.com A berhasil: 104.20.23.154 dan 172.66.147.243. Learner belum memahami 'IP yang dikembalikan' sebagai jawaban DNS; DNS sukses tidak otomatis membuktikan TCP.
- Test-NetConnection example.com:443 = TcpTestSucceeded True. RemoteAddress salah satu hasil DNS; SourceAddress lokal, NextHop router. TCP failure tidak selalu DNS failure.
- Learner melaporkan 10 TCP connections pada Select-Object First 10, namun tidak menyertakan output sehingga owning PID belum dapat dinilai.
- RemoteAddress adalah IP endpoint di sisi lain koneksi (tidak selalu publik). OwningProcess berisi PID; dapat dicari lewat Win32_Process. Penjelasan current vs historical benar.

## Simulated Challenge #021
- C1 private IP; C2 same /24: benar.
- C3 default gateway disebut menyamarkan IP lokal; itu mencampurkan gateway dengan NAT. Gateway mengarahkan ke route lain; NAT fungsi terpisah.
- C4 lupa DNS server; DNS server memetakan domain ke alamat; destination IP adalah tujuan komunikasi.
- C5–C6 ProcessGuid sama pada Event ID 1, 22, 3 serta QueryResults cocok DestinationIp, waktu terurut: indikasi correlation satu process instance, tetapi isi komunikasi belum terbukti.
- C7 source/destination port: benar.
- C8 port443/TCP tidak membuktikan aman/HTTPS: benar.
- C9 tidak memberikan dua hipotesis (benign dan potentially malicious).
- C10 menjelaskan perlunya signature, SHA256 dan metadata; masih perlu Assessment eksplisit beserta Evidence Gap dan dua Next Actions.

## Fokus remediasi non-blocking jika nilai PASS
Subnet PrefixLength, AddressState/loopback, DNS success vs TCP success, default gateway vs NAT, Assessment vs Next Action, drafting competing hypotheses. Final grade ditentukan setelah mini-project.


## Mini-project progress baru — current TCP Established → executable (evidence aktual)

- Pelajar mengirim contoh koneksi Established dari private local endpoint port 65514 ke remote 104.89.118.72:443, `OwningProcess = 10512`.
- Query current `Win32_Process` dengan PID10512 berhasil mengidentifikasi `msedgewebview2.exe`, ParentProcessId4100, lokasi Microsoft Edge WebView2, dan command-line role `network.mojom.NetworkService` dengan `--webview-exe-name=Widgets.exe`.
- **Valid correlation dalam snapshot:** OwningProcess cocok dengan PID dari process query. Tidak memverifikasi historical instance atau remote domain/reputasi.
- **Preliminary assessment:** Needs Investigation. Path dan commandline konsisten dengan fungsi WebView2/Windows Widgets tetapi belum ada pemeriksaan publisher/signature/hash dan parent name. Port443 tidak membuktikan keamanan.
- Parent PID4100 belum diperiksa; command untuk investigasi berikutnya telah diberikan dan output belum diterima.
- Mini SOC Report dan nilai final Day21 tetap pending.

## Follow-up Parent Process (bukti pengguna)

- `Get-CimInstance Win32_Process -Filter "ProcessId = 4100"` berhasil mengidentifikasi parent dari proses pemilik koneksi.
- PID4100 adalah `msedgewebview2.exe`; ParentProcessId=6200. Binary WebView2 version 156.0.4314.58. Commandline mengacu pada embedded browser dan `Widgets.exe`.
- Korelasi snapshot: Established TCP owner PID10512 → current WebView2 PID10512 → current WebView2 PID4100. PID6200 belum diperiksa.
- Penilaian sementara: fungsi normal Windows Widgets/WebView2 merupakan hipotesis masuk akal; belum terbukti aman atau berbahaya. Perlu cek parent PID6200, file signature/hash.
- Jangan memublikasikan user profile path, token/handles, hostname atau identifier khusus perangkat.
- Mini SOC Report dan nilai final Day21 masih pending.

## Final Day 21 Assessment — supersedes initial IN PROGRESS

**PASS — 78/100 (formatif mentor; batas PASS 76).** Score based on demonstrated learner work, not standardized certification. User has not independently authored the full case report; final report below is **MENTOR-ASSISTED**.

### Final evidence submitted
- Current Established TCP endpoint has OwningProcess PID10512.
- Current PID10512 `msedgewebview2.exe`, PPID4100; network service utility, WebView2 command-line flags associated with Widgets.
- Current PID4100 `msedgewebview2.exe`, PPID6200; embedded browser WebView2 for Widgets.
- Current PID6200 `Widgets.exe`, PPID1416, installed under Windows Client WebExperience WindowsApps package. Command line includes `-ServerName:Microsoft.Windows.DashboardServer`.
- `Get-AuthenticodeSignature` against specified WebView2 binary returned **Valid / Signature verified**. Full certificate signer subject truncated in submitted output.
- `Get-FileHash` was invoked, but no SHA256 output was pasted. Status: **NOT VERIFIED FROM OUTPUT**.
- Exact role/ownership of remote destination is unverified; port443 is not proof of HTTPS content or safety. PID1416 not identified. Event ID1/3/22 historical ProcessGuid correlation for this case not performed.

### Rubrik penilaian
| Komponen | Maks | Nilai |
|---|---:|---:|
| Pemahaman konsep | 20 | 15 |
| Ketepatan mengumpulkan dan membaca evidence | 25 | 24 |
| Analisis dan penalaran | 25 | 18 |
| Keputusan dan tindak lanjut | 15 | 11 |
| Dokumentasi dan komunikasi | 15 | 10 |
| **Total** | **100** | **78** |

### Mini SOC Report — DRAFT BY MENTOR, not learner-independent
**Case:** Current Network & Process Investigation, WebView2/Widgets.
**Observation:** Windows machine Ethernet has private IPv4 /24, default IPv4 route via local gateway. DNS query for example.com returned two A records; TCP connection test to example.com:443 succeeded. Separately, an actual Established TCP connection was observed from a local ephemeral port to public remote IP 104.89.118.72 on TCP443, owned by PID10512.
**Process evidence:** current chain observed PID10512 WebView2 (network utility) → PPID4100 WebView2 (embedded browser) → PPID6200 Widgets.exe. Widgets.exe points to Windows Client WebExperience package. Process names and executable paths consistent with Windows Widgets.
**File evidence:** Authenticode Status Valid and message Signature verified on WebView2 binary. Full signer subject and SHA-256 not provided in pasted output.
**Hypotheses:** benign: Widgets/WebView2 communicating for normal feature content. potentially malicious: component spoofing/abuse or unexpected activity, not demonstrated by present evidence.
**Assessment:** Activity appears **consistent with legitimate Widgets/WebView2**; no affirmative malicious indicator in the supplied observations. Still insufficient to make an absolute safe verdict or validate actual remote service identity.
**Evidence gaps:** Parent PID1416, signer subject, hash output, destination ownership, current-to-historical event GUID correlation, and independent case reporting.
**Next actions (optional):** inspect actual certificate signer and hash output; if risk/context warrants, check PID1416 and related network telemetry from same instance/time.

### Progression and weakness reinforcement
- PASS grants progression to Day22 without retaking entire Day21.
- Minor wording not corrected. Important topics to revisit through *different examples*: IPv4 subnet prefix, default gateway vs NAT, DNS success vs TCP success, Assessment vs Next Action, and competing hypotheses.
- Independence rating: **Terpandu** for final report; **bantuan terbatas** for selected commands/output interpretation. Not rated fully independent.
