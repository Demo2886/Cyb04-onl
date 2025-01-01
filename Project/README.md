
<p align="left">
    <a href="https://t.me/darkcat28"><img src="https://badgen.net/badge/icon/%40darkcat28?icon=telegram&label=TG" /></a>
    <a href="https://www.linkedin.com/in/%D0%B0%D0%BB%D0%B5%D0%BA%D1%81%D0%B0%D0%BD%D0%B4%D1%80-%D0%BD%D0%B5%D1%84%D0%B5%D0%B4%D0%B8%D0%BD-b34629114/"><img src="https://badgen.net/badge/blog/linkedin/green?icon=chrome&label" /></a>
</p>

### Hi there 👋

<div id="header" align="center">
  <img src="https://raw.githubusercontent.com/Demo2886/Cyb04-onl/master/%F0%9F%92%80InfoMysor%F0%9F%92%80/img/giphy.webp"/>
</div>

<div align="center">
  <h2>🐍 My Project 🐍</h2>
  <br>
  <img alt="snake eating my contributions" src="https://raw.githubusercontent.com/salesp07/salesp07/output/github-contribution-grid-snake.svg" />
  
  <br/>
</div>


<details>
<summary><b>Установить SIEM систему.</b></summary>

## Выбрана система Wazuh.
### Установка Wazuh на Ubuntu 22.04.
**Прежде всего, обязательно обновите систему.**

```bash
sudo apt update
```

**Загрузите и запустите помощник установки Wazuh.**

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

**Как только помощник завершит установку, в выходных данных будут показаны учетные данные для доступа и сообщение, подтверждающее, что установка прошла успешно.**

```bash
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
    User: admin
    Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```

**Вы можете найти пароли для всех пользователей индексатора Wazuh и Wazuh API в файле wazuh-passwords.txt внутри wazuh-install-files.tar. Чтобы их распечатать, выполните следующую команду:**

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

**Установленные Wazuh agent на платформах Linux и Windows.**
![img](/Project/img/installAgent.png)

**Сборка логов Wazuh agent на платформах Linux.**
![img](/Project/img/linux.png)

**Сборка логов Wazuh agent на платформах Windows.**
![img](/Project/img/Win.png)

</details>

💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀

<details>
<summary><b>Playbook для устранения уязвимости CVE-2021-41773 на веб-сервере Apache.</b></summary>

#### 1. **Описание уязвимости**
CVE-2021-41773 — это уязвимость, связанная с путевым обходом (Path Traversal) и раскрытием файловой системы на серверах, работающих с Apache HTTP Server версии 2.4.49. Она позволяет злоумышленникам получить доступ к файлам, находящимся за пределами корневого каталога веб-сервера, при определённых конфигурациях. При активированном параметре "require all denied" в конфигурации злоумышленник может использовать специально сформированные запросы для обхода ограничений доступа и чтения конфиденциальных файлов.

#### 2. **Цель**
Обновить уязвимую версию веб-сервера Apache до безопасной версии, а также убедиться, что конфигурация сервера надёжно защищена от подобных атак.

#### 3. **План действий**

##### 3.1. **Проверка текущей версии веб-сервера Apache**
1. Подключитесь к серверу с правами администратора.
2. Выполните команду для проверки версии Apache:
   ```bash
   apachectl -v
   ```
   Либо:
   ```bash
   httpd -v
   ```
3. Если версия Apache — 2.4.49, сервер уязвим к CVE-2021-41773. Если версия 2.4.50, также следует провести обновление, так как эта версия уязвима к другой уязвимости (CVE-2021-42013).

##### 3.2. **Резервное копирование конфигураций**
1. Сделайте резервную копию конфигурационных файлов Apache, чтобы избежать потери данных при обновлении:
   ```bash
   cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
   cp -r /etc/httpd/conf.d/ /etc/httpd/conf.d.bak/
   ```

