# Day 01 — Evaluasi

### — Evaluasi

**Status: DEVELOPING → READY dengan catatan kecil.**

Secara keseluruhan, untuk seseorang yang benar-benar mulai dari 0, hasilmu **sudah cukup baik untuk lanjut**, terutama karena kamu tidak asal menjawab. Kamu mulai menunjukkan pola berpikir investigasi: mencari kemungkinan lain sebelum menyimpulkan malware.

Ada beberapa konsep yang perlu saya luruskan sebelum Day 2.

---

### 1. Review Challenge

#### 1. Process mana yang diperiksa?

Jawabanmu:

> memilih proses dengan CPU Usage tinggi

**Benar sebagai titik awal, tetapi belum cukup.**

CPU 78% memang layak diperiksa karena merupakan **anomaly/performance indicator**.

Namun SOC Analyst tidak berpikir:

> CPU tinggi = mencurigakan.

Kita berpikir:

> **Why is this process consuming 78% CPU?**

Misalnya:

```text
Firefox → CPU 78%
```

bisa saja:

* membuka website berat
* menjalankan JavaScript
* video
* browser extension
* update
* legitimate workload
* atau sesuatu yang malicious

Jadi CPU usage adalah **signal**, bukan verdict.

---

#### 2. CPU tinggi = malware?

Jawabanmu: **belum tentu.**

✅ Benar.

Tetapi alasanmu tentang spesifikasi perangkat perlu sedikit diperbaiki.

CPU 90% **bukan berarti CPU tidak mampu**, dan bukan pula otomatis karena suhu tinggi.

Yang kita cari adalah **context**.

Contoh:

```text
chrome.exe
CPU: 90%
```

Pertanyaan berikutnya:

```text
What website?
What process?
What parent process?
What command line?
Which user?
How long?
What network connection?
```

Itulah pola pikir SOC.

---

### 3. Evidence berikutnya

Kamu memilih **memory usage**.

✅ Bisa menjadi evidence tambahan.

Tetapi saya ingin kamu mulai memperluas cara berpikir.

Kalau melihat:

```text
Process A
CPU: 78%
Memory: 500 MB
```

jangan hanya:

> "Memory-nya besar."

Tanyakan:

```text
Is this normal for this application?
What is the process name?
Where is the executable located?
Who started it?
What is its parent process?
What command line was used?
Does it make network connections?
How long has it been running?
```

Ini akan menjadi kebiasaan penting.

---

## 4. Containment

Di sini ada **satu koreksi penting**.

Kamu mengatakan:

> menutup aplikasi abnormal

Untuk troubleshooting komputer biasa, itu masuk akal.

Tetapi dalam **incident investigation**, kita tidak boleh terlalu cepat mengubah evidence.

Misalnya kita menemukan:

```text
suspicious.exe
CPU 90%
```

Kemudian langsung:

```text
End Process
```

Kita mungkin kehilangan:

* process state
* network connection
* command line context
* volatile evidence
* informasi yang membantu investigation

Jadi SOC Analyst biasanya:

**observe → collect evidence → assess → respond**

bukan langsung:

**see anomaly → kill process**

Nanti kita akan belajar kapan containment memang diperlukan.

---

## 5. Informasi yang kurang

Kamu menjawab:

> suhu

Ini **bisa relevan untuk troubleshooting**, tetapi untuk SOC investigation bukan evidence utama.

Yang lebih penting justru:

* process name
* PID
* executable path
* parent process
* command line
* user
* start time
* network connection
* file/signature
* related events/logs

Kamu sendiri mengatakan:

> belum mengerti parent process

**Bagus.**

Itu justru menjadi salah satu materi yang akan kita pelajari.

---

## REVIEW SELF TEST

### 1. CPU

Jawabanmu:

> seperti manusia memproses di meja (RAM)

🟡 **Konsepnya sudah mengarah benar.**

Analogi kamu bagus:

```text
RAM = meja kerja
CPU = orang yang mengerjakan
Storage = lemari penyimpanan
```

Tetapi CPU bukan sekadar "orang yang memproses".

Lebih tepat:

> **CPU executes instructions.**

Ini terminology yang perlu kamu biasakan.

