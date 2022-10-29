|     Nama    |     NRP    |
| :--------- |:--------    |
|Muhammad Amin | 5025201251 |
|Aiffah Kiysa Waafi | 5025201202 |
|Elthan Ramanda B | 5025201092 |

## 1. Membuat Topologi sesuai gambar dibawah. WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet 
<img src="image\imageTopologiSoal.png">

### Network configuration for node (router) Ostania
<img src="image\ostania.png">

### Network configuration for node WISE
<img src="image\wise.png">

Untuk node yang lain tinggal diganti address dan gateway sesuai dengan eth dari ostania (gateway) dan address juga sesuai dari gatewaynya.

### Hasil Topologi F05
<img src="image\topologiHasil.png">

Untuk mengakses internet, yang pertama kita perlu setup NAT di Ostania
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sudo sysctl -p
```
kemudian jalankan command berikut
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.31.0.0/16
```
Untuk mengetahui IP DNS dapat dilihat pada file resolv.conf
```
cat /etc/resolv.conf
```

Pada node-node yang lain, agar dapat mengakses internet, jalankan command berikut sesuai IP DNS
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
Kemudian kita sudah dapat melakukan ping ke google
```
ping google.com -c 2
```

### Hasil ping google pada garden
<img src="image\ping_google_garden.png">

### Hasil ping google pada eden
<img src="image\ping_google_eden.png">

### Hasil ping google pada sss
<img src="image\ping_google_sss.png">

### Hasil ping google pada berlint
<img src="image\ping_google_berlint.png">

## 2. Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

Melakukan instalasi bind9 di wise
```
apt-get update
apt-get install bind9 -y
```
Kemudian ubah file /etc/resolv.conf untuk connect ke IP WISE

### Membuat domain wise.f05.com
Buat direktori wise didalam bind
```
mkdir /etc/bind/wise
```
Buat domain dengan mengisi konfigurasi file named.conf.local seperti berikut
```
echo "
zone \"wise.f05.com\" {
	type master;
	file \"/etc/bind/wise/wise.f05.com\";
};
" > /etc/bind/named.conf.local
```
Copy file db.lokal ke file wise.f05.com
```
cp /etc/bind/db.local /etc/bind/wise/wise.f05.com
```
Set domain name dengan menambahkan CNAME untuk wise.f05.com dan sesuaikan IP WISE
```
echo ";
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA    wise.f05.com. root.wise.f05.com. (
                             2           ; Serial
                        604800           ; Refresh
                         86400           ; Retry
                       2419200           ; Expire
                        604800 )         ; Negative Cache TTL
;
@       IN      NS      wise.f05.com.
@       IN      A       10.31.2.2 ; IP WISE
www     IN      CNAME   wise.f05.com.
@       IN      AAAA    ::1
" > /etc/bind/wise/wise.f05.com
```

Kemudian restart bind9 dan jalankan ping wise.f05.com
```
service bind9 restart
```
Jalankan ping pada client sss atau garden
<img src="image\ping_wise_sss.png">

## 3. Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden
Untuk membuat subdomain kita tinggal menambahkan konfigurasi www.eden dengan IP eden pada file wise.f05.com
```
echo ";
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA    wise.f05.com. root.wise.f05.com. (
                             2           ; Serial
                        604800           ; Refresh
                         86400           ; Retry
                       2419200           ; Expire
                        604800 )         ; Negative Cache TTL
;
@       IN      NS      wise.f05.com.
@       IN      A       10.31.2.2 ; IP WISE
www     IN      CNAME   wise.f05.com.
eden     IN    A     10.31.3.3
www.eden    IN A    10.31.3.3
@       IN      AAAA    ::1
" > /etc/bind/wise/wise.f05.com
```
Kemudian restart bind9 dan www.wise.f05.com dapat diakses


## 4. Buat juga reverse domain untuk domain utama 
Buka command pada wise, kemudian pada file named.conf.local tambahkan konfigurasi untuk reverse DNS
```
zone "2.31.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/2.31.10.in-addr.arpa";
};
```
sehingga file named.conf.local isinya menjadi seperti berikut

Kemudian reverse DNS dengan copy db.local ke file address berikut
```
cp /etc/bind/db.local /etc/bind/wise/2.31.10.in-addr.arpa
```
ubah juga setting untuk reverse DNS
```
echo ";
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA    wise.f05.com. root.wise.f05.com. (
                             2           ; Serial
                        604800           ; Refresh
                         86400           ; Retry
                       2419200           ; Expire
                        604800 )         ; Negative Cache TTL
;
2.31.10.in-addr.arpa.       IN      NS         wise.f05.com.
2                           IN      PTR        wise.f05.com. ; Byte ke-4 WISE

" > /etc/bind/wise/2.31.10.in-addr.arpa
```

## 5. Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama 

## 6. Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation 

## 7. Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

## 8. Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

## 9. Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home