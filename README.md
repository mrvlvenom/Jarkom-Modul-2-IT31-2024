# Jarkom-Modul-2-IT31-2024

| No | Nama | NRP |
|---|---|---|
| 1 | M. Januar Eko Wicaksono | 50272221006 |
| 2 | SUbkhan Mas Udi | 50272221044 |

## SOAL SHIFT MODUL 2
### PROBLEM
---
1.	Untuk membantu pertempuran di Erangel, kamu ditugaskan untuk membuat jaringan komputer yang akan digunakan sebagai alat komunikasi. Sesuaikan rancangan Topologi dengan rancangan dan pembagian yang berada di link yang telah disediakan, dengan ketentuan nodenya sebagai berikut :
    •	DNS Master akan diberi nama Pochinki, sesuai dengan kota tempat dibuatnya server tersebut
    •	Karena ada kemungkinan musuh akan mencoba menyerang Server Utama, maka buatlah DNS Slave Georgopol yang mengarah ke Pochinki
    •	Markas pusat juga meminta dibuatkan tiga Web Server yaitu Severny, Stalber, dan Lipovka. Sedangkan Mylta akan bertindak sebagai Load Balancer untuk server-server tersebut.

### SOLUTION
---
a. Untuk problem pertama kita disuruh membuat topologi sesuai dengan link yang ditentukan. Seperti gambar dibawah:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/topologi_soal.png)

b. Dan kita membuatnya pada Web GUI GNS3 dan seperti gambar dibawah:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/topologi_jawaban.png)

c. Setelah itu kita konfigurasikan IP disemua server, dengan menambahkan prefix plotting kelompok kita yaitu: 192.232:
- Erangel
```bash
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.232.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.232.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.232.3.1
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 192.232.4.1
	netmask 255.255.255.0
```
- Pochinki
```bash
auto eth0
iface eth0 inet static
	address 192.232.1.2
	netmask 255.255.255.0
	gateway 192.232.1.1
```
- Quarry
```bash
auto eth0
iface eth0 inet static
	address 192.232.2.2
	netmask 255.255.255.0
	gateway 192.232.2.1
```
- Shelter
```bash
auto eth0
iface eth0 inet static
	address 192.232.2.3
	netmask 255.255.255.0
	gateway 192.232.2.1
```
- Georgopol
```bash
auto eth0
iface eth0 inet static
	address 192.232.3.2
	netmask 255.255.255.0
	gateway 192.232.3.1
```
- Gatka
```bash
auto eth0
iface eth0 inet static
	address 192.232.3.3
	netmask 255.255.255.0
	gateway 192.232.3.1
```
- Severny
```bash
auto eth0
iface eth0 inet static
	address 192.232.4.2
	netmask 255.255.255.0
	gateway 192.232.4.1
```
- Stalber
```bash
auto eth0
iface eth0 inet static
	address 192.232.4.3
	netmask 255.255.255.0
	gateway 192.232.4.1
```
- Lipovka
```bash
auto eth0
iface eth0 inet static
	address 192.232.4.4
	netmask 255.255.255.0
	gateway 192.232.4.1
```
- Mylta
```bash
auto eth0
iface eth0 inet static
	address 192.232.4.5
	netmask 255.255.255.0
	gateway 192.232.4.1
```

d. Kemudian kita buat di dalam server pochinki dan georgopol pada file /root/.bashrc untuk auto install bind9 dan provision dengan menambahkan isi didalamnya dengan:
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.232.0.0/16
```

Ingat-ingat IP tersebut karena IP tersebut merupakan IP DNS, lalu ketikkan command ini di node ubuntu yang lain echo nameserver [IP DNS] > /etc/resolv.conf. Jika pada kasus contoh maka command-nya adalah echo nameserver 192.232.1.2 (IP Pochinki) > /etc/resolv.conf.

dan Sebelum melanjutkan kita coba 'ping google.com' untuk mengecek tiap server, webserver, dll apakah sudah connect atau belum, jika sudah maka lanjut ke step berikutnya.

e. Dan untuk menjalankannya kita menggunakan file .sh untuk menjalankan secara runtut. Untuk soal pertama ini menginstall terlebih dahulu untuk bind9 dan mengkonfigurasi tiap domain yang ada di soal, code nya seperti dibawah ini:
```bash
#!/bin/bash

