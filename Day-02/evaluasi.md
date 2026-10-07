# Day 02 — Evaluasi

### Evaluasi DAY 2 — PASS ✅

Kamu **lulus Day 2**. Status: **READY FOR NEXT DAY**.

Yang paling penting, cara berpikirmu mulai bergeser dari:

> “CPU tinggi = mencurigakan”

menjadi:

> “CPU tinggi adalah indicator. Sekarang saya perlu mencari context dan evidence untuk menjelaskan kenapa tinggi.”

Itu adalah perubahan mindset yang sangat penting untuk SOC.

---

## 1. Penilaian Hands-On

#### Task 1 — CPU ✅

Datamu lengkap:

* Ryzen 7 8845HS
* 8 cores
* 16 logical processors
* Base 3.80 GHz
* Current 2.35 GHz
* Utilization 3%

Kamu juga tidak menganggap current speed 2.35 GHz sebagai masalah. Itu benar. CPU tidak harus selalu berjalan pada base/max clock; frekuensi dapat berubah mengikuti workload dan power management.

---

#### Task 2 — Memory ✅

Datamu cukup lengkap:

* 16 GB RAM
* In use 9.1 GB
* Available 3.7 GB
* Committed 12.3/18.7 GB
* Cached 4.5 GB
* 5600 MT/s

Catatan penting:

**9.1 GB in use tidak berarti RAM-mu “hampir penuh dan bermasalah”.**

Windows juga menggunakan memory untuk cache dan kebutuhan sistem. Jadi angka memory harus dilihat bersama konteks keseluruhan.

---

#### Task 3 — Disk ✅

Datamu juga sudah benar sebagai snapshot kondisi saat itu:

* Capacity 477 GB
* Active time 1%
* Read 16 KB/s
* Write 127 KB/s

Yang perlu kamu ingat:

> **Disk usage rendah tidak berarti storage bagus atau buruk.**

Itu hanya menunjukkan aktivitas disk pada saat pengukuran relatif rendah.

---

#### Task 4 — Process ✅

Kamu berhasil mengambil:

* Process name
* PID
* CPU
* Memory

Itulah telemetry dasar yang nanti akan sangat sering kamu lihat dalam SOC.

---

## 2. Task 5 — Baseline ✅

Ini justru bagian yang bagus.

Discord:

* CPU 0.6% → 3.1%
* Memory 280.6 MB → 537 MB

Spotify:

* CPU 0.1% → 4%
* Memory 247.9 MB → 525 MB

Artinya kamu mulai melihat:

**baseline → activity → change**

Dan observasimu masuk akal:

> membuka halaman/fitur baru → aplikasi melakukan pekerjaan lebih banyak → CPU/RAM dapat meningkat.

Tetapi ada satu koreksi:

> Foto/video yang ada di storage **tidak langsung “diolah oleh RAM.”**

Model sederhananya lebih tepat:

**Storage → data dibaca ke RAM → CPU/GPU memproses → hasil ditampilkan**

RAM adalah tempat kerja sementara untuk data/instruction yang sedang digunakan. **RAM bukan tempat eksekusi. CPU yang mengeksekusi instruction**, sementara GPU juga dapat menangani pekerjaan graphics/rendering.

Untuk tahap sekarang, gunakan model sederhana itu. Nanti kita bedah lebih teknis.

---

## 3. Challenge — Saya nilai 7.5/10 ⭐

Kamu berhasil menemukan 5 kandidat:

1. Executable path
2. PID
3. Memory
4. CPU usage
5. Network connection

Ini **sudah bagus untuk pemula**.

Tapi ada hal yang sangat penting:

### Tidak semua yang kamu sebut sebenarnya memiliki kekuatan yang sama sebagai evidence.

Misalnya:

#### CPU Usage

`unknown.exe = 92% CPU`

Ini **indicator**, bukan bukti bahwa malicious.

#### Memory

`unknown.exe = 180 MB`

Juga **indicator**.

#### PID

`PID 4216`

Ini lebih tepat disebut **identifier/context**, bukan evidence yang menunjukkan maliciousness.

PID membantu kita berkata:

> “Saya sedang menyelidiki process yang mana?”

Bukan:

> “PID 4216 berarti process berbahaya.”

Ini perbedaan penting.

---

## 4. Jadi sebenarnya ada berapa Evidence?

Tidak ada angka tetap.

Dalam SOC, kamu biasanya mengumpulkan **beberapa jenis evidence dari beberapa sumber**, kemudian menghubungkannya.

Untuk kasus:

```text
unknown.exe
CPU: 92%
PID: 4216
```

Saya ingin kamu berpikir seperti ini:

#### A. Process evidence

**1. Executable Path**

Contoh:

```text
C:\Users\Buya\AppData\Local\Temp\unknown.exe
```

Pertanyaan:

> Apakah lokasi ini masuk akal untuk aplikasi tersebut?

Bandingkan dengan misalnya:

```text
C:\Windows\System32\...
```

atau folder instalasi aplikasi yang memang dikenal.

Path saja belum membuktikan malicious, tetapi bisa menjadi clue penting.

---

#### B. Process Relationship

**2. Parent Process**

Kita ingin tahu:

> Siapa yang menjalankan `unknown.exe`?

Misalnya:

```text
explorer.exe
   └── unknown.exe
```

versus:

```text
winword.exe
   └── powershell.exe
        └── unknown.exe
```

Context-nya jadi sangat berbeda.

Ini salah satu evidence yang **lebih bernilai** daripada sekadar CPU usage.

---

#### C. Command Line

**3. Command Line**

Misalnya process dijalankan sebagai:

```text
unknown.exe
```

atau:

```text
unknown.exe -update
```

atau bahkan:

```text
powershell.exe -enc ...
```

Command line dapat memberi tahu **apa yang sebenarnya process coba lakukan**.

---

#### D. User

**4. User / Account**

Siapa yang menjalankan process?

Contoh:

```text
Buya
SYSTEM
Administrator
```

Pertanyaannya:

> Apakah process tersebut masuk akal dijalankan oleh account tersebut?

---

#### E. Network

**5. Network Connection**

Ini yang tadi kamu bilang belum tahu cara melihatnya.

Sekarang kamu bisa mulai belajar.

Di Windows:

```text
Win + R
```

ketik:

```text
resmon
```

Lalu:

**Network → TCP Connections**

Cari PID:

```text
4216
```

Kamu bisa melihat koneksi seperti:

```text
PID     Local Address     Remote Address       State
4216    192.168.x.x       185.x.x.x:443        ESTABLISHED
```

Jadi kita dapat bertanya:

> Process 4216 berkomunikasi dengan siapa?

Ini tidak langsung berarti malicious. Browser normal juga akan memiliki banyak network connections.

Yang kita cari adalah **context**:

* remote IP/domain
* port
* connection state
* kapan koneksi terjadi
* apakah connection masuk akal untuk process tersebut

---

## 5. Bahkan bisa menggunakan command line

Nanti SOC Analyst akan nyaman dengan CLI.

Untuk melihat network connection:

```cmd
netstat -ano
```

`-o` menampilkan PID.

Kemudian kita cocokkan:

```text
unknown.exe → PID 4216
```

dengan:

```text
TCP connection → PID 4216
```

Kamu sedang mulai melakukan sesuatu yang sangat penting:

**correlation**

Yaitu menghubungkan satu evidence dengan evidence lain.

---

## 6. Evidence lain yang nanti harus kamu kenal

Selain 5 tadi, ada:

| Evidence           | Pertanyaan                                            |
| ------------------ | ----------------------------------------------------- |
| Executable Path    | File ini berada di mana?                              |
| Parent Process     | Siapa yang menjalankannya?                            |
| Command Line       | Process menjalankan perintah apa?                     |
| User               | Siapa yang menjalankan?                               |
| Network Connection | Berkomunikasi dengan siapa?                           |
| Digital Signature  | File ditandatangani publisher yang valid?             |
| File Hash          | File ini sebenarnya file apa?                         |
| Start Time         | Kapan process mulai?                                  |
| Persistence        | Apakah process dibuat agar jalan lagi setelah reboot? |
| File Changes       | Apakah process membuat/mengubah file?                 |
| Event Logs         | Apakah Windows mencatat aktivitas terkait?            |

