# Jarkom-Modul-5-E26-2023
| Nama | NRP |
|----------|----------|
| Handitanto Herprasetyo | 5025201077 |
| Nadiah Nuri Aisyah  | 5025211210 |  

<h2>Prefix IP</h2>

192.219.X.X

### Tugas pertama, buatlah peta wilayah sesuai berikut ini:
<img src=soal.png>
Keterangan:	Richter adalah DNS Server
- Revolte adalah DHCP Server 
- Sein dan Stark adalah Web Server
- Jumlah Host pada SchwerMountain adalah 64
- Jumlah Host pada LaubHills adalah 255
- Jumlah Host pada TurkRegion adalah 1022
- Jumlah Host pada GrobeForest adalah 512

### Untuk menghitung rute-rute yang diperlukan, gunakan perhitungan dengan metode VLSM. Buat juga pohonnya, dan lingkari subnet yang dilewati.

<img src=subnet.png>
<img src=treevlsm.png>
<img src=pembagian-ip.png>

### Kemudian buatlah rute sesuai dengan pembagian IP yang kalian lakukan.

<img src=rute.png>

### Tugas berikutnya adalah memberikan ip pada subnet SchwerMountain, LaubHills, TurkRegion, dan GrobeForest menggunakan bantuan DHCP.

Soal
--

### 1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Aura menggunakan iptables, tetapi tidak ingin menggunakan MASQUERADE.

```bash
ip_gateway=$(ip -4 addr show eth0 | grep inet | awk '{ print substr( $2, 1, length($2)-3 ) }')
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source $ip_gateway`
```
`ip -4 addr show eth0`: Bagian perintah ini menggunakan perintah ip untuk menampilkan alamat IPv4 yang ditetapkan ke antarmuka jaringan eth0. Opsi -4 menentukan bahwa ia hanya akan menampilkan alamat IPv4.

`grep inet`: Output dari perintah sebelumnya kemudian disalurkan (|) ke grep, yang digunakan untuk memfilter baris yang mengandung kata kunci "inet". Ini membantu mengekstrak baris yang berisi informasi alamat IPv4.

`awk '{ print substr( $2, 1, length($2)-3 ) }'`: Output dari grep kemudian disalurkan ke awk, yang merupakan alat pemrosesan teks yang kuat. Dalam perintah awk ini:

`$2` merujuk pada kolom kedua dari input, yang merupakan alamat IPv4.
`substr( $2, 1, length($2)-3 )` mengekstrak substring dari alamat IPv4. Dimulai dari karakter pertama (1) dan terus ke panjang alamat IPv4 dikurangi 3. Ini dilakukan untuk menghapus "/24" atau informasi subnet yang serupa.

`ip_gateway = $(...)`: Terakhir, seluruh perintah diapit oleh $(...) untuk melakukan substitusi perintah. Ini berarti bahwa output dari perintah di dalam tanda kurung ditetapkan ke variabel ip_gateway

### 2. Kalian diminta untuk melakukan drop semua TCP dan UDP kecuali port 8080 pada TCP.
```bash
iptables -A INPUT -p tcp -j DROP
iptables -A INPUT -p udp -j DROP
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
```
### 3. Kepala Suku North Area meminta kalian untuk membatasi DHCP dan DNS Server hanya dapat dilakukan ping oleh maksimal 3 device secara bersamaan, selebihnya akan di drop.
```bash
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

`-A INPUT`: Opsi ini menambahkan aturan ke akhir tabel yang ditentukan (INPUT dalam kasus ini). Tabel INPUT digunakan untuk paket masuk yang ditujukan untuk sistem lokal.

`-p icmp`: Ini menentukan protokol yang akan dicocokkan, dalam hal ini, ICMP (Internet Control Message Protocol), yang biasanya digunakan untuk permintaan ping.

`-m connlimit`: Modul ini memungkinkan Anda untuk mencocokkan jumlah koneksi yang dimiliki host dalam jangka waktu tertentu.

`--connlimit-above 3`: Opsi ini menetapkan ambang batas untuk jumlah koneksi bersamaan. Aturan akan cocok jika jumlah koneksi bersamaan melebihi 3.

`--connlimit-mask 0`: Opsi ini menentukan mask yang akan diterapkan saat memeriksa koneksi bersamaan. Mask 0 berarti bahwa perbandingan harus dilakukan pada basis per-IP, mempertimbangkan semua koneksi terlepas dari port sumber tertentu.

`-j DROP`: Jika kondisi yang ditentukan dalam aturan terpenuhi (misalnya, jika jumlah koneksi bersamaan di atas 3), tindakan yang dilakukan adalah drop (membuang) paket. Dengan kata lain, setiap permintaan ping tambahan di luar batas yang ditentukan akan di-drop, yang secara efektif membatasi jumlah ping simultan dari satu sumber.


### 4. Lakukan pembatasan sehingga koneksi SSH pada Web Server hanya dapat dilakukan oleh masyarakat yang berada pada GrobeForest.
```bash
iptables -A INPUT -p tcp --dport 22 -s 192.219.4.2/22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

### 5. Selain itu, akses menuju WebServer hanya diperbolehkan saat jam kerja yaitu Senin-Jumat pada pukul 08.00-16.00.
```bash
iptables -A INPUT -m time --timestart 08:00 --timestop 16:00 –weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -j DROP
```

### 6. Lalu, karena ternyata terdapat beberapa waktu di mana network administrator dari WebServer tidak bisa stand by, sehingga perlu ditambahkan rule bahwa akses pada hari Senin - Kamis pada jam 12.00 - 13.00 dilarang dan akses di hari Jumat pada jam 11.00 - 13.00 juga dilarang.
```bash
iptables -A INPUT -m time --timestart 08:00 --timestop 12:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -m time --timestart 08:00 --timestop 11:00 --weekdays Fri -j ACCEPT
iptables -A INPUT -m time --timestart 13:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -m time --timestart 12:00 --timestop 13:00 --weekdays Mon,Tue,Wed,Thu -j DROP
iptables -A INPUT -m time --timestart 11:00 --timestop 13:00 --weekdays Fri -j DROP
```