# Check for root privileges
if [ "$(id -u)" -ne 0 ]; then
    cat "Please run this script as root"
    exit 1
fi

config() {
    # Check if BIND9 is installed
    if ! dpkg -l | grep -q bind9; then
        apt-get update 
        apt-get install -y bind9 nano 
    fi

    # Create directory for zone files if it doesn't exist
    mkdir -p /etc/bind/jarkom/

    # Create a zone configuration
    cat <<EOL > /etc/bind/named.conf.local
zone "airdrop.it31.com" {
    type master;
    notify yes;
    also-notify { 192.232.3.2; }; 
    allow-transfer { 192.232.3.2; };
    file "/etc/bind/jarkom/airdrop.it31.com";
};

zone "redzone.it31.com" {
    type master;
    notify yes;
    also-notify { 192.232.3.2; }; 
    allow-transfer { 192.232.3.2; };
    file "/etc/bind/jarkom/redzone.it31.com";
};

zone "loot.it31.com" {
    type master;
    notify yes;
    also-notify { 192.232.3.2; }; 
    allow-transfer { 192.232.3.2; };
    file "/etc/bind/jarkom/loot.it31.com";
};

zone "mylta.it31.com" {
    type master;
    notify yes;
    also-notify { 192.232.3.2; }; 
    allow-transfer { 192.232.3.2; };
    file "/etc/bind/jarkom/mylta.it31.com";
};

zone "tamat.it31.com" {
    type master;
    notify yes;
    also-notify { 192.232.3.2; }; 
    allow-transfer { 192.232.3.2; };
    file "/etc/bind/jarkom/tamat.it31.com";
};
EOL

    # Copy db.local to new files
    cp /etc/bind/db.local /etc/bind/jarkom/airdrop.it31.com
    cp /etc/bind/db.local /etc/bind/jarkom/redzone.it31.com
    cp /etc/bind/db.local /etc/bind/jarkom/loot.it31.com
    cp /etc/bind/db.local /etc/bind/jarkom/mylta.it31.com
    cp /etc/bind/db.local /etc/bind/jarkom/tamat.it31.com

    # Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/airdrop.it31.com
\$TTL 604800
@ IN SOA airdrop.it31.com. root.airdrop.it31.com. (
    $(date +"%Y%m%d%H") ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ); Negative Cache TTL
@ IN NS airdrop.it31.com.
@ IN A 192.232.4.3
www IN CNAME airdrop.it31.com.
EOL

cat <<EOL > /etc/bind/jarkom/redzone.it31.com
\$TTL 604800
@ IN SOA redzone.it31.com. root.redzone.it31.com. (
    $(date +"%Y%m%d%H") ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ); Negative Cache TTL
@ IN NS redzone.it31.com.
@ IN A 192.232.4.2
www IN CNAME redzone.it31.com.
EOL

cat <<EOL > /etc/bind/jarkom/loot.it31.com
\$TTL 604800
@ IN SOA loot.it31.com. root.loot.it31.com. (
    $(date +"%Y%m%d%H") ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ); Negative Cache TTL
@ IN NS loot.it31.com.
@ IN A 192.232.4.5
www IN CNAME loot.it31.com.
EOL

cat <<EOL > /etc/bind/jarkom/mylta.it31.com
\$TTL 604800
@ IN SOA mylta.it31.com. root.mylta.it31.com. (
    $(date +"%Y%m%d%H") ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ); Negative Cache TTL
@ IN NS mylta.it31.com.
@ IN A 192.232.3.2
www IN CNAME mylta.it31.com.
EOL

cat <<EOL > /etc/bind/jarkom/tamat.it31.com
\$TTL 604800
@ IN SOA tamat.it31.com. root.tamat.it31.com. (
    $(date +"%Y%m%d%H") ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ); Negative Cache TTL
@ IN NS tamat.it31.com.
@ IN A 192.232.3.2
www IN CNAME tamat.it31.com.
EOL
}
    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```

kemudian di running mengguanakan:
```bash
chmod +x pochinki.sh
./pochinki.sh
```
maka akan jalan dengan sendirinya. Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik.
![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_pochinki1.png)

### PROBLEM
---
2. 	Karena para pasukan membutuhkan koordinasi untuk mengambil airdrop, maka buatlah sebuah domain yang mengarah ke Stalber dengan alamat airdrop.xxxx.com dengan alias www.airdrop.xxxx.com dimana xxxx merupakan kode kelompok. Contoh : airdrop.it01.com

### SOLUTION
---
Untuk solusi nomor 2 ini, kita ubah untuk domain www.airdrop.xxx.com mengarah ke stalber kita ubah isi dalam file yang berada di /etc/bind/jarkom/airdrop.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/airdrop.it31.com
\$TTL 604800
@   IN SOA airdrop.it31.com. root.airdrop.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  airdrop.it31.com.
@   IN  A   192.232.4.3 //IP Stalber
www IN CNAME airdrop.it31.com.
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running mengguanakan:
```bash
chmod +x no2.sh
./no2.sh
```
Hasilnya akan seperti seblumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping www.airdrop.it31.com.

### PROBLEM
---
3.	Para pasukan juga perlu mengetahui mana titik yang sedang di bombardir artileri, sehingga dibutuhkan domain lain yaitu redzone.xxxx.com dengan alias www.redzone.xxxx.com yang mengarah ke Severny

### SOLUTION
---
Untuk solusi nomor 3 ini, kita ubah untuk domain www.redzone.xxx.com mengarah ke severny kita ubah isi dalam file yang berada di /etc/bind/jarkom/redzone.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/redzone.it31.com
\$TTL 604800
@   IN SOA redzone.it31.com. root.redzone.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  redzone.it31.com.
@   IN  A   192.232.4.2 //IP Severny
www IN CNAME redzone.it31.com.
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running mengguanakan:
```bash
chmod +x no3.sh
./no3.sh
```
Hasilnya akan seperti seblumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping www.redzone.it31.com.

### PROBLEM
---
4.	Markas pusat meminta dibuatnya domain khusus untuk menaruh informasi persenjataan dan suplai yang tersebar. Informasi persenjataan dan suplai tersebut mengarah ke Mylta dan domain yang ingin digunakan adalah loot.xxxx.com dengan alias www.loot.xxxx.com

### SOLUTION
---
Untuk solusi nomor 4 ini, kita ubah untuk domain www.loot.xxx.com mengarah ke mylta kita ubah isi dalam file yang berada di /etc/bind/jarkom/loot.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/loot.it31.com
\$TTL 604800
@   IN SOA loot.it31.com. root.loot.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  loot.it31.com.
@   IN  A   192.232.4.5
www IN CNAME loot.it31.com.
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running mengguanakan:
```bash
chmod +x no4.sh
./no4.sh
```
Hasilnya akan seperti seblumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping www.loot.it31.com.

### PROBLEM
---
5.	Pastikan domain-domain tersebut dapat diakses oleh seluruh komputer (client) yang berada di Erangel
   
### SOLUTION
---
cara pingnya menggunakan:
```bash
ping www.xxx.xxx.com -c 5
```
1. Hasil dari domain www.airdrop.it31.com menuju webserver stalber

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no2c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no2c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no2c3.png)
   
2. Hasil dari domain www.redzone.it31.com menuju webserver severny

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no3c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no3c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no3c3.png)
   
3. Hasil dari domain www.loot.it31.com menuju webserver mylta

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no4c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no4c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no4c3.png)

### PROBLEM
---
6. Beberapa daerah memiliki keterbatasan yang menyebabkan hanya dapat mengakses domain secara langsung melalui alamat IP domain tersebut. Karena daerah tersebut tidak diketahui secara spesifik, pastikan semua komputer (client) dapat mengakses domain redzone.xxxx.com melalui alamat IP Severny (Notes : menggunakan pointer record)
   
### SOLUTION
---
Untuk solusi nomor 6 ini, kita ubah untuk domain www.redzone.xxx.com mengarah ke severny kita ubah isi dalam file yang berada di /etc/bind/jarkom/redzone.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini dengan menambahkan Reverse DNS (Record PTR) pada domain www.redzone.it31.com:
Pertama dengan menginstall dns pada setiap client menggunakan file .sh  dibawah:
```bash
#!/bin/bash

