# Pengembangan Analisis Perbandingan Snort dan Suricata dalam Mendeteksi Serangan Metasploit

## 1. Gambaran Penelitian

README ini merupakan panduan implementasi dan pengembangan dari materi penelitian **“Penerapan Intrusion Detection dan Prevention System untuk Mendeteksi Serangan Metasploit Menggunakan Snort dan Wireshark”**. Metodologi utama yang digunakan tetap **PPDIOO (Prepare, Plan, Design, Implement, Operate, Optimize)**.

Pengembangan dilakukan dengan menambahkan Suricata sebagai objek pembanding terhadap Snort. Penambahan tersebut tidak mengubah alur dasar penelitian: menyiapkan lingkungan, merancang topologi, mengimplementasikan IDS, menjalankan simulasi Remote Exploit menggunakan Metasploit, mengamati alert, dan menganalisis lalu lintas menggunakan Wireshark.

### 1.1 Tujuan

1. Menerapkan Snort sebagai Intrusion Detection System (IDS) untuk memantau lalu lintas jaringan.
2. Menguji kemampuan Snort dalam mendeteksi lalu lintas yang dihasilkan oleh skenario Remote Exploit menggunakan Metasploit.
3. Menganalisis paket jaringan dan alert menggunakan Wireshark.
4. Mengembangkan penelitian dengan menguji Suricata pada skenario dan lingkungan yang sama.
5. Membandingkan hasil deteksi dan penggunaan sumber daya berdasarkan bukti pengujian.

### 1.2 Batasan dan prinsip pengujian

- Pengujian dilakukan pada mesin virtual milik sendiri dan jaringan laboratorium yang terisolasi.
- Kali Linux berperan sebagai attacker; Linux Mint berperan sebagai target sekaligus sensor IDS.
- Snort dan Suricata diuji secara bergantian, bukan dijalankan bersamaan saat pengukuran.
- Skenario Metasploit, target, alamat jaringan, durasi, dan jumlah pengulangan dijaga sama untuk kedua IDS.
- Fokus pengamatan adalah deteksi dan pencatatan alert. Jangan mengaktifkan pemblokiran inline apabila penelitian sedang mengukur IDS/passive detection.
- Hasil, versi perangkat lunak, rule, dan konfigurasi harus dicatat berdasarkan kondisi aktual. Jangan mengisi hasil berdasarkan perkiraan.

------------------------------------------------------------------------

## 2. Informasi Lingkungan Penelitian

| Parameter                     | Konfigurasi aktual                                                |
|:------------------------------|:------------------------------------------------------------------|
| Platform virtualisasi         | VMware Workstation                                                |
| Sistem operasi attacker       | Kali Linux                                                        |
| Sistem operasi target dan IDS | Linux Mint 22.3 (Zena)                                            |
| Kernel target saat persiapan  | `6.14.0-37-generic`                                               |
| Interface NAT Kali            | `eth0`                                                            |
| Interface LAN Kali            | `eth1`                                                            |
| Interface NAT Linux Mint      | `ens33`                                                           |
| Interface LAN Linux Mint      | `ens37`                                                           |
| IP LAN Kali                   | `192.168.56.10/24`                                                |
| IP LAN Linux Mint             | `192.168.56.20/24`                                                |
| Subnet pengujian              | `192.168.56.0/24`                                                 |
| Jaringan virtual              | LAN Segment 1                                                     |
| IDS utama sesuai materi       | Snort                                                             |
| IDS pengembangan/pembanding   | Suricata                                                          |
| Tools pengujian               | Metasploit Framework, tcpdump, Wireshark                          |
| Tools pendukung               | Apache2, PHP, MySQL/MariaDB, Barnyard2, BASE (sesuai alur materi) |

> **Catatan perbedaan lingkungan:** materi acuan mencantumkan Ubuntu Desktop 20.04, Snort 2.9.20, BASE 1.4.5, Wireshark 3.2.3, Kali Linux 6.0, dan VirtualBox 7.0. Implementasi ini menggunakan VMware Workstation dan Linux Mint 22.3 sesuai lingkungan aktual. Versi perangkat lunak yang terpasang harus diperiksa dan dicatat; versi pada materi merupakan acuan penelitian, bukan klaim versi yang sedang terpasang.

------------------------------------------------------------------------

# METODOLOGI PPDIOO

Materi acuan menggunakan enam tahapan PPDIOO. Tahap **Prepare** dan **Plan** dilaksanakan secara beriringan karena persiapan dan perencanaan saling berkaitan. Urutan berikut dipakai sebagai struktur utama README.

## A. Prepare — Persiapan

### A.1 Tujuan

Mengumpulkan informasi, mengidentifikasi kebutuhan penelitian, menyiapkan perangkat, dan melakukan konfigurasi awal agar penelitian dapat dijalankan dengan terarah. Tahap ini juga digunakan untuk menemukan kendala awal dan memperbaikinya sebelum implementasi.

### A.2 Identifikasi kebutuhan penelitian

Komponen yang dibutuhkan:

| Komponen                    | Peran                                                      |
|:----------------------------|:-----------------------------------------------------------|
| Kali Linux                  | Mesin attacker untuk menjalankan Metasploit                |
| Linux Mint                  | Mesin target dan tempat IDS dijalankan                     |
| Snort                       | IDS utama sesuai materi                                    |
| Suricata                    | IDS tambahan untuk pengembangan/perbandingan               |
| Metasploit Framework        | Membuat lalu lintas skenario Remote Exploit                |
| Wireshark                   | Menganalisis paket dan komunikasi jaringan                 |
| tcpdump                     | Merekam lalu lintas menjadi file PCAP                      |
| Apache2, PHP, MySQL/MariaDB | Komponen pendukung alur Web BASE pada materi               |
| Barnyard2                   | Membaca/meneruskan output unified Snort menuju penyimpanan |
| BASE                        | Antarmuka web untuk menampilkan alert Snort                |

### A.3 Periksa sistem operasi dan kernel Linux Mint

Jalankan pada Linux Mint:

```bash
cat /etc/os-release
uname -r
```

Catat hasil aktual:

| Parameter             | Hasil aktual        |
|:----------------------|:--------------------|
| Distribusi            | Linux Mint          |
| Versi                 | 22.3 (Zena)         |
| Kernel saat persiapan | `6.14.0-37-generic` |

<img width="1281" height="321" alt="image" src="https://github.com/user-attachments/assets/4996752d-5c29-4a5f-b5be-7ff02b72da82" />


### A.4 Periksa sumber daya mesin

```bash
free -h
df -h /
lsblk -f
```

Catat kapasitas RAM, ruang penyimpanan, dan disk virtual. Informasi ini menjadi kondisi awal sebelum IDS dijalankan.

<img width="1281" height="301" alt="image" src="https://github.com/user-attachments/assets/9634a7a1-c4d2-44f3-84fa-748cb275525b" />


### A.5 Perbarui sistem dan pasang utilitas awal

```bash
sudo apt update
sudo apt upgrade -y
```

Pasang utilitas jaringan dan pemantauan:

```bash
sudo apt install -y curl wget git nano net-tools iproute2 tcpdump ethtool htop sysstat jq
```

Fungsi utilitas:

