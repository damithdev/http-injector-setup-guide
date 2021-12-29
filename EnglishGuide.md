## Disclaimer! This guide is for educational purposes only!!! 

## Get yourself a domain name
If you already have a domain you can use that if not you can get a free domain at [duckdns](duckdns.org)

## Obtaining a free VPS

### Requirements

* A payment card (credit card/debit card that support online transaction without 3-D Secure)

### Fees

* Free forever, maximum traffic is 10 TB per month at 50 Mbps.

### Creating Oracle Cloud account

Login to https://www.oracle.com/cloud/free/ and click **Start for free**.

Free tier can't change or add region after registration. Use the nearest region to your location (unless your goal is to avoid your country routing/blocking). If you have a problem with the SMS token, use the customer service chat from the site.

## Setup Bitvise client

Download and install from https://www.bitvise.com/ssh-client-download

## Create VM

Login to https://www.oracle.com/cloud/sign-in.html and use cloud account name that has been emailed from Oracle, then login with the username (email) and password you've picked.

In Oracle console, click Create a VM instance

In Image and shape section, edit and change Image to Canonical Ubuntu 20.04 Minimal.

In the Add SSH keys section, click Save Private Key. Open Bitvise, click Client key manager, Import. Change filetype filter from Bitvise Keypair Files to All Files, then pick the recently downloaded file. Click Import, close the dialog, return to Oracle and click Create. Wait until instance status change from Provisioning to Running. Find Public IP Address field, click Copy

Return to Bitvise, paste IP address to Host. Change Username to ubuntu, ensure Initial Method is set to publickey. Set Client Key to Auto, then click Login. In Host Key Verification dialog, click Accept and Save. After the connection is complete, click New Terminal Console

## Opening Network Access

In Oracle, in Primary VNIC section, click Subnet. In the new window, click Default Security List, then click Add Ingress Rules, adjust the content as the following image then click Add Ingress Rules.

<br><img src="https://github.com/neneeen/own-vpn-for-everyone/blob/3c194c1fbf8ad2a9cb5d3b78eb25b95246b93ab8/images/oracle6.png" width="645">

Also create the same rule but substitute port 443 for port 80.

After that don't forget to change your duckdns ip to your vps public ip.

# Setting up v2fly server

Go back to the bitvise terminal console and start running these commands.

## Configure iptables
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

## Install the required packages

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
certbot certonly --nginx --non-interactive --agree-tos --email xvincent1000@gmail.com -d <yourdomain>.duckdns.org
```

## Setup V2Fly

```
v2ctl uuid
```
Copy the output and save it in notepad. You will need this later.

```
echo $RANDOM | md5sum | head -c 32; echo;
```
Copy the output and save it in notepad. You will need this later.

```
nano /usr/local/etc/v2ray/config.json
```

Then paste the following content(don't forget to delete existing content and change the required values in bracket)
```
{
  "inbounds": [{
    "port": <CHANGE THIS TO WHATEVER PORT YOU LIKE EXCEPT 80 OR 443, Save this for later>,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "<CHANGE THIS TO THE OUTPUT OF THE v2ctl uuid COMMAND>",
          "level": 1,
          "alterId": <CHANGE THIS TO WHATEVER NUMBER YOU LIKE MINIMUM 32. HIGHER IS BETTER FOR SECURITY, BUT LOWER SPEED>
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/<CHANGE THIS WITH THE OUTPUT OF echo $RANDOM>"
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

save and exit using ctrl+s and ctrl+x

### Example of a complete /usr/local/etc/v2ray/config.json file. DON'T USE THIS ONE!!! Because if people know your id,AlterID, path and Domain people can use your server(think domain as your username and ID as your password)
<details>
  
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

## Nginx configuration

```
echo "" > /etc/nginx/sites-available/default
```

```
nano /etc/nginx/sites-available/default
```

Same as before paste the following content to the file, don't forget to subtitute the values in the brackets.

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

    ssl_certificate /etc/letsencrypt/live/<YOUR DOMAIN>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<YOUR DOMAIN>/privkey.pem;
    
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location /<OUTPUT FROM echo $RANDOM> {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:<THE PORT THAT YOU HAVE SETUP BEFORE>;
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

## Example of a complete /etc/nginx/sites-available/default file.

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

## Finally reset and enable everything

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

# Client Config

# Konfigurasi Client

## Android
Download v2rayNG_x.x.xx.apk from [https://github.com/2dust/v2rayNG/releases/](https://github.com/2dust/v2rayNG/releases/) or you can install it from the google playstore

Press the (+) button on the top right >> Type manually[Vmess]

```
Configuration file
---------------------------------------------------------------
Remarks        : <FILL IN WITH WHATEVER YOU WANT>
Address        : <YOUR DOMAIN>
Port           : 443
UUID           : <OUTPUT FROM v2ctl uuid>
AlterId        : <ALTERID THAT YOU HAVE SET IN THE SERVER>
Security       : Auto

Transport
---------------------------------------------------------------
Network        : ws
request host   : 
path           : /<OUTPUT FROM echo $RANDOM>
tls            : tls
SNI            : <USE THE DOMAIN THAT YOU WANT TO INJECT THE HTTP HEADER WITH. for example if you want to inject the learning quota you can use "meet.google.com" or the chatting quota you can use "web.whatsapp.com">
allowInsecure  : true
```

### Example of a complete config
<details>
  
```
Configuration file
---------------------------------------------------------------
Remarks        : Singapore Server
Address        : vincenttjia.duckdns.org
Port           : 443
UUID           : 5c8713c8-4d98-933f-4460-54adc8c07c71
AlterId        : 64
Security       : Auto

Transport
---------------------------------------------------------------
Network        : ws
request host   : 
path           : /063f04131db66c38e76202c9fae75a12
tls            : tls
SNI            : meet.google.com
allowInsecure  : true
```
  
</details>


## Windows
For windows I have not tested if it works with hotspot from the phone.

Download v2rayN-Core.zip dari [https://github.com/2dust/v2rayN/releases/](https://github.com/2dust/v2rayN/releases/)

Extract the zip file then run v2rayN.exe

Servers > Add VMess server

```
Server
------------------------------------------------------------------
Address            : <YOUR DOMAIN>
Port               : 443
UUID               : <OUTPUT FROM v2ctl uuid>
AlterId            : <ALTERID THAT YOU HAVE SET IN THE SERVER>
Encryption method  : Auto
Alias              : <FILL IN WITH WHATEVER YOU WANT>

Transport
------------------------------------------------------------------
Transport protocol : ws
Camouflage type    : none
Camouflage domain  : 
Path               : /<OUTPUT FROM echo $RANDOM>
TLS                : TLS
allowInsecure      : true
SNI                : <USE THE DOMAIN THAT YOU WANT TO INJECT THE HTTP HEADER WITH. for example if you want to inject the learning quota you can use "meet.google.com" or the chatting quota you can use "web.whatsapp.com">
```

### Example of a complete config
<details>
  
```
Server
------------------------------------------------------------------
Address            : vincenttjia.duckdns.org
Port               : 443
UUID               : 5c8713c8-4d98-933f-4460-54adc8c07c71
AlterId            : 64
Encryption method  : Auto
Alias              : Singapore Server

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

  
References:
  - https://github.com/neneeen/own-vpn-for-everyone