##### 3.3. **Обновление Apache**
1. Обновите Apache до версии 2.4.51 или более новой (в которой устранена уязвимость CVE-2021-41773):
   - Для серверов на базе **Debian/Ubuntu**:
     ```bash
     sudo apt update
     sudo apt install apache2
     ```
   - Для серверов на базе **CentOS/RHEL**:
     ```bash
     sudo yum update httpd
     ```

2. После обновления проверьте новую версию Apache:
   ```bash
   apachectl -v
   ```

##### 3.4. **Проверка конфигурации безопасности**
1. Откройте конфигурационный файл Apache:
   ```bash
   sudo nano /etc/httpd/conf/httpd.conf
   ```
   или
   ```bash
   sudo nano /etc/apache2/apache2.conf
   ```
2. Убедитесь, что в конфигурации правильно настроены директивы безопасности:
   - Убедитесь, что в разделе конфигурации ваших директорий присутствуют следующие строки:
     ```apache
     <Directory />
         Require all denied
     </Directory>
     ```
   - Проверьте, что директивы `Options` и `AllowOverride` настроены корректно для предотвращения возможности злоупотребления пользовательскими запросами.
   
3. Перезапустите Apache для применения изменений:
   - Для **Debian/Ubuntu**:
     ```bash
     sudo systemctl restart apache2
     ```
   - Для **CentOS/RHEL**:
     ```bash
     sudo systemctl restart httpd
     ```

##### 3.5. **Проверка устранения уязвимости**
1. Проверьте, что уязвимость закрыта, используя специализированные инструменты для сканирования уязвимостей, такие как **Nmap** с соответствующими скриптами или **Nessus**.
2. Если у вас есть возможность, выполните тестирование на наличие уязвимости CVE-2021-41773 вручную, сформировав запрос типа:
   ```
   http://<server_ip>/?/../../../../../../etc/passwd
   ```
   Если запрос возвращает ошибку 403 или 404, это означает, что сервер защищён.

##### 3.6. **Мониторинг и логирование**
1. Включите и настройте ведение логов для мониторинга подозрительных запросов:
   - Убедитесь, что директива `LogLevel` установлена в `warn` или выше:
     ```apache
     LogLevel warn
     ```
   - Проверьте файлы журналов для выявления попыток эксплуатации уязвимости:
     ```bash
     tail -f /var/log/httpd/access_log /var/log/httpd/error_log
     ```

##### 3.7. **Заключительные действия**
1. Оповестите команду IT и SOC о завершении обновления и устранении уязвимости.
2. Внесите изменения в документацию о конфигурации сервера и проведённых обновлениях.

#### 4. **Важные замечания**
- CVE-2021-41773 также была исправлена в версии Apache 2.4.51 и выше, поэтому рекомендуется всегда использовать актуальные версии серверного ПО.
- Важно регулярно обновлять серверное ПО и следить за публикациями о новых уязвимостях.
- Проверяйте конфигурацию безопасности веб-сервера после каждого обновления или изменения конфигурационных файлов.

#### 5. **Последующие действия и улучшения безопасности**

После устранения уязвимости CVE-2021-41773, рекомендуется провести следующие шаги для повышения общей безопасности веб-сервера и предотвращения подобных инцидентов в будущем.

##### 5.1. **Обновление системы и компонентов**
1. Убедитесь, что все установленные пакеты и зависимости на сервере обновлены до последних стабильных версий для устранения других возможных уязвимостей:
   - На **Debian/Ubuntu**:
     ```bash
     sudo apt update && sudo apt upgrade
     ```
   - На **CentOS/RHEL**:
     ```bash
     sudo yum update
     ```

2. Отключите или удалите ненужные модули Apache, которые могут представлять дополнительную поверхность для атак. Например, если не используется CGI, отключите его:
   ```bash
   sudo a2dismod cgi
   ```

