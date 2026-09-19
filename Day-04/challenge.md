CHALLENGE

Sekarang kita kembali ke kasus Day 3.

Kamu menemukan:

unknown.exe

PID: 4216
CPU: 92%
Memory: 180 MB

Parent:
firefox.exe
PID: 5500

Executable Path:
C:\Users\Buya\AppData\Local\Temp\unknown.exe

Command Line:
unknown.exe -update

Kemudian kamu mendapatkan file evidence:

Size:
3.8 MB

Creation Time:
2026-09-18 14:05

 Last Write Time:
2026-09-18 14:05

Digital Signature:
NotSigned

SHA-256:
7B3F...A912
Tugas kamu:

1. Sebutkan minimal 5 observation dari kasus tersebut.
- kita bisa melihat bahwa aplikasi tersebut tidak resmi
- kita mempunya kode hash yng nantinya bisa di check di total virus
- kita punya informasi dibuatnya file dan kapan terakhir di buka
- kita punya info besarnya file 3.8mb , ini harus dicari tau apakah normal file ini punya besaran 3.8mb
- filenya berada di temp, apakah itu normal atau tidak

2. Mana yang merupakan identifier, mana yang merupakan evidence/context?
identifier itu metadata dan digital signature
kalua context itu sha256

3. Apa arti SHA-256 7B3F...A912?
kode hash khusus yang diberikan untuk file tersebut, ya tidak khusus sih
kalua ada file yg isinya sama. hashnya juga sama

4. Apakah NotSigned otomatis berarti malicious? Jelaskan.
tidak juga, karena bisa jadi pembuatnya seorang individu yg belum punya uang untuk mendaftarkan aplikasi karena mahal, atau bisa jadi aplikasi tersebut open source yg belum didaftarkan. biasanya tujuan2nya untuk perbuatan baik

5. Apakah file di Temp otomatis berarti malicious? Jelaskan.
tidak, ada beberapa aplikasi yg membutuhkan temp, untuk install untuk file sementara dll

6. Evidence apa yang ingin kamu cari berikutnya? 
belum ada bayangan
