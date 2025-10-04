# Jarkom-Modul-1-2025-K-18

# Praktikum Komunikasi Data dan Jaringan Komputer Modul 1 - K18

## Anggota Kelompok

| NRP        | Nama                            |
|:----------:|:-------------------------------:|
| 5027231046 | Ryan Adya Purwanto              |
| 5027241092 | Muhammad Khairul Yahya          |


## Topologi Jaringan

Topologi jaringan dibangun di GNS3 dengan arsitektur sebagai berikut:

### Komponen Jaringan
- **Router**: 1x Eru (ervn-debi) ---> pusat, gerbang ke internet
- **Switches**: 2x Ethernet Switch (Switch1, Switch2)
- **Clients**: 4x Ainur (Melkor, Manwe, Varda, Ulmo) (ervn-debi)
- **Internet**: 1x NAT / Cloud

### Diagram Koneksi

[gambar]

### Koneksi Topologi
- `NAT` ↔ `Eru` (eth0)
- `Eru` (eth1) ↔ `Switch1`
- `Eru` (eth2) ↔ `Switch2`  
- `Switch1` ↔ `Melkor` (eth0) & `Manwe` (eth0)
- `Switch2` ↔ `Varda` (eth0) & `Ulmo` (eth0)

## Persiapan

- Pastikan template docker image di GNS3 tersedia (ervn-debi / envn-debi sesuai server).
- Siapkan Wireshark capture pada link yang relevan (Eru<->Switch1, Eru<->Switch2, link client->switch).
- Direktori kerja: `/root/praktikum_mdoul1` pada setiap node (opsional).
- Pastiin semua node jalan di GNS3 (mati? restart aja)
- Wireshark jangan lupa jalan, kalau lupa capture, ngulang.

## Konfigurasi
> **Catatan:** Jalankan perintah-perintah ini di node yang sesuai (Eru / Melkor / Manwe / Varda / Ulmo). Gunakan `sudo` jika tidak sebagai root.

## Walkthrough Pengerjaan Soal

### Soal 1: Pembangunan Topologi

**Deskripsi:** Eru sebagai Router membuat dua Switch. Switch 1 menuju Melkor dan Manwe. Switch 2 menuju Varda dan Ulmo.

**Langkah-langkah:**
1. Membuat proyek baru di GNS3
2. Menambahkan 5 node ervn-debi (untuk Eru, Melkor, Manwe, Varda, Ulmo), 2 Ethernet Switch, dan 1 NAT
3. Menghubungkan perangkat dengan kabel virtual sesuai deskripsi topologi
4. Menyalakan semua node

**Script:**
```bash
# Tidak ada script khusus, dilakukan melalui GUI GNS3
```

**Bukti:**
[gambar topologi]

---

### Soal 2&3: Konektivitas Dasar & Routing

- **Walkthrough:** Di tahap ini, kita "menghidupkan" Eru. Pertama, Eru dihubungkan ke internet. Kemudian, fitur IP forwarding diaktifkan. Ini seperti mengubah Eru dari sekadar komputer biasa menjadi "petugas lalu lintas" yang pintar, yang bisa mengarahkan paket data antar jaringan yang berbeda (dari jaringan Melkor ke jaringan Varda, dan sebaliknya).

- **Perintah (Di Eru):**
```bash
# (Soal 2) Verifikasi koneksi internet Eru
ping -c 3 8.8.8.8

# (Soal 3) Aktifkan IP Forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```

- **Verifikasi (Di Melkor):**
```bash
ping -c 3 192.220.1.3  # Tes ke Manwe (satu jaringan)
ping -c 3 192.220.2.2  # Tes ke Varda (beda jaringan)
```

- **Bukti**:

[kumpulan ping]

### Soal 4: Koneksi Internet untuk Client

-**Walkthrough:** Client memiliki IP privat (192.220.x.x) yang tidak dikenali di internet. Agar mereka bisa online, kita perlu memasang "penyamaran" di Eru menggunakan iptables. Aturan MASQUERADE ini membuat semua client seolah-olah menggunakan IP publik milik Eru saat mengakses internet.

-**Peirntah (Di Eru):**
```bash
apt-get update && apt-get install -y iptables
/sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

- **Verifikasi (Di Melkor):**
```bash
ping -c 3 8.8.8.8
```

- **Bukti:**

[melkor ping ke internet]

### Soal 5: Konfigurasi Permanen (Revisi)

- **Walkthrough:** Masalah klasik di Linux adalah konfigurasi sementara akan hilang saat reboot. Untuk mengatasinya, kita menambahkan perintah IP forwarding dan NAT ke dalam file ~/.bashrc di Eru. File ini seperti "catatan pengingat" yang dibaca dan dijalankan oleh sistem setiap kali Eru booting atau login, sehingga Eru otomatis menjadi router setiap saat.

- **Perintah (Di Eru):** 
```bash
echo 'echo 1 > /proc/sys/net/ipv4/ip_forward' >> ~/.bashrc
echo '/sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE' >> ~/.bashrc

```

- **Pembuktian:**
1. Reboot node Eru dari GNS3.
2. Setelah Eru menyala, langsung tes `ping -c 3 8.8.8.8` dari **Melkor** tanpa konfigurasi manual.

- **Bukti:**
[ping berhasil ke melkor setelah di reboot]

---

### Soal 6: Packet Sniffing Manwe (Revisi)

- **Walkthrough:** Tugas ini adalah menyadap komunikasi Manwe. Terdapat kendala saat menggunakan wget karena link Google Drive dan masalah sertifikat. Solusinya adalah: atasi masalah DNS, gunakan wget dengan flag --no-check-certificate untuk mengunduh halaman HTML-nya, lalu secara manual salin-tempel konten script yang benar ke dalam file traffic.sh. Setelah itu, script bisa dijalankan sambil direkam oleh Wireshark.

- **Perintah (Di Manwe)**: 
```bash
# Atasi masalah DNS (jika perlu)
echo "nameserver 8.8.8.8" > /etc/resolv.conf

# Atasi masalah sertifikat & unduh halaman HTML
wget --no-check-certificate "[https://drive.google.com/drive/folders/1ULr_Fik1O0_79zUng41POMZtdzJTugVR?usp=sharing](https://drive.google.com/drive/folders/1ULr_Fik1O0_79zUng41POMZtdzJTugVR?usp=sharing)" -O traffic.sh

# Buka 'traffic.sh' dengan nano, hapus isinya, lalu tempel konten script yang benar.

