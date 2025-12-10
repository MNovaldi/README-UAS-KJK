
# DoS Attack Simulation and Mitigation on IoT Smart Light Devices Using Snort IPS

### 1 Deskripsi Umum

Prototype ini bertujuan untuk:
Mensimulasikan serangan Denial of Service (DoS) terhadap perangkat IoT Bardi Smart Light Bulb dan mengevaluasi efektivitas Intrusion Prevention System (IPS) berbasis Snort dalam mendeteksi dan memblokir serangan tersebut.

Pada eksperimen ini digunakan:

- 1 laptop (berperan sebagai attacker, defender, dan IPS sekaligus)
- Wireshark untuk analisis trafik
- Snort sebagai IDS/IPS
- hping3 untuk simulasi DoS
- IoT Bardi Smart Light Bulb sebagai target


### 2 Tahapan Implementasi

Pertama mengindetifikasi perangkat

```ip a```

```nmap -sn 10.139.136.0/24```

Tujuannya untuk menemukan IP Bardi smart light bulb

### 3 Capture Trafik Awal
Menangkap trafik normal saat IoT berfungsi dengan baik

```sudo wireshark &```

``` ip.addr == 10.139.136.223```

### 4 Simulasi Serangan DoS
Serangan dilakukan tanpa flood ekstrem, dibatasi agar terlihat beda saat IPS aktif & tidak aktif:

Serangan ringan

```sudo hping3 -S -p 80 --count 200 10.139.136.223```

Serangan Intens

```sudo hping3 -S --flood -p 80 10.139.136.223```

### 5 Konfigurasi Snort sebagai IPS
Aktifkan snort inline:

```sudo snort -Q --daq afpacket -i eth0 -c /etc/snort/snort.conf -A console```

Mode -Q artinya:
Snort aktif sebagai IPS, bisa memblok trafik

Rule DoS Detection

Tambahkan ke file

```sudo nano /etc/snort/rules/local.rules```

Isi rule: 

```drop tcp any any -> 10.139.136.223 80 \```

(msg:"SYN FLOOD DOS BLOCKED"; flags:S; \
threshold:type both, track by_src, count 20, seconds 3; \
sid:1000002; rev:1;)


Penjelasan rule:
drop      : Paksa Snort memblokir paket
flags:S	  : Deteksi paket SYN
threshold : > 20 paket SYN dalam 3 detik
by_src    : Berdasarkan IP penyerang
sid	      : ID rule

### 6 Hasil Snort (IPS Aktif)

```[drop] "SYN FLOOD DOS BLOCKED"```

```{TCP} 10.139.136.21 -> 10.139.136.223:80```

Dapat dilihat Paket berhasil dideteksi & diblok oleh IPS

### 7 Analisis Data

perbedaan antara kondisi sebelum dan sesudah aktivasi IPS. Saat serangan SYN Flood dilakukan tanpa IPS, trafik meningkat drastis dan menyebabkan delay serta gangguan respons pada perangkat Bardi Smart Bulb. Setelah Snort IPS diaktifkan, sistem berhasil mendeteksi dan memblokir paket serangan yang ditandai dengan munculnya notifikasi [drop] pada log Snort. Meskipun paket masih terpantau di Wireshark, serangan tidak mencapai perangkat target, sehingga stabilitas kontrol perangkat tetap terjaga. Hal ini menunjukkan bahwa Snort IPS efektif dalam memitigasi serangan DoS pada lingkungan IoT skala kecil.















