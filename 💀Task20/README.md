### Установка и тестирование Suricata IDS

Suricata — это высокопроизводительная система обнаружения и предотвращения вторжений (IDS/IPS). Она позволяет анализировать сетевой трафик в реальном времени и выявлять подозрительную активность. В этом руководстве будут описаны шаги по установке Suricata, добавлению пользовательского правила для обнаружения FIN или SYN сканирования, а также перезагрузке службы.

#### 1. Установка Suricata

##### Установка в Ubuntu/Debian

1. Обновите систему и установите необходимые зависимости:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install software-properties-common -y
   ```

2. Добавьте репозиторий Suricata, чтобы получить последнюю версию:
   ```bash
   sudo add-apt-repository ppa:oisf/suricata-stable
   sudo apt update
   ```

3. Установите Suricata:
   ```bash
   sudo apt install suricata -y
   ```
![img](/Cyb04-onl/💀Task20/img/install.png)

4. Далее в файле /etc/default/suricata сверим значение параметра IFACE с именем внешнего интерфейса хоста:

    ```bash
    $ cat /etc/default/suricata | grep IFACE
    IFACE=eth0
    $ ip a
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel  state UP group default qlen 1000
        link/ether fa:16:3e:29:e3:e3 brd ff:ff:ff:ff:ff:ff
        inet 45.145.64.243/29 brd 45.145.64.247 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe29:e3e3/64 scope link 
           valid_lft forever preferred_lft forever
    ```
    Обратите внимание на строку ```IFACE=eth0``` и ```2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>```. Проверим, чтобы это же значение стояло в файле ```/etc/suricata/suricata.yaml``` в блоках ```pcap```, ```pfring``` и ```af-packet```. По умолчанию там установлено eth0 — совпадает с именем внешнего интерфейса.

#### 2. Добавление правила для FIN или SYN сканирования

После установки Suricata вы можете добавить пользовательское правило для обнаружения сетевого сканирования с использованием флагов FIN или SYN.

1. Перейдите в каталог с правилами и добавьте следующее собственное правило:
   ```bash
   sudo nano /var/lib/suricata/rules/local.rules
   ```

2. Добавьте следующее правило для обнаружения SYN-сканирования:
   ```bash
   alert tcp any any -> any any (flags:S; msg:"SYN scan detected"; sid:1000001; rev:1;)
   ```

3. Добавьте правило для обнаружения FIN-сканирования:
   ```bash
   alert tcp any any -> any any (flags:F; msg:"FIN scan detected"; sid:1000002; rev:1;)
   ```

   - **flags:S** — устанавливает фильтр для SYN-флага.
   - **flags:F** — устанавливает фильтр для FIN-флага.
   - **sid** — уникальный идентификатор правила.
   - **msg** — сообщение, которое будет записано в журнал при срабатывании правила.

4. Сохраните файл и закройте редактор.

5. Отредактируйте раздел и добавьте следующую выделенную - local.rulesстроку:
   ```bash
    sudo nano /etc/suricata/suricata.yaml
   ```
    Отредактируйте раздел и добавьте следующую ``` - local.rules``` строку:
   ```bash
    . . .
    rule-files:
        - suricata.rules
        - local.rules
    . . .
   ```

#### 3. Перезагрузка Suricata

После добавления правил нужно перезапустить Suricata, чтобы изменения вступили в силу.

##### В Ubuntu/Debian:
```bash
sudo systemctl restart suricata
```

#### 4. Проверка корректности

Чтобы убедиться, что Suricata работает с новыми правилами, можно использовать команду для проверки конфигурации:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Если конфигурация верна, вы увидите сообщение о успешной проверке.

#### 5. Тестирование правил

Для тестирования правил можно использовать утилиту `nmap` для выполнения SYN и FIN сканирования на локальной машине или в сети.

1. Установите `nmap`, если его нет:
   ```bash
   sudo apt install nmap -y  # для Ubuntu/Debian
   ```

2. Выполните SYN сканирование:
   ```bash
   sudo nmap -sS <IP-адрес>
   ```

3. Выполните FIN сканирование:
   ```bash
   sudo nmap -sF <IP-адрес>
   ```

4. После выполнения сканирования проверьте логи Suricata. Они обычно находятся в каталоге `/var/log/suricata/fast.log` или `/var/log/suricata/eve.json`:

   ```bash
   sudo tail -f /var/log/suricata/fast.log
   ```

   Вы должны увидеть сообщения о срабатывании правил для SYN или FIN сканирования.

![img](/Cyb04-onl/💀Task20/img/scan.png)


#### 6. Заключение

Теперь Suricata настроена для обнаружения SYN и FIN сканирования. Если вам нужно добавить другие правила или настроить более сложные параметры, вы можете редактировать конфигурационные файлы в `/etc/suricata/` и добавлять новые правила в `/etc/suricata