# Set nameserver kembali ke Ip Erangel
echo 'nameserver 192.168.122.1' > /etc/resolv.conf

# Install package dnsutils
apt-get update
apt-get install dnsutils -y

# Kembalikan nameserver ke Ip Pochinki & Georgopol
echo '
nameserver 192.232.1.2
nameserver 192.232.3.2' > /etc/resolv.conf
```
kemudian di running menggunakan:
```bash
chmod +x no6c.sh
./no6c.sh
```
Setelah instalasi selesai, kemudian kembali pada server pochinki untuk mengganti pada /etc/bind/jarkom/redzone.it31.com
```bash
#!/bin/bash

# Buat reverse DNS (Record PTR)
echo 'zone "3.232.192.in-addr.arpa" {
    type master;
    file "/etc/bind/jarkom/3.232.192.in-addr.arpa";
};' >> /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/3.232.192.in-addr.arpa

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     redzone.it31.com. redzone.it31.com. (
                        2024050301      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.232.192.in-addr.arpa    IN      NS      redzone.it31.com.
2                       IN      PTR       192.232.4.5     ; IP Mylta' > /etc/bind/jarkom/3.232.192.in-addr.arpa

service bind9 restart
```
kemudian di running menggunakan:
```bash
chmod +x no6.sh
./no6.sh
```
Hasilnya akan seperti sebelumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping IP dari mylta dan berhasil seperti gambar dibawah:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no6c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no6c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no6c3.png)

### PROBLEM
---
7.	Akhir-akhir ini seringkali terjadi serangan siber ke DNS Server Utama, sebagai tindakan antisipasi kamu diperintahkan untuk membuat DNS Slave di Georgopol untuk semua domain yang sudah dibuat sebelumnya

### SOLUTION
---
Untuk problem nomor 7 ini caranya sama seperti awal-awal ketika kita mengkonfigurasi pochinki sebagai dns master, dengan mengganti master menjadi slave dan IP nya diganti menggunakan IP Pochinki (DNS MAster) untuk menghubungkannya. Dengan menggunakan file ,sh berikut:
```bash
#!/bin/bash

# Check for root privileges
if [ "$(id -u)" -ne 0 ]; then
    echo "Please run this script as root"
    exit 1
fi

config() {
    # Check if BIND9 is installed
    if ! dpkg -l | grep -q bind9; then
        apt-get update 
        apt-get install -y bind9 nano 
    fi

    # Create directory for zone files if it doesn't exist
    mkdir -p /etc/bind/jarkom/

    # Create a zone configuration
    cat <<EOL > /etc/bind/named.conf.local
zone "airdrop.it31.com" {
    type slave;
    masters { 192.232.1.2; }; 
    file "/var/lib/bind/jarkom/airdrop.it31.com";
};

zone "redzone.it31.com" {
    type slave;
    masters { 192.232.1.2; };
    file "/var/lib/bind/jarkom/redzone.it31.com";
};

zone "loot.it31.com" {
    type slave;
    masters { 192.232.1.2; };
    file "/var/lib/bind/jarkom/loot.it31.com";
};

zone "mylta.it31.com" {
    type slave;
    masters { 192.232.1.2; };
    file "/var/lib/bind/jarkom/mylta.it31.com";
};

zone "tamat.it31.com" {
    type slave;
    masters { 192.232.1.2; };
    file "/var/lib/bind/jarkom/tamat.it31.com";
};
EOL

#     # Copy db.local to new files
#     cp /etc/bind/db.local /etc/bind/jarkom/airdrop.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/redzone.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/loot.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/medkit.airdrop.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/siren.redzone.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/log.siren.redzone.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/mylta.it31.com
#     cp /etc/bind/db.local /etc/bind/jarkom/tamat.it31.com

#     # Configure each domain
#     for domain in "airdrop.it31.com" "redzone.it31.com" "loot.it31.com" "medkit.airdrop.it31.com" "siren.redzone.it31.com" "log.siren.redzone.it31.com" "mylta.it31.com" "tamat.it31.com"; do
#         cat <<EOL > "/etc/bind/jarkom/$domain"
# \$TTL 604800
# @ IN SOA $domain. root.$domain. (
#     $(date +"%Y%m%d00") ; Serial
#     604800 ; Refresh
#     86400 ; Retry
#     2419200 ; Expire
#     604800 ); Negative Cache TTL
# @ IN NS $domain.
# @ IN A 192.232.1.2
# www IN CNAME $domain.
# EOL
#     done

    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```

kemudian di running menggunakan:
```bash
chmod +x georgopol.sh
./georgopol.sh
```
maka akan jalan dengan sendirinya. Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik ketika DNS master di hentikan.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_georgopol1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no6c1.png)

### PROBLEM
---
8.	Kamu juga diperintahkan untuk membuat subdomain khusus melacak airdrop berisi peralatan medis dengan subdomain medkit.airdrop.xxxx.com yang mengarah ke Lipovka

### SOLUTION
---
Untuk problem nomor 8 ini, kita tambahkan subdomain medkit.airdrop.it31.com pada domain airdrop.it31.com yang mengarah ke lipovka, dengan cara file .sh berikut:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/airdrop.it31.com
\$TTL 604800
@   IN SOA airdrop.it31.com. root.airdrop.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  airdrop.it31.com.
@   IN  A   192.232.4.3
www IN CNAME airdrop.it31.com.
medkit  IN  A   192.232.4.3
@   IN AAAA ::1
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running menggunakan:
```bash
chmod +x no8.sh
./no8.sh
```
maka akan jalan dengan sendirinya. Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no8c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no8c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no8c3.png)

### PROBLEM
---
9.	Terkadang red zone yang pada umumnya di bombardir artileri akan dijatuhi bom oleh pesawat tempur. Untuk melindungi warga, kita diperlukan untuk membuat sistem peringatan air raid dan memasukkannya ke subdomain siren.redzone.xxxx.com dalam folder siren dan pastikan dapat diakses secara mudah dengan menambahkan alias www.siren.redzone.xxxx.com dan mendelegasikan subdomain tersebut ke Georgopol dengan alamat IP menuju radar di Severny

### SOLUTION
---
a. Pertama kita tambahkan dulu subdomain siren.redzone/it31.com pada server pochinki /etc/bind/jarkom/redzone.it31.com dengan file .sh dibawah:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/redzone.it31.com
\$TTL 604800
@   IN SOA redzone.it31.com. root.redzone.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  redzone.it31.com.
@   IN  A   192.232.4.2
192.232.4.2 IN  PTR redzone.it31.com.
siren   IN  A   192.232.3.2 
ns1 IN  A   192.232.3.2
its IN  NS  ns1
@   IN  AAAA    ::1
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```

kemudian di running menggunakan:
```bash
chmod +x no9p.sh
./no9p.sh
```
- Kemudian edit file /etc/bind/named.conf.options pada Pochinki.
- Kemudian comment dnssec-validation auto; dan tambahkan baris berikut pada /etc/bind/named.conf.options.
```bash
  allow-query{any;};
```
- Setelah itu restart bind9 dengan:
```bash
service bind9 restart
```
- Kemudian membuat konfigurasi pada server georgopol, pertama sama seperti sebelumnya dengan comment dnssec-validation auto; dan tambahkan baris berikut pada /etc/bind/named.conf.options.
```bash
allow-query{any;};
```
- Setelah itu menambahkan subdomain pada config local georgopol dan membuat folder delegasi baru dan konfigurasikan seperti file .sh dibawah ini:
```bash
#!/bin/bash

# Create directory for zone files if it doesn't exist
    mkdir -p /etc/bind/delegasi

# Buat reverse DNS (Record PTR)
echo 'zone "siren.redzone.it31.com" {
    type master;
    file "/etc/bind/delegasi/siren.redzone.it31.com";
};' >> /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/delegasi/siren.redzone.it31.com

echo '
;
; BIND data file for local loopback interface
;
\$TTL 604800
@   IN SOA siren.redzone.it31.com. root.siren.redzone.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  siren.redzone.it31.com.
@   IN  A   192.232.3.2
log   IN  A   192.232.3.2' > /etc/bind/delegasi/siren.redzone.it31.com

service bind9 restart

```
kemudian di running menggunakan:
```bash
chmod +x no9ga.sh
./no9ga.sh
```

Setelah semua dijalanin, kita cek Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no9c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no9c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no9c3.png)

### PROBLEM
---
10.	Markas juga meminta catatan kapan saja pesawat tempur tersebut menjatuhkan bom, maka buatlah subdomain baru di subdomain siren yaitu log.siren.redzone.xxxx.com serta aliasnya www.log.siren.redzone.xxxx.com yang juga mengarah ke Severny

### SOLUTION
---

Untuk problem nomor 10 ini, kita tambahkan subdomain log.siren.redzone.it31.com pada domain redzone.it31.com yang juga mengarah ke severny seperti nomor 9 tadi. Jadi kita hanya menambahkan subdomain aja pada konfigurasinya, dengan cara file .sh berikut:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/redzone.it31.com
\$TTL 604800
@   IN SOA redzone.it31.com. root.redzone.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  redzone.it31.com.
@   IN  A   192.232.4.2
192.232.4.2 IN  PTR redzone.it31.com.
siren   IN  A   192.232.3.2
www IN  CNAME   siren.redzone.it31.com
log.siren IN  A   192.232.3.2    
ns1 IN  A   192.232.3.2
its IN  NS  ns1
@   IN  AAAA    ::1
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running menggunakan:
```bash
chmod +x no10.sh
./no10.sh
```
maka akan jalan dengan sendirinya. Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no10c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no10c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no10c3.png)

