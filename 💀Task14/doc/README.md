## Настройка правил `iptables`, который разрешает все соединения по портам 80 и 443, а также разрешает подключение к порту 22 только из внутренней сети. 

### Сохраните этот код в файл, например, `setup_iptables.sh`:

```bash
#!/bin/bash

# Сброс всех текущих правил
iptables -F
iptables -X

# Установка политики по умолчанию
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Разрешение соединений по 80 и 443 портам
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Разрешение подключения к 22 порту только из внутренней сети (например, 192.168.1.0/24)
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Разрешение уже установленных соединений
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Логирование (по желанию)
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: " --log-level 4

echo "Правила iptables настроены."
```

### Инструкции по использованию:

1. Сохраните скрипт в файл `setup_iptables.sh`.
2. Сделайте файл исполняемым:
   ```bash
   chmod +x setup_iptables.sh
   ```
3. Запустите скрипт с правами суперпользователя:
   ```bash
   sudo ./setup_iptables.sh
   ```

### Примечания:
- Необходимо изменить диапазон IP-адресов (192.168.1.0/24) на тот, который соответствует вашей внутренней сети.

## Чтобы очистить все правила `iptables`, установить `UFW` (Uncomplicated Firewall) и настроить аналогичные правила, выполните следующие шаги.

### Скрипт для настройки `UFW`

Создаем скрипт, например, `setup_ufw.sh`, который будет содержать все необходимые команды:

```bash
#!/bin/bash

# Сброс всех текущих правил iptables
iptables -F
iptables -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# Установка UFW
apt update
apt install -y ufw

# Включение UFW
ufw enable

# Разрешение соединений по 80 и 443 портам
ufw allow 80/tcp
ufw allow 443/tcp

# Разрешение подключения к 22 порту только из внутренней сети (например, 192.168.1.0/24)
ufw allow from 192.168.1.0/24 to any port 22

# Проверка статуса и правил UFW
ufw status verbose

echo "Настройки UFW завершены."
```

### Инструкции по использованию:

1. **Создайте файл**:
   ```bash
   nano setup_ufw.sh
   ```

2. **Скопируйте и вставьте содержимое**:
   Вставьте приведенный выше код в редактор.

3. **Сохраните файл**:
   Нажмите `CTRL + X`, затем `Y`, и нажмите `Enter`.

4. **Сделайте файл исполняемым**:
   ```bash
   chmod +x setup_ufw.sh
   ```

5. **Запустите скрипт с правами суперпользователя**:
   ```bash
   sudo ./setup_ufw.sh
   ```

Теперь ваш сервер будет защищен с помощью `UFW`, и все необходимые правила будут настроены.
