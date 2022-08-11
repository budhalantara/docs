## Update & Upgrade
```
sudo apt update && sudo apt upgrade
```

## Clean Setup
Install nginx
```
sudo apt install nginx
```
Install nginx-extras (more_clear_headers)
```
sudo apt install nginx-extras
or
sudo apt install libnginx-mod-http-headers-more-filter
```
Configure /etc/nginx/nginx.conf
```nginx
http {
	...

	# max body 200mb
	client_max_body_size 200M;

	# Remove Server Header
	more_clear_headers Server;

	# proxy
	proxy_http_version 1.1;
	proxy_cache_bypass $http_upgrade;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
	proxy_set_header Host $host;
	proxy_set_header X-Forwarded-For $remote_addr;

	# Cloudflare original visitor IP
	# https://support.cloudflare.com/hc/en-us/articles/200170786-Restoring-original-visitor-IPs
	# https://www.cloudflare.com/ips/
	# https://danielmiessler.com/blog/getting-real-ip-addresses-using-cloudflare-nginx-and-varnish/
	set_real_ip_from 103.21.244.0/22;
	set_real_ip_from 103.22.200.0/22;
	set_real_ip_from 103.31.4.0/22;
	set_real_ip_from 104.16.0.0/13;
	set_real_ip_from 104.24.0.0/14;
	set_real_ip_from 108.162.192.0/18;
	set_real_ip_from 131.0.72.0/22;
	set_real_ip_from 141.101.64.0/18;
	set_real_ip_from 162.158.0.0/15;
	set_real_ip_from 172.64.0.0/13;
	set_real_ip_from 173.245.48.0/20;
	set_real_ip_from 188.114.96.0/20;
	set_real_ip_from 190.93.240.0/20;
	set_real_ip_from 197.234.240.0/22;
	set_real_ip_from 198.41.128.0/17;
	set_real_ip_from 2400:cb00::/32;
	set_real_ip_from 2606:4700::/32;
	set_real_ip_from 2803:f800::/32;
	set_real_ip_from 2405:b500::/32;
	set_real_ip_from 2405:8100::/32;
	set_real_ip_from 2a06:98c0::/29;
	set_real_ip_from 2c0f:f248::/32;
	real_ip_header CF-Connecting-IP;
	...
}
```
Install PHP 7.4
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.4-fpm php7.4-mysqli php7.4-mbstring php7.4-curl php7.4-xml php7.4-json
```
Laravel 8 ext
```
sudo apt install php7.4-bcmath php7.4-xml
```
Codeigniter 4 ext
```
sudo apt install php7.4-mysqlnd php7.4-intl
```
Install phpMyAdmin
```
curl -O https://files.phpmyadmin.net/phpMyAdmin/4.9.7/phpMyAdmin-4.9.7-all-languages.tar.xz
curl -O https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.tar.gz

export PMA_PATH=/var/www/html/pma
tar -xvf phpMyAdmin-5.1.1-all-languages.tar.gz
mv phpMyAdmin-5.1.1-all-languages $PMA_PATH
mkdir $PMA_PATH/tmp
chmod 777 $PMA_PATH/tmp
```
Config phpMyAdmin
```
cp $PMA_PATH/config.sample.inc.php $PMA_PATH/config.inc.php
nano $PMA_PATH/config.inc.php
```
```php
...
$cfg['blowfish_secret'] = 'random64length';
...
$cfg['Servers'][$i]['host'] = '127.0.0.1';
...
```
listen aws rds
```
socat tcp-listen:3306,reuseaddr,fork tcp:endpoint.rds.amazonaws.com:3306 &
```
Domain server config
```nginx
server {
	listen 80;
	listen [::]:80;
	listen 443 ssl http2;
	listen [::]:443 ssl http2;

	ssl_certificate /etc/ssl/cf_example.com.pem;
	ssl_certificate_key /etc/ssl/cf_example.com.key;

	server_name example.com;

	access_log /var/log/nginx/example.com.log;
	error_log /var/log/nginx/example.com.error.log;

	root /var/www/example.com;
	index index.html index.php;
	...
	# normal files
	...
	location / {
		try_files $uri $uri/ =404;
	}
	...
	# rewrite vue app
	location / {
		try_files $uri $uri/ /index.html;
	}
	...
	# rewrite vue app if using subfolder
	location /sub {
		try_files $uri $uri/ /sub/index.html;
	}
	...
	# reverse proxy
	location / {
		proxy_pass http://127.0.0.1:3000;
	}
	...
	# reverse proxy if using subpath
	location /sub/ {
		proxy_pass http://127.0.0.1:3000/;
	}
	...
	# pass PHP scripts to FastCGI server
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
	}
	...
	# php using alias
	location /pma {
        alias /home/ubuntu/app/prod/pma;
        index index.php index.html;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }
    }
	...
	# block .git
	location ~ /\.git {
		return 404;
	}
	...
	# block .ini
	location ~ ^.+\.ini$ {
		return 404;
	}
}
```
link config
```
sudo ln -s */server-available */server-enabled
```
Install NVM https://github.com/nvm-sh/nvm#installing-and-updating
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
Install Node.js
```
nvm install v14.7.0
```
Install Node.js pnpm & pm2 package
```
npm i -g pnpm pm2
```

Install cerbot
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install python-certbot-nginx
```
Install cerbot ubuntu 20
```
sudo apt install python3-certbot-nginx
```
Obtaining an SSL Certificate
```
sudo certbot --nginx -d example.com -d www.example.com
```