##### 5.2. **Ограничение доступа к конфиденциальным файлам**
1. Убедитесь, что важные системные файлы (например, `/etc/passwd`, конфигурационные файлы с паролями) недоступны через веб-сервер. Это можно сделать, добавив соответствующие правила в конфигурацию:
   - Для CentOS/RHEL:
     ```apache
     <FilesMatch "^\.ht">
         Require all denied
     </FilesMatch>
     ```
   - Для Ubuntu/Debian:
     ```apache
     <Files ~ "^\.ht">
         Require all denied
     </Files>
     ```

##### 5.3. **Усиление конфигурации Apache**
1. Активируйте модули безопасности такие как `mod_security` и `mod_evasive` для защиты от атак типа SQL-инъекций, XSS, brute-force и DoS:
   - Установка на **Debian/Ubuntu**:
     ```bash
     sudo apt install libapache2-mod-security2 libapache2-mod-evasive
     ```
   - Установка на **CentOS/RHEL**:
     ```bash
     sudo yum install mod_security mod_evasive
     ```

2. Настройте файлы конфигурации вышеупомянутых модулей для повышения уровня защиты:
   - Добавьте в конфигурацию правила для блокировки вредоносных запросов и ограничьте частоту запросов от одного IP-адреса.

##### 5.4. **Настройка HTTPS**
1. Убедитесь, что сервер использует безопасные протоколы и шифры для передачи данных по HTTPS. Если HTTPS не настроен, настройте его с помощью **Let`s Encrypt**:
   
   ```bash
   sudo apt install certbot python3-certbot-apache
   sudo certbot --apache
   ```

2. Обновите конфигурацию SSL для использования современных протоколов (например, TLS 1.2 и 1.3) и отключения устаревших версий, таких как SSLv3 и TLS 1.0:
   ```apache
   SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
   SSLCipherSuite HIGH:!aNULL:!MD5
   ```

##### 5.5. **Регулярные аудиты безопасности**
1. Регулярно проводите внутренние и внешние аудиты безопасности веб-сервера. Используйте инструменты для анализа конфигурации и поиска уязвимостей:
   - **Nessus** для периодического сканирования на уязвимости.
   - **Qualys** для оценки безопасности конфигурации веб-приложений.
   - **OWASP ZAP** или **Burp Suite** для анализа на наличие уязвимостей в веб-приложении.

2. Настройте автоматическое оповещение о новых уязвимостях в используемых версиях ПО, подписавшись на рассылки безопасности (например, от Apache и Linux-дистрибутивов).

##### 5.6. **Обучение и повышение осведомлённости**
1. Организуйте внутреннее обучение для членов команды IT и SOC о методах выявления и устранения подобных уязвимостей.
2. Разработайте и внедрите процедуры быстрого реагирования на инциденты, связанные с уязвимостями веб-серверов.

#### 6. **Заключение**
Устранение уязвимости CVE-2021-41773 — это важный шаг в защите вашего веб-сервера Apache. Однако важно понимать, что безопасность — это процесс, требующий постоянного улучшения. Следуя шагам в данном плейбуке, вы не только устраните текущую уязвимость, но и повысите общую безопасность инфраструктуры, снизив риск появления новых уязвимостей в будущем.

#### 7. **Ресурсы и ссылки**
- [Официальный сайт Apache HTTP Server](https://httpd.apache.org/)
- [Let`s Encrypt](https://letsencrypt.org/)
- [Руководство по безопасности Apache](https://httpd.apache.org/docs/2.4/misc/security_tips.html)
- [CVE-2021-41773 на NVD](https://nvd.nist.gov/vuln/detail/CVE-2021-41773)
- [Модуль ModSecurity](https://modsecurity.org/)

</details>

💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀

<details>
<summary><b>Пример простого скрипта на Bash для автоматизированной проверки URL через API VirusTotal.</b></summary>

### Перед использованием убедитесь, что у вас есть API-ключ VirusTotal, и вы заменили `<YOUR_API_KEY>` на свой собственный ключ

### Скрипт: `check_url_virustotal.sh`

```bash
#!/bin/bash

# Ваш API-ключ VirusTotal
API_KEY="<YOUR_API_KEY>"

# URL для проверки
URL_TO_CHECK=$1

# Проверка, передан ли URL в качестве аргумента
if [ -z "$URL_TO_CHECK" ]; then
  echo "Использование: $0 <URL>"
  exit 1
fi

# Отправка запроса на VirusTotal
response=$(curl -s --request POST \
  --url https://www.virustotal.com/vtapi/v2/url/scan \
  --form apikey="$API_KEY" \
  --form url="$URL_TO_CHECK")

# Извлечение scan_id из ответа
scan_id=$(echo "$response" | jq -r '.scan_id')

if [ "$scan_id" == "null" ]; then
  echo "Ошибка: не удалось отправить URL на проверку."
  echo "Ответ: $response"
  exit 1
fi

echo "URL отправлен на проверку. Scan ID: $scan_id"
echo "Ожидание результатов..."

# Ожидание перед запросом результата (можно настроить)
sleep 15

# Получение результатов анализа
result=$(curl -s --request GET \
  --url "https://www.virustotal.com/vtapi/v2/url/report?apikey=$API_KEY&resource=$scan_id")

# Вывод результатов
positives=$(echo "$result" | jq -r '.positives')
total=$(echo "$result" | jq -r '.total')
permalink=$(echo "$result" | jq -r '.permalink')

if [ "$positives" == "null" ]; then
  echo "Ошибка: не удалось получить результаты анализа."
  echo "Ответ: $result"
  exit 1
fi

echo "Результаты анализа:"
echo "Положительных срабатываний: $positives из $total"
echo "Подробнее: $permalink"
```

---

### Шаги для использования:

1. Убедитесь, что у вас установлен `jq` (утилита для обработки JSON). Установить можно с помощью:
   ```bash
   sudo apt-get install jq   # Для Ubuntu/Debian
   sudo yum install jq       # Для CentOS/RHEL
   ```
2. Сохраните скрипт в файл, например, `check_url_virustotal.sh`.
3. Сделайте скрипт исполняемым:
   ```bash
   chmod +x check_url_virustotal.sh
   ```
4. Запустите скрипт, передав URL для проверки:
   ```bash
   ./check_url_virustotal.sh https://example.com
   ```

![img](/Project/img/chekVirus.png)
![img](/Project/img/onl.png)

---

### Что делает скрипт:

1. Отправляет указанный URL на анализ с помощью API VirusTotal.
2. Извлекает `scan_id` из ответа.
3. Ждёт 15 секунд (можно изменить это время), чтобы дать VirusTotal возможность завершить анализ.
4. Запрашивает результаты анализа и отображает:
   - Количество положительных срабатываний (`positives`).
   - Общее количество проверок (`total`).
   - Ссылку на полный отчёт на сайте VirusTotal.

---

### Примечания:
- VirusTotal API имеет ограничения на количество запросов (обычно 4 запроса в минуту для бесплатной версии). Учитывайте это, если планируете частое использование.
- Если вы работаете с большим количеством URL, можно модифицировать скрипт для чтения списка URL из файла и проверять их по очереди.


</details>

💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀

<details>
<summary><b>Установка setoolkit на ubuntu.</b></summary>

[Установка Social-Engineer Toolkit (SET)](/💀Task9/README.md)

</details>

💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀💀

<details>
<summary><b>Установите Elasticsearch с пакетом Debian.</b></summary>

![img](/Project/img/elk.png)

**Загрузите и установите публичный ключ подписи:**

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

**Сохраните определение репозитория в /etc/apt/sources.list.d/elastic-8.x.list:**

```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

**Вы можете установить пакет Elasticsearch Debian с помощью:**

```bash
sudo apt-get update && sudo apt-get install elasticsearch
```

**Скопируйте вывод терминала команды установки в локальный файл. В частности, вам понадобится пароль для встроенной учетной записи эластичного суперпользователя. Вывод также содержит команды, позволяющие запускать Elasticsearch как службу, которые вы будете использовать на следующем шаге.**

```bash
--------------------------- Security autoconfiguration information ------------------------------

Authentication and authorization are enabled.
TLS for the transport and HTTP layers is enabled and configured.

The generated password for the elastic built-in superuser is : <ELASTIC_PASSWORD>

If this node should join an existing cluster, you can reconfigure this with
'/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'
after creating an enrollment token on your existing cluster.

You can complete the following actions at any time:

Reset the password of the elastic built-in superuser with 
'/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.

Generate an enrollment token for Kibana instances with 
 '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.

Generate an enrollment token for Elasticsearch nodes with 
'/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node'.
```

**Рекомендуем хранить `elastic` пароль как переменную среды в вашей оболочке. Например:**

```bash
export ELASTIC_PASSWORD="your_password"
```

**Выполните следующие две команды, чтобы Elasticsearch работал как служба с использованием systemd. Это позволяет Elasticsearch запускаться автоматически при перезагрузке хост-системы. Подробности об этом и следующих шагах можно найти в разделе «Запуск Elasticsearch с помощью systemd».**

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```

**Настройте узел Elasticsearch для подключения.**
**Откройте файл конфигурации Elasticsearch в текстовом редакторе, например vim:**
```bash
sudo vim /etc/elasticsearch/elasticsearch.yml
```

**Раскомментируйте строку network.host: 192.168.0.1 и замените адрес на `localhost`. Например:**
```bash
network.host: localhost
```

**Теперь пришло время запустить службу Elasticsearch:**
```bash
sudo systemctl start elasticsearch.service
```
**Убедитесь, что Elasticsearch работает правильно.**

```bash
sudo curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

**Если все в порядке, команда возвращает такой ответ:**
```bash
{
  "name" : "Cp9oae6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_C_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "{version_qualified}",
    "build_type" : "{build_type}",
    "build_hash" : "f27399d",
    "build_flavor" : "default",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "{lucene_version}",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```
**Сгенерируйте токен регистрации узла:**

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```
**Скопируйте сгенерированный токен регистрации из выходных данных команды. Срок действия токена регистрации составляет 30 минут. Если команда elasticsearch-reconfigure-node возвращает ошибку «Неверный токен регистрации», попробуйте создать новый токен.**

## Установите Kibana с пакетом Debian

```bash
sudo apt-get update && sudo apt-get install kibana
```

**Запустите команду elasticsearch-create-enrollment-token с опцией -s kibana, чтобы сгенерировать токен регистрации Kibana:**
**Скопируйте сгенерированный токен регистрации из выходных данных команды.**
```bash
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
```
**Откройте файл конфигурации Kibana для редактирования:**
```bash
sudo vim /etc/kibana/kibana.yml
```
**Раскомментируйте строку server.host: localhost и замените адрес по умолчанию значением inet, которое вы скопировали из команды ifconfig. Например:**
```bash
server.host: 10.128.0.28
```
**Запустите сервис Kibana:**
```bash
sudo systemctl start kibana.service
```
**Запустите команду статуса, чтобы получить подробную информацию о сервисе Kibana.**
```bash
sudo systemctl status kibana
```
**Запуск Kibana может занять минуту или две, поэтому обновите страницу, если вы не видите подсказку сразу.**

**В выводе команды status URL-адрес отображается:**
- Адрес хоста для доступа к Kibana
- Шестизначный код подтверждения
- Например:

```bash
Kibana has not been configured.
Go to http://10.128.0.28:5601/?code=<code> to get started.
```
**Запишите код подтверждения.**

**Откройте веб-браузер по внешнему IP-адресу хост-компьютера Kibana, например: `http://<kibana-host-address>:5601`.**

**Когда Kibana запустится, вам будет предложено предоставить токен регистрации. Вставьте токен регистрации Kibana, который вы создали ранее.**

**Если вам будет предложено ввести код подтверждения `<code>`, скопируйте и вставьте шестизначный код, возвращенный командой статуса. Затем дождитесь завершения настройки.**

**Укажите `elastic` в качестве имени пользователя и укажите пароль `<ELASTIC_PASSWORD>`.**

</details>


