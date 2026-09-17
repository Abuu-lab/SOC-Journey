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