---

### 2. RAM vs Storage

Jawabanmu:

> RAM seperti meja kerja, storage tempat data mentah.

✅ **Bagus.**

Analogi ini bisa kita pertahankan.

```text
Storage = lemari
RAM     = meja
CPU     = pekerja
```

Ketika program berjalan, OS mengatur penggunaan RAM dan CPU.

---

### 3. Operating System

Jawabanmu:

> perantara aplikasi dengan hardware

✅ **Benar.**

Tambahkan satu konsep:

OS bukan hanya "penerjemah".

OS juga **mengelola resources**:

```text
CPU
RAM
Storage
Network
Devices
Processes
Users
```

Ini akan sangat penting nanti.

---

### 4. Application

Jawabanmu:

> sesuatu yang memudahkan pekerjaan pengguna...

🟢 **Benar secara konsep.**

Contoh yang kamu berikan juga tepat.

---

## 5. Process

Di sini ada koreksi penting.

Kamu mengatakan:

> process adalah suatu data yang menampilkan beberapa aplikasi atau system yang sedang berjalan.

🟡 **Hampir, tetapi definisinya perlu diperbaiki.**

Process bukan "data yang menampilkan aplikasi".

Lebih tepat:

> **A process adalah instance dari program yang sedang dieksekusi oleh operating system.**

Misalnya:

```text
firefox.exe
```

adalah executable/program.

Ketika dijalankan:

```text
firefox.exe
       ↓
Windows creates process
       ↓
PID 6804
```

Process kemudian memiliki berbagai resources/context, misalnya:

* memory
* CPU time
* handles
* security context
* threads
* network activity

Ini alasan process sangat penting dalam SOC.

---

## 6. Application → Process

Jawabanmu:

> ketika aplikasi sedang berjalan maka kita dapat melihat processnya

🟢 **Benar.**

Dan ini penting:

Satu application **tidak selalu = satu process**.

Contoh browser dapat membuat banyak processes.

Misalnya:

```text
firefox.exe → PID 1000
firefox.exe → PID 1001
firefox.exe → PID 1002
```

Nama executable bisa sama, tetapi masing-masing adalah **process berbeda**.

---

## 7. PID

Jawabanmu:

> nomor khusus untuk membedakan system/aplikasi yang berjalan

🟢 **Benar.**

Tetapi ada satu kesalahan yang perlu kita hilangkan:

> "PID berfungsi sebagai pendeteksi mana malware mana aplikasi asli."

❌ **Tidak.**

PID hanya **identifier**.

Contoh:

```text
notepad.exe
PID 1234
```

PID 1234 tidak berarti legitimate.

Malware juga mendapatkan PID.

Misalnya:

```text
malware.exe
PID 5678
```

Jadi:

**PID ≠ malware detector**

PID membantu kita **mengidentifikasi dan melacak process**.

---

## 8. Kenapa process penting untuk SOC?

Jawabanmu cukup bagus.

🟢 Kamu sudah mulai berpikir ke arah SOC:

> CPU usage abnormal
> memory usage abnormal
> process analysis

Nanti kita tingkatkan:

```text
Process
 ↓
Parent Process
 ↓
Command Line
 ↓
User
 ↓
File Path
 ↓
Network
 ↓
Logs
```

---

## 9. Task Manager vs tasklist

Jawabanmu:

> Task Manager lebih ramah pemula karena GUI

🟢 Benar.

Tapi perbedaan pentingnya:

**Task Manager** memberikan graphical interface dan informasi lebih mudah dibaca.

**tasklist** memberikan process listing melalui command line.

SOC Analyst perlu command line karena nanti kita akan bekerja dengan server/Linux/remote systems yang mungkin **tidak memiliki GUI**.

---

## 10. CPU 90% = malicious?

Jawabanmu:

> bisa jadi tapi tidak selalu

🟢 **Ini jawaban yang benar.**

Dan kamu sudah mulai berpikir seperti analyst ketika mengatakan:

> kita harus melihat beberapa kemungkinan terlebih dahulu.

Pertahankan kebiasaan itu.

---

## SATU HAL YANG SAYA INGIN KAMU HILANGKAN