# Jalankan script sambil direkam Wireshark
chmod +x traffic.sh && ./traffic.sh
```

- **Analisis:** Di Wireshark, terapkan filter ip.addr == 192.220.1.3 untuk mengisolasi semua lalu lintas data dari dan ke Manwe.

- **Bukti:** 
[ss wireshark di filter]

### Soal 7: Konfigurasi FTP Server (Revisi)

- **Walkthrough:** vsftpd di-install di Eru. Untuk mengatasi error vsftpd dan writable root, struktur direktori yang aman dibuat: user ainur "dipenjara" di direktori /home/ainur/ftp_root yang dimiliki oleh root, tetapi diberi hak tulis di sub-direktori /upload. User melkor diblokir sepenuhnya dengan tidak memasukkannya ke dalam userlist yang berfungsi sebagai "daftar tamu".

- **Verifikasi (Di Manwe):**

1. **Tes ainur:** ftp 192.220.1.1 21, login, cd upload, lalu put file. Hasil: Berhasil, membuktikan hak read-write.

2. **Tes melkor:** ftp 192.220.1.1 21, login. Hasil: Gagal dengan 530 Permission denied, membuktikan melkor diblokir.

- **Bukti:** 
[Screenshot terminal Manwe yang menunjukkan keberhasilan ainur dan kegagalan melkor.]

### Soal 8&9: Analisis FTP (Upload & Download)

- **Walkthrough:** Sesi FTP direkam untuk melihat "bahasa" asli protokol ini. Saat pengguna mengetik put (upload), perintah yang dikirim adalah STOR. Saat mengetik get (download), perintah yang dikirim adalah RETR. Untuk mengubah akses menjadi read-only, izin direktori upload diubah dengan chmod 555, yang menghapus hak write.

- **Analisis:**

1. **Upload (Soal 8):** Filter ftp di Wireshark menunjukkan perintah `Request: STOR.`
2. **Download (Soal 9):** Filter ftp di Wireshark menunjukkan perintah `Request: RETR.`

- **Verifikasi Read-Only (Di Manwe):** Coba put file sebagai ainur. Hasil: Gagal dengan 553 Could not create file.

- **Bukti:**
[ss STOR & RETR terus terminal manwe untuk kegagalan put]

### Soal 10: Analis Serangan Ping

- **Walkthrough:** Ini adalah stress test sederhana. Dengan ping -c 100, kita mengirim 100 "surat" ke Eru secara berturut-turut. Hasilnya dianalisis untuk melihat apakah ada "surat" yang hilang (packet loss) atau waktu balasnya melambat (avg rtt).
- **Perintah (Di Melkor):** ping -c 100 192.220.1.1
- **Analisis:** Hasil 0% packet loss dan avg rtt yang rendah membuktikan Eru sangat tangguh dan tidak terpengaruh.
- **Bukti:** 
[Screenshot ringkasan hasil ping di terminal Melkor.]

## Soal 11: Analisis Kelemahan Telnet

- **Walkthrough:** Telnet adalah protokol lawas untuk remote control. Kelemahan utamanya adalah tidak adanya enkripsi. Dengan merekam sesi login Telnet dan menggunakan fitur Follow > TCP Stream di Wireshark, kita bisa merekonstruksi seluruh percakapan persis seperti yang terjadi, termasuk melihat username dan password dalam bentuk teks biasa.
- **Analisis:** Di Wireshark, filter telnet, lalu Follow > TCP Stream.
- **Bukti:** 
[Screenshot jendela "Follow TCP Stream" yang dengan jelas menampilkan **username dan password dalam bentuk teks biasa**].

## Soal 12: Pemindaian Port

- **Walkthrough:** netcat (nc) adalah "pisau Swiss Army" untuk jaringan. Dengan mode pemindaian (-zv), kita bisa mengetuk "pintu-pintu" (port) di Melkor. Respons Connection refused berarti pintu tersebut tertutup rapat, membuktikan tidak ada layanan yang berjalan.

- **Perintah (Di Eru):** nc -zv -w 2 192.220.1.2 21 80 666

- **Bukti:**
[ss eru nampilin connection refused buat semua port]

## Soal 13: Analisis Keamanan SSH

- **Walkthrough:** SSH adalah pengganti Telnet yang aman. Kuncinya ada di **enkripsi**. Sesi dimulai dengan "jabat tangan" aman (Key Exchange) untuk membuat kunci rahasia. Setelah itu, semua data "dibungkus" dalam enkripsi. Jika kita coba intip dengan Follow > TCP Stream, yang terlihat hanyalah data acak, kontras dengan Telnet.

- **Analisis:** Di Wireshark, filter ssh. Paket-paket ditandai sebagai Encrypted packet. Follow > TCP Stream hanya menampilkan data acak.

- **Bukti:**
[ss follow tcp stream dari ssh yang ke enkripsi, terus dibandingin sama telnet (11)]


## Konfigurasi Jaringan Awal (`/etc/network/interfaces`)

### Eru (Router)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback

# Interface ke Internet (NAT)
auto eth0
iface eth0 inet dhcp

# Interface ke Switch 1 (Jaringan Melkor & Manwe)
auto eth1
iface eth1 inet static
address 192.220.1.1
netmask 255.255.255.0

# Interface ke Switch 2 (Jaringan Varda & Ulmo)
auto eth2
iface eth2 inet static
address 192.220.2.1
netmask 255.255.255.0
```

### Melkor (Client)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.220.1.2
netmask 255.255.255.0
gateway 192.220.1.1
dns-nameservers 8.8.8.8
```

### Manwe (Client)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.220.1.3
netmask 255.255.255.0
gateway 192.220.1.1
dns-nameservers 8.8.8.8
```

### Varda (Client)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.220.2.2
netmask 255.255.255.0
gateway 192.220.2.1
dns-nameservers 8.8.8.8
```

### Ulmo (Client)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.220.2.3
netmask 255.255.255.0
gateway 192.220.2.1
dns-nameservers 8.8.8.8
```

## Peralatan yang Digunakan
- **Simulator Jaringan:** GNS3 (Web UI & Client)
- **Sistem Operasi Node:** Docker Image ervn-debi
- **Alat Analisis Jaringan:** Wireshark
- **Alat Bantu:** `vsftpd`, `openssh-server`, `telnetd`, `netcat`, `iptables`

## Kesimpulan
Praktikum Modul 1 berhasil diselesaikan dengan baik. Kami telah mempelajari konfigurasi jaringan dasar, manajemen server FTP, analisis protokol, dan perbandingan keamanan antara Telnet dan SSH. Kendala teknis berhasil diatasi dengan troubleshooting yang sistematis.