### PROBLEM
---
11.	Setelah pertempuran mereda, warga Erangel dapat kembali mengakses jaringan luar, tetapi hanya warga Pochinki saja yang dapat mengakses jaringan luar secara langsung. Buatlah konfigurasi agar warga Erangel yang berada diluar Pochinki dapat mengakses jaringan luar melalui DNS Server Pochinki

### SOLUTION
---

Untuk problem nomor 11 ini, kita tinggal menggunakan DNS Forwarder yang digunakan untuk mengarahkan DNS Server ke IP yang ingin dituju.
a. Edit file /etc/bind/named.conf.options pada server EniesLobby
b. Uncomment pada bagian ini:
```bash
forwarders {
    192.168.122.1; //IP dari erangel
};
```
c. kemudian Comment pada bagian ini
```bash
// dnssec-validation auto;
```
Dan tambahkan
```bash
allow-query{any;};
```
d. Setelah itu di restart
```bash
service bind9 restart
```
Dan tambahkan nameserver IP erangel pada setiap client trus coba ping google.com, dan ping salah satu domain harusnya jika nameserver pada file /etc/resolv.conf di client diubah menjadi IP Pochinki maka akan di forward ke IP DNS GNS3 yaitu IP nameserver yang ada di Erangel dan bisa mendapatkan koneksi. dan hasilnya seperti dibawah ini:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no11c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no11c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no11c3.png)

### PROBLEM
---
12.	Karena pusat ingin sebuah website yang ingin digunakan untuk memantau kondisi markas lainnya maka deploy lah webiste ini (cek resource yg lb) pada severny menggunakan apache
    
### SOLUTION
---
Pada problem nomor 12 ini, karena menggunakan web server dan harus menginstall apache, lynx dan php, maka kita install terlebih dahulu tiap worker mulai dari 3 client (gatka, shelter dan quarry) dan 1 webserver yaitu severny. Dengan cara menjalankan file.sh berikut:
```bash
#!/bin/bash

apt-get update
apt-get install apache2
service apache2 start
apt-get install php
sudo apt-get install lynx
apt-get install libapache2-mod-php7.0
a2enmod php7.0
apt-get wget //khusus severny
apt-get unzip //khusus severny
```

Kemudian pada saat webserver Severny dilakukan download source pada soal yang diperintahkan dengan cara berikut:
```bash
wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=11S6CzcvLG-dB0ws1yp494IURnDvtIOcq' -O 'dir-listing.zip'
wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1xn03kTB27K872cokqwEIlk8Zb121HnfB' -O 'lb.zip'

unzip -o dir-listing.zip -d dir-listing
unzip -o lb.zip -d lb

cp default.conf /etc/apache2/sites-available/default.conf
rm /etc/apache2/sites-available/000-default.conf

cp ./lb/worker/index.php /var/www/html/index.php

a2ensite default.conf

service apache2 restart
```
Setelah itu, coba cek di tiap client untuk cek apakah index.php sudah berjalan atau belum dengan cara ke IP Severny:
```bash
lynx http://192.232.4.2/index.php
```

Dan hasilnya seperti ini:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c3.png)

### PROBLEM
---
13. Tapi pusat merasa tidak puas dengan performanya karena traffic yag tinggi maka pusat meminta kita memasang load balancer pada web nya, dengan Severny, Stalber, Lipovka sebagai worker dan Mylta sebagai Load Balancer menggunakan apache sebagai web server nya dan load balancernya

### SOLUTION
---
a. Pertama lakukan instalasi terlebih dahulu pada web server mylta:
```bash
#!/bin/bash

apt-get update
apy-get install -y bind9
apt-get install apache2
service apache2 start
apt-get install php
apt-get install lynx
apt-get install libapache2-mod-php7.0
a2enmod php7.0
apt-get wget 
apt-get unzip 
```

b. Setelah itu, masukkan konfigurasi pada /etc/apache2/sites-available/loadBalance.conf pada mylta:
```bash
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.

        ServerName 192.168.1.2


        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        <Proxy balancer://mycluster>
        BalancerMember http://192.168.4.5
        </Proxy>
        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
ServerName 127.0.0.1
```

c. Kemudian konfigurasi juga pada stalber:
```bash
wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=11S6CzcvLG-dB0ws1yp494IURnDvtIOcq' -O 'dir-listing.zip'
wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1xn03kTB27K872cokqwEIlk8Zb121HnfB' -O 'lb.zip'

unzip -o dir-listing.zip -d dir-listing
unzip -o lb.zip -d lb

cp default.conf /etc/apache2/sites-available/default.conf
rm /etc/apache2/sites-available/000-default.conf

cp ./lb/worker/index.php /var/www/html/index.php

a2ensite default.conf

service apache2 restart
```
dengan default config yang sama seperti nomor 12.

