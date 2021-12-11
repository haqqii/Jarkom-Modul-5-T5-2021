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
    address 10.42.0.5
    netmask 255.255.255.252
    broadcast 10.42.0.7
 
auto eth2
iface eth2 inet static
    address 10.42.0.1
    netmask 255.255.255.252
    broadcast 10.42.0.3
' > /etc/network/interfaces
```
```
iptables -t nat -A POSTROUTING -s 10.42.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.2
```

kami menggunakan command -t nat NAT Table pada -A POSTROUTING chain untuk -j SNAT mengubah source address yang awalnya berupa private IPv4 address yang memiliki 16-bit blok dari private IP addresses yaitu -s 10.42.0.0/21 menjadi --to-source 192.168.122.2 IP eth0 Foosha yaitu 192.168.122.2 karena Foosha merupakan router yang terhubung ke internet melalui eth0.

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
iptables -A FORWARD -d 10.42.0.16/29 -i eth0 -p tcp -m tcp --dport 80 -j DROP
```
Kita menggunakan -A FORWARD untuk menyaring paket dengan -p tcp -m tcp yaitu protokol TCP dari luar topologi menuju ke DHCP Server JIPANGU dan DNS Server DORIKI (yang berada di satu subnet yang sama yaitu -d 10.42.0.16/29), dimana akses SSH (yang memiliki --dport 80 port 80) yang masuk ke DHCP Server JIPANGU dan DNS Server DORIKI melalui interfaces eth0 dari DHCP Server JIPANGU dan DNS Server DORIKI dengan command -i eth0 untuk DROP kami gunakan command -j DROP

## Testing

### Elena
```
nmap -p 80 10.42.2.2
```
