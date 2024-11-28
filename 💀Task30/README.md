
## Установка WAF (Nginx + ModSecurity3)

Требования:
- Сервер Ubuntu 22.04.
- Пользователь root.

### Шаг 1: Установка необходимых зависимостей

```bash
apt update -y 
apt upgrade -y
apt install g++ flex bison curl apache2-dev doxygen libyajl-dev ssdeep liblua5.2-dev libgeoip-dev libtool dh-autoreconf libcurl4-gnutls-dev libxml2 libpcre++-dev libxml2-dev git liblmdb-dev libpkgconf3 lmdb-doc pkgconf zlib1g-dev libssl-dev -y
```

### Шаг 2: Компиляция ModSecurity3

По умолчанию пакет ModSecurity не включен в репозиторий Ubuntu по умолчанию. Поэтому вам придется скомпилировать его из исходников.

```bash
wget https://github.com/SpiderLabs/ModSecurity/releases/download/v3.0.8/modsecurity-v3.0.8.tar.gz
```
После завершения загрузки извлеките загруженный файл с помощью следующей команды:

```bash
tar -xvzf modsecurity-v3.0.8.tar.gz
```
Далее перейдите в извлеченный каталог и настройте его с помощью следующей команды:

```bash
cd modsecurity-v3.0.8 
./build.sh 
./configure
```
Далее установите его с помощью следующей команды:

```bash
make 
make install
```

### Установите Nginx с поддержкой ModSecurity 3

Далее вам нужно будет установить Nginx с поддержкой ModSecurity. Сначала загрузите коннектор ModSecurity-nginx с помощью следующей команды:

```bash
cd ~
git clone https://github.com/SpiderLabs/ModSecurity-nginx.git
```
Затем загрузите исходный код Nginx с помощью следующей команды:

```bash
wget https://nginx.org/download/nginx-1.20.2.tar.gz
```
Далее извлеките исходный код Nginx с помощью следующей команды:

```bash
tar xzf nginx-1.20.2.tar.gz
```
Далее создайте пользователя для Nginx с помощью следующей команды:

```bash
useradd -r -M -s /sbin/nologin -d /usr/local/nginx nginx
```
Далее измените директорию на исходник Nginx и настройте ее с помощью следующей команды:

```bash
cd nginx-1.20.2
./configure --user=nginx --group=nginx --with-pcre-jit --with-debug --with-compat --with-http_ssl_module --with-http_realip_module --add-dynamic-module=/root/ModSecurity-nginx --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log
```
Далее установите его с помощью следующей команды:

```bash
make
make modules
make install
```
Далее создайте символическую ссылку Nginx с помощью следующей команды:

```bash
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
```
Далее проверьте версию Nginx с помощью следующей команды:

```bash
nginx -v
```

### Настройка Nginx с помощью ModSecurity
Скопируйте примеры конфигурационных файлов с помощью следующей команды:

```bash
cp ~/modsecurity-v3.0.8/modsecurity.conf-recommended /usr/local/nginx/conf/modsecurity.conf
cp ~/modsecurity-v3.0.8/unicode.mapping /usr/local/nginx/conf/
```
Далее сделайте резервную копию файла конфигурации Nginx:

```bash
cp /usr/local/nginx/conf/nginx.conf{,.bak}
```
Далее отредактируйте файл конфигурации Nginx:

```bash
nano /usr/local/nginx/conf/nginx.conf
```
Удалите строки по умолчанию и добавьте следующие строки:

```bash
load_module modules/ngx_http_modsecurity_module.so;
user  nginx;
worker_processes  1;
pid        /run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  nginx.example.com;
        modsecurity  on;
        modsecurity_rules_file  /usr/local/nginx/conf/modsecurity.conf;
        access_log  /var/log/nginx/access_example.log;
        error_log  /var/log/nginx/error_example.log;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
Сохраните и закройте файл, затем включите ModSecurity с помощью следующей команды:

```bash
sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /usr/local/nginx/conf/modsecurity.conf
```

### Установка основных правил ModSecurity
Набор основных правил OWASP ModSecurity предоставляет набор правил для обнаружения и защиты от широкого спектра атак, включая минимальную десятку ложных срабатываний OWASP.
Сначала скачайте набор правил OWASP с помощью следующей команды:

```bash
cd ~
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git /usr/local/nginx/conf/owasp-crs
```
Затем переименуйте crs-setup.conf.example в файл crs-setup.conf:

```bash
cp /usr/local/nginx/conf/owasp-crs/crs-setup.conf{.example,}
```
Далее определите правила с помощью следующей команды:

```bash
echo -e "Include owasp-crs/crs-setup.conf
Include owasp-crs/rules/*.conf" >> /usr/local/nginx/conf/modsecurity.conf
```
Затем проверьте Nginx на наличие ошибок конфигурации с помощью следующей команды:

```bash
nginx -t
```
Если все в порядке, вы получите следующий вывод:

```bash
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

### Создание сервисного файла systemd для nginx

Далее вам нужно будет создать файл сервиса systemd для управления сервисом Nginx. Чтобы вы могли запускать и останавливать сервис Nginx через системы. Вы можете создать его с помощью следующей команды:

```bash
nano /etc/systemd/system/nginx.service
```
Добавьте следующие строки:

```bash
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

Сохраните и закройте файл, затем перезагрузите систему с помощью следующей команды:

```bash
systemctl daemon-reload
systemctl start nginx
systemctl enable nginx
```

### Проверка работы Nginx с ModSecurity
Далее проверьте работоспособность Nginx с помощью следующей команды:

```bash
curl localhost?doc=/bin/ls
```
Вы также можете проверить журнал безопасности Mode, используя следующую команду:    

```bash
tail /var/log/modsec_audit.log
```

![img](/💀Task30/img/waf.png)
![img](/💀Task30/img/test.png)

### Заключение
Поздравляю! вы успешно установили ModSecurity с Nginx на Ubuntu 22.04. Теперь вы можете внедрить ModSecurity в свою производственную среду для защиты от DDoS-атак. Не стесняйтесь спрашивать меня, если у вас есть какие-либо вопросы.
