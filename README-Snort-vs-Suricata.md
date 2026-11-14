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
| VMware Workstation | Versi terpasang dicatat saat penelitian | Menjalankan VM        |
| Linux Mint         | 22.3 (Zena)                             | Target dan sensor IDS |
| Kali Linux         | Versi terpasang dicatat saat penelitian | Attacker              |
| Suricata           | Versi terpasang dicatat saat penelitian | IDS pembanding        |
| tcpdump            | Versi paket terpasang                   | Merekam PCAP          |
| Wireshark          | Versi terpasang dicatat saat penelitian | Analisis paket        |

### B.3 Rencana perangkat keras/virtual

| Perangkat  | Peran               | IP jaringan pengujian |
|:-----------|:--------------------|:----------------------|
| Linux Mint | Server/target + IDS | `192.168.56.20/24`    |
| Kali Linux | Penyerang           | `192.168.56.10/24`    |

Spesifikasi vCPU, RAM, dan disk masing-masing VM harus dicatat dari konfigurasi VMware. Jangan mengisi spesifikasi yang belum diperiksa.

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

<!-- SS C.1 — Konfigurasi Network Adapter Kali Linux di VMware -->
<!-- SS C.2 — Konfigurasi Network Adapter Linux Mint di VMware -->
<!-- SS C.3 — Topologi jaringan LAN Segment 1 -->

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

<!-- SS C.4 — Output ip -br addr dan ip route Kali Linux -->

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

<!-- SS C.5 — Output ip -br addr dan ip route Linux Mint -->

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

Menerapkan seluruh komponen sesuai rencana dan desain. Tahap implementasi mengikuti alur pada materi: instalasi Snort, penambahan rules, instalasi komponen pendukung, konfigurasi Snort, pengujian konfigurasi, instalasi BASE, konfigurasi BASE, dan pemeriksaan tampilan alert.

> **Catatan kompatibilitas:** materi acuan menggunakan kombinasi Snort 2.9.20, Barnyard2, dan BASE 1.4.5. Komponen tersebut merupakan stack yang lebih lama. Pada Linux Mint 22.3, kompatibilitas paket dan dependensi harus diperiksa. Catat error dan versi aktual; jangan menyatakan BASE berhasil sebelum antarmukanya benar-benar dapat menampilkan alert.

### D.2 Instalasi Snort

Periksa paket yang tersedia:

```bash
apt-cache policy snort
```

Instal Snort apabila paket tersedia dan versi/dependensinya sesuai dengan rancangan:

```bash
sudo apt install -y snort
```

Verifikasi versi:

```bash
snort -V
```

Catat versi Snort yang benar-benar terpasang. Jika versi berbeda dari 2.9.20 yang dicantumkan materi, tuliskan perbedaannya pada dokumentasi.

<!-- SS D.1 — Pemeriksaan paket Snort -->
<!-- SS D.2 — Instalasi Snort -->
<!-- SS D.3 — Versi Snort -->

### D.3 Konfigurasi Snort dan HOME_NET

Periksa file konfigurasi:

```bash
ls -l /etc/snort/
```

Buat salinan konfigurasi sebelum perubahan:

```bash
sudo cp /etc/snort/snort.conf /etc/snort/snort.conf.backup
```

Buka konfigurasi:

```bash
sudo nano /etc/snort/snort.conf
```

Sesuaikan konfigurasi jaringan internal/HOME_NET dengan jaringan laboratorium. Untuk rancangan ini, jaringan pengujian adalah:

```text
192.168.56.0/24
```

Periksa juga direktori rules dan pastikan rule yang diperlukan dimuat oleh konfigurasi.

> Lokasi file dan sintaks dapat berbeda menurut versi Snort. Gunakan file konfigurasi yang benar-benar tersedia pada sistem.

<!-- SS D.4 — Konfigurasi HOME_NET Snort -->
<!-- SS D.5 — Konfigurasi rule Snort -->

### D.4 Penambahan dan pemeriksaan Rules Snort

Materi menyebutkan penambahan rules sebagai bagian dari konfigurasi Snort. Periksa rule yang tersedia:

```bash
ls -l /etc/snort/rules/
```

Dokumentasikan:

- Nama file rule.
- Sumber rule.
- Versi atau tanggal pembaruan.
- Rule yang diaktifkan.
- Tujuan rule dalam skenario pengujian.

Jika menggunakan rule khusus, simpan isi rule dan jelaskan pola lalu lintas yang hendak dideteksi. Jangan mengklaim rule mendeteksi serangan sebelum diuji.

