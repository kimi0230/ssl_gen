# SSL GEN
###### tags: `Develop` `SSL`
cerbot 比較方便

各種申請Tool: https://letsencrypt.org/docs/client-options/
* [Certbot](##1.Certbot)  (建議使用)
* [Acme](##2.Acme)
* [Caddy](##3.Caddy)
* [Dehydrated](##4.Dehydrated)


申請限制[Rate Limit](https://letsencrypt.org/docs/rate-limits/)
* Names/Certificate：單一 certificate 限制 100 個 hostname
* Certificates/Domain：每個 domain 每個禮拜最多 20 個 certificate，但 renew 不計算在 quota 內 (需要憑證內的 hostname 與之前完全一樣)
* Certificates/FQDNset：相同 hostname 的憑證每個禮拜最多發出五個

## 1.Certbot
* Docker:https://hub.docker.com/r/staticfloat/nginx-certbot/
* https://certbot.eff.org/
* Document: https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins
* [Ubuntu+nginx](https://certbot.eff.org/lets-encrypt/ubuntuother-nginx)

基本使用
* 全自動 (自備 HTTP 伺服器) certbot
* 半自動（自備 HTTP 伺服器，不調整 HTTP 伺服器設定）certbot certonly
* webroot （自備 HTTP 伺服器，自行設定 acme-challenge 部分）certbot certonly --webroot
* 手動（自備 HTTP 伺服器 、其他主機）certbot certonly --manual
* DNS 套件 certbot certonly --dns-PLUGIN
* Standalone（Certbot 提供獨立 HTTP 伺服器部分）certbot certonly --standalone

### Docker
https://hub.docker.com/r/staticfloat/nginx-certbot/
##### staticfloat/nginx-cerbot

```shell=
/scripts/run_certbot.sh
```
Helper function to ask certbot for the given domain(s).  Must have defined the.
EMAIL environment variable, to register the proper support email address.
```shell=
. ./util.sh get_certificate demo.kttech.ga char0x00@gmail.com)

```

Given a domain name, return true if a renewal is required (last renewal
ran over a week ago or never happened yet), otherwise return false.
```shell=
echo "$(. ./util.sh is_renewal_required demo.kttech.ga)"
```


---

## 2.Acme
https://github.com/acmesh-official/acme.sh

https://my.freenom.com/clientarea.php

### Gen
acme.sh  --issue  -d localhost --standalone

### Renew
./acme.sh  --renew   -d localhost  \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please

### 1.Install
```shell=
curl https://get.acme.sh | sh -s email=char0x00@gmail.com
```

### 2.發行 cert
```shell
acme.sh --issue -d example.com -w /home/wwwroot/example.com
```
or
```shell
acme.sh --issue -d example.com -w /var/www/html
```


Example:
```shell
/home/ubuntu/.acme.sh/acme.sh --issue -d testapi.petdev.info -w /home/ubuntu/pawbo-api/src/assets/
```
---
## 3.Caddy
類似nginx 
https://github.com/caddyserver/caddy
教學: https://junyussh.github.io/p/caddy-web-server/

### 1. Install
```shell=
brew install caddy
```

### 2. Config
cinfig file
```jsonld=
{
    "email" kimi0230@gmail.com
}


localhost {
    reverse_proxy /crwn-clothing/* 127.0.0.1:3000
	encode gzip
    file_server
	log {
		output file access.log {
			roll_keep_for 5s
		}
	}
}

localhost:7788 {
	reverse_proxy /* 127.0.0.1:5566
	file_server
}

```
caddy adapt --config /path/to/Caddyfile
caddy reload

### 3. run
```
caddy run
```
check caddy server `http://localhost:2019/` you can see browser show `404 page not found`

### 4. 自動啟動
...

---

## 4.Dehydrated 
教學: https://tinyl.io/4vsD
(阿仁)
https://github.com/dehydrated-io/dehydrated

### 1. Download
```shell
apt-get install curl
```
##### 方法一: 下載最新 release 的 dehydrated 並且解開，目前是 0.4.0：
$ # refer: https://github.com/lukas2511/dehydrated/releases
```shell
curl -LO https://github.com/lukas2511/dehydrated/archive/v0.4.0.tar.gz
tar -zxv -f v0.4.0.tar.gz
cd dehydrated-0.4.0/
```

##### 方法二: 或是透過 Git 下載最新版本：
```
curl -LO https://raw.githubusercontent.com/lukas2511/dehydrated/master/dehydrated
```

### 1. Install

```shell=
mkdir /etc/dehydrated/
cp dehydrated /etc/dehydrated/
chmod a+x /etc/dehydrated/dehydrated

mkdir -p /var/www/dehydrated/
```

#### nginx 檔案補上
location /.well-known/acme-challenge/ {
    alias /var/www/dehydrated/;
}

#### 第一次使用才需要
```shell=
/etc/dehydrated/dehydrated --register --accept-terms 
```

### 2. 發行
注意子網域名稱不可以有 "."
```shell=
/etc/dehydrated/dehydrated -c -d test-api.pawbo.com 
```

成功後產生的檔案都在 /etc/dehydrated/certs/letsencrypt.tw/

成功的話會有類似的輸出：
```
# INFO: Using main config file /etc/dehydrated/config.sh
Processing letsencrypt.tw
+ Signing domains...
+ Generating private key...
+ Generating signing request...
+ Requesting challenge for letsencrypt.tw...
+ Responding to challenge for letsencrypt.tw...
+ Challenge is valid!
+ Requesting certificate...
+ Checking certificate...
+ Done!
+ Creating fullchain.pem...
+ Done!
```

### 4. nginx 檔案補上
```json=
ssl_certificate /etc/dehydrated/certs/josera.pawbo.com/fullchain.pem;
ssl_certificate_key /etc/dehydrated/certs/josera.pawbo.com/privkey.pem;
```

### 5. 重load nginx
``` shell
service nginx reload
```

### 6. 設定cron
接下來設定 /etc/cron.d/dehydrated-letsencrypt_tw (因為 /etc/cron.d/ 裡面的檔名不能有 . 這個符號，用 _ 取代)，讓 cron 每天自動檢查並更新：
```shell
vi /etc/crobtab
```

```shell
0 4 1 * * root sleep $(expr $(printf "\%d" "0x$(hostname | md5sum | cut -c 1-8)") \% 86400); ( /etc/dehydrated/dehydrated -c -d spring-test.pawbo.com; /usr/sbin/service nginx reload ) > /tmp/dehydrated-spring-test.pawbo.com.log 2>&1
```

### 手動更新憑證
( /etc/dehydrated/dehydrated -c -d test-pawbolive.pawbo.com; /usr/sbin/service nginx reload ) > /tmp/dehydrated-test-pawbolive.pawbo.com.log 2>&1




/etc/dehydrated/certbot-auto renew --dry-run
/etc/dehydrated/certbot-auto renew --quiet --no-self-upgrade --post-hook "systemctl restart nginx"


### 三個月更新憑證方法
```
cd /etc/nginx/sites-available/
vi test-api.pawbo.com
```

修改如下:

```    
    location /.well-known/acme-challenge/ {
      alias /var/www/dehydrated/;
    }
    #rewrite ^(.*)$  https://$host$1 permanent;
    
    #ssl_certificate /etc/dehydrated/certs/jenkins.pawbo.com/fullchain.pem;
    #ssl_certificate_key /etc/dehydrated/certs/jenkins.pawbo.com/privkey.pem;
```
重啟 nginx
```
systemctl restart nginx
```    
    
cd /etc/dehydrated/certs/ 刪除網址目錄

重新要憑證
```
/etc/dehydrated/dehydrated -c -d test-api.pawbo.com
```

```
cd /etc/nginx/sites-enabled/
vi test-api.pawbo.com
```
再把檔案回復如下:
```
    #location /.well-known/acme-challenge/ {
    #  alias /var/www/dehydrated/;
    #}
    rewrite ^(.*)$  https://$host$1 permanent;
    
    ssl_certificate /etc/dehydrated/certs/jenkins.pawbo.com/fullchain.pem;
    ssl_certificate_key /etc/dehydrated/certs/jenkins.pawbo.com/privkey.pem;
```

```
systemctl restart nginx 
```

---



#### Note
##### 1. nginx 設定 ssl
https://www.maxlist.xyz/2020/01/19/docker-nginx/

##### 2. mac 設定 hosts
sudo vi /etc/hosts
```
127.0.0.1 kk.test.caddy
```
dscacheutil -flushcache




# Reference
* https://caddyserver.com/docs/caddyfile/concepts#structure
* https://junyussh.github.io/p/caddy-web-server/
* https://caddyserver.com/docs/caddyfile/directives/log#syntax