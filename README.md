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