| Utilitas       | Kegunaan                                  |
|:---------------|:------------------------------------------|
| `iproute2`     | Memeriksa interface dan routing           |
| `tcpdump`      | Merekam paket ke PCAP                     |
| `ethtool`      | Memeriksa informasi interface             |
| `htop`         | Memantau CPU dan RAM                      |
| `sysstat`      | Mencatat statistik resource melalui `sar` |
| `curl`, `wget` | Mengambil berkas dari jaringan            |
| `jq`           | Membaca data JSON                         |

Jika APT menampilkan sumber `cdrom://`, periksa **Software Sources** dan pastikan media instalasi tidak lagi menjadi sumber repositori aktif pada instalasi permanen.

<img width="1283" height="511" alt="Cuplikan layar 2026-11-14 113519" src="https://github.com/user-attachments/assets/559ab47c-75d0-42a2-8557-9fec24336fa1" />


### A.6 Pemeriksaan awal konektivitas

Dari Kali Linux:

```bash
ping -c 4 192.168.56.20
```

Dari Linux Mint:

```bash
ping -c 4 192.168.56.10
```

Koneksi antar-VM harus berhasil sebelum tahap desain dan implementasi dilanjutkan.

<img width="1277" height="257" alt="image" src="https://github.com/user-attachments/assets/42af2aea-277e-4c6f-8bc0-ed6afe571853" />

<img width="1281" height="237" alt="image" src="https://github.com/user-attachments/assets/e6970b9e-909a-40c9-b400-dbd3b2799f5b" />


### A.7 Kriteria penyelesaian Prepare

- Sistem operasi dan resource VM telah diperiksa.
- Kebutuhan perangkat keras dan perangkat lunak telah diidentifikasi.
- Sistem telah diperbarui dan utilitas dasar tersedia.
- Kedua VM dapat berkomunikasi melalui jaringan pengujian.
- Kendala persiapan telah dicatat dan diselesaikan.

------------------------------------------------------------------------

## B. Plan — Perencanaan

### B.1 Tujuan

Menentukan perangkat keras, perangkat lunak, versi, fungsi setiap komponen, dan kebutuhan pengujian. Perencanaan ini menjadi dasar untuk membuat desain jaringan dan menjalankan implementasi.

### B.2 Rencana perangkat lunak

Materi acuan mencantumkan perangkat lunak berikut. Versi aktual harus diperiksa kembali pada sistem yang digunakan.

| No. | Komponen             | Versi acuan dalam materi | Fungsi dalam penelitian                            |
|----:|:---------------------|:-------------------------|:---------------------------------------------------|
|   1 | Ubuntu Desktop       | 20.04                    | Sistem operasi server/target pada penelitian acuan |
|   2 | Snort                | 2.9.20                   | Menangkap dan mendeteksi serangan yang masuk       |
|   3 | BASE                 | 1.4.5                    | Menampilkan alert Snort melalui antarmuka web      |
|   4 | Wireshark            | 3.2.3                    | Menganalisis lalu lintas jaringan                  |
|   5 | Kali Linux           | 6.0                      | Mesin attacker                                     |
|   6 | Metasploit Framework | Tidak dicantumkan        | Menjalankan simulasi Remote Exploit                |
|   7 | VirtualBox           | 7.0                      | Virtualisasi pada penelitian acuan                 |

Pengembangan pada lingkungan aktual:

| Komponen           | Lingkungan aktual                       | Fungsi                |
|:-------------------|:----------------------------------------|:----------------------|
| VMware Workstation Pro 26H1 | 26.0.0.25388281 | Menjalankan VM        |
| Linux Mint         | 22.3 (Zena)                             | Target dan sensor IDS |
| Kali Linux         | 6.19.14+kali-amd64 | Attacker              |
| Suricata           | 7.0.3 | IDS pembanding        |
| tcpdump            | 4.99.4                   | Merekam PCAP          |
| Wireshark          | 4.6.4 (v4.6.4-0-g93282876538d) | Analisis paket        |
| Snort              | 2.9.20 GRE (Build 82)                   | Menangkap dan mendeteksi serangan yang masuk       |

### B.3 Rencana perangkat keras/virtual

| Perangkat  | Peran               | IP jaringan pengujian |
|:-----------|:--------------------|:----------------------|
| Linux Mint | Server/target + IDS | `192.168.56.20/24`    |
| Kali Linux | Penyerang           | `192.168.56.10/24`    |


| Komponen | Kali Linux | Linux Mint | Windows Host   |
| -------- | ---------- | ---------- | -------------- |
| Storage  | 25 GB      | 30 GB      | Sisa kapasitas |
| RAM      | 4 GB       | 4 GB       | Sisa RAM       |
| CPU      | 2 Core     | 2 Core     | Prosesor host  |

### B.4 Rencana prosedur pengujian

1. Menyiapkan jaringan virtual dan target.
2. Menginstal serta mengonfigurasi Snort dan rule.
3. Menyiapkan komponen pendukung Snort dan Web BASE sesuai alur materi.
4. Menguji konfigurasi Snort sampai berhasil.
5. Menjalankan skenario Remote Exploit menggunakan Metasploit.
6. Mencatat alert dan merekam paket jaringan.
7. Menganalisis file PCAP menggunakan Wireshark.
8. Mengulangi skenario yang sama menggunakan Suricata sebagai pengembangan penelitian.
9. Mengevaluasi hasil deteksi dan penggunaan resource.

### B.5 Variabel yang dijaga konsisten

| Variabel               | Ketentuan                     |
|:-----------------------|:------------------------------|
| Attacker               | Kali Linux yang sama          |
| Target                 | Linux Mint yang sama          |
| Jaringan               | LAN Segment 1                 |
| IP attacker dan target | Tetap                         |
| Layanan target         | Sama untuk Snort dan Suricata |
| Skenario Metasploit    | Sama                          |
| Jumlah pengulangan     | Sama                          |
| Durasi pengujian       | Sama                          |
| Aktivitas normal       | Sama                          |
| Metode pencatatan      | Sama                          |

### B.6 Kriteria penyelesaian Plan

- Perangkat keras dan perangkat lunak telah ditentukan.
- Fungsi setiap komponen telah jelas.
- Skenario dan variabel pengujian telah direncanakan.
- Kebutuhan bukti dan pencatatan telah ditentukan.

------------------------------------------------------------------------

## C. Design — Perancangan

### C.1 Tujuan

Merancang topologi jaringan dan hubungan antarperangkat berdasarkan kebutuhan penelitian. Desain mempertimbangkan fungsi perangkat, jalur lalu lintas, kinerja, dan keamanan jaringan.

### C.2 Topologi jaringan aktual

```text
                          INTERNET
                             |
                    +----------------+
                    |   VMware NAT   |
                    +-------+--------+
                            |
             +--------------+--------------+
             |                             |
       Adapter 1: NAT                Adapter 1: NAT
             |                             |
    +--------+---------+          +--------+---------+
    |    KALI LINUX    |          |    LINUX MINT    |
    |                  |          |                  |
    | Attacker         |          | Target + IDS     |
    |                  |          |                  |
    | eth0             |          | ens33            |
    | NAT              |          | NAT              |
    |                  |          |                  |
    | eth1             |          | ens37            |
    | 192.168.56.10    |          | 192.168.56.20    |
    +--------+---------+          +--------+---------+
             |                             |
             +-------------+---------------+
                           |
                    LAN Segment 1
                    192.168.56.0/24
```