Jangan terlalu cepat menggunakan pola:

```text
Abnormal
   ↓
Malware
```

Gunakan:

```text
Observation
     ↓
Hypothesis
     ↓
Evidence
     ↓
Analysis
     ↓
Conclusion
```

Contohnya:

```text
Observation:
Firefox CPU 90%

        ↓

Hypothesis:
Firefox mungkin melakukan aktivitas abnormal

        ↓

Evidence:
Process tree
Command line
Network connection
Executable path
User
Browser activity

        ↓

Analysis:
Apakah aktivitas tersebut expected?

        ↓

Conclusion:
Benign / Suspicious / Malicious
```

**Ini salah satu fondasi terpenting untuk perjalanan SOC kamu.**

---

## HASIL DAY 1

| Skill                  | Status   |
| ---------------------- | -------- |
| Hardware fundamentals  | 🟢       |
| CPU/RAM/Storage        | 🟢       |
| Operating System       | 🟢       |
| Application            | 🟢       |
| Process                | 🟡       |
| PID                    | 🟢       |
| Task Manager           | 🟢       |
| Command line dasar     | 🟢       |
| Security reasoning     | 🟡       |
| Evidence thinking      | 🟡       |
| Parent Process         | 🔴 Belum |
| Malware identification | 🔴 Belum |

#### Overall

**DEVELOPING — READY FOR DAY 2**

Saya **tidak akan menahanmu hanya karena parent process belum dipahami**, karena memang itu belum menjadi materi Day 1. Kita akan membahasnya ketika sudah relevan.

---

## DAY 1 SUDAH SELESAI

Dan ada hal yang cukup bagus dari hasilmu:

Kamu tidak mencoba membuat jawabanmu terdengar "cybersecurity banget". Kamu menjelaskan berdasarkan pemahamanmu sendiri dan bahkan menulis:

> "belum mengerti"

Itu justru cara belajar yang bagus.

Mulai sekarang saya akan lebih ketat.

Kalau jawabanmu salah, saya akan bilang salah. Kalau benar sebagian, saya akan bilang sebagian. **Tidak akan saya naikkan level hanya supaya cepat selesai.**

### Status akhir:

**DAY 1 ✅ PASSED**

Kamu **boleh lanjut ke DAY 2**.

Saat siap, cukup kirim:

**`Day 2`**

dan saya akan memberikan materi yang merupakan **kelanjutan langsung dari Day 1**, bukan topik random.


### Challenge dan jawaban

1. Process mana yang pertama kali ingin kamu periksa ?
saya akan memilih proses dengan CPU Usage tinggi, karena tidak masuk akal, biasanya rendah penggunaannya
2. apakah CPU usage tinggi otomatis berarti malware?
belum tentu mungkin bisa jadi spesifikasi perangkat kita yang tidak mumpuni untuk membuka aplikasi tersebut
3. evidence apa yang ingin kamu cari berikutnnya
penggunaan memory, yang terlalu besar. 
4. apakah kamu langsung melakukan containment
karena saya masih awam dan belum tau lebih dalam, saya akan melakukan penutupan aplikasi yang abnormal, seperti cpu usage yang besar lalu Ketika masih lag saya akan mulai menutup aplikasi dengan penggunaan memory yang besar
5. informasi apa yang masih kurang?
mungkin, informasi panasnya suhu. bisa menyebabkan lag

### Hasil mini-project

MY COMPUTER SECURITY BASELINE

1. Hardware
CPU:AMD Ryzen 7 8845HS 
RAM: 16GB
Storage: 477GB
GPU: Radeon 780M Graphics

2. Operating System
OS: Microsoft Windows 11 Pro
Version: 10.0.28000 N/A Build 28000
Architecture: x64 - based PC

3. Current User
Username: Buya

4. Running Processes
 Name: steam.exe
   PID: 4324
   Status: Running
   CPU: 00
   Memory : 18.436K

2. Name: Cloudflare WARP.exe
   PID: 11852
   Status: Running
   CPU: 00
   Memory: 4968K 

3. Name: firefox.exe
   PID: 6804
   Status:  Running
   CPU: 00
   Memory: 348.884K