<!-- SS D.6 — Direktori dan rule Snort yang digunakan -->

### D.5 Instalasi komponen pendukung Web BASE

Mengikuti diagram alir materi, komponen pendukung yang disiapkan adalah Apache2, PHP, MySQL, dan Barnyard2. BASE digunakan untuk menampilkan alert Snort melalui antarmuka web.

Periksa ketersediaan paket terlebih dahulu:

```bash
apt-cache policy apache2 php mysql-server mariadb-server
```

Instal komponen web dan database yang tersedia serta sesuai dengan kebutuhan BASE:

```bash
sudo apt install -y apache2 php mysql-server
```

Jika paket `mysql-server` tidak tersedia pada repositori, periksa alternatif database yang disediakan distribusi dan dokumentasikan pilihan aktual. Jangan mengganti komponen tanpa mencatat perbedaannya.

Periksa layanan:

```bash
systemctl status apache2
systemctl status mysql
```

> Instalasi Barnyard2 dan BASE 1.4.5 dapat memerlukan dependensi serta konfigurasi tambahan. Ikuti dokumentasi versi yang digunakan dan catat setiap perubahan. Jangan menganggap keduanya sudah terpasang hanya karena Apache2 dan database berjalan.

<!-- SS D.7 — Instalasi Apache2, PHP, dan database -->
<!-- SS D.8 — Status Apache2 dan database -->

### D.6 Instalasi dan konfigurasi Barnyard2

Barnyard2 merupakan komponen pada alur implementasi artikel untuk memproses output Snort dan meneruskannya ke penyimpanan yang digunakan BASE.

Sebelum instalasi, periksa ketersediaan sumber paket dan dependensi:

```bash
apt-cache policy barnyard2
```

Jika paket tidak tersedia, dokumentasikan metode instalasi yang dipilih, sumber paket, versi, dan dependensinya. Jangan menggunakan paket yang tidak jelas asalnya.

Setelah tersedia, lakukan konfigurasi sesuai format output Snort dan database yang dipakai. Pastikan parameter database, lokasi spool/output, serta hak akses sesuai dengan konfigurasi aktual.

<!-- SS D.9 — Instalasi Barnyard2 -->
<!-- SS D.10 — Konfigurasi Barnyard2 -->

### D.7 Instalasi dan konfigurasi BASE

Materi acuan menyebut BASE versi 1.4.5 untuk menyimpan/menampilkan alert melalui Web BASE. Siapkan BASE sesuai versi dan dependensi yang kompatibel dengan sistem.

Pastikan:

1. Apache2 berjalan.
2. PHP dan ekstensi yang dibutuhkan tersedia.
3. Database telah dibuat dan dapat diakses.
4. Output Snort dapat dibaca oleh Barnyard2.
5. BASE terhubung ke database yang benar.

Setelah konfigurasi, akses antarmuka BASE melalui browser menggunakan alamat lokal sesuai konfigurasi Apache2.

> Jangan menuliskan status “BASE berhasil” sebelum halaman BASE dapat dibuka dan alert hasil pengujian benar-benar tampil. Jika BASE 1.4.5 tidak kompatibel dengan lingkungan aktual, catat kendalanya sebagai keterbatasan implementasi. Jangan diam-diam mengganti BASE dengan dashboard lain karena hal tersebut mengubah komponen pada prosedur acuan.

<!-- SS D.11 — Konfigurasi database BASE -->
<!-- SS D.12 — Halaman Web BASE -->
<!-- SS D.13 — Alert Snort yang tampil pada BASE -->

### D.8 Pengujian konfigurasi Snort

Jalankan pemeriksaan konfigurasi:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

Jika pemeriksaan gagal:

1. Baca pesan error.
2. Periksa sintaks konfigurasi.
3. Periksa lokasi dan nama file rules.
4. Periksa HOME_NET dan interface.
5. Perbaiki konfigurasi.
6. Ulangi pemeriksaan sampai tidak ada error fatal.

Jika pemeriksaan berhasil, lanjutkan ke pengoperasian Snort.

<!-- SS D.14 — Hasil pengujian konfigurasi Snort -->

### D.9 Persiapan interface pemantauan

Interface LAN Linux Mint yang digunakan:

```text
ens37
```

Periksa opsi Snort sesuai versi:

```bash
snort --help
```

Jalankan Snort dalam mode IDS/passive detection pada interface pengujian. Perintah final harus mengikuti sintaks versi Snort yang benar-benar terpasang dan lokasi output alert aktual.

<!-- SS D.15 — Snort berjalan dan memantau ens37 -->

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
