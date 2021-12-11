# Jarkom-Modul-5-T5-2021

### Laporan Resmi Pengerjaan Sesi Lab Jaringan Komputer
Nama Anggota Kelompok :
1. Shavica Ihya Q A L (05311940000013)
2. Gerry Putra Fresnando (05311940000031)
3. Mohammad Ibadul Haqqi (05311940000037)

# Soal

Setelah kalian mempelajari semua modul yang telah diberikan, Luffy ingin meminta bantuan untuk terakhir kalinya kepada kalian. Dan kalian dengan senang hati mau membantu Luffy.

1. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy
2. Keterangan :
Doriki adalah DNS Server
Jipangu adalah DHCP Server
Maingate dan Jorge adalah Web Server
Jumlah Host pada Blueno adalah 100 host
Jumlah Host pada Cipher adalah 700 host
Jumlah Host pada Elena adalah 300 host
Jumlah Host pada Fukurou adalah 200 host
3. Karena kalian telah belajar subnetting dan routing, Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM. setelah melakukan subnetting,
4. Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.
5. Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.


# Jawaban

### Perhitungan VLSM
Berikut adalah topologi dan pembagian subnet

Berikut Tree yang kami dapat :

Berikut Perhitungan IP yang kami dapat:

### Konfigurasi pada setiap node

Foosha
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.44.0.1
	netmask 255.255.255.252

auto eth2
iface eth2 inet static
	address 10.44.0.5
	netmask 255.255.255.252
```

Guanhao
```
auto eth0
iface eth0 inet static
	address 10.44.0.6
	netmask 255.255.255.252
	gateway 10.44.0.5

auto eth1
iface eth1 inet static
	address 10.44.2.1
	netmask 255.255.254.0
	gateway 10.44.0.5

auto eth2
iface eth2 inet static
	address 10.44.1.1
	netmask 255.255.255.0
	gateway 10.44.0.5

auto eth3
iface eth3 inet static
	address 10.44.0.17
	netmask 255.255.255.248
	gateway 10.44.0.5
```

Water7
```
auto eth0
iface eth0 inet static
	address 10.44.0.2
	netmask 255.255.255.252
  gateway 10.44.0.1

auto eth1
iface eth1 inet static
	address 10.44.0.129
	netmask 255.255.255.128
  gateway 10.44.0.1

auto eth2
iface eth2 inet static
	address 10.44.4.1
	netmask 255.255.252.0
  gateway 10.44.0.1

auto eth3
iface eth3 inet static
	address 10.44.0.9
	netmask 255.255.255.248
  gateway 10.44.0.1
```

Blueno, Cipher, Elena, Fukurou
```
auto eth0
iface eth0 inet dhcp
```

Doriki
```
auto eth0
iface eth0 inet static
	address 10.44.0.10
	netmask 255.255.255.248
  gateway 10.44.0.9
```

# NO 1
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
    address 10.44.0.5
    netmask 255.255.255.252
    broadcast 10.44.0.7
 
auto eth2
iface eth2 inet static
    address 10.44.0.1
    netmask 255.255.255.252
    broadcast 10.44.0.3
' > /etc/network/interfaces
```
```
iptables -t nat -A POSTROUTING -s 10.44.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.2
```

kami menggunakan command -t nat NAT Table pada -A POSTROUTING chain untuk -j SNAT mengubah source address yang awalnya berupa private IPv4 address yang memiliki 16-bit blok dari private IP addresses yaitu -s 10.44.0.0/21 menjadi --to-source 192.168.122.2 IP eth0 Foosha yaitu 192.168.122.2 karena Foosha merupakan router yang terhubung ke internet melalui eth0.

Untuk testing, kami mencoba kirim ping keluar pada Foosha dan Jipangu

### Foosha
```
ping 8.8.8.8
```
### Jipangu
```
ping 8.8.8.8
```



# NO 2
Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang memiliki ip DHCP dan DNS Server demi menjaga keamanan.

### Foosha
```
iptables -A FORWARD -d 10.44.0.16/29 -i eth0 -p tcp -m tcp --dport 80 -j DROP
```
Kita menggunakan -A FORWARD untuk menyaring paket dengan -p tcp -m tcp yaitu protokol TCP dari luar topologi menuju ke DHCP Server JIPANGU dan DNS Server DORIKI (yang berada di satu subnet yang sama yaitu -d 10.44.0.16/29), dimana akses SSH (yang memiliki --dport 80 port 80) yang masuk ke DHCP Server JIPANGU dan DNS Server DORIKI melalui interfaces eth0 dari DHCP Server JIPANGU dan DNS Server DORIKI dengan command -i eth0 untuk DROP kami gunakan command -j DROP

