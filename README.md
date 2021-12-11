# Jarkom-Modul-5-D11-2021

Anggota kelompok:
- 05111940000067 - Fika Nur Aini
- 05111940000095 - Fuad Elhasan Irfani
- 05111940000203 - Fidhia Ainun Khofifah

---

### `(A)` Topologi jaringan

![image](https://user-images.githubusercontent.com/73324192/145664783-d6b31fca-6697-4fb4-891c-11381597a258.png)

### `(B)` Subnetting

- Untuk subnetting, kami menggunakan metode VLSM. Penghitungannya menggunakan bantuan http://vlsmcalc.net/

![image](https://user-images.githubusercontent.com/73324192/145664809-770dcc80-c823-4e72-a4dc-e10ca9fd0a98.png)
![image](https://user-images.githubusercontent.com/73324192/145664836-f44ad290-edb7-4d75-a25c-48e8fccc17bf.png)

### `(C)` Routing

- Assign IP statis pada semua node sesuai dengan pembagian IP. Berikut contohnya pada Foosha.

![image](https://user-images.githubusercontent.com/73324192/145665147-1eff9379-69c4-47f0-ac89-211530e28424.png)

- Selanjutnya konfigurasikan routing pada Foosha.

![image](https://user-images.githubusercontent.com/73324192/145664982-9e2458a3-65f0-41fd-864a-2ec37d6fa245.png)
![image](https://user-images.githubusercontent.com/73324192/145665065-c17e409b-defb-4efe-bba1-c5d20a92d8a9.png)

### `(D)` IP dinamis pada subnet Blueno, Cipher, Fukurou, dan Elena

- Ubah konfigurasi network keempat subnet tersebut menjadi DHCP

![image](https://user-images.githubusercontent.com/73324192/145665239-1d2df6e8-2f13-4c1f-ac11-7f7aba8a4845.png)

- Pada Jipangu, install DHCP Server. Konfigurasikan `/etc/default/isc-dhcp-server` seperti berikut. Pada `INTERFACES` diisikan `eth0`.

![image](https://user-images.githubusercontent.com/73324192/145665288-8a8c010c-9175-44c9-97b2-62d8f7793068.png)

- Konfigurasikan `/etc/dhcp/dhcpd.conf` seperti berikut. IP berdasarkan pembagian VLSM. Kosongkan konfigurasi subnet yang tidak memerlukan IP dinamis. Domain name server diisikan IP Doriki sebagai DNS Server.

![image](https://user-images.githubusercontent.com/73324192/145665370-fdbd602e-4cbf-43b0-9c51-6219290d1ce6.png)

- Kemudian restart DHCP dengan `service isc-dhcp-server restart`.

- Selanjutnya install DHCP Relay pada semua router. Lalu konfigurasikan `/etc/default/isc-dhcp-relay` seperti berikut. Servers diisikan dengan IP Jipangu. Pada Interfaces dikosongkan agar router auto mendeteksi interface mana yang merequest DHCP.

![image](https://user-images.githubusercontent.com/73324192/145665515-8772fbef-005f-4c46-ac24-f26224da66ce.png)

- Lalu restart DHCP Relay dengan `/etc/init.d/isc-dhcp-relay restart`

- Sekarang semua client mendapatkan IP dinamis.

![image](https://user-images.githubusercontent.com/73324192/145665622-605ec01b-6e0e-411c-a85c-daf730573125.png)

### ` 1. ` Konfigurasi iptables agar bisa mengakses keluar tanpa menggunakan MASQUERADE

- Pada modul-modul sebelumnya, eth0 pada Foosha selalu menggunakan IP dinamis. Untuk kali ini assign IP statis pada eth0 Foosha.

![image](https://user-images.githubusercontent.com/73324192/145665755-bce783eb-2621-4f0e-8eb3-43f9d153f34a.png)

- Lalu jalankan konfigurasi iptables berikut `iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 192.168.122.2`
- `192.168.122.2` adalah IP statis Foosha
- `SNAT --to 192.168.122.2` berarti semua node yang mengakses keluar akan menggunakan IP Foosha. Karena Foosha bisa mengakses keluar maka semua node juga bisa mengakses keluar.
- Edit `/etc/resolv.conf` pada semua node dan tambahkan nameserver yang ada pada Foosha

![image](https://user-images.githubusercontent.com/73324192/145666046-1870d7a3-d3dd-4d86-b990-f5f947700d3a.png)

- Sekarang semua node bisa mengakses keluar

![image](https://user-images.githubusercontent.com/73324192/145666034-1f9db434-cea0-4c7d-a2b0-4c6ebdc93b4c.png)

### ` 2. ` Drop semua akses HTTP dari luar Topologi pada DHCP Server dan DNS Server

- Pada Foosha jalankan konfigurasi iptables berikut `iptables -A FORWARD -i eth0 -p tcp --destination-port 80 -d 192.197.7.131 -j DROP` dan `iptables -A FORWARD -i eth0 -p tcp --destination-port 80 -d 192.197.7.130 -j DROP`. IP Tersebut adalah IP Jipangu (DHCP Server) dan IP Doriki (DNS Server). Tapi kami belum tahu apakah konfigurasi ini benar atau tidak

### ` 3. ` Membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan, selebihnya didrop

- Pada Jipangu (DHCP Server) dan Doriki (DNS Server) tambahkan iptables rule berikut `iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP`. Koneksi ICMP akan didrop apabila sudah ada 3 koneksi ICMP bersamaan.

### ` 4. ` Akses subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis

- Tambahkan rule pada Doriki (DNS Server), sehingga akses selain di waktu berikut akan ditolak.
`
iptables -A INPUT -s 192.197.7.0/25 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 192.197.0.0/22 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT

iptables -A INPUT -s 192.197.7.0/25 -j REJECT
iptables -A INPUT -s 192.197.0.0/22 -j REJECT
`

#### Kendala praktikum keseluruhan
- Tidak tahu bagaimana cara testingnya apakah iptables bekerja sesuai yang diinginkan atau tidak.
- Kurang paham iptables