Keterangan:

- Adapter 1 pada kedua VM menggunakan NAT untuk kebutuhan internet dan pembaruan.
- Adapter 2 pada kedua VM menggunakan LAN Segment 1 untuk lalu lintas pengujian.
- Kali Linux mengirimkan lalu lintas skenario melalui `eth1`.
- Linux Mint menerima lalu lintas melalui `ens37`.
- Snort atau Suricata memantau interface `ens37` secara bergantian.
- Wireshark digunakan untuk menganalisis hasil perekaman paket.



<img width="426" height="55" alt="image" src="https://github.com/user-attachments/assets/36d18db0-9b10-41be-bdc9-dc26de0515f9" />

<img width="435" height="54" alt="image" src="https://github.com/user-attachments/assets/0aa9dd56-1b50-4144-abd7-a77556c79265" />

<img width="389" height="54" alt="image" src="https://github.com/user-attachments/assets/aa4772ba-b7b0-4393-bf5e-c9f506a2a947" />

### C.3 Verifikasi interface Kali Linux

```bash
ip -br addr
ip route
```

Rencana interface:

| Interface | Fungsi         | IP                   |
|:----------|:---------------|:---------------------|
| `eth0`    | NAT / Internet | Periksa hasil aktual |
| `eth1`    | LAN Segment 1  | `192.168.56.10/24`   |

<img width="1282" height="338" alt="image" src="https://github.com/user-attachments/assets/36ae5a44-c188-4070-92d4-c949db07c937" />


### C.4 Verifikasi interface Linux Mint

```bash
ip -br addr
ip route
```

Rencana interface:

| Interface | Fungsi         | IP                   |
|:----------|:---------------|:---------------------|
| `ens33`   | NAT / Internet | Periksa hasil aktual |
| `ens37`   | LAN Segment 1  | `192.168.56.20/24`   |

Interface LAN tidak menggunakan default gateway. Default route internet menggunakan interface NAT.

<img width="1281" height="202" alt="image" src="https://github.com/user-attachments/assets/666c30e0-105e-4332-a569-fd3da5a38b2a" />


### C.5 Penetapan target dan layanan

Linux Mint berfungsi sebagai target yang menerima lalu lintas dari Kali Linux. Sebelum pengujian, tentukan layanan dan versi yang memang disiapkan untuk skenario laboratorium.

Catat informasi berikut:

| Parameter               | Nilai aktual                           |
|:------------------------|:---------------------------------------|
| Nama layanan target     | Diisi setelah ditetapkan               |
| Versi layanan           | Diisi setelah diperiksa                |
| Port layanan            | Diisi setelah diperiksa                |
| Alamat target           | `192.168.56.20`                        |
| Skenario Remote Exploit | Diisi sesuai modul yang dipilih        |
| Alasan pemilihan target | Diisi berdasarkan kebutuhan penelitian |

Jangan mengekspos layanan yang sengaja dibuat rentan ke jaringan publik atau jaringan kampus. Pengujian hanya dilakukan pada VM dan jaringan laboratorium yang dikendalikan peneliti.

### C.6 Rancangan alur sistem

```text
Persiapan VM dan jaringan
          |
          v
Perancangan topologi
          |
          v
Implementasi Snort IDS
          |
          v
Konfigurasi dan pengujian Snort
          |
          v
Simulasi Remote Exploit
          |
          v
Pencatatan alert dan paket
          |
          v
Analisis menggunakan Wireshark
          |
          v
Pengembangan pengujian Suricata
          |
          v
Analisis dan perbandingan
```

### C.7 Kriteria penyelesaian Design

- Topologi jaringan telah ditentukan.
- Jalur lalu lintas attacker menuju target telah jelas.
- Interface pemantauan telah ditentukan.
- Layanan target dan skenario pengujian telah direncanakan.
- Alur implementasi telah terdokumentasi.

------------------------------------------------------------------------

## D. Implement — Implementasi

### D.1 Tujuan

Menerapkan seluruh komponen sesuai rencana dan desain. Tahap implementasi mengikuti diagram alir instalasi dan konfigurasi Snort IDS pada materi acuan, dimulai dari instalasi Snort, instalasi komponen pendukung, konfigurasi dan pengujian Snort, hingga instalasi serta konfigurasi BASE.

Suricata ditambahkan sebagai pengembangan penelitian untuk membandingkan kemampuan deteksi dan penggunaan sumber daya dengan Snort.

> **Catatan kompatibilitas:** Materi acuan menggunakan kombinasi Snort 2.9.20, Barnyard2, dan BASE 1.4.5. Komponen tersebut merupakan stack yang lebih lama. Pada Linux Mint 22.3, kompatibilitas paket dan dependensi harus diperiksa. Catat versi dan kondisi aktual setiap komponen. Jangan menyatakan instalasi atau integrasi berhasil sebelum dilakukan verifikasi.

### D.2 Instalasi Snort

Snort merupakan Intrusion Detection System (IDS) utama dalam penelitian ini. Instalasi dilakukan pada Linux Mint yang berperan sebagai target sekaligus sensor IDS.

Periksa ketersediaan paket Snort pada repositori:

```bash
apt-cache policy snort
```

Jika paket tersedia dan versi serta dependensinya sesuai dengan kebutuhan penelitian, lakukan instalasi:

```bash
sudo apt install -y snort
```

Setelah instalasi selesai, periksa versi Snort:

```bash
snort -V
```

Pastikan informasi versi dan build yang ditampilkan sesuai dengan paket yang terpasang. Catat versi aktual untuk kebutuhan dokumentasi dan perbandingan.

<img width="1281" height="492" alt="Instalasi dan versi Snort" src="https://github.com/user-attachments/assets/7669d5ce-b807-440c-9980-abdd14df583d" />

### D.3 Instalasi Komponen Pendukung Snort

Mengikuti diagram alir pada materi acuan, implementasi Snort dilanjutkan dengan instalasi komponen pendukung yang diperlukan untuk pemrosesan dan penampilan alert melalui Web BASE.

Komponen yang disiapkan meliputi:

| Komponen | Fungsi |
|---|---|
| Apache2 | Web server untuk menjalankan antarmuka BASE |
| PHP | Menyediakan lingkungan pemrosesan aplikasi BASE |
| MySQL/MariaDB | Menyimpan data alert yang diproses |
| Barnyard2 | Memproses output Snort dan meneruskannya ke database |

#### 1. Pemeriksaan ketersediaan paket

Periksa paket yang tersedia pada repositori Linux Mint:

```bash
apt-cache policy apache2 php mysql-server mariadb-server barnyard2
```

Pemeriksaan ini dilakukan untuk mengetahui ketersediaan paket dan versi yang dapat digunakan. Tidak semua komponen harus tersedia langsung melalui repositori bawaan.

#### 2. Instalasi Apache2, PHP, dan database

Instal komponen web serta database sesuai paket yang tersedia:

```bash
sudo apt install -y apache2 php mysql-server
```

Apabila `mysql-server` tidak tersedia, periksa alternatif MariaDB:

```bash
sudo apt install -y apache2 php mariadb-server
```

Gunakan salah satu sistem database sesuai paket yang benar-benar terpasang. Jangan memasang keduanya tanpa kebutuhan yang jelas.

Periksa status Apache2:

```bash
systemctl status apache2
```

Jika menggunakan MySQL:

```bash
systemctl status mysql
```

Jika menggunakan MariaDB:

```bash
systemctl status mariadb
```

<img width="1281" height="675" alt="Instalasi komponen Apache2, PHP, dan database" src="https://github.com/user-attachments/assets/f7132133-e69c-4c00-abbb-f562197efcf9" />

#### 3. Instalasi Barnyard2 dari Source Code

Berdasarkan hasil pemeriksaan repositori, paket Barnyard2 tidak tersedia untuk dipasang secara langsung melalui APT. Oleh karena itu, instalasi dilakukan dengan mengompilasi source code dari repositori GitHub.

Periksa ketersediaan paket:

```bash
apt-cache policy barnyard2
```

Unduh source code Barnyard2:

```bash
git clone https://github.com/firnsy/barnyard2.git
cd barnyard2
```

Pasang dependensi yang diperlukan untuk proses kompilasi:

```bash
sudo apt install -y libtool autoconf automake libpcap-dev libmariadb-dev libmariadb-dev-compat libdaq-dev libdumbnet-dev
```

Jalankan proses persiapan dan konfigurasi:

```bash
./autogen.sh
```

```bash
./configure --with-mysql \
--with-mysql-includes=/usr/include/mariadb \
--with-mysql-libraries=/usr/lib/x86_64-linux-gnu
```

Pada proses kompilasi, dilakukan penyesuaian kompatibilitas terhadap source code Barnyard2, yaitu penghapusan deklarasi `typedef int SOCKET;` yang bertentangan dengan definisi tipe pada header sistem serta penyediaan nama header `dnet.h` melalui symbolic link ke `dumbnet.h`.

Setelah dependensi dan konfigurasi terpenuhi, lakukan kompilasi:

```bash
make
```

Jika kompilasi berhasil, pasang Barnyard2:

```bash
sudo make install
```

Verifikasi hasil instalasi dan versi Barnyard2:

```bash
barnyard2 -V
```

Hasil verifikasi menunjukkan:

```text
Barnyard2
Version 2.1.14 (Build 337)
By Ian Firns (SecurixLive)
```

Executable Barnyard2 terpasang pada:

```text
/usr/local/bin/barnyard2
```

Dengan demikian, instalasi Barnyard2 berhasil dilakukan melalui source code meskipun paketnya tidak tersedia pada repositori APT.

Konfigurasi integrasi Barnyard2 dengan output Snort dan database dijelaskan pada bagian D.7.

<img width="1283" height="300" alt="image" src="https://github.com/user-attachments/assets/f3466dbb-34f8-41d9-b50c-d287be7382d8" />

> Status komponen harus dicatat berdasarkan pemeriksaan aktual. Keberhasilan instalasi Barnyard2 belum membuktikan bahwa integrasi Snort, database, dan BASE telah berhasil.

### D.4 Konfigurasi Snort dan Penyiapan Rules

Setelah Snort dan komponen pendukung disiapkan, tahap berikutnya adalah mengonfigurasi Snort agar dapat memantau jaringan laboratorium dan menggunakan rules yang sesuai dengan kebutuhan pengujian.

#### 1. Pemeriksaan direktori konfigurasi

Periksa direktori konfigurasi Snort:

```bash
ls -l /etc/snort/
```

Buat salinan cadangan file konfigurasi sebelum melakukan perubahan:

```bash
sudo cp /etc/snort/snort.conf /etc/snort/snort.conf.backup
```

Buka file konfigurasi:

```bash
sudo nano /etc/snort/snort.conf
```

> Lokasi file konfigurasi dapat berbeda bergantung pada metode instalasi dan versi Snort yang digunakan. Gunakan lokasi aktual pada sistem.

#### 2. Konfigurasi HOME_NET

HOME_NET merupakan variabel yang digunakan Snort untuk menentukan jaringan internal yang dipantau.

Sesuaikan nilai HOME_NET dengan subnet laboratorium:

```text
192.168.56.0/24
```

Penetapan tersebut sesuai dengan jaringan LAN Segment 1 yang menghubungkan Kali Linux sebagai attacker dan Linux Mint sebagai target sekaligus sensor IDS.

<img width="1278" height="292" alt="Konfigurasi HOME_NET" src="https://github.com/user-attachments/assets/ca8c349c-26c3-47de-9ecf-72a0a6133e1e" />

#### 3. Pemeriksaan konfigurasi rules

Pada file `snort.conf`, periksa bagian konfigurasi rules dan baris `include` yang digunakan untuk memuat file rules.

Baris konfigurasi yang aktif digunakan untuk memanggil file rules, sedangkan baris yang diawali karakter `#` merupakan komentar sehingga tidak dijalankan sebagai perintah konfigurasi.

Pastikan file rules yang diperlukan untuk skenario penelitian telah dimasukkan sesuai lokasi direktori yang tersedia.

<img width="1281" height="712" alt="Konfigurasi include rules bagian pertama" src="https://github.com/user-attachments/assets/207c132c-6413-472b-ac7c-3c7dfa3b50c8" />

<img width="1281" height="633" alt="Konfigurasi include rules bagian kedua" src="https://github.com/user-attachments/assets/047c8714-3800-463b-94d9-2d2c14754a89" />

<img width="1281" height="634" alt="Konfigurasi include rules bagian ketiga" src="https://github.com/user-attachments/assets/4f8bd269-b795-474b-9508-801c54e562ab" />

#### 4. Pemeriksaan direktori rules

Periksa daftar file rules yang tersedia:

```bash
ls -l /etc/snort/rules/
```

Pemeriksaan ini bertujuan mengetahui file rules yang tersedia pada direktori Snort.

Dokumentasikan informasi berikut:

- Nama file rules.
- Sumber rules.
- Versi atau tanggal pembaruan, jika diketahui.
- File rules yang dimuat melalui konfigurasi `snort.conf`.
- Tujuan penggunaan rules dalam skenario pengujian.

Perlu diperhatikan bahwa keberadaan file dalam direktori `/etc/snort/rules/` tidak otomatis berarti file tersebut aktif. File harus dipanggil melalui konfigurasi `include`. Selain itu, aturan individual di dalam file juga dapat dinonaktifkan melalui komentar.

Jika menggunakan rules khusus, dokumentasikan isi dan tujuan rules tersebut. Jangan menyimpulkan suatu rules mampu mendeteksi serangan sebelum dilakukan pengujian.

<img width="1281" height="760" alt="Daftar file rules Snort bagian pertama" src="https://github.com/user-attachments/assets/798a49d7-a2c8-4bcd-a6ff-3533af367e1c" />

<img width="1282" height="577" alt="Daftar file rules Snort bagian kedua" src="https://github.com/user-attachments/assets/97937888-7144-43d2-8566-ac984e7497f9" />

### D.5 Pengujian Konfigurasi Snort

Setelah konfigurasi jaringan dan rules selesai disiapkan, lakukan pemeriksaan untuk memastikan Snort dapat membaca file konfigurasi dan rules tanpa kesalahan fatal.

Jalankan perintah:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

Opsi `-T` digunakan untuk menguji konfigurasi tanpa menjalankan pemantauan lalu lintas secara normal.

Perhatikan pesan yang ditampilkan pada terminal.

Jika pengujian gagal:

1. Periksa pesan kesalahan yang ditampilkan.
2. Periksa sintaks pada `snort.conf`.
3. Periksa lokasi dan nama file rules.
4. Periksa konfigurasi HOME_NET.
5. Periksa dependensi dan kompatibilitas rules.
6. Perbaiki kesalahan, kemudian ulangi pengujian.

Jika pengujian berhasil, konfigurasi Snort dinyatakan lolos pemeriksaan dan implementasi dapat dilanjutkan ke tahap instalasi BASE.

<img width="1283" height="746" alt="image" src="https://github.com/user-attachments/assets/8ef7bb15-c1da-4ce0-a6bb-850a08cc172a" />


### D.6 Instalasi BASE

BASE (Basic Analysis and Security Engine) merupakan antarmuka berbasis web yang digunakan untuk menampilkan dan memeriksa alert Snort melalui browser.

Materi acuan menggunakan BASE versi 1.4.5. Oleh karena itu, versi tersebut dijadikan acuan dalam pengembangan implementasi ini.

Sebelum instalasi, periksa ketersediaan sumber BASE dan dependensi yang diperlukan. Pastikan versi PHP, database, serta Apache2 yang digunakan sesuai dengan kebutuhan aplikasi.

Jika BASE 1.4.5 tidak tersedia melalui repositori, metode instalasi harus mengikuti sumber distribusi atau dokumentasi versi yang digunakan. Catat sumber, versi, dan penyesuaian dependensi yang dilakukan.

Jangan menganggap BASE telah terinstal hanya karena Apache2, PHP, dan database sudah berjalan.

> Apabila BASE 1.4.5 mengalami kendala kompatibilitas dengan Linux Mint 22.3, dokumentasikan kendala tersebut. Jangan mengganti BASE dengan dashboard lain tanpa menjelaskan bahwa terdapat perubahan dari komponen pada materi acuan.

### D.7 Konfigurasi dan Pengujian Web BASE

Setelah BASE disiapkan, lakukan konfigurasi agar aplikasi dapat terhubung ke database dan menampilkan data alert yang dihasilkan Snort.

#### 1. Persiapan database

Siapkan database khusus untuk menyimpan data alert. Konfigurasikan nama database, akun pengguna, kata sandi, dan hak akses sesuai kebutuhan instalasi.

Pastikan database dapat diakses oleh komponen yang digunakan. Jangan mencantumkan kata sandi atau informasi autentikasi asli ke dalam README publik.

#### 2. Konfigurasi Barnyard2

Sesuaikan konfigurasi Barnyard2 dengan output Snort dan database yang telah disiapkan.

Pastikan parameter berikut sesuai dengan kondisi aktual:

- Lokasi output atau spool Snort.
- Format output yang digunakan Snort.
- Lokasi file konfigurasi Barnyard2.
- Nama database.
- Akun pengguna database.
- Hak akses koneksi database.

Setelah konfigurasi selesai, jalankan Barnyard2 sesuai metode pengoperasian yang digunakan dan periksa apakah proses dapat membaca output Snort serta meneruskan data ke database.

#### 3. Konfigurasi BASE

Sesuaikan konfigurasi BASE agar dapat terhubung ke database yang telah disiapkan.

Periksa kembali:

- Alamat server database.
- Nama database.
- Akun pengguna database.
- Hak akses database.
- Konfigurasi Apache2 dan PHP.
- Lokasi direktori aplikasi BASE.

#### 4. Pengujian antarmuka Web BASE

Pastikan layanan Apache2 dan database berjalan. Kemudian akses alamat Web BASE melalui browser sesuai konfigurasi Apache2.

Lakukan pemeriksaan berikut:

1. Halaman BASE dapat diakses.
2. BASE dapat terhubung ke database.
3. Data alert dapat dibaca.
4. Alert Snort dari pengujian dapat ditampilkan pada halaman BASE.

Apabila halaman BASE tidak dapat diakses atau alert tidak tampil, periksa konfigurasi Apache2, PHP, database, Barnyard2, output Snort, dan hak akses yang digunakan.

Keberhasilan instalasi BASE tidak hanya ditentukan oleh halaman web yang dapat dibuka, tetapi juga oleh keberhasilan integrasi data alert Snort hingga tampil pada antarmuka BASE.

**Dokumentasi:**

- Screenshot konfigurasi database BASE.
- Screenshot konfigurasi Barnyard2.
- Screenshot halaman Web BASE.
- Screenshot alert Snort yang berhasil ditampilkan pada BASE, jika pengujian berhasil.

### D.8 Implementasi Suricata sebagai Pengembangan

Suricata ditambahkan setelah implementasi dasar Snort untuk memperluas penelitian menjadi analisis perbandingan. Penambahan ini bukan bagian dari komponen utama pada materi acuan.

Periksa paket:

```bash
apt-cache policy suricata
```

Instal:

```bash
sudo apt install -y suricata
```

Periksa versi:

```bash
suricata -V
```

Cadangkan konfigurasi:

```bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.backup
```

Buka konfigurasi:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Sesuaikan HOME_NET dan interface sesuai jaringan laboratorium. Interface pengujian adalah `ens37`.

Periksa dan perbarui rules jika `suricata-update` tersedia:

```bash
suricata-update --version
sudo suricata-update
```

Uji konfigurasi:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Periksa opsi yang tersedia:

```bash
suricata --help
```

Jalankan dalam mode IDS/passive detection. Catat versi, sumber rules, HOME_NET, interface, dan lokasi log.

**Dokumentasi:**

- Screenshot versi dan konfigurasi Suricata.
- Screenshot hasil pengujian konfigurasi Suricata.

### D.9 Kriteria Penyelesaian Implement

- Snort telah terpasang dan versinya tercatat.
- Komponen pendukung Snort telah diperiksa dan status aktualnya dicatat.
- HOME_NET dan konfigurasi rules telah disesuaikan dengan jaringan laboratorium.
- Konfigurasi Snort lolos pemeriksaan atau kendalanya telah didokumentasikan.
- BASE dan integrasinya dengan Snort telah diuji, atau kendala kompatibilitasnya telah dicatat.
- Suricata telah disiapkan sebagai komponen pengembangan dan konfigurasi telah diuji.

### D.10 Implementasi Suricata sebagai pengembangan

Suricata ditambahkan setelah implementasi dasar Snort untuk memperluas penelitian menjadi analisis perbandingan. Penambahan ini bukan bagian dari komponen utama pada materi acuan.

Periksa paket:

```bash
apt-cache policy suricata
```

Instal:

```bash
sudo apt install -y suricata
```

Periksa versi:

```bash
suricata -V
```

Cadangkan konfigurasi:

```bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.backup
```

Buka konfigurasi:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Sesuaikan HOME_NET dan interface sesuai jaringan laboratorium. Interface pengujian adalah `ens37`.

Periksa dan perbarui rules jika `suricata-update` tersedia:

```bash
suricata-update --version
sudo suricata-update
```

Uji konfigurasi:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Periksa opsi yang tersedia:

```bash
suricata --help
```

Jalankan dalam mode IDS/passive detection. Catat versi, sumber rules, HOME_NET, interface, dan lokasi log.

<!-- SS D.16 — Versi dan konfigurasi Suricata -->
<!-- SS D.17 — Hasil pengujian konfigurasi Suricata -->

### D.11 Kriteria penyelesaian Implement