## Debug

Сообщение, которое вы видите:

```
W: detect: No rule files match the pattern /var/lib/suricata/rules/suricata.rules
```

указывает на то, что Suricata не может найти файл `suricata.rules` в указанной директории `/var/lib/suricata/rules/`. Это предупреждение (W) о том, что нет правил для анализа трафика, поэтому Suricata не сможет корректно работать в режиме обнаружения.

### Возможные причины и решения:

1. **Отсутствие правил**:
   По умолчанию Suricata может не иметь загруженных правил для анализа сетевого трафика. Для решения этой проблемы можно загрузить правила, например, с помощью `suricata-update`.

2. **Некорректный путь к правилам**:
   Возможно, конфигурационный файл `suricata.yaml` указывает на неверное расположение файлов правил. Проверьте, что путь к файлам правил правильный.

### Шаги для решения проблемы:

#### 1. Обновите правила с помощью suricata-update

`suricata-update` — это инструмент для загрузки и обновления правил. Выполните следующие команды, чтобы загрузить последние сигнатуры правил:

1. Запустите обновление правил:
   ```bash
   sudo suricata-update
   ```

2. После успешного обновления правил перезагрузите Suricata:
   ```bash
   sudo systemctl restart suricata
   ```

3. Проверьте, что теперь Suricata видит правила:
   ```bash
   sudo suricata -T -c /etc/suricata/suricata.yaml
   ```

   Теперь предупреждение должно исчезнуть, и вывод команды должен быть примерно таким:

   ```
   i: suricata: This is Suricata version 7.0.7 RELEASE running in SYSTEM mode
   i: Configuration provided was successfully loaded. Exiting.
   ```

#### 2. Проверьте путь к правилам в конфигурации

Если проблема не исчезла, возможно, путь к правилам в конфигурации `suricata.yaml` неверен.

1. Откройте файл конфигурации:
   ```bash
   sudo nano /etc/suricata/suricata.yaml
   ```

2. Найдите секцию `rule-files` и проверьте правильность пути:
   ```yaml
   rule-files:
     - /var/lib/suricata/rules/suricata.rules
   ```

   Убедитесь, что путь к правилам существует и содержит правильные файлы. Если нет, измените путь или скопируйте правила в нужный каталог.

3. После внесения изменений сохраните файл и перезапустите Suricata:
   ```bash
   sudo systemctl restart suricata
   ```

#### 3. Проверьте наличие и содержимое файла правил

Убедитесь, что файл правил действительно существует в указанной директории:

1. Проверьте наличие файла:
   ```bash
   ls /var/lib/suricata/rules/suricata.rules
   ```

2. Если файла нет, скопируйте или создайте его вручную.

#### Итог:

После выполнения вышеуказанных шагов, предупреждение должно исчезнуть, и команда `sudo suricata -T -c /etc/suricata/suricata.yaml` должна выводить успешное сообщение о загрузке конфигурации, например:

```
i: suricata: This is Suricata version 7.0.7 RELEASE running in SYSTEM mode
i: Configuration provided was successfully loaded. Exiting.
```

Если проблема продолжает возникать, проверьте конфигурацию и пути к правилам в файле `suricata.yaml`.