d. Kemudian configurasi pada mylta menggunakan file.sh dibawah:

```bash
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_balancer
a2enmod lbmethod_byrequests

cp loadBalance.conf /etc/apache2/sites-available/loadBalance.conf
rm 000-default.conf

a2ensite loadBalance.conf

service apache2 restart
```

e. Maka dijalankan akan menghasilkan seperti hasil dibawah:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no12c3.png)

### PROBLEM
---
14.	Mereka juga belum merasa puas jadi pusat meminta agar web servernya dan load balancer nya diubah menjadi nginx

### SOLUTION
---



### PROBLEM
---
15.	Markas pusat meminta laporan hasil benchmark dengan menggunakan apache benchmark dari load balancer dengan 2 web server yang berbeda tersebut dan meminta secara detail dengan ketentuan:
    o	Nama Algoritma Load Balancer
    o	Report hasil testing apache benchmark 
    o	Grafik request per second untuk masing masing algoritma. 
    o	Analisis

### SOLUTION
---

### PROBLEM
---
16. Karena dirasa kurang aman karena masih memakai IP, markas ingin akses ke mylta memakai mylta.xxx.com dengan alias www.mylta.xxx.com (sesuai web server terbaik hasil analisis kalian)

### SOLUTION
---
a. Pertama kita konfigurasikan pada pochinki untuk ditambahkan seperti file.sh tersebut:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/mylta.it31.com
\$TTL 604800
@   IN SOA mylta.it31.com. root.mylta.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  mylta.it31.com.
@   IN  A   192.232.4.5
www IN CNAME mylta.it31.com.
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running menggunakan:
```bash
chmod +x no16.sh
./no16.sh
```
maka akan jalan dengan sendirinya. Dan hasilnya seperti dibawah berarti sudah berjalan dengan baik.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no16c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no16c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no16c3.png)

