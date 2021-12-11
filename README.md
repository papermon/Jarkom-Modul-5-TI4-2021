# Jarkom-Modul-5-TI4-2021

**Oleh:**
  * Muhammad Nur Fauzan (05311940000035)
  * Ghimnastiar Al Abiyyuna (05311940000042)
  * Christopher Benedict (05311840000024)

---
#  TOPOLOGI
Langkah 1 yaitu membuat topologi jaringan sesuai dengan modul 
![image](https://cdn.discordapp.com/attachments/811534608800677891/919227550734098512/unknown.png)

**Dengan Keterangan**
  * Doriki adalah DNS Server
  * Jipangu adalah DHCP Server
  * Maingate dan Jorge adalah Web Server
  * Jumlah Host pada Blueno adalah 100 host
  * Jumlah Host pada Cipher adalah 700 host
  * Jumlah Host pada Elena adalah 300 host
  * Jumlah Host pada Fukurou adalah 200 host

## Soal B
Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM. setelah melakukan subnetting,
Disini kami menggunakan teknik VLSM. Dari hasil pembagian subnet, kami mendapatkan sejumlah 8 subnet yang terdiri atas 6 subnet untuk router-router dan router-client (A1, A2, A3, A4,A5,A6) dan 2 subnet untuk router-server (A7 dan A8) Seperti gambar berikut ini. 

![image](https://user-images.githubusercontent.com/75864703/145680416-661a3213-ce9a-4422-a26f-832bc3ca473f.png)
![image](https://user-images.githubusercontent.com/73151866/145680547-8401d5b0-820b-4335-a474-fb7a4657cc43.png)



1. Foosha
```
# masquerade dengan netmask terbesar /21
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.213.0.0/21

#Kiri Foosha

# ke a2 Blueno
route add -net 192.213.0.128 netmask 255.255.255.128 gw 192.213.0.2

# ke a1 client Cipher
route add -net 192.213.4.0 netmask 255.255.252.0 gw 192.213.0.2

# ke a8 client Doriki & Jipangu
route add -net 192.213.0.16 netmask 255.255.255.248 gw 192.213.0.2


#Kanan Foosha

# ke a5 Elena
route add -net 192.213.2.0 netmask 255.255.254.0 gw 192.213.0.6

# ke a6 client Fukurou
route add -net 192.213.1.0 netmask 255.255.255.0 gw 192.213.0.6
```
2. Water 7
pertama menjalankan command ```echo nameserver 192.168.122.1 > /etc/resolv.conf``` agar dapat melakukan ping 

```
#kanan Water 7

#ke a7 Jorge & Maingate
route add -net 192.213.0.8 netmask 255.255.255.248 gw 192.213.0.1

# ke a5 Elena
route add -net 192.213.2.0 netmask 255.255.254.0 gw 192.213.0.1

# ke a6 client Fukurou
route add -net 192.213.1.0 netmask 255.255.255.0 gw 192.213.0.1

# ke a4 Guanhao
route add -net 192.213.0.4 netmask 255.255.255.252 gw 192.213.0.1
```

3. Guanhao

```
#ke kiri 

#ke a2 water 7
route add -net 10.42.0.128 netmask 255.255.255.128 gw 192.213.0.5

#ke a2 Blueno
route add -net 10.42.0.128 netmask 255.255.255.128 gw 192.213.0.5

#ke a1 Cipher
route add -net 10.42.4.0 netmask 255.255.252.0 gw 192.213.0.5

#ke a7 Jipangu & Doriki
route add -net 10.42.0.16 netmask 255.255.255.248 gw 192.213.0.5

```
## Soal D
Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya


### Jipangu
Pertama kami menghubungkan ke internet dan melakukan install DHCP Server menggunakan command pada JIPANGU adalah 
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get install isc-dhcp-server
```
kemudian install nano dengan perintah ```apt-get install nano```
dan edit konfigurasi seperti berikut.
```
echo 
‘
INTERFACES="eth0"
’> /etc/default/isc-dhcp-server
```

### Foosha
Selanjutnya konfigurasi DHCP relay pada FOOSHA dengan menginstall DHCP Relay menggunakan
command ```apt-get install isc-dhcp-relay -y```
kemudian edit konfigurasi dengan setting sysctl untuk IP forwarding dan edit konfigurasi isc-dhcp-relay dengan server jipangu lalu dilakukan restart dhcp relay seperti berikut ini.
```
echo
‘
net.ipv4.ip_forward=1
‘ > /etc/sysctl.conf
sysctl -p 
echo '
SERVERS="192.213.0.18"
INTERFACES="eth1 eth2"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```
### Water7
sama seperti sebelumnya konfigurasi DHCP relay pada WATER7 dengan menginstall DHCP Relay menggunakan
command ```apt-get install isc-dhcp-relay -y```
kemudian edit konfigurasi dengan setting sysctl untuk IP forwarding dan edit konfigurasi isc-dhcp-relay dengan server jipangu lalu dilakukan restart dhcp relay seperti berikut ini.
```
apt-get install isc-dhcp-relay -y
echo
‘
net.ipv4.ip_forward=1
‘ > /etc/sysctl.conf
sysctl -p
echo '
SERVERS="192.213.0.18"
INTERFACES="eth0 eth1 eth2 eth3"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```
### Guanhao
sama seperti sebelumnya konfigurasi DHCP relay pada Guanhao dengan menginstall DHCP Relay menggunakan
command ```apt-get install isc-dhcp-relay -y```
namun tidak setting sysctl untuk IP forwarding dan edit konfigurasi isc-dhcp-relay dengan server jipangu lalu dilakukan restart dhcp relay seperti berikut ini.
```
apt-get install isc-dhcp-relay -y
echo '
SERVERS="192.213.0.18"
INTERFACES="eth0 eth1 eth2 eth3"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```
Agar DHCP Server JIPANGU dapat berjalan dengan lancar, maka kami melakukan deklarasi subnet yang terkoneksi pada JIPANGU yaitu subnet A8, A1, A6, dan A5 pada file ```etc/dhcp/dhcpd.conf```

### Jipangu
Selanjutnya dilakukan deklarasi untuk subnet A8 ini hanya harus dideklarasikan, tetapi tidak harus memilikisettingan DHCP
```
echo
'
#A8
subnet 192.213.0.16 netmask 255.255.255.248 {
}
 
#A1 (CIPHER)
subnet 192.213.4.0 netmask 255.255.252.0 {
range 192.213.4.2 192.213.4.702;
option routers 192.213.4.1;
option broadcast-address 192.213.7.255;
option domain-name-servers 192.213.0.19 ,202.46.129.2;
default-lease-time 600;
max-lease-time 7200;
}
 
#A2 (BLUENO)
subnet 192.213.0.128 netmask 255.255.255.128 {
range 192.213.0.130 192.213.1.230;
option routers 192.213.0.129;
option broadcast-address 192.213.0.255;
option domain-name-servers 192.213.0.19 ,202.46.129.2;
default-lease-time 600;
max-lease-time 7200;
}
 
#A6 (FUKUROU)
subnet 192.213.1.0 netmask 255.255.255.0 {
range 192.213.1.2 192.213.1.202;
option routers 192.213.1.1;
option broadcast-address 192.213.1.255;
option domain-name-servers 192.213.0.19 ,202.46.129.2;
default-lease-time 600;
max-lease-time 7200;
}
 
#A5 (ELENA)
subnet 192.213.2.0 netmask 255.255.254.0 {
range 192.213.2.2 192.213.2.302;
option routers 192.213.2.1;
option broadcast-address 192.213.3.255;
option domain-name-servers 192.213.0.19 ,202.46.129.2;
default-lease-time 600;
max-lease-time 7200;
}
' > /etc/dhcp/dhcpd.conf
```
## NO 1
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.

Pada Foosha dilakukan konfigurasi seperti berikut ini.
### Foosha

```
echo '
auto eth0
iface eth0 inet static
    address 192.168.122.2
    netmask 255.255.255.0
    broadcast 192.168.122.1
auto eth1
iface eth1 inet static
    address 192.213.0.5
    netmask 255.255.255.252
    broadcast 192.213.0.7
 
auto eth2
iface eth2 inet static
    address 192.213.0.1
    netmask 255.255.255.252
    broadcast 192.213.0.3
' > /etc/network/interfaces
```

```
iptables -t nat -A POSTROUTING -s 192.213.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.2
```

kami menggunakan command ```-t nat``` NAT Table pada ```-A POSTROUTING chain``` untuk ```-j SNAT``` mengubah source address yang awalnya berupa private IPv4 address yang memiliki 16-bit blok dari private IP addresses yaitu ```-s 192.213.0.0/21``` menjadi ```--to-source 192.168.122.2``` IP eth0 Foosha yaitu ```192.168.122.2``` karena Foosha merupakan router yang terhubung ke internet melalui eth0. 

> Testing

Untuk testing, kami mencoba kirim ping keluar pada Foosha dan Jipangu
### Foosha
```
ping google.com
```
### Jipangu
```
ping google.com
```
## NO 2
Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang memiliki ip DHCP dan DNS Server demi menjaga keamanan.

> Setting
### Foosha
```
iptables -A FORWARD -d 192.213.0.16/29 -i eth0 -p tcp -m tcp --dport 80 -j DROP
```

Kita menggunakan ```-A FORWARD``` untuk menyaring paket dengan``` -p tcp -m tcp``` yaitu protokol TCP dari luar topologi menuju ke DHCP Server JIPANGU dan DNS Server DORIKI (yang berada di satu subnet yang sama yaitu ```-d 192.213.0.16/29```), dimana akses SSH (yang memiliki ```--dport 80 port 80```) yang masuk ke DHCP Server JIPANGU dan DNS Server DORIKI melalui interfaces eth0 dari DHCP Server JIPANGU dan DNS Server DORIKI dengan command ```-i eth0``` untuk DROP kami gunakan command ```-j DROP```

> Testing
### Chiper
```
nmap -p 80 192.213.7.130
```
![image](https://user-images.githubusercontent.com/73151866/145680618-86f345df-2f56-4534-943e-bc1f5e995e26.png)


## NO 3
Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

> Setting
### Jipangu
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```
### Doriki
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

Kita menggunakan command ```-A INPUT``` untuk menyaring paket dan command ```-p icmp``` agar protokol ICMP atau ping yang masuk dibatasi kami gunakan juga ``-m connlimit --connlimit-above 3`` yang artinya hanya sebatas maksimal 3 koneksi saja yang dapat tersambung dan command ```--connlimit-mask 0``` untuk memperbolehkan akses darimana saja, sehingga lebih dari itu akan di DROP command-j DROP


### Jorge
```
ping 192.213.7.130
```
![image](https://user-images.githubusercontent.com/73151866/145680796-ba048f60-337a-4cf3-be2f-a8b2088d72d5.png)
### Maingate
```
ping 192.213.7.130
```
![image](https://user-images.githubusercontent.com/73151866/145680788-19c1261b-e0ef-408b-a1ee-8971fe0b270b.png)
### Blueno
```
ping 192.213.7.130
```
![image](https://user-images.githubusercontent.com/73151866/145680780-027c6272-a411-424b-8849-ac95d2394766.png)
### Cipher
```
ping 192.213.7.130
```
![image](https://user-images.githubusercontent.com/73151866/145680777-0202f51c-d79b-4001-a5a9-cc9fb9cd1d2b.png)
## NO 4
Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada  pukul 07.00 - 15.00 pada hari Senin sampai Kamis.

> Setting
### Doriki
```
# cipher
iptables -A INPUT -s 192.213.4.0/22 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 192.213.4.0/22 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 192.213.4.0/22 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT

# blueno
iptables -A INPUT -s 192.213.0.128/25 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 192.213.0.128/25 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 192.2130.128/25 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
```

- Di sini kami menggunakan ```-A INPUT``` untuk menyaring paket yang masuk dari ```-s 192.213.4.0/22``` subnet CIPHER dan 192.213.0.128/25 subnet BLUENO
- Untuk ```-m time --weekdays Fri, Sat,Sun``` di jam berapapun pada hari Jumat, Sabtu dan Minggu agar ditolak dengan command ```-j REJECT ```

- kemudian ditambahkan command  ``` -A INPUT -m time --timestart 00:00 --timestop 06:59``` untuk menyaring paket di waktu jam 00:00 sampai dengan jam 06:59 ```--weekdays Mon,Tue,Wed,Thu``` pada hari Senin, Selasa, Rabu, Kamis agar ditolak kami tambahkan command ```-j REJECT```

- Dan ditambahkan command ```-A INPUT -m time --timestart 15:01 --timestop 23:59``` untuk menyaring paket di waktu jam 15:01 sampai dengan jam 23:59 ```--weekdays Mon,Tue,Wed,Thu``` pada hari Senin, Selasa, Rabu, Kamis agar ditolak dengan command ```-j REJECT```


![image](https://user-images.githubusercontent.com/73151866/145680876-68da4d6e-cbb2-443a-9ed9-fe0c1bd4c4c9.png)
## NO 5
Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.

> Setting
### Doriki
```
iptables -A INPUT -s 192.213.2.0/23 -m time --timestart 07:00 --timestop 15:00 -j REJECT

iptables -A INPUT -s 192.213.1.0/24 -m time --timestart 07:00 --timestop 15:00 -j REJECT
```
Di sini digunakan ```-A INPUT``` untuk menyaring paket yang masuk dari ```-s 192.213.2.0/23``` subnet ELENA dan ```192.213.1.0/24``` subnet Fukurou ```-m time --timestart 07:00 --timestop 15:00``` di waktu jam 07:00 sampai dengan jam 15:00  di hari apapun untuk batasan aksesnya dan digunakan command ```-j REJECT``` agar ditolak jika diluar batas waktunya. 

## NO 6
Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate

```
iptables -A PREROUTING -t nat -p tcp -d 192.213.0.19 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.213.0.11:80

iptables -A PREROUTING -t nat -p tcp -d 192.213.0.19 -j DNAT --to-destination 192.213.0.10:80 
```

Pada kasus ini kami menggunakan  Load Balancing untuk mendistribusikan koneksi. Kami menggunakan ```-A PREROUTING``` pada ```-t nat``` untuk mengubah destination IP yang awalnya menuju ke ```192.213.0.19``` DNS Server DORIKI menjadi ke ```192.213.0.11:80``` Server JORGE port 80 dan ```192.213.0.10:80``` Server MAINGATE port 80.

