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
