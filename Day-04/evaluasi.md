# Day 04 — Evaluasi

Catatan ini mempertahankan jawaban, koreksi, dan penilaian yang tercatat. Sebagian jawaban hanya tersedia sebagai ringkasan mentor; pertanyaan atau jawaban lengkap yang tidak tercatat tidak direkonstruksi. Skenario latihan tidak dianggap sebagai incident pada endpoint aktual.

## Challenge

### 6. Challenge nomor 1 — ⚠️

Kamu menulis:

> “aplikasi tersebut tidak resmi”

Dari data:

```text
Digital Signature: NotSigned
```

kita **belum boleh mengatakan “tidak resmi.”**

Yang bisa kita katakan:

> **File tersebut tidak memiliki signature yang dapat diverifikasi oleh pemeriksaan Authenticode yang dilakukan.**

Kemudian:

> **This is an observation, not a malware verdict.**

Kamu sebenarnya sudah memahami ini di nomor 4 dan 5. Jadi masalahnya terutama pada penggunaan istilah.

---
### 7. Challenge nomor 6 — ❌ belum terjawab

Kamu menulis:

> “belum ada bayangan”

Nah, ini bagian yang perlu kita latihan.

Dari kasus:

```text
unknown.exe
CPU 92%
Temp
Parent = firefox.exe
NotSigned
SHA-256 = 7B3F...A912
```

Evidence berikutnya yang logis misalnya:

```text
1. Network Connection
2. Parent Process / Process Tree lebih lanjut
3. Command Line lebih lengkap
4. File origin / Zone information
5. Event Logs
6. Threat Intelligence lookup berdasarkan SHA-256
```

Tidak harus semuanya.

Yang penting sekarang kamu mulai bisa mengatakan:

> “Evidence yang saya punya belum cukup, jadi saya perlu mencari evidence tambahan.”

Itu inti investigation.

---
#### Challenge dan jawaban

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

## Mini-project / Investigasi

#### Hasil file investigation

File Investigation

##### File
Name: notepad.exe
Path: C:\Windows\...

##### Metadata
Name           : notepad.exe
Length         : 356352
CreationTime   : 9/9/2026 10:54:17 PM
LastWriteTime  : 9/9/2026 10:54:17 PM
LastAccessTime : 9/18/2026 10:21:09 PM
Attributes     : Archive

##### Digital Signature
Status: Valid
Signer/Publisher: DC91E564D5BC1E3A8E02D6A8508682ABEA8A2443

##### Hash
Algorithm: SHA-256
SHA-256: 87CA8213820A0ECAC9CD131B5E6BE9268A56E4E121482A5816EE7BAEFC9CE042

##### CMD vs PowerShell
PowerShell SHA-256:
CMD SHA-256:
Match: YES

##### Observation

Apa yang saya temukan?
saya menemukan informasi dalam evidence metadata yang penting untuk mendukung investigasi seperti tgl dibuat tgl edit jam dan menit, saya menemukan kode hash juga kode ini bisa berubah total hanya karena 1 karakter yg berubah, digital signature bisa mengetahui bahwa ini resmi atau ngga

Apa arti metadata file?
informasi tentang kapan file dibuat kapan file di edit kapan terakhir di akses attributenya apa

Apa arti Digital Signature?
seperti pengakuan digital , bersertifikat atau tidaknya sebuah aplikasi resmi atau tidaknya. dan berbayar

Apa arti SHA-256?
kode hash unik , bisa dijadikan salah satu indicator untuk mengetahui apakah itu malware atau bukan. kode atau perintah virus umumnya sama jadi kode hashnya juga sama
tapi bisa jadi bukan karena peretas punya cara untuk mengelabuhi dengan menambahkan huruf atau string didalam kode virus. sehingga tidak bisa terdeteksi

Apa perbedaan Hash dengan Digital Signature?
Digital Signature memberitahu bahwa file tersebut resmi atau tidak, sudah terdaftar atau tidak
kalau hash dia hanya kode unik yang mengatasnamakan file tersebut, kode hash bisa sama dengan file lainnya asalkan isinya sama persis 100% tapi kalua beda 1 karakter akan berubah total kode hash nya

##### Investigation Thinking

Apakah file ini otomatis aman hanya karena mempunyai
Digital Signature? TIDAK. karena peretas bisa memanipulasi dengan mengambil sertifikat asli dari perusahaan aplikasi tersebut , mungkin yg dinamakan private key

