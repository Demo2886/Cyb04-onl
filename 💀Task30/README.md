
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



### Вызовите 5 различных сработок WAF Правила, применяя уязвимости в приложении

Запустите контейнер с уязвимым приложением.

```bash
docker run -d -p 8080:80 saurav7055/xvwa
```

![img](/💀Task30/img/XVWA.png)



###  **SQL-инъекция (SQL Injection)**

**Описание уязвимости**: SQL-инъекции происходят, когда пользовательский ввод передаётся к базе данных без должной фильтрации. Это позволяет злоумышленнику выполнять произвольные SQL-запросы.

**Пример атаки**:
```sql
35.228.67.180/login?username=' OR 1=1 --&password=test
```

**Что происходит**: Введённый код `' OR 1=1 --` позволяет обойти аутентификацию, так как условие `OR 1=1` всегда истинно.

**Сработка WAF**: WAF распознает характерные элементы SQL-инъекций (например, `' OR`, '--' ) и блокирует запрос.

![img](/💀Task30/img/SQLin.png)

###  **XSS (Межсайтовый скриптинг, Cross-Site Scripting)**

**Описание уязвимости**: XSS-атаки позволяют злоумышленнику внедрить вредоносный JavaScript-код в веб-страницу, который затем будет выполнен у других пользователей.

**Пример атаки**:
```html
35.228.67.180/search?q=<script>alert('XSS')</script>
```

**Что происходит**: Вредоносный JavaScript исполняется на стороне пользователя, что может привести к краже cookies, перенаправлению на вредоносные сайты и другим атакам.

**Сработка WAF**: WAF обнаружит попытку внедрения вредоносного кода (теги `<script>`, функция `alert()` и т.д.) и заблокирует запрос.

![img](/💀Task30/img/XSS.png)

###  **Командная инъекция (Command Injection)**

**Описание уязвимости**: Командная инъекция происходит, когда приложение передаёт пользовательский ввод напрямую в командную строку операционной системы без должной фильтрации.

**Пример атаки**:
```bash
35.228.67.180/data?file=report.txt;ls -la;
```

**Что происходит**: В этом примере злоумышленник пытается выполнить команду `ls -la`, которая выводит список файлов на сервере.

**Сработка WAF**: WAF обнаружит подозрительные символы, такие как `;` и команды оболочки (например, `ls`), и предотвратит выполнение запроса.

![img](/💀Task30/img/CI.png)

###  **LFI (Local File Inclusion)**

**Описание уязвимости**: LFI позволяет злоумышленнику получить доступ к файлам на сервере, используя уязвимую функцию, которая обрабатывает пользовательский ввод в путях к файлам.

**Пример атаки**:
```bash
35.228.67.180/page?file=../../../../etc/passwd
```

**Что происходит**: Злоумышленник пытается получить доступ к критически важному файлу `/etc/passwd`, который содержит информацию о пользователях системы.

**Сработка WAF**: WAF обнаруживает характерные шаблоны для LFI-атак (например, `../`) и блокирует запрос.

![img](/💀Task30/img/LFI.png)