Socat Autostart
```
[Install]
WantedBy=multi-user.target
Type=forking

[Service]
ExecStart=/usr/bin/socat tcp-listen:3306,reuseaddr,fork tcp:pandahack.cqedmedqwrbk.ap-southeast-1.rds.amazonaws.com:3306
ExecStop=/bin/kill -s TERM $MAINPID
Restart=always
```

```
sudo systemctl enable service
```

Puppeteer
```
sudo apt install chromium-browser
export PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
export PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
```


https://docs.gitlab.com/ee/ssh/index.html

```
aws ec2 create-security-group --group-name cloudflare-https --description "only allow cloudflare https"
aws ec2 authorize-security-group-ingress --group-name cloudflare-https --ip-permissions IpProtocol=tcp,FromPort=443,ToPort=443,Ipv6Ranges='[{CidrIpv6=2400:cb00::/32},{CidrIpv6=2606:4700::/32},{CidrIpv6=2803:f800::/32},{CidrIpv6=2405:b500::/32},{CidrIpv6=2405:8100::/32},{CidrIpv6=2a06:98c0::/29},{CidrIpv6=2c0f:f248::/32}]',IpRanges=[{CidrIp=103.21.244.0/22},{CidrIp=103.22.200.0/22},{CidrIp=103.31.4.0/22},{CidrIp=104.16.0.0/13},{CidrIp=104.24.0.0/14},{CidrIp=108.162.192.0/18},{CidrIp=131.0.72.0/22},{CidrIp=141.101.64.0/18},{CidrIp=162.158.0.0/15},{CidrIp=172.64.0.0/13},{CidrIp=173.245.48.0/20},{CidrIp=188.114.96.0/20},{CidrIp=190.93.240.0/20},{CidrIp=197.234.240.0/22},{CidrIp=198.41.128.0/17}]
aws ec2 create-security-group --group-name cloudflare-http --description "only allow cloudflare http"
aws ec2 authorize-security-group-ingress --group-name cloudflare-http --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,Ipv6Ranges='[{CidrIpv6=2400:cb00::/32},{CidrIpv6=2606:4700::/32},{CidrIpv6=2803:f800::/32},{CidrIpv6=2405:b500::/32},{CidrIpv6=2405:8100::/32},{CidrIpv6=2a06:98c0::/29},{CidrIpv6=2c0f:f248::/32}]',IpRanges=[{CidrIp=103.21.244.0/22},{CidrIp=103.22.200.0/22},{CidrIp=103.31.4.0/22},{CidrIp=104.16.0.0/13},{CidrIp=104.24.0.0/14},{CidrIp=108.162.192.0/18},{CidrIp=131.0.72.0/22},{CidrIp=141.101.64.0/18},{CidrIp=162.158.0.0/15},{CidrIp=172.64.0.0/13},{CidrIp=173.245.48.0/20},{CidrIp=188.114.96.0/20},{CidrIp=190.93.240.0/20},{CidrIp=197.234.240.0/22},{CidrIp=198.41.128.0/17}]
```

auto https nginx
```nginx
server {
    listen 80;
    listen [::]:80;

    server_name example.com;

    if ($host = example.com) {
        return 301 https://$host$request_uri;
    }

    return 404;
}
```