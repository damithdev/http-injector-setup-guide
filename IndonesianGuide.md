## Disclaimer! Panduan ini hanya bersifat educational!!! 

# Konfigurasi Server

## Dapatkan domain Gratis
Jika sudah punya domain dapat digunakan, jika belum bisa mendapatkan domain gratis di [duckdns](https://www.duckdns.org)

## Dapatkan VPS Gratis

### Prasyarat

* Kartu kredit/kartu debit yang mendukung transaksi online *tanpa* 3-D Secure (hampir semua kartu kredit berlogo Visa & MasterCard, kartu debit Jenius)

### Biaya

* Gratis selamanya, kuota 10 TB tiap bulan.

### Membuat akun Oracle Cloud

Masuk ke https://www.oracle.com/cloud/free/ dan klik **Start for free**. Oracle akan membuat dua test charge, yang pertama sekitar 1 USD, yang kedua sekitar 10 USD beberapa hari kemudian, keduanya akan dibatalkan setelahnya. Kecuali [upgrade](https://www.oracle.com/cloud/free/faq.html), Oracle *tidak akan* menarik biaya walaupun kredit gratis 200 dollar sudah habis atau 30 hari telah lewat. VM free tier tetap akan berjalan selamanya.

Free tier tidak bisa mengganti/menambah region setelah pembayaran. Untuk pengguna Indonesia gunakan region Singapura.

Jika menemui masalah tentang token SMS, kontak customer service chat dari situs Oracle. Setelah memasukkan informasi kartu pembayaran, proses dari Oracle bisa memakan waktu beberapa jam sebelum akun diaktifkan dan bisa membuat VM. Selama menunggu email dari Oracle yang memberikan Cloud Account name, install dulu Bitvise client

### Setup Bitvise client

Download dan jalankan installer Bitvise client dari https://www.bitvise.com/ssh-client-download

### Membuat VM

Masuk ke https://www.oracle.com/cloud/sign-in.html dan gunakan cloud account name dari email Oracle, lalu login dengan username (email) dan password yang sudah diset sebelumnya.

Setelah masuk di console, klik Create a VM instance

Di bagian Image and shape, edit dan ganti Image ke Canonical Ubuntu 20.04 Minimal.

Di bagian Add SSH keys, klik Save Private Key. Buka Bitvise, klik Client key manager, Import. Ganti filetype dari Bitvise Keypair Files ke All Files, lalu pilih file yang baru didownload. Klik Import dan pastikan ada entry baru. Tutup dialog, kembali ke halaman Oracle dan klik Create. Tunggu sampai status instance berubah dari Provisioning menjadi Running. Cari field Public IP Address, klik Copy

Kembali ke Bitvise, paste IP address ke Host. Ubah Username ke ubuntu, pastikan Initial Method diset ke publickey. Set Client Key ke Auto, lalu klik Login. Di dialog Host Key Verification, klik Accept and Save. Setelah koneksi sukses, klik New Terminal Console

Jangan lupa mengganti IP address di duckdns menjadi IP dari VPS tersebut

### Membuka Akses Jaringan

Di halaman Oracle, di bagian  Primary VNIC, klik Subnet. Di halaman yang terbuka, klik Default Security List, lalu klik Add Ingress Rules, dan sesuaikan isinya seperti dibawah ini lalu klik Add Ingress Rules.

<br><img src="https://github.com/neneeen/own-vpn-for-everyone/blob/3c194c1fbf8ad2a9cb5d3b78eb25b95246b93ab8/images/oracle6.png" width="645">

Lakukan hal yang sama namun port 443 di ganti port 80

# Setting server v2fly
Kembali ke terminal bitvise lalu masukan perintah-perintah dibawah ini.

## Konfigurasi IPTable
```
sudo su
```

```
iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
```

```
iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
```

```
netfilter-persistent save
```

## Install package yang dibutuhkan

```
apt update
```

```
apt install nginx certbot python3-certbot-nginx -y
```

```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```


## Setup Certbot

```
certbot certonly --nginx --non-interactive --agree-tos --email xvincent1000@gmail.com -d <domainanda>.duckdns.org
```

## Setup V2Fly

```
v2ctl uuid
```
Copy outputnya dan di simpan di notepad. akan dibutuhkan untuk nanti.

```
echo $RANDOM | md5sum | head -c 32; echo;
```
Copy outputnya dan di simpan di notepad. akan dibutuhkan untuk nanti.

```
nano /usr/local/etc/v2ray/config.json
```

Paste tulisan dibawah ini, jangan lupa menghapus tulisan yang sudah ada dan mengganti semua text yang ada di dalam bracket
```
{
  "inbounds": [{
    "port": <GANTI INI DENGAN ANGKA DIANTARA 1-65535 SELAIN 80 ATAU 443, Simpan ini untuk nanti>,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "<GANTI INI DENGAN OUTPUT DARI COMMAND v2ctl uuid>",
          "level": 1,
          "alterId": <GANTI INI DENGAN ANGKA BERAPAPUN MINIMAL 32. Semakin tinggi semakin baik untuk keamanan, namun kecepatan makin melambat>
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/<GANTI INI DENGAN OUTPUT DARI COMMAND echo $RANDOM>"
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

Gunakan ctrl+s dan ctrl + x. untuk save and exit.

### Contoh dari file /usr/local/etc/v2ray/config.json yang sudah jadi
<details>

<b>JANGAN COPAS CONTOH!!!</b> Karena jika orang lain mengetahui ID, AlterID, domain dan path anda, server anda dapat digunakan siapa saja(Anggap domain sebagai username dan ID sebagai password)
```
{
  "inbounds": [{
    "port": 12345,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "5c8713c8-4d98-933f-4460-54adc8c07c71",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/063f04131db66c38e76202c9fae75a12"
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```
  
</details>

## Konfigurasi nginx

```
echo "" > /etc/nginx/sites-available/default
```

```
nano /etc/nginx/sites-available/default
```

Sama seperti sebelumnya paste konten dibawah ini ke dalam file tersebut, jangan lupa menganti value dari bracket

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate /etc/letsencrypt/live/<DOMAIN ANDA>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<DOMAIN ANDA>/privkey.pem;
    
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location /<OUTPUT DARI echo $RANDOM> {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:<PORT YANG ANDA TELAH SETTING>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        # Show real IP if you enable V2Ray access log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
       try_files $uri $uri/ =404;
    }
}
```

### Contoh dari file /etc/nginx/sites-available/default yang sudah jadi.
<details>

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate /etc/letsencrypt/live/vincenttjia.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/vincenttjia.duckdns.org/privkey.pem;
    
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location /063f04131db66c38e76202c9fae75a12 {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:12345;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        # Show real IP if you enable V2Ray access log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
       try_files $uri $uri/ =404;
    }
}
```
  
</details>

## Finalisasi

```
systemctl restart v2ray
```

```
systemctl enable v2ray
```

```
systemctl restart nginx
```

```
systemctl enable nginx
```

# Konfigurasi Client

## Android
Download v2rayNG_x.x.xx.apk dari [https://github.com/2dust/v2rayNG/releases/](https://github.com/2dust/v2rayNG/releases/) atau jika kalian malas bisa di cari di playstore

Tombol + diatas kanan
Type manually[Vmess]
```
Configuration file
---------------------------------------------------------------
Remarks        : <BEBAS>
Address        : <DOMAIN KALIAN>
Port           : 443
UUID           : <OUTPUT DARI v2ctl uuid>
AlterId        : <ALTERID YANG TELAH KALIAN SETEL DI SERVER>
Security       : Auto

Transport
---------------------------------------------------------------
Network        : ws
request host   : 
path           : /<OUTPUT DARI echo $RANDOM>
tls            : tls
SNI            : <SILAHAKAN GUNAKAN DOMAIN YANG INGIN KALIAN INJECT. contoh kuota belajar "meet.google.com", kuota youtube "youtube.com">
allowInsecure  : true
```

### Contoh dari config yang sudah jadi
<details>
  
```
Configuration file
---------------------------------------------------------------
Remarks        : Server Singapore
Address        : vincenttjia.duckdns.org
Port           : 443
UUID           : 5c8713c8-4d98-933f-4460-54adc8c07c71
AlterId        : 64
Security       : Auto

Transport
---------------------------------------------------------------
Network        : ws
request host   : 
path           : /<OUTPUT DARI echo $RANDOM>
tls            : tls
SNI            : <SILAHAKAN GUNAKAN DOMAIN YANG INGIN KALIAN INJECT. contoh kuota belajar "meet.google.com", kuota youtube "youtube.com">
allowInsecure  : true
```
  
</details>


## Windows
Untuk windows saya belum pernah ngetest hotspot apakah berhasil atau tidak.
Download v2rayN-Core.zip dari [https://github.com/2dust/v2rayN/releases/](https://github.com/2dust/v2rayN/releases/)
Extract zip nya lalu jalankan v2rayN.exe
Servers > Add VMess server

```
Server
------------------------------------------------------------------
Address            : <DOMAIN KALIAN>
Port               : 443
UUID               : <OUTPUT DARI v2ctl uuid>
AlterId            : <ALTERID YANG TELAH KALIAN SETEL DI SERVER>
Encryption method  : Auto
Alias              : <BEBAS>

Transport
------------------------------------------------------------------
Transport protocol : ws
Camouflage type    : none
Camouflage domain  : 
Path               : /063f04131db66c38e76202c9fae75a12
TLS                : TLS
allowInsecure      : true
SNI                : meet.google.com
```

### Contoh dari config yang sudah jadi
<details>
  
```
Server
------------------------------------------------------------------
Address            : vincenttjia.duckdns.org
Port               : 443
UUID               : 5c8713c8-4d98-933f-4460-54adc8c07c71
AlterId            : 64
Encryption method  : Auto
Alias              : Server Singapore

Transport
------------------------------------------------------------------
Transport protocol : ws
Camouflage type    : none
Camouflage domain  : 
Path               : /063f04131db66c38e76202c9fae75a12
TLS                : TLS
allowInsecure      : true
SNI                : meet.google.com
```
  
</details>

  
Refrensi:
  - https://github.com/neneeen/own-vpn-for-everyone