- Snort telah terpasang dan versinya tercatat.
- Rules Snort telah ditentukan dan didokumentasikan.
- Konfigurasi Snort lolos pemeriksaan.
- Komponen pendukung Web BASE telah diperiksa dan status aktualnya dicatat.
- Snort dapat dijalankan pada interface pengujian.
- Suricata telah disiapkan sebagai komponen pengembangan dan konfigurasi telah diuji.

------------------------------------------------------------------------

## E. Operate — Pengoperasian dan Pengujian

### E.1 Tujuan

Menjalankan sistem yang telah dibangun, menguji fungsinya, mengidentifikasi kendala, dan memastikan sistem bekerja sesuai tujuan penelitian. Tahap ini merupakan pelaksanaan skenario Remote Exploit dan pengamatan hasil deteksi.

### E.2 Persiapan sebelum pengujian

Sebelum setiap percobaan:

1. Pastikan Kali Linux dan Linux Mint terhubung melalui LAN Segment 1.
2. Pastikan layanan target aktif dan siap menerima koneksi.
3. Pastikan hanya IDS yang sedang diuji berjalan.
4. Pastikan interface pemantauan adalah `ens37`.
5. Pastikan konfigurasi dan rules telah lolos pemeriksaan.
6. Catat waktu sistem kedua VM.
7. Siapkan pencatatan alert dan file PCAP.
8. Catat kondisi awal CPU dan RAM.

### E.3 Pengoperasian Snort

Aktifkan Snort pada Linux Mint dengan mode IDS/passive detection. Perintah harus disesuaikan dengan opsi versi yang terpasang.

Pastikan proses berjalan dan output alert tercatat. Jika Web BASE berhasil dikonfigurasi, periksa apakah alert yang dihasilkan dapat ditampilkan pada halaman tersebut.

<!-- SS E.1 — Snort aktif pada interface pengujian -->
<!-- SS E.2 — Alert Snort pada log atau Web BASE -->

### E.4 Simulasi Remote Exploit menggunakan Metasploit

Dari Kali Linux, jalankan skenario Remote Exploit yang telah ditentukan terhadap layanan target milik sendiri di Linux Mint.

Skenario dan parameter harus dicatat sebelum pengujian. Gunakan target, layanan, port, dan parameter yang sama untuk pengujian Snort maupun Suricata.

Catatan skenario:

| Parameter                 | Hasil aktual    |
|:--------------------------|:----------------|
| ID pengujian              |                 |
| Tanggal dan waktu         |                 |
| Modul/skenario Metasploit |                 |
| Payload                   |                 |
| IP attacker               | `192.168.56.10` |
| IP target                 | `192.168.56.20` |
| Port target               |                 |
| Status eksekusi skenario  |                 |
| Bukti koneksi/hasil       |                 |

Lakukan pengujian hanya pada target laboratorium yang dikendalikan sendiri. Jangan mengarahkan modul eksploitasi ke sistem publik atau pihak lain.

<!-- SS E.3 — Skenario Metasploit pada Kali Linux -->
<!-- SS E.4 — Status hasil skenario pengujian -->

### E.5 Rekam paket jaringan

Pada Linux Mint, rekam lalu lintas di interface LAN:

```bash
sudo tcpdump -i ens37 -nn -s0 -w snort-test-01.pcap
```

Jalankan skenario dari Kali saat perekaman berlangsung. Setelah skenario selesai, hentikan tcpdump dengan `Ctrl+C`.

Buat nama file berbeda untuk setiap percobaan:

```text
snort-test-01.pcap
snort-test-02.pcap
snort-test-03.pcap
```

<!-- SS E.5 — tcpdump merekam paket pada ens37 -->

### E.6 Pengujian lalu lintas normal

Selain skenario serangan, jalankan aktivitas normal yang telah ditentukan. Pengujian ini digunakan untuk mengamati alert yang muncul ketika tidak ada skenario serangan.

Catat jenis aktivitas, durasi, jumlah alert, dan alert yang tidak relevan dengan aktivitas normal.

<!-- SS E.6 — Aktivitas lalu lintas normal -->
<!-- SS E.7 — Hasil alert pada lalu lintas normal -->

### E.7 Pencatatan resource ketika Snort aktif

Pantau CPU dan RAM pada Linux Mint:

```bash
htop
```

CPU:

```bash
sar -u 1
```

Memori:

```bash
sar -r 1
```

Gunakan durasi dan interval pencatatan yang sama untuk pengujian Suricata.

<!-- SS E.8 — Resource ketika Snort aktif -->

### E.8 Pengulangan pengujian menggunakan Suricata

Setelah seluruh pengujian Snort selesai dan hasilnya disimpan:

1. Hentikan Snort.
2. Pastikan Snort tidak lagi memproses lalu lintas.
3. Aktifkan Suricata pada interface `ens37`.
4. Pastikan Suricata menghasilkan log.
5. Jalankan skenario Metasploit yang sama.
6. Rekam paket dengan tcpdump.
7. Jalankan aktivitas normal yang sama.
8. Catat CPU dan RAM dengan metode yang sama.
9. Simpan seluruh hasil dengan ID percobaan yang sepadan.

Contoh file PCAP:

```text
suricata-test-01.pcap
suricata-test-02.pcap
suricata-test-03.pcap
```

<!-- SS E.9 — Snort telah dihentikan -->
<!-- SS E.10 — Suricata aktif pada ens37 -->
<!-- SS E.11 — Alert Suricata -->
<!-- SS E.12 — Resource ketika Suricata aktif -->

### E.9 Format pencatatan setiap percobaan

| Parameter           | Snort | Suricata |
|:--------------------|:------|:---------|
| ID pengujian        |       |          |
| Skenario            |       |          |
| Waktu mulai         |       |          |
| Waktu alert pertama |       |          |
| Status skenario     |       |          |
| Alert relevan       |       |          |
| Alert tidak relevan |       |          |
| Rata-rata CPU       |       |          |
| Puncak CPU          |       |          |
| Rata-rata RAM       |       |          |
| Nama file PCAP      |       |          |
| Catatan             |       |          |

### E.10 Kriteria penyelesaian Operate

- Skenario Metasploit berhasil dijalankan atau status kegagalannya terdokumentasi.
- Alert Snort dan Suricata telah dicatat secara terpisah.
- PCAP untuk setiap sesi tersedia.
- Aktivitas normal telah diuji.
- Penggunaan CPU dan RAM telah dicatat dengan metode yang sama.
- Semua bukti diberi nama berdasarkan ID pengujian.

------------------------------------------------------------------------

## F. Optimize — Analisis, Evaluasi, dan Optimasi

### F.1 Tujuan

Menganalisis data hasil pengoperasian, mengevaluasi kinerja sistem, mengidentifikasi bagian yang perlu diperbaiki, dan menentukan optimasi yang relevan. Kesimpulan harus didasarkan pada hasil aktual.

### F.2 Analisis awal file PCAP di Wireshark

Buka file PCAP hasil pengujian Snort dan Suricata menggunakan Wireshark.

Mulai dengan melihat ringkasan file:

**Statistics → Capture File Properties**

Catat informasi seperti jumlah paket dan rentang waktu perekaman.

<!-- SS F.1 — Capture File Properties -->

### F.3 Analisis Conversations

Buka:

**Statistics → Conversations**