Dan masih banyak lagi.

Jadi jangan merasa kamu harus menghafal semuanya sekarang.

**Kita akan mempelajarinya satu per satu.**

---

## 7. Cara berpikir SOC yang harus mulai kamu gunakan

Misalkan muncul:

```text
unknown.exe
CPU 92%
PID 4216
```

Jangan langsung:

> “Malware.”

Gunakan:

#### Observation

```text
unknown.exe menggunakan CPU 92%.
```

#### Hypothesis

```text
A. Legitimate heavy task
B. Malicious activity
```

#### Evidence

Cari:

```text
Path
Parent
Command line
User
Network
Signature
Hash
Logs
File activity
```

#### Analysis

Gabungkan semuanya.

#### Conclusion

Baru tentukan:

```text
Benign
Suspicious
Malicious
```

Perhatikan:

**CPU 92% adalah awal investigation, bukan akhir investigation.**

---

## 8. Self-Test — hasilmu

#### Q1 CPU vs RAM — ⚠️ perlu koreksi

Kamu menulis:

> CPU menjalankan process dari storage > RAM > process > CPU

Lebih tepat:

> **RAM menyimpan data dan instructions yang sedang digunakan. CPU mengeksekusi instructions tersebut.**

Model sederhananya:

```text
Storage
   ↓
RAM
   ↓
CPU executes instructions
   ↓
Application/process bekerja
```

Ini penyederhanaan untuk belajar dasar, tapi cukup untuk sekarang.

---

#### Q2 RAM vs Storage ✅

Analogi meja/wajan yang kamu gunakan cukup bagus.

Intinya sudah benar:

**Storage = persistent**

**RAM = temporary/volatile**

---

#### Q3 CPU 90% otomatis malicious? ✅

Jawabanmu tepat.

---

#### Q4 Baseline ✅

Bagus.

Bahkan contoh:

> user biasanya login jam 8 pagi, tiba-tiba login jam 2 malam

adalah contoh bagaimana **behavioral baseline** digunakan dalam security monitoring.

Belum tentu malicious, tetapi deviation perlu diperiksa.

---

#### Q5 Baseline penting? ✅

Benar.

---

#### Q6 One application → multiple processes ✅

Benar.

---

#### Q7 Same executable name → malware? ✅

Benar.

Harus melihat:

* PID
* path
* parent
* command line
* user
* network
* dan evidence lain.

---

#### Q8 CPU + RAM ⚠️

Bagian ini masih perlu diperbaiki.

RAM bukan:

> tempat mengolah memory.

Lebih tepat:

> **RAM adalah working memory yang menyimpan data dan instructions yang sedang dibutuhkan process. CPU melakukan computation/execution.**

Contoh sederhana:

Spotify sedang berjalan.

RAM bisa menyimpan data/instructions Spotify yang sedang digunakan.

CPU kemudian bekerja menjalankan instructions tersebut.

---

#### Q9 High RAM + Low CPU → malicious? ✅

Jawabanmu bagus.

Kamu bahkan mulai mengenali kemungkinan:

**memory leak**

Itu bagus sekali.

---

#### Q10 unknown.exe ✅

Jawabanmu sebenarnya sudah berkembang.

Kamu sudah sadar:

> “CPU tinggi saja tidak cukup.”

Itulah target utama Day 2.

---

## 9. Nilai keseluruhan

| Bagian                  | Hasil               |
| ----------------------- | ------------------- |
| Hands-on                | ✅ PASS              |
| Performance observation | ✅ PASS              |
| Baseline concept        | ✅ PASS              |
| Challenge reasoning     | ✅ PASS              |
| CPU concept             | ✅                   |
| RAM concept             | ⚠️ perlu diperbaiki |
| Evidence mindset        | ✅ DEVELOPING        |
| Investigation mindset   | ✅ DEVELOPING        |

### Status akhir:

## **DAY 2 — PASS ✅**

Kamu **READY FOR NEXT DAY**.

Tetapi ada satu hal yang saya ingin kamu tanamkan mulai sekarang:

> **Indicator ≠ Evidence ≠ Conclusion**

Contoh:

```text
CPU 92%
        ↓
Indicator

Executable path
Parent process
Command line
Network
Signature
Logs
        ↓
Evidence / Context

        ↓
Analysis

        ↓
Conclusion
```

Ini sudah mulai masuk cara berpikir **SOC Analyst**, meskipun kamu masih sangat awal.

Besok materi berikutnya sebaiknya mulai masuk ke **process, PID, executable, parent process, command line**, karena itu akan menjawab keraguanmu yang sekarang paling besar.


### Challenge dan jawaban

CHALLENGE 

Hypothesis A

Process tersebut legitimate tetapi sedang melakukan computationally intensive task.

Hypothesis B

Process tersebut malicious dan sedang melakukan aktivitas yang tidak diinginkan.

Tugas kamu:

Tentukan evidence apa yang ingin kamu cari untuk membedakan Hypothesis A dan B.

Minimal cari 5 evidence.

Format:

1. Evidence: Executable path apk
   Karena: kita lihat apakah apk tersebut asli atau tidak dengan melihat file location. trs di check apakah itu cloning atau tidak

2. Evidence: PID
   Karena: melihat PID jika sudah melihat proses mana yang memakan cpu usage besar, soalnya jika tidak ada PID kita kesulitan menentukan mana yg cpu usage nya besar karena kadang Namanya sama semua

3. Evidence: Memory
   Karena: kita bisa melihat memory, apakah normal memory yang di perlukan untuk aplikasi tersebut. jika ada salah satu yang tidak biasa penggunaannya seperti terlalu besar. kita patut mecurigai. tapi juga belum tentu

4. Evidence: CPU Usage
   Karena: apakah benar aplikasi tersebut membutuhkan cpu usage yg besar, contoh aplikasi seperti notepad kan tidak butuh rendering video, tapi kok menggunakan cpu usage yg besar, itu juga patut dicurigai

5. Evidence: Network Connect
   Karena: mungkin ini juga termasuk salah satu berbahaya. contoh kita perlu memeriksa arus koneksi, apakah besar lalulintasnya. contoh aplikasi firefox padahal kita tidak melihat video streaming atau mendownload kita hanya melakukan aktifitas scrolling ringan sedikit video banyak text seperti utasan di thread atau baca2 postingan di web. tapi lalulintas data begitu besar seperti sedang upload video atau download file besar (hanya kemungkinan tpi belum tau caranya melihat lalulintas data)

ini hanya isi kepalaku pemula awam

### Hasil mini-project

MINI PROJECT
PERFORMANCE BASELINE
IDLE

Process: Discord.exe
CPU: 0.6%
Memory: 280,6MB
Disk: 0.1 MB/s

Process : Spotify.exe
CPU: 0.1%
Memory: 247,9MB
Disk: 0 MB/s

ACTIVE

process: Discord.exe
CPU: 3.1%
Memory: 537MB
Disk: 0.1 MB/s

Process: Spotify.exe
CPU: 4%
Memory: 525MB
Disk: 0.1/s

OBSERVATION

Apa yang berubah?
CPU Usage, Memory, Disk

Process yang paling banyak menggunakan CPU:
aku belum tau pasti cuman yang aku liat dari discord dan spotify, mereka hamper memiliki cpu usage dan memory sma2 naik Ketika aktif dan sama2 turun Ketika sedang afk. tapi menurutku yang membuat cpu usage naik adalah Ketika aku buka shop di discord yang menampilkan berbagai macam pernak Pernik mungkin render animasi itu yang bikin naik. Begitu juga dengan spotify saya berpindah halaman satu ke halaman lain mencari lagu terus ada render video foto atau iklan

Process yang paling banyak menggunakan memory:
sama dengan jawaban diatas. untuk memory naik mungkin saat permintaan user klik tombol dan layer interface berganti terus muncul video dan foto baru. lalu os meminta permintaan foto dan video yang mana menurut aku itu adalah memori di ssd lalu diolah di ram

Kesimpulan:
jika kita mengetahu kenaikan cpu usage dan memory dalam sebuah aplikasi umumnya di angka berapa berapa... nanti Ketika kenaikannya abnormal atau diluar kebiasaan. bisa dijadikan evidence

menurut pendapat saya yg awam