## Testing

### Elena
```
nmap -p 80 10.44.2.2
```


# NO 3
Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

## Setting
### Jipangu
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

### Doriki
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

Kita menggunakan command -A INPUT untuk menyaring paket dan command -p icmp agar protokol ICMP atau ping yang masuk dibatasi kami gunakan juga -m connlimit --connlimit-above 3 yang artinya hanya sebatas maksimal 3 koneksi saja yang dapat tersambung dan command --connlimit-mask 0 untuk memperbolehkan akses darimana saja, sehingga lebih dari itu akan di DROP command-j DROP

## Testing
### Elena
```
ping 10.44.0.18
```

### Fukurou
```
ping 10.44.0.18
```

### Blueno
```
ping 10.44.0.18
```

### Cipher
```
ping 10.44.0.18
```


# NO 4
Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.

## Setting
### Doriki
```
# cipher
iptables -A INPUT -s 10.44.4.0/22 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 10.44.4.0/22 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 10.44.4.0/22 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT

# blueno
iptables -A INPUT -s 10.44.0.128/25 -m time --weekdays Fri,Sat,Sun -j REJECT
iptables -A INPUT -s 10.44.0.128/25 -m time --timestart 00:00 --timestop 06:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
iptables -A INPUT -s 10.44.0.128/25 -m time --timestart 15:01 --timestop 23:59 --weekdays Mon,Tue,Wed,Thu -j REJECT
```

- Di sini kami menggunakan -A INPUT untuk menyaring paket yang masuk dari -s 10.44.4.0/22 subnet CIPHER dan 10.44.0.128/25 subnet BLUENO

- Untuk -m time --weekdays Fri, Sat,Sun di jam berapapun pada hari Jumat, Sabtu dan Minggu agar ditolak dengan command -j REJECT

- kemudian ditambahkan command -A INPUT -m time --timestart 00:00 --timestop 06:59 untuk menyaring paket di waktu jam 00:00 sampai dengan jam 06:59 --weekdays Mon,Tue,Wed,Thu pada hari Senin, Selasa, Rabu, Kamis agar ditolak kami tambahkan command -j REJECT

- Dan ditambahkan command -A INPUT -m time --timestart 15:01 --timestop 23:59 untuk menyaring paket di waktu jam 15:01 sampai dengan jam 23:59 --weekdays Mon,Tue,Wed,Thu pada hari Senin, Selasa, Rabu, Kamis agar ditolak dengan command -j REJECT

## Testing
### Cipher
```
ping 10.44.0.19
```

### Blueno
```
ping 10.44.0.19
```


# NO 5
Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.

## Setting
### Doriki
```
iptables -A INPUT -s 10.44.2.0/23 -m time --timestart 07:00 --timestop 15:00 -j REJECT

iptables -A INPUT -s 10.44.1.0/24 -m time --timestart 07:00 --timestop 15:00 -j REJECT
```

Di sini digunakan -A INPUT untuk menyaring paket yang masuk dari -s 10.44.2.0/23 subnet ELENA dan 10.44.1.0/24 subnet Fukurou -m time --timestart 07:00 --timestop 15:00 di waktu jam 07:00 sampai dengan jam 15:00 di hari apapun untuk batasan aksesnya dan digunakan command -j REJECT agar ditolak jika diluar batas waktunya.

## Testing
### Elena
```
ping 10.44.0.19
```

### Fukuro
```
ping 10.44.0.19
```


# NO 6
Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate
```
iptables -A PREROUTING -t nat -p tcp -d 10.44.0.19 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.44.0.11:80

iptables -A PREROUTING -t nat -p tcp -d 10.44.0.19 -j DNAT --to-destination 10.44.0.10:80
```

Pada kasus ini kami menggunakan Load Balancing untuk mendistribusikan koneksi. Kami menggunakan -A PREROUTING pada -t nat untuk mengubah destination IP yang awalnya menuju ke 10.44.0.19 DNS Server DORIKI menjadi ke 10.44.0.11:80 Server JORGE port 80 dan 10.44.0.10:80 Server MAINGATE port 80.