Amati komunikasi antar-host, jumlah paket, alamat sumber dan tujuan, serta volume data. Identifikasi komunikasi antara Kali Linux (`192.168.56.10`) dan Linux Mint (`192.168.56.20`).

<!-- SS F.2 — Statistics Conversations -->

### F.4 Analisis Protocol Hierarchy

Buka:

**Statistics → Protocol Hierarchy**

Amati protokol yang muncul pada file PCAP. Gunakan hasil aktual untuk menentukan protokol yang perlu dianalisis lebih lanjut. Jangan mengasumsikan protokol atau port sebelum memeriksa hasil capture.

<!-- SS F.3 — Statistics Protocol Hierarchy -->

### F.5 Pemeriksaan paket dan komunikasi

Periksa informasi berikut:

- IP sumber dan IP tujuan.
- Port sumber dan port tujuan.
- Protokol.
- Waktu paket.
- Urutan komunikasi.
- Flag dan informasi TCP yang relevan.
- Pola komunikasi yang berkaitan dengan skenario Metasploit.

Gunakan display filter Wireshark sesuai protokol dan alamat yang benar-benar ditemukan pada PCAP.

<!-- SS F.4 — Detail paket komunikasi attacker dan target -->

### F.6 Follow TCP Stream dan analisis HTTP (jika tersedia)

Jika PCAP berisi komunikasi TCP yang relevan, pilih paket lalu gunakan:

**Follow → TCP Stream**

Fitur ini membantu melihat urutan komunikasi dalam satu aliran TCP.

Jika skenario menghasilkan lalu lintas HTTP, lakukan pemeriksaan tambahan:

- Filter HTTP.
- Periksa header `Host`.
- Periksa metode dan URI permintaan, misalnya GET.
- Periksa respons server.
- Cocokkan waktu permintaan dan respons.

Langkah analisis HTTP hanya dilakukan jika protokol tersebut memang terdapat pada PCAP. Jangan memaksakan analisis HTTP pada skenario yang tidak menghasilkan lalu lintas HTTP.

<!-- SS F.5 — Follow TCP Stream -->
<!-- SS F.6 — Pemeriksaan HTTP apabila tersedia -->

### F.7 Ekspor objek HTTP dan pemeriksaan file (jika tersedia)

Jika terdapat objek HTTP yang dapat diekspor, gunakan:

**File → Export Objects → HTTP**

Simpan objek yang relevan untuk pemeriksaan lebih lanjut. Hitung hash SHA-256 jika terdapat file hasil ekspor:

```bash
sha256sum nama-file
```

Hash dapat diperiksa menggunakan layanan reputasi seperti VirusTotal sesuai kebijakan dan kebutuhan penelitian. Jangan mengunggah data privat atau file yang bukan milik peneliti.

Analisis ekspor objek, hash, dan reputasi file merupakan langkah lanjutan yang hanya relevan jika skenario menghasilkan objek file. Pada skenario reverse shell yang tidak membawa objek HTTP, bagian ini dicatat sebagai tidak berlaku.

<!-- SS F.7 — Export Objects HTTP jika tersedia -->
<!-- SS F.8 — Hash SHA-256 jika ada objek file -->

### F.8 Korelasi alert dengan PCAP

Cocokkan waktu alert Snort atau Suricata dengan paket pada PCAP.

Catat:

|                                                   |
|:--------------------------------------------------|
| ID IDS Waktu Waktu IP sumber IP tujuan Kesesuaian |
| paket alert                                       |

------------------------------------------------------------------------

Korelasi digunakan untuk memastikan alert berkaitan dengan aktivitas pengujian yang diamati. Alert tidak langsung dianggap benar hanya karena muncul pada waktu yang berdekatan; periksa isi paket dan konteks skenario.

<!-- SS F.9 — Korelasi alert dengan paket Wireshark -->

### F.9 Perbandingan hasil deteksi

| ID  | Skenario | Snort | Suricata | Keterangan |
|:----|:---------|:------|:---------|:-----------|
| 1   |          |       |          |            |
| 2   |          |       |          |            |
| 3   |          |       |          |            |

Keterangan:

- **Terdeteksi:** terdapat alert yang relevan dan didukung bukti lalu lintas.
- **Tidak terdeteksi:** tidak terdapat alert relevan pada sesi yang valid.
- **Tidak valid:** skenario gagal dijalankan atau bukti tidak cukup untuk evaluasi.

### F.10 Perbandingan waktu deteksi

| ID  | Skenario | Waktu alert Snort | Waktu alert Suricata |
|:----|:---------|------------------:|---------------------:|
| 1   |          |                   |                      |
| 2   |          |                   |                      |
| 3   |          |                   |                      |

Titik awal pengukuran harus konsisten, yaitu waktu aktivitas uji dimulai hingga alert relevan pertama dihasilkan. Jelaskan cara memperoleh timestamp dan satuan waktu yang digunakan.

### F.11 Perbandingan penggunaan resource

| IDS      | Rata-rata CPU | Puncak CPU | Rata-rata RAM |
|:---------|--------------:|-----------:|--------------:|
| Snort    |               |            |               |
| Suricata |               |            |               |

Gunakan interval sampling, durasi, spesifikasi VM, dan beban pengujian yang sama.

### F.12 Perhitungan Detection Rate

Detection Rate menghitung persentase skenario serangan valid yang berhasil dideteksi.

```text
Detection Rate =
(Jumlah skenario serangan valid yang terdeteksi /
 Jumlah seluruh skenario serangan valid) x 100%
```

### F.13 Evaluasi false positive

False positive adalah alert yang muncul pada aktivitas normal tetapi tidak didukung bukti bahwa aktivitas tersebut merupakan serangan dalam skenario penelitian.

Catat jumlah false positive dan definisikan unit penghitungan secara konsisten. Jika menghitung false positive rate, tentukan terlebih dahulu unit evaluasi dan penyebutnya. Jangan mencampur jumlah alert dengan jumlah aktivitas normal tanpa definisi.

### F.14 Evaluasi dan optimasi

Berdasarkan hasil analisis:

1. Identifikasi skenario yang tidak terdeteksi.
2. Periksa rule yang digunakan dan kesesuaiannya dengan lalu lintas.
3. Periksa konfigurasi HOME_NET dan interface pemantauan.
4. Periksa apakah alert sesuai dengan paket pada PCAP.
5. Identifikasi penggunaan CPU atau RAM yang tinggi.
6. Tentukan perubahan konfigurasi atau rules yang diperlukan.
7. Jika konfigurasi diubah, ulangi pengujian dengan skenario dan kondisi yang sama.
8. Catat hasil sebelum dan sesudah optimasi.

Optimasi tidak boleh dilakukan dengan mengubah banyak variabel sekaligus tanpa pencatatan. Tujuannya adalah meningkatkan fungsi sistem berdasarkan temuan pengujian, bukan membuat salah satu IDS terlihat lebih unggul.

### F.15 Ringkasan evaluasi

| Parameter                    | Snort | Suricata | Hasil pengamatan |
|:-----------------------------|:------|:---------|:-----------------|
| Detection Rate               |       |          |                  |
| Waktu deteksi                |       |          |                  |
| False positive               |       |          |                  |
| Rata-rata CPU                |       |          |                  |
| Rata-rata RAM                |       |          |                  |
| Kesesuaian alert dengan PCAP |       |          |                  |

