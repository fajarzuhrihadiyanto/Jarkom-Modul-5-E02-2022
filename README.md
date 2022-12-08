# Laporan Resmi Praktikum Jaringan Komputer Modul 5

## Anggota Kelompok
- 5025201248 - Fajar Zuhri Hadiyanto
- 5025201047 - Sidrotul Munawaroh

## A (Topologi)

![A](https://user-images.githubusercontent.com/52820619/206482782-6879974b-5e52-4c33-a251-895e0ab717e6.png)

## B (Subnetting)

Subnetting menggunakan VLSM
![B](https://user-images.githubusercontent.com/52820619/206483036-26e2053a-52b7-4976-b2f2-3b5685043525.png)

## C (Routing)
### Routing pada Strix

![C1](https://user-images.githubusercontent.com/52820619/206483738-e8ff59b8-0bf4-4d9b-869b-96742e3b78dd.png)

### Routing Pada Westalis

![C2](https://user-images.githubusercontent.com/52820619/206483873-7c47fe63-1e91-40cd-8ea3-7bc2e65036fe.png)

### Routing Pada Ostanis

![C3](https://user-images.githubusercontent.com/52820619/206483932-06822267-a5a0-4e9e-8306-84829caca6d4.png)

## D (Dynamic IP)

### Konfigurasi DHCP Server

Pada server WISE yang berperan sebagai DHCP server, lakukan instalasi dhcp server dengan perintah `apt-get update`, lalu `apt-get install isc-dhcp-server -y`. Setelah itu, buka file `/etc/default/isc-dhcp-server`, lalu masukkan line berikut
```
INTERFACES="eth0"
```

lalu buka file `/etc/dhcp/dhcpd.conf`, lalu tambahkan line berikut
```
subnet 192.193.0.0 netmask 255.255.252.0 {
    range 192.193.0.2 192.193.3.254;
    option routers 192.193.0.1;
    option broadcast-address 192.193.3.255;
    option domain-name-servers 192.193.7.131;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.193.4.0 netmask 255.255.254.0 {
    range 192.193.4.2 192.193.5.254;
    option routers 192.193.4.1;
    option broadcast-address 192.193.5.255;
    option domain-name-servers 192.193.7.131;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.193.6.0 netmask 255.255.255.0 {
    range 192.193.6.2 192.193.6.254;
    option routers 192.193.6.1;
    option broadcast-address 192.193.6.255;
    option domain-name-servers 192.193.7.131;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.193.7.0 netmask 255.255.255.128 {
    range 192.193.7.2 192.193.7.126;
    option routers 192.193.7.1;
    option broadcast-address 192.193.7.127;
    option domain-name-servers 192.193.7.131;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.193.7.128 netmask 255.255.255.248 {
}

subnet 192.193.7.144 netmask 255.255.255.252 {
}

subnet 192.193.7.148 netmask 255.255.255.252 {
}
```

lalu, lakukan restart dhcp server dengan perintah `service isc-dhcp-server restart`

### Konfigurasi DHCP Relay

Untuk setiap router yang berperan sebagai DHCP relay (dalam hal ini yaitu semua router), lakukan instalasi dhcp relay dengan perintah `apt-get update`, lalu `apt-get install isc-dhcp-relay`, untuk opsi2 yang diminta, kosongkan saja terlebih dahulu. Setelah itu buka file `/etc/default/isc-dhcp-relay`, lalu tambahkan line berikut
```
SERVERS="192.193.7.130"
INTERFACES="eth1 eth2"
OPTIONS=""
```

Setelah itu, restart dhcp relay dengan perintah `service isc-dhcp-relay restart`

### Konfigurasi IP Client

Pada client, lakukan konfigurasi ip sebagai berikut
```
auto eth0
iface eth0 inet dhcp
```

Setelah itu, restart client.

## Nomor 1 (Mengakses Internet keluar tanpa MASQUERADE)

gunakan postrouting pada nat table dengan jump ke SNAT, lalu ubah source addressnya menjadi address dari router pada interface yang terhubung ke nat (dalam kasus ini yaitu eth0) menggunakan perintah `iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 192.168.122.10`

![1](https://user-images.githubusercontent.com/52820619/206488209-846417ed-9e2f-46c6-8ba6-1b7cf96dc931.png)

## Nomor 2 (Blok semua tcp dan udp dari luar jaringan)

Gunakan perintah sebagai berikut

```
iptables -A INPUT -m iprange --src-range 192.193.0.0-192.193.7.151 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
iptables -A INPUT -p udp -j DROP
```

![image](https://user-images.githubusercontent.com/52820619/206488747-82c266b0-1558-4c97-b22c-8792c28af8c9.png)

## Noomor 3 (Pembatasan ICMP pada DNS dan DHCP Server)

Gunakan perintah sebagai berikut pada DNS dan DHCP Server `iptables -A INPUT -p icmp -m conntrack --ctstate NEW -m connlimit --connlimit-mask 21 --connlimit-above 2 -j DROP`

## Nomor 4 (Akses Web Server berdasarkan Waktu)

Lakukan instalasi Web Server pada server Garden dan SSS (tidak akan dijelaskan disini). Pada router Ostania (yang terdekat dengan kedua server tsb), tambahkan rule sebagai berikut

```
iptables -A FORWARD -p tcp -d 192.193.7.138,192.193.7.139 -m multiport --dport 80,443 -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A FORWARD -p tcp -d 192.193.7.138,192.193.7.139 -m multiport --dport 80,443 -j DROP
```

Hasil pengujian pada Work hour
![4a](https://user-images.githubusercontent.com/52820619/206490680-74353ba1-b427-4f2a-ab29-8b3d71ffd93a.png)

Hasil pengujian pada Weekdays non work hour
![4b](https://user-images.githubusercontent.com/52820619/206490690-5cdfcc69-be28-446c-b2ff-781dfa16ba46.png)

Hasil pengujian pada Weekend
![4c](https://user-images.githubusercontent.com/52820619/206490693-bf0c77f1-1b8e-4f2f-bd46-2f8aadd8e583.png)


