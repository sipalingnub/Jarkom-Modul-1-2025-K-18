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
- NAT ↔ Eru (eth0)
- Eru (eth1) ↔ Switch1
- Eru (eth2) ↔ Switch2  
- Switch1 ↔ Melkor (eth0) & Manwe (eth0)
- Switch2 ↔ Varda (eth0) & Ulmo (eth0)

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
![Topologi Jaringan](images/topologi.png)

---

### Soal 2: Koneksi Internet untuk Router

**Deskripsi:** Membuat Eru dapat tersambung ke internet.

**Langkah-langkah:**
```bash
# Di konsol Eru
apt-get update && apt-get install -y isc-dhcp-client
dhclient eth0
ping -c 3 8.8.8.8
```

**Script:**
```bash
#!/bin/bash
# Konfigurasi internet untuk router Eru
apt-get update
apt-get install -y isc-dhcp-client
dhclient eth0
ping -c 3 8.8.8.8
```
**Walkthrough:** Di sini kita pastikan router Eru bisa internetan. DHCP dipasang supaya Eru dapat IP otomatis dari NAT. Tes ping ke `8.8.8.8` (Google DNS) buat bukti. Kalau reply jalan, berarti aman. Kalau nggak, berarti salah konfigurasi interface.

**Bukti:**
![Ping Internet Eru](images/ping-8.8.8.8-eru.png)

---

### Soal 3: Konektivitas Antar Client

**Deskripsi:** Memastikan semua Ainur (Client) dapat terhubung satu sama lain.

**Langkah-langkah:**
```bash
# Di konsol Eru - aktifkan IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Di konsol Melkor - test koneksi
ping -c 3 192.220.1.3  # Ke Manwe (satu jaringan)
ping -c 3 192.220.2.2  # Ke Varda (beda jaringan)
```

**Script:**
```bash
#!/bin/bash
# Di Eru - Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Di Melkor - Test konektivitas
echo "Testing connectivity to Manwe..."
ping -c 3 192.220.1.3
echo "Testing connectivity to Varda..."
ping -c 3 192.220.2.2
```
**Walkthrough:** Linux defaultnya bermasalah, dia nggak mau forward paket antar jaringan. Makanya kita aktifin IP forwarding manual. Setelah nyala, client bisa saling ping lewat Eru. Kalua masih gagal, kemungkinan routing table atau IP address salah.

**Bukti:**
![Ping Antar Client](images/ping-melkor-manwe.png)

---

### Soal 4: Koneksi Internet untuk Client

**Deskripsi:** Membuat semua Client dapat tersambung ke internet.

**Langkah-langkah:**
```bash
# Di konsol Eru - setup NAT
apt-get update && apt-get install -y iptables
/sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Di konsol Melkor - test internet
ping -c 3 8.8.8.8
```

**Script:**
```bash
#!/bin/bash
# Di Eru - Setup NAT Masquerade
apt-get update
apt-get install -y iptables
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Di Melkor - Test internet connectivity
echo "Testing internet connection..."
ping -c 3 8.8.8.8
```
**Walkthrough:** Client masih belum bisa internetan kalau cuman forward doang. Jadi NAT dipasang biar alamat private client diubah jadi public. Intinya, semua request client dipinjemin IP si Eru. Setelah aturan ini, client bisa ping ke internet.

**Bukti:**
![Ping Internet Client](images/ping-melkor-8.8.8.8.png)

---

### Soal 5: Konfigurasi Permanen

**Deskripsi:** Konfigurasi tidak boleh hilang saat semua node di-restart.

**Langkah-langkah:**
```bash
# Setup rc.local untuk IP forwarding
echo '#!/bin/sh -e' > /etc/rc.local
echo 'echo 1 > /proc/sys/net/ipv4/ip_forward' >> /etc/rc.local
echo 'exit 0' >> /etc/rc.local
chmod +x /etc/rc.local

# Install iptables-persistent
apt-get install -y iptables-persistent
```

**Script:**
```bash
#!/bin/bash
# Setup konfigurasi permanen di Eru

# Buat rc.local untuk IP forwarding
cat > /etc/rc.local << 'EOF'
#!/bin/sh -e
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables-restore < /etc/iptables/rules.v4
exit 0
EOF

chmod +x /etc/rc.local

# Install iptables-persistent
apt-get install -y iptables-persistent

# Simpan aturan iptables
iptables-save > /etc/iptables/rules.v4
```
**Walkthrough:** Masalah klasik: tiap reboot, iptables reset. Jadi kita install iptables-persistent biar rules NAT otomatis nyala pas booting. Anggap aja kayak auto-save di game.

**Bukti:**
![Konfigurasi Permanen](images/reboot-test.png)

---

### Soal 6: Packet Sniffing Komunikasi Manwe

**Deskripsi:** Menyusup ke komunikasi Manwe dan Eru saat file `traffic.sh` dijalankan.

**Langkah-langkah:**
1. Start Capture di GNS3 pada kabel antara Manwe dan Switch1
2. Jalankan script traffic.sh di Manwe
3. Stop capture setelah 10 detik
4. Filter dengan: `ip.addr == 192.220.1.3`

**Script traffic.sh:**
```bash
#!/bin/bash
# Script traffic.sh untuk menghasilkan traffic
echo "Starting traffic generation..."
for i in {1..5}; do
    ping -c 2 192.220.1.1  # Ping ke Eru
    sleep 1
done
echo "Traffic generation completed."
```
**Walkthrough:** Disuruh capture paket dari Manwe ke Eru. Bisa pakai tcpdump langsung di Eru atau capture Wireshark di link. Hasilnya nunjukin paket ICMP/TCP lewat, bukti bahwa routing udah benar.

**Bukti:**
![Packet Sniffing](images/soal6-packet-sniffing.png)

---

### Soal 7: Konfigurasi FTP Server

**Deskripsi:** Membuat FTP server di Eru dengan 2 user: ainur (read/write) dan melkor (tanpa akses).

**Langkah-langkah:**
```bash
# Install vsftpd
apt-get update && apt-get install -y vsftpd

# Buat user dan direktori
useradd -m ainur && passwd ainur
useradd -s /usr/sbin/nologin melkor && passwd melkor
mkdir -p /home/ainur/ftp_root/share_files

# Konfigurasi vsftpd
# Edit /etc/vsftpd.conf dengan:
# local_enable=YES, write_enable=YES, anonymous_enable=NO, chroot_local_user=YES
```

**Script:**
```bash
#!/bin/bash
# Konfigurasi FTP Server di Eru

# Install vsftpd
apt-get update
apt-get install -y vsftpd

# Buat user ainur
useradd -m ainur
echo "ainur:password123" | chpasswd

# Buat user melkor tanpa shell login
useradd -s /usr/sbin/nologin melkor
echo "melkor:password123" | chpasswd

# Buat struktur direktori FTP
mkdir -p /home/ainur/ftp_root/share_files
chown root:root /home/ainur/ftp_root
chown ainur:ainur /home/ainur/ftp_root/share_files
usermod -d /home/ainur/ftp_root ainur

# Konfigurasi vsftpd
cat > /etc/vsftpd.conf << 'EOF'
listen=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
anonymous_enable=NO
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
EOF

# Buat userlist
echo "ainur" > /etc/vsftpd.userlist

# Restart service
systemctl restart vsftpd
```
**Walkthrough:** Di sini kita bikin FTP server. `ainur` dikasih akses penuh, sedangkan `melkor` dibuat user dummy (pakai /usr/sbin/nologin biar gk bisa login). Folder share_files jadi lokasi upload/download. Tujuannya biar bisa bedain hak akses antar user.

**Bukti:**
![FTP Login](images/soal7-ftp-login.png)

---

### Soal 8: Analisis Upload FTP (STOR)

**Deskripsi:** Menganalisis proses upload file dari Ulmo ke Eru.

**Langkah-langkah:**
1. Start Capture pada kabel Ulmo ↔ Switch2
2. Dari Ulmo, konek FTP ke Eru, login sebagai ainur, dan upload file
3. Stop capture dan filter dengan `ftp`
4. Cari perintah `STOR`

**Script FTP Upload:**
```bash
#!/bin/bash
# Script untuk upload file via FTP dari Ulmo
ftp -n 192.220.2.1 << EOF
user ainur password123
put testfile.txt
quit
EOF
```

**Bukti:**
![FTP STOR](images/soal8-ftp-stor.png)

---

### Soal 9: Analisis Download FTP (RETR) & Read-Only

**Deskripsi:** Menganalisis proses download dari Eru ke Manwe, lalu mengubah akses ainur menjadi read-only.

**Langkah-langkah:**
```bash
# Capture paket FTP download
# Ubah permission menjadi read-only
chmod 555 /home/ainur/ftp_root/share_files
```