Apakah file otomatis malicious jika tidak mempunyai
Digital Signature? tidak juga, karena bisa jadi pembuatnya seorang individu yg belum punya uang untuk mendaftarkan aplikasi karena mahal, atau bisa jadi aplikasi tersebut open source yg belum didaftarkan. biasanya tujuan2nya untuk perbuatan baik

Apakah hash sendiri dapat membuktikan file malicious?
tidak, file malware A bisa memiliki hash yg sama dengan malware A lainnya
malware B memiliki kode hash yg berbeda dengan malware A karena isi kode virus didalamnya berbeda

tapi didalam file tersebut peretas bisa menambahkan spasi untuk mengelabuhi antivirus, karena menambahkan spasi atau 1 huruf bisa merubah hash seluruhnya

## Active Recall / Self-test

### CORRECTION DRILL DAY 4

Jawab 5 pertanyaan ini saja:

**1.** Apa perbedaan:

```text
SHA-256
vs
Digital Signature
```

**2.** Mana yang merupakan **identifier**, mana yang merupakan **context/evidence**?

```text
SHA-256
Creation Time
Digital Signature
PID
```

**3.** Apakah:

```text
Digital Signature = Valid
```

berarti file pasti aman? Jelaskan.

**4.** Apakah:

```text
Digital Signature = NotSigned
```

berarti file pasti malware? Jelaskan.

**5.** Kasus:

```text
unknown.exe
CPU 92%
Temp
Parent = Firefox
NotSigned
SHA-256 = 7B3F...A912
```

Sebutkan **5 evidence berikutnya** yang ingin kamu cari.

Tidak perlu command dulu. Saya ingin memastikan **cara berpikirmu** benar sebelum kita lanjut.

## Penilaian dan Hasil Praktik

EVALUATION

**Status: DEVELOPING ⚠️**

Hands-on kamu **PASS**, tetapi pemahaman konsep masih perlu beberapa koreksi sebelum saya nyatakan Day 4 PASS penuh.

---
### 1. Hands-On ✅ PASS

Kamu berhasil mendapatkan:

```text
Metadata
Digital Signature
SHA-256
CMD hash
PowerShell hash
```

Dan kamu menemukan bahwa hasil PowerShell dan CMD sama kecuali kapitalisasi huruf. Itu **benar**. Hash hexadecimal yang sama dapat ditulis uppercase atau lowercase tanpa mengubah nilainya. 

Ini latihan yang bagus karena kamu sekarang sudah mengalami sendiri:

```text
PowerShell
    ↓
SHA-256
    ↓
CMD
    ↓
SHA-256
    ↓
same result
```

---
### 2. Metadata — ✅

Pemahamanmu sudah cukup benar.

Kamu memahami bahwa metadata bisa memberikan informasi seperti:

* creation time
* last write time
* last access time
* attributes

Data nyata yang kamu dapat juga menunjukkan itu. 

---
### 3. Digital Signature — ⚠️ KOREKSI PENTING

Kamu mengatakan:

> “seperti pengakuan digital, bersertifikat atau tidaknya sebuah aplikasi resmi atau tidaknya dan berbayar.”

Bagian **“berbayar”** dan **“resmi/tidak resmi”** harus kita hilangkan.

Digital Signature **bukan tanda bahwa aplikasi itu berbayar**.

Lebih tepat:

> **Digital Signature membantu memverifikasi identitas signer/publisher dan integritas file yang ditandatangani.**

Jadi misalnya:

```text
Signer: Microsoft Corporation
Status: Valid
```

kita memperoleh informasi bahwa signature tersebut dapat diverifikasi dan berkaitan dengan certificate signer tersebut.

Tetapi:

```text
Valid Signature
≠
100% Safe
```

Dan:

```text
Unsigned
≠
Malware
```

Kamu sudah benar pada dua konsep terakhir itu. 

##### Satu lagi:

Pada hasilmu:

```text
SignerCertificate:
DC91E564...
```

itu **bukan nama publisher**. Itu merupakan informasi/identifier certificate yang ditampilkan oleh PowerShell dalam output tersebut. Jadi ketika kamu mengatakan:

> “gak ada tulisan siapa publishernya”

✅ benar berdasarkan output yang kamu dapat.

Kita belum belajar cara mengambil nama signer dari certificate tersebut.

---
### 4. Hash — ⚠️ konsepnya sudah dekat

Kamu mengatakan:

> “kode hash unik”

Lebih tepat:

> **Hash adalah nilai yang dihasilkan dari isi file melalui hash function.**

Analogi:

```text
File
 ↓
SHA-256
 ↓
Fingerprint-like value
```

Jadi SHA-256:

```text
87CA8213820A...
```

adalah **hash dari file tertentu pada kondisi tertentu**. 

Kamu benar bahwa:

> file dengan isi identik → hash yang sama.

Dan perubahan pada data biasanya menghasilkan hash yang berbeda.

---
### 5. Koreksi paling penting: Identifier vs Context ❌

Pada Challenge kamu menulis:

> identifier itu metadata dan digital signature
> context itu SHA256

Ini **terbalik**.

Yang lebih tepat:

##### Identifier

```text
SHA-256
PID
filename
```

Mereka membantu **mengidentifikasi sesuatu**.

Contohnya:

```text
SHA-256 = 7B3F...A912
```

adalah identifier/fingerprint untuk file tersebut.

##### Evidence / Context

```text
Creation Time
Last Write Time
Digital Signature
Executable Path
Command Line
Parent Process
Network Connection
```

Informasi tersebut membantu kita **memahami dan menilai context**.

Jadi:

```text
SHA-256
→ Identifier

Digital Signature
→ Evidence / Context

Creation Time
→ Evidence / Context
```

---
### 8. Hash sebagai IOC — ⚠️

Kamu mengatakan:

> “indicator pendeteksi yang cepat hanya dengan menggunakan hash, kita bisa mencari malware dengan nilai hash yang sama lalu mengeksekusinya.”

Bagian **“mencari malware dengan nilai hash yang sama”** arahnya benar.

Tetapi jangan mengatakan:

> “lalu mengeksekusinya.”

Dalam SOC, kita umumnya menggunakan hash untuk:

```text
File
 ↓
SHA-256
 ↓
IOC
 ↓
Search / Match
 ↓
Threat Intelligence / endpoint telemetry
```

Misalnya mencari apakah hash tersebut pernah terlihat pada endpoint lain atau dikenal sebagai malicious hash.

**Kita tidak perlu menjalankan file tersebut untuk melakukan hash matching.**

Dan benar bahwa attacker dapat mengubah file sehingga hash-nya berubah, tetapi itu berarti **hash matching bergantung pada file yang sama secara cryptographic identity**, bukan berarti hash “gampang dikelabui” secara universal.

---
### 9. Q10 — sebenarnya ini jawaban penting

Kamu belum tahu kenapa kita membandingkan:

```powershell
Get-FileHash ...
```

dengan:

```cmd
certutil -hashfile ...
```

Jawabannya sederhana:

> **Untuk memastikan kita memahami bahwa dua tools/interface berbeda dapat menghasilkan identifier cryptographic yang sama untuk file dan algoritma yang sama.**

Misalnya:

```text
PowerShell:
87CA8213820A...

CMD:
87CA8213820A...
```

Maka kita tahu:

```text
Same file
+
Same algorithm
=
Same SHA-256
```

Ini juga melatih kamu supaya tidak bergantung pada satu tool saja.

---
### 10. Ada satu hal bagus dari jawabanmu

Saya suka bagian ini:

> “mungkin evidence lain dan cara membaca commandline”

Itu menunjukkan kamu mulai memahami bahwa investigation **belum selesai setelah menemukan hash**.

Urutannya mulai menjadi:

```text
Process
 ↓
Executable
 ↓
Metadata
 ↓
Signature
 ↓
Hash
 ↓
???
 ↓
Further investigation
```

Bagus.

---
### 11. Penilaian Day 4

| Materi                         | Status |
| ------------------------------ | ------ |
| File path                      | ✅      |
| File metadata                  | ✅      |
| Digital Signature command      | ✅      |
| SHA-256 command                | ✅      |
| CMD `certutil`                 | ✅      |
| Hash concept                   | ⚠️     |
| Digital Signature concept      | ⚠️     |
| Identifier vs evidence/context | ❌      |
| Challenge investigation        | ⚠️     |
| Evidence-based mindset         | ✅      |
#### Kesimpulan

**Hands-on: PASS ✅**

**Theory: DEVELOPING ⚠️**

Jadi saya **belum memberikan Day 4 PASS penuh**.

Bukan karena kamu belum bisa command. Justru command-mu sudah berjalan dengan baik. Yang perlu diperbaiki adalah **cara membaca hasilnya**.

---

