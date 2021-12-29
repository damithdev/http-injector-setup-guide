## Disclaimer! This guide is only for educational purposes!!! 

## Get yourself a domain name
If you already have a domain you can use that if not you can get a free domain at [duckdns](duckdns.org)

## Obtaining a free VPS

Follow this guide [https://github.com/neneeen/own-vpn-for-everyone/blob/main/guides/OracleEnglish.md](https://github.com/neneeen/own-vpn-for-everyone/blob/main/guides/OracleEnglish.md) up until you get shell access.
After "In the new window, type sudo su and press Enter"

After that don't forget to change your duckdns ip to your vps public ip.

Also don't forget to configure the security list in oracle cloud to allow port 80/tcp and 443/tcp from 0.0.0.0/0. you can look this up in [open network access](https://github.com/neneeen/own-vpn-for-everyone/blob/main/guides/OracleEnglish.md#open-network-access)

# Setting up v2fly server

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



```
nano /etc/nginx/sites-available/default
```

Same as before paste the following content but 