### Self-test dan jawaban

SELF TEST


Apa perbedaan fungsi CPU dan RAM?

CPU: Untuk menjalankan process dari storage > ram >process> cpu
Ram: itu seperti memory sementara dia Ketika computer mati memorinya hilang. istilahnya tempat mengolah memori

Apa perbedaan RAM dan Storage?

Storage : itu tempat penyimpanan Ketika computer mati file masih ada
Ram : itu tempat mengolah memori seperti meja/wajan semakin besar meja atau wajannya semakin cepat proses mengolahnya tapi dia sifatnya sementara Ketika dimatikan memori filenya hilang

Jika sebuah process menggunakan CPU 90%, apakah otomatis malicious?

tidak, kita butuh Analisa lebih jauh mendalam lagi. bisa saja aplikasi yang kita buka sedang merendering video. atau ekstrak file atau sedang download. aplikasi tersebut memang sedang melakukan pekerjaan berat. contoh seperti aplikasi autocad

Apa yang dimaksud baseline?

baseline itu seperti kondisi wajar atau umumnya terjadi. contoh di suatu aplikasi umumnya cpu usage kisaran 2-10% tapi ini kok tiba2 jadi 50% itu sangat tidak wajar. atau bisa saja terjadi pada karyawan biasanya login di jam 8 pagi saat jam kerja , kok tiba2 login pukul 2 malam itu bisa jadi mencurigakan tapi kita belum bisa menyimpulkan apakah karyawan itu melakukan hal jahat atau tidak

Mengapa baseline penting dalam security investigation?

seperti yang kujelaskan sebelumnya, kita bisa melihat indicator kebiasaan kerja aplikasi pada umumnya lalu berubah drastic atau kita bisa melihat karyawan kita umumnya dalam kurun Waktu 6 bulan dia melakukan aktifitas berulang tiap hari, tapi suatu Ketika dia melakukan login yg tidak biasa

Apakah satu application bisa memiliki beberapa process?

bisa, dikarenakan bisa jadi aplikasi tersebut berjalan di belakang layer seperti mendownload, atau melakukan keamanan, pencegahan virus dll

Apakah beberapa process dengan executable name yang sama otomatis malware?

tidak, kita harus melihat beberapa kemungkinan. nama yang sama bisa jadi cpu usage berbeda, memori usage berbeda, PID, lalu lintas jaringan dll

Process menggunakan CPU dan RAM untuk apa?

ram itu menyimpan proses yang dijalankan user contoh sperti klik, Gerakan mouse, save gambar, copy gambar tapi hanya sebatas menyimpan kode perintah
lalu yang meng eksekusi perintah tersebut adalah CPU , dia bertugas menjalankan perintah kode dari ram agar bisa ditampikan di layer pengguna

Jika sebuah process menggunakan RAM sangat besar tetapi CPU rendah, apakah otomatis malicious?

10. belum tentu, bisa jadi user sedang membuka banyak aplikasi dalam 1 Waktu atau membuka banyak tab browser
atau bisa jadi sedang menggunakan photo editor atau video editor di dalam aplikasi tersebut kan user sedang editing foto dan video Ketika edit kan user menyimpan foto dan video sebelum di render nah penyimpanan sementara itu memerlukan ram besar. atau bisa jadi bug pada aplikasi yang developernya mungkin kelupaan memberi perintah membuang memori sementara sehingga memori tertumpuk di ram. jadi aplikasi tersebut terasa sangat berat

Dalam kasus:

unknown.exe
CPU 92%
PID 4216

apa yang masih perlu kamu ketahui sebelum mengambil kesimpulan?

sebenernya banyak, aku belum bner2 pandai untuk menentukan evidence. masih ragu menentukan evidence
aku hanya berputar putar Ketika melihat cpu usage tinggi , ahh ini mencurigakan
memori tinggi ah ini mencurigakan juga tapi 2 itu sekarang agak lumayan teratasi karena aku harus melihat indicator normal sebuah aplikasi berjalan
lalu evidence lain lalu lintas network aku juga belum bisa mengetahui cara melihatnya bagaimana
dan mungkin aku harus diberitahu ada berapa sih evidence itu contoh2nya
