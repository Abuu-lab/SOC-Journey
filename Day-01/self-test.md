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
