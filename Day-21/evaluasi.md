# Day 21 — Evaluasi Sementara

**Status: IN PROGRESS, belum PASS/belum dinilai final.** Active Recall, Hands-on 1–6 dan Challenge telah dijawab sebagian besar; mini-report mandiri belum diserahkan. Koreksi mengutamakan salah konsep penting, bukan kekurangan redaksi.

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