### PROBLEM
---
17. Agar aman, buatlah konfigurasi agar mylta.xxx.com hanya dapat diakses melalui port 14000 dan 14400.

### SOLUTION
---

### PROBLEM
---
18. Apa bila ada yang mencoba mengakses IP mylta akan secara otomatis dialihkan ke www.mylta.xxx.com

### SOLUTION
---
Untuk solusi nomor 18 ini, kita ubah untuk domain www.redzone.xxx.com mengarah ke severny kita ubah isi dalam file yang berada di /etc/bind/jarkom/mylta.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini dengan menambahkan Reverse DNS (Record PTR) pada domain www.mylta.it31.com:
Pertama dengan menginstall dns pada setiap client menggunakan file .sh  dibawah:
```bash
#!/bin/bash

# Install package dnsutils
apt-get update
apt-get install dnsutils -y

```
kemudian di running menggunakan:
```bash
chmod +x no6c.sh
./no6c.sh
```
Setelah instalasi selesai, kemudian kembali pada server pochinki untuk mengganti pada /etc/bind/jarkom/redzone.it31.com
```bash
#!/bin/bash

echo 'zone "4.236.192.in-addr.arpa.mylta" {
    type master;
    file "/etc/bind/jarkom/4.236.192.in-addr.arpa.mylta";
};' >> /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/4.232.192.in-addr.arpa

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/4.232.192.in-addr.arpa
$TTL    604800
@       IN      SOA     mylta.it31.com. mylta.it31.com. (
                        2024050301      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
4.232.192.in-addr.arpa    IN      NS      mylta.it31.com.
2                       IN      PTR       192.232.4.5     ; IP mylta' > /etc/bind/jarkom/3.232.192.in-addr.arpa

EOL

    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running menggunakan:
```bash
chmod +x no6.sh
./no6.sh
```
Hasilnya akan seperti sebelumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping IP dari mylta dan berhasil seperti gambar dibawah:

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no18c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no18c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no18c3.png)

### PROBLEM
---
19. Karena probset sudah kehabisan ide masuk ke salah satu worker buatkan akses direktori listing yang mengarah ke resource worker2

### SOLUTION
---

### PROBLEM
---
20. Worker tersebut harus dapat di akses dengan tamat.xxx.com dengan alias www.tamat.xxx.com

### SOLUTION
---
Untuk solusi nomor 2 ini, kita ubah untuk domain www.airdrop.xxx.com mengarah ke stalber kita ubah isi dalam file yang berada di /etc/bind/jarkom/tamat.it31.com yang sudah dibuat sebelumnya. Dengan isinya menggunakan file .sh untuk di overwrite menggunakan cat seperti code dibawah ini:
```bash
#!/bin/bash

# Configure each domain
    config() {
cat <<EOL > /etc/bind/jarkom/tamat.it31.com
\$TTL 604800
@   IN SOA tamat.it31.com. root.tamat.it31.com. (
            $(date +"%Y%m%d%H"); Serial
                        604800 ; Refresh
                        86400  ; Retry
                        2419200 ; Expire
                        604800 ); Negative Cache TTL
@   IN  NS  tamat.it31.com.
@   IN  A   192.232.4.2
www IN CNAME tamat.it31.com.
EOL


    # Restart BIND to apply changes
    service bind9 restart
}

# Call the function to execute the configuration
config
```
kemudian di running mengguanakan:
```bash
chmod +x no2-.sh
./no20.sh
```
Hasilnya akan seperti seblumnya, Kemudian setelah berhasil dijalankan maka kita coba di setiap client untuk ping www.airdrop.it31.com.

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no20c1.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no20c2.png)

![](https://github.com/mrvlvenom/Jarkom-Modul-2-IT31-2024/blob/main/img/hasil_no20c3.png)