**Script:**
```bash
#!/bin/bash
# Di Eru - Ubah permission menjadi read-only
chmod 555 /home/ainur/ftp_root/share_files

# Di Manwe - Test download dan upload
echo "Testing FTP operations..."
ftp -n 192.220.1.1 << EOF
user ainur password123
get testfile.txt downloaded_file.txt
put newfile.txt
quit
EOF
```
**Walkthrough 8 & 9:** FTP gak aman karena semua perintah dan data dikirim plaintext. Kalau Ulmo upload file, di capture keliatan perintah `STOR`. Kalau Manwe download, muncul `RETR`. Jadi gampang banget orang iseng intip isinya. 

**Bukti:**
![FTP RETR](images/soal9-ftp-retr.png)

---

### Soal 10: Simulasi Serangan Ping Flood

**Deskripsi:** Menguji ketahanan Eru dengan mengirim 100 paket ping dari Melkor.

**Langkah-langkah:**
```bash
# Di konsol Melkor
ping -c 100 192.220.1.1
```

**Script:**
```bash
#!/bin/bash
# Ping flood test dari Melkor
echo "Starting ping flood test..."
ping -c 100 192.220.1.1
echo "Ping flood test completed."
```
**Walkthrough:** Tujuan soal ini buat ngetest flooding. Client nge-ping router berkali-kali sampe traffic penuh. Di capture keliatan banjir ICMP masuk ke Eru.

**Bukti:**
![Ping Flood](images/soal10-ping-flood.png)

---

### Soal 11: Analisis Kelemahan Telnet

**Deskripsi:** Membuktikan Telnet mengirim username dan password sebagai plain text.

**Langkah-langkah:**
1. Install telnetd di Melkor
2. Start Capture pada kabel Eru ↔ Switch1
3. Dari Eru, koneksi telnet ke Melkor dan login
4. Stop capture dan Follow TCP Stream

**Script:**
```bash
#!/bin/bash
# Di Melkor - Setup telnet server
apt-get update
apt-get install -y telnetd
systemctl start inetd

# Buat user untuk testing
useradd -m testuser
echo "testuser:testpass" | chpasswd

# Di Eru - Test telnet connection
telnet 192.220.1.2
```
**Walkthrough:** Telnet udah kuno, semua data dikirim tanpa enkripsi. Username & password bisa kelihatan jelas di Wireshark (Follow TCP Stream). Jadi ini bukti nyata kalau Telnet nggak aman.

**Bukti:**
![Telnet Plaintext](images/soal11-telnet-plaintext.png)

---

### Soal 12: Pemindaian Port dengan Netcat

**Deskripsi:** Memindai port 21, 80, dan 666 di Melkor dari Eru.

**Langkah-langkah:**
```bash
# Di konsol Eru
nc -zv -w 2 192.220.1.2 21 80 666
```

**Script:**
```bash
#!/bin/bash
# Port scanning dengan netcat dari Eru
echo "Scanning ports on Melkor (192.220.1.2)..."
nc -zv -w 2 192.220.1.2 21
nc -zv -w 2 192.220.1.2 80
nc -zv -w 2 192.220.1.2 666
echo "Port scanning completed."
```
**Walkthrough:** Netcat dipake buat scanning port. Outputnya nunjukin port mana aja yang terbuka. Teknik ini biasanya dipakai attacker buat reconnaissance.

**Bukti:**
![Netcat Scan](images/soal12-netcat-scan.png)

---

### Soal 13: Analisis Keamanan SSH

**Deskripsi:** Membuktikan keamanan SSH dengan menyadap sesi login dari Varda ke Eru.

**Langkah-langkah:**
1. Install openssh-server di Eru
2. Start Capture pada kabel Varda ↔ Switch2
3. Dari Varda, koneksi SSH ke Eru dan login
4. Stop capture dan Follow TCP Stream

**Script:**
```bash
#!/bin/bash
# Di Eru - Setup SSH server
apt-get update
apt-get install -y openssh-server
systemctl start ssh
systemctl enable ssh

# Buat user untuk testing
useradd -m sshuser
echo "sshuser:sshpass123" | chpasswd

# Di Varda - Test SSH connection
ssh sshuser@192.220.2.1
```

**Walkthrough:** SSH kebalikan dari Telnet: semua data terenkripsi. Di Wireshark kita cuman bisa lihat ciphertext, gak bisa lihat passwordnya. Inilah kenapa SSH masih dipakai sampai sekarang.

**Bukti:**
![SSH Encrypted](images/soal13-ssh-encrypted.png)

---

## Konfigurasi Jaringan Lengkap

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