4. Name: Spotify.exe
   PID: 8120
   Status: Running
   CPU: 01
   Memory: 191.840K 

5. Observations

Apa yang saya pelajari:
- Membuka program atau aplikasi yang masih running
- melihat PID nomor proses disetiap aplikasi yg running
- melihat cpu usage
- melihat memory usage
- melihat informasi PC atau laptop yg lebih komplit melalui CMD menggunakan command systeminfo
bisa melihat OS name, OS version, manufaktur, architecture

Hal yang belum saya pahami:
- belum mengerti cara membedakan apakah cpu dengan usage tinggi terkena malware atau tidak
- belum bisa membedakan apakah memory dengan usage tinggi terkena malware atau tidak
- belum mengerti apa itu parent proses

### Self-test dan jawaban

SELF TEST

1. apa fungsi CPU?
dia seperti manusia memproses di meja (ram)

2. apa perbedaan RAM dan Storage ?
Ram itu seperti meja kerja dia mendapat data mentah dari storage lalu mengolahnya di meja yaitu RAM tapi yang mengolah adalah CPU (manusia)

3. apa fungsi operating System
dia membaca kerja aplikasi, atau sebagai perantara aplikasi dengan hardware seperti ram,cpu,storage. jadi aplikasi tidak perlu membaca kode yg begitu rumit dari hardware, cukup OS yang mengolahnya. 

4. apa yang dimaksud aplikasi?
aplikasi adalah sesuatu yang memudahkan pekerjaan pengguna atau punya maksud dan tujuan yang di inginkan pengguna. dia memproses data lalu menampilkan dalam bentuk grafis yang mudah dibaca pengguna. biasa kita liat seperti spotify ,FB, chrome dll

5. apa yang dimaksud process?
process adalah suatu data yang menampilkan beberapa aplikasi atau system yang sedang berjalan. tiap system yang berjalan akan diberi PID

6. apa hubungan application dengan process?
sangat berhubungan sekali, karena Ketika aplikasi sedang berjalan maka kita dapat melihat processnya, entah itu berjalan secara terlihat maupun dibelakang layer dan itu ada PID nya sendiri sendiri

7. apa fungsi PID?
PID adalah nomor yang diberikan khusus pada system yang berjalan atau aplikasi yang berjalan, biasanya digunakan untuk membedakan . karena kita kadang melihat process dari sebuah aplikasi yang begitu bnyak. contoh firefox saya melihat 10 firefox berjalan bersamaan tapi PID nya berbeda . itu juga berfungsi sebagai pendeteksi mana malware mana aplikasi asli

8. seperti yang saya jelaskan di no 7. PID sangat penting karena akan menjadi senjata untuk melihat mana aplikasi palsu mana asli. dan process itu memperlihatkan banyak info yang bagus untuk analisis kerentanan. seperti cpu usage yang abnormal ataupun memory usage yang abnormal

9.apa perbedaan melihat process melalui task manager dan tasklist dari CMD?
perbedaannya task manage lebih ramah pemula karena ada GUI, kalua dari tasklist mungkin karena saya pemula jadi masih kurang memahami bagaimana cara proses membacanya agar lebih cepat

10. jika sebuah process menggunakan CPU 90%, apakah itu bukti bahwa process tersebut malicious? jelaskan!
bisa jadi tapi tidak selalu. bisa saja karena panasnya CPU membuat berat aplikasinya
bisa jadi karena aplikasinya speknya terlalu tinggi untuk CPUnya jadi berat
tapi bisa jadi iya, mungkin Ketika kita crosscheck lebih dalam ada aplikasi double yang abnormal, dan kita lihat PIDnya berbeda PID 1 dia rendah penggunaannya PID 2 dia tinggi penggunaannya. kemungkinan itu adalah aplikasi palsu/malware. kita harus melihat beberapa kemungkinan terlebih dahulu

### Screenshot praktik

![Bukti praktik 1](../assets/Day-01/Screenshot%202026-09-16%20203920.png)

![Bukti praktik 2](../assets/Day-01/Screenshot%202026-09-16%20204028.png)

![Bukti praktik 3](../assets/Day-01/Screenshot%202026-09-16%20204046.png)