<!-- SS F.10 — Ringkasan alert Snort -->
<!-- SS F.11 — Ringkasan alert Suricata -->
<!-- SS F.12 — Tabel/grafik perbandingan -->

### F.16 Kriteria penyelesaian Optimize

- File PCAP telah dianalisis.
- Alert telah dikorelasikan dengan bukti paket.
- Hasil Snort dan Suricata telah dibandingkan.
- Parameter deteksi dan resource telah dievaluasi.
- Perbaikan yang dilakukan dan hasilnya telah dicatat.
- Kesimpulan tidak melampaui bukti yang diperoleh.

------------------------------------------------------------------------

# 3. Struktur Penyimpanan Hasil Penelitian

Simpan bukti setiap tahap agar hasil tidak tertukar.

```text
hasil-penelitian/
├── prepare-plan/
│   ├── spesifikasi-vm/
│   └── konfigurasi-jaringan/
├── design/
│   └── topologi/
├── snort/
│   ├── konfigurasi/
│   ├── rules/
│   ├── pcap/
│   ├── alert/
│   ├── resource/
│   └── catatan-pengujian/
├── suricata/
│   ├── konfigurasi/
│   ├── rules/
│   ├── pcap/
│   ├── alert/
│   ├── resource/
│   └── catatan-pengujian/
└── optimize/
    ├── analisis-wireshark/
    ├── perbandingan/
    └── evaluasi/
```

Gunakan ID percobaan yang sama untuk kedua IDS, misalnya `test-01`, agar hasilnya mudah dibandingkan.

------------------------------------------------------------------------

# 4. Troubleshooting

## 4.1 Linux Mint tidak mendapat internet

Periksa interface dan routing:

```bash
ip -br addr
ip route
```

Pastikan Adapter 1 menggunakan NAT dan memperoleh alamat serta default gateway.

Uji konektivitas:

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

## 4.2 Kali tidak dapat terhubung ke Linux Mint

Periksa:

```bash
ip -br addr
```

Pastikan:

- Kali menggunakan `192.168.56.10/24` pada `eth1`.
- Linux Mint menggunakan `192.168.56.20/24` pada `ens37`.
- Kedua VM menggunakan LAN Segment 1 yang sama.
- Interface LAN dalam keadaan aktif.

## 4.3 APT mengalami error repositori CD-ROM

Periksa **Software Sources**. Pastikan repositori CD-ROM/ISO tidak aktif setelah instalasi permanen. Kemudian:

```bash
sudo apt update
```

## 4.4 Snort gagal membaca konfigurasi

Periksa versi dan konfigurasi:

```bash
snort -V
ls -l /etc/snort/
sudo snort -T -c /etc/snort/snort.conf
```

Periksa pesan error, sintaks, lokasi rules, dan HOME_NET.

## 4.5 Suricata gagal membaca konfigurasi

```bash
suricata -V
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Periksa pesan error dan pastikan konfigurasi sesuai versi Suricata.

## 4.6 Tidak ada alert IDS

Periksa:

1. IDS memantau `ens37`.
2. Lalu lintas benar-benar melewati LAN Segment 1.
3. Rule yang sesuai telah dimuat.
4. HOME_NET sesuai jaringan.
5. Skenario uji menghasilkan lalu lintas yang dimaksud.
6. Log yang diperiksa berasal dari sesi pengujian yang benar.
7. Waktu sistem dan timestamp telah diperiksa.

## 4.7 Alert tidak muncul pada BASE

Periksa status Apache2, database, Barnyard2, konfigurasi output Snort, koneksi database, dan konfigurasi BASE. Pastikan aliran output Snort sampai ke database sebelum menyimpulkan bahwa tampilan BASE bermasalah.

------------------------------------------------------------------------

# 5. Ringkasan Hubungan README dengan Materi Acuan

| Tahap PPDIOO | Prosedur yang diacu dari materi                                                                             | Implementasi pada README                                                                                       |
|:-------------|:------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------|
| Prepare      | Pengumpulan informasi, analisis kebutuhan, konfigurasi awal, identifikasi masalah                           | Pemeriksaan VM, OS, resource, utilitas, dan konektivitas                                                       |
| Plan         | Menentukan hardware dan software                                                                            | Daftar perangkat, versi, fungsi, serta rencana pengujian                                                       |
| Design       | Merancang topologi jaringan                                                                                 | Topologi VMware, interface, IP, target, dan alur sistem                                                        |
| Implement    | Instalasi Snort, rules, Apache2/PHP/MySQL/Barnyard2, konfigurasi Snort, uji, instalasi dan konfigurasi BASE | Prosedur implementasi Snort dan komponen pendukung; Suricata ditambahkan sebagai pengembangan                  |
| Operate      | Menjalankan sistem dan melakukan pengujian                                                                  | Simulasi Remote Exploit dengan Metasploit, pencatatan alert, perekaman paket, dan pengujian lalu lintas normal |
| Optimize     | Analisis data, evaluasi, dan peningkatan kinerja                                                            | Analisis Wireshark, korelasi alert-PCAP, perbandingan, evaluasi, dan optimasi                                  |

------------------------------------------------------------------------

# 6. Kesimpulan Penelitian

Bagian ini diisi setelah seluruh tahapan PPDIOO selesai. Kesimpulan harus menjawab tujuan penelitian berdasarkan hasil aktual.

Aspek yang dibahas:

1. Keberhasilan implementasi Snort sebagai IDS.
2. Kemampuan Snort mendeteksi skenario Remote Exploit.
3. Hasil analisis paket menggunakan Wireshark.
4. Hasil pengujian Suricata sebagai pengembangan penelitian.
5. Perbedaan hasil deteksi dan waktu alert.
6. Perbedaan penggunaan CPU dan RAM.
7. Keterbatasan implementasi, termasuk kompatibilitas komponen Web BASE jika ditemukan.
8. Perbaikan yang dilakukan pada tahap Optimize.

------------------------------------------------------------------------

# 7. Referensi

## Materi utama

- Asri Yohana Sirait, Rivka Salvita Silalahi, dan Samuel Volder Naibaho, “Penerapan Intrusion Detection Dan Prevention System Untuk Mendeteksi Serangan Metasploit Menggunakan Snort dan Wireshark,” *Jurnal Quancom*, Vol. 3, No. 2, Desember 2025, hlm. 9–18. DOI: [10.62375/jqc.v3i2.428](https://doi.org/10.62375/jqc.v3i2.428).

## Dokumentasi teknis pendukung

- Snort Documentation: https://docs.snort.org/
- Suricata Documentation: https://docs.suricata.io/
- Wireshark User’s Guide: https://www.wireshark.org/docs/
- Metasploit Documentation: https://docs.metasploit.com/
- Linux Mint: https://linuxmint.com/
- Ubuntu Documentation: https://documentation.ubuntu.com/

> **Catatan akhir:** README ini adalah panduan pengembangan implementasi berdasarkan materi acuan. Metodologi PPDIOO dan urutan prosedur utama dipertahankan. Penyesuaian dilakukan pada lingkungan virtual yang benar-benar digunakan, sedangkan pengujian Suricata, pengukuran resource, dan perbandingan kinerja merupakan pengembangan tambahan. Seluruh versi, konfigurasi, hasil alert, waktu deteksi, dan kesimpulan harus diisi berdasarkan bukti aktual.
