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

Problem :
karena iptables tidak kelihatan sudah behrasil apa belum jadi dijalankan
```bash
iptables -t nat -L
```
agar iptables ter-save, maka ada command tambahan diatas, agar jika ter-restart tetep aman

## 2. Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

Jalankan shell script dibawah ini untuk meng-install service bind9 yang akan dibutuhkan

```bash
apt clean
apt update
apt upgrade
apt install -y bind9
clear
service bind9
```
Setelah selesai baru kita jalankan bash untuk setup domainnya
```bash
#setup DNS Master (domain name) at wise

mkdir /etc/bind/wise

#Buat domain	
echo "
zone \"wise.f05.com\" {
	type master;
	file \"/etc/bind/wise/wise.f05.com\";
};

" > /etc/bind/named.conf.local

rndc reload

cp /etc/bind/db.local /etc/bind/wise/wise.f05.com

#Set domain name
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

#DONE set domain name with CNAME

service bind9 restart
clear
echo "Set Up www.wise.f05.com done"
```
Setelah selesai setup, pada node client, ubah file /etc/resolv.conf untuk mengarah ke IP Wise (10.31.2.2)
```
#Change nameserver to IP WISE
echo "nameserver 10.31.2.2" > /etc/resolv.conf
ping wise.f05.com -c 2				   
```
				   
Jalankan ping pada client sss atau garden

<img src="image\ping_wise_sss.png">

Problem:
Pastikan nameserver mengarah ke IP Wise 10.31.2.2

## 3. Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden


```bash
#Tambahkan prefix eden. dan www.eden. pada file /etc/bind/wise/wise.f05.com
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
untuk memastikan apakah sudah bisa maka dapat melakukan ping pada eden dan www.eden

![image](https://user-images.githubusercontent.com/94375772/198834828-55d33ae7-b238-4b84-8792-8bc800145ba7.png)
		   
## 4. Buat juga reverse domain untuk domain utama 
 
 Untuk membuat reverse, kita perlu menambah zone pada file /etc/bind/named.conf.local pada WISE
 ```bash
 #Adding zone reverse dns
mkdir /etc/bind/wise
echo "
zone \"2.31.10.in-addr.arpa\" {
    type master;
    file \"/etc/bind/wise/2.31.10.in-addr.arpa\";
};
" >> /etc/bind/named.conf.local

rndc reload

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

rndc reload
service bind9 restart
clear
echo "Reverse DNS Added"
 ```

Hasil reverse DNS
<img src="image\reverse_dns.png">

## 5. Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama 

jalankan command ini untuk menambahkan Berlint dengan IP 10.31.3.2 sebagai DNS Slave
```bash
echo "
zone \"wise.f05.com\" {
	also-notify { 10.31.3.2; };
	allow-transfer { 10.31.3.2; };
};
" >> /etc/bind/named.conf.local
service bind9 restart
```
Pada node Berlint, jalankan command ini untuk menjadi DNS Slave dari DNS Master WISE dengan IP 10.31.2.2
```bash
echo "
zone \"wise.f05.com\" {
    type slave;
    masters { 10.31.2.2; };
    file \"/var/lib/bind/wise.f05.com\";
};" > /etc/bind/named.conf.local
service bind9 restart
```
setelah itu untuk mencoba, stop service bind9 pada WISE
```bash
service bind9 stop
```
seteleha itu coba ping www.wise.f05.com dari client 
```bash
ping www.wise.f05.com -c 2
```
<img src="image\5. Slave.jpg">

Problem:
ada lupa spasi di:
```bash
echo"
blablabla "
```
harusnya
```bash
echo "
blablabla "
```

## 6. Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation 

jalankan command bash dibawah ini, konfigurasi hampir sama seperti DNS Master wise, tapi butuh diatur /etc/bind/named.conf.options nya
```bash
#kalau gak diatur ini, saat ngeping akan ke website random seperti saat ngeping wise tapi nameserver masih ke NAT
echo "
options {
        directory \"/var/cache/bind\";
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
" > /etc/bind/named.conf.options

echo "
zone \"operation.wise.f05.com\"{
        type master;
        file \"/etc/bind/operation/operation.wise.f05.com\";
}; " >> /etc/bind/named.conf.local
mkdir /etc/bind/operation
echo "
\$TTL    604800
@       IN      SOA     operation.wise.f05.com. root.operation.wise.f05.com. (
                                2      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.f05.com.
@               IN      A       10.31.3.3       ;ip Eden
www             IN      CNAME   operation.wise.f05.com.
" > /etc/bind/operation/operation.wise.f05.com
service bind9 restart
```
<img src="image\6. ping.jpg">

problem:
saat selesai konfigurasi tanpa ngubah named.conf.options, saat ngeping akan ke website random seperti saat ngeping wise tapi nameserver masih ke NAT

## 7. Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

tinggal tambahkan CNAME strix dan www.strix pada file /etc/bind/operation/operation.wise.f05.com
```bash
echo "
strix IN CNAME operation.wise.f05.com.
www.strix IN CNAME operation.wise.f05.com.
" >> /etc/bind/operation/operation.wise.f05.com
service bind9 restart
```
<img src="image\7.strix.jpg">

## 8. Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

## 9. Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home

## 10. Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com

## 11. Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja

## 12. Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache

## 13. Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js

## 14. Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500

## 15. dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy

## 16. dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com

## 17. Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!
