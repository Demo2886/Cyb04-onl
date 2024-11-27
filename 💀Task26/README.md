# Пример интерактивного Bash-скрипта, который позволяет автоматизировать установку часто используемых утилит, таких как `wget`, `curl`, `docker` и `docker-compose`. Скрипт проверяет, установлены ли утилиты, и предлагает их установить, если они отсутствуют. Если утилита уже установлена, он выводит соответствующее сообщение и предлагает выбрать другие утилиты для установки.

```bash
#!/bin/bash

# Цвета для вывода
GREEN='\033[0;32m'
RED='\033[0;31m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Функция для проверки установки пакета
check_installed() {
    if command -v $1 &> /dev/null; then
        echo -e "${GREEN}$1 уже установлен.${NC}"
        return 0
    else
        echo -e "${RED}$1 не установлен.${NC}"
        return 1
    fi
}

# Функция для установки пакета
install_package() {
    local package=$1
    echo -e "${BLUE}Устанавливаю $package...${NC}"
    sudo apt-get update -y && sudo apt-get install -y $package

    if check_installed $package; then
        echo -e "${GREEN}$package успешно установлен.${NC}"
    else
        echo -e "${RED}Ошибка при установке $package.${NC}"
    fi
}

# Меню выбора утилит
menu() {
    echo "Выберите утилиты для установки:"
    echo "1) wget"
    echo "2) curl"
    echo "3) docker"
    echo "4) docker-compose"
    echo "5) Обновить систему (apt-get update & upgrade)"
    echo "6) Выйти"
    read -p "Введите номер действия: " choice
    case $choice in
        1)
            if ! check_installed wget; then
                install_package wget
            fi
            ;;
        2)
            if ! check_installed curl; then
                install_package curl
            fi
            ;;
        3)
            if ! check_installed docker; then
                echo -e "${BLUE}Устанавливаю Docker...${NC}"
                sudo apt-get update -y
                sudo apt-get install -y \
                    ca-certificates \
                    curl \
                    gnupg \
                    lsb-release

                sudo mkdir -m 0755 -p /etc/apt/keyrings
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

                echo \
                  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
                  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

                sudo apt-get update -y
                sudo apt-get install -y docker-ce docker-ce-cli containerd.io
                sudo usermod -aG docker $USER

                if check_installed docker; then
                    echo -e "${GREEN}Docker успешно установлен.${NC}"
                else
                    echo -e "${RED}Ошибка при установке Docker.${NC}"
                fi
            fi
            ;;
        4)
            if ! check_installed docker-compose; then
                echo -e "${BLUE}Устанавливаю Docker Compose...${NC}"
                sudo curl -L "https://github.com/docker/compose/releases/download/$(curl --silent "https://api.github.com/repos/docker/compose/releases/latest" | grep -Po '"tag_name": "\K.*?(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                sudo chmod +x /usr/local/bin/docker-compose

                if check_installed docker-compose; then
                    echo -e "${GREEN}Docker Compose успешно установлен.${NC}"
                else
                    echo -e "${RED}Ошибка при установке Docker Compose.${NC}"
                fi
            fi
            ;;
        5)
            echo -e "${BLUE}Обновление системы...${NC}"
            sudo apt-get update -y && sudo apt-get upgrade -y
            echo -e "${GREEN}Система успешно обновлена.${NC}"
            ;;
        6)
            echo "Выход..."
            exit 0
            ;;
        *)
            echo "Неверный выбор, попробуйте снова."
            ;;
    esac
}

# Основной цикл
while true; do
    menu
done
```

### Что делает этот скрипт (продолжение):

1. **Цветной вывод**: Зеленый цвет указывает на успешную установку или наличие программы, красный — на отсутствие или ошибку, синий — на процесс установки.
   
2. **Функции**:
   - **`check_installed`** — проверяет, установлена ли утилита. Если она установлена, выводит соответствующее сообщение и возвращает `0`. Если утилита не найдена — выводит сообщение об отсутствии и возвращает `1`.
   - **`install_package`** — отвечает за установку пакетов. Перед установкой выполняется обновление списка доступных пакетов с помощью `sudo apt-get update`. После установки вызывается функция `check_installed`, чтобы убедиться, что пакет был успешно установлен.
   
3. **Меню**:
   - Пользователю предлагается выбрать утилиту для установки.
   - Если утилита уже установлена, пользователь увидит сообщение о том, что она уже присутствует в системе.
   - Если утилита не установлена, она будет загружена и установлена.
   - Также есть возможность обновить систему с помощью опции `5` (выполняется `apt-get update` и `apt-get upgrade`).
   - Опция `6` позволяет выйти из скрипта.

4. **Установка Docker**:
   - Для установки Docker скрипт добавляет необходимые репозитории, загружает ключи GPG и устанавливает Docker через пакетный менеджер `apt`. Также добавляет текущего пользователя в группу Docker, чтобы можно было использовать Docker без `sudo`.

5. **Установка Docker Compose**:
   - Docker Compose скачивается напрямую с GitHub, с использованием последней версии, полученной из API GitHub. После скачивания бинарник помещается в `/usr/local/bin` и ему назначаются права на выполнение.

### Как использовать скрипт:

1. Сохраните скрипт в файл, например, `install_tools.sh`.
   
2. Сделайте скрипт исполняемым:
   ```bash
   chmod +x install_tools.sh
   ```

3. Запустите скрипт:
   ```bash
   ./install_tools.sh
   ```

4. Вы увидите интерактивное меню, в котором сможете выбрать нужные утилиты для установки.

### Пример работы:

```bash
Выберите утилиты для установки:
1) wget
2) curl
3) docker
4) docker-compose
5) Обновить систему (apt-get update & upgrade)
6) Выйти
Введите номер действия: 1
wget не установлен.
Устанавливаю wget...
...
wget успешно установлен.
```

![img](/💀Task26/img/inst.png)
![img](/💀Task26/img/ins.doc.png)

### Возможные улучшения:
- **Поддержка других дистрибутивов**: Сейчас скрипт рассчитан на работу с `apt` (пакетный менеджер для Debian/Ubuntu). Можно добавить поддержку других пакетных менеджеров, таких как `yum` для CentOS или `dnf` для Fedora.
- **Логирование**: Для более удобного отслеживания процессов установки можно добавить логирование действий в файл.
- **Обработка ошибок**: Добавить более продвинутую обработку ошибок, например, если установка прерывается, уведомить пользователя и предложить варианты решения (например, повторить попытку).

### Этот скрипт полезен для автоматизации установки часто используемых программ и может быть адаптирован под ваши нужды, добавив дополнительные утилиты или изменив логику работы.


# Пример простого PowerShell-скрипта, который выводит подробную информацию о системе (свободное место на дисках, RAM, CPU и т.д.), а также может использоваться для установки программ (например, установку файла MSI).

### Скрипт для вывода информации о системе и установки программы:

```powershell
# Вывод информации о системе
Write-Host "Системная информация:" -ForegroundColor Cyan

# Информация о процессоре
$cpu = Get-WmiObject Win32_Processor
Write-Host "Процессор: " $cpu.Name
Write-Host "Количество ядер: " $cpu.NumberOfCores
Write-Host "Количество логических процессоров: " $cpu.NumberOfLogicalProcessors
Write-Host "Частота: " $cpu.MaxClockSpeed "MHz"

# Информация о памяти (RAM)
$ram = Get-WmiObject Win32_PhysicalMemory
$totalRam = ($ram.Capacity | Measure-Object -Sum).Sum / 1GB
Write-Host "Оперативная память (RAM): " [math]::Round($totalRam, 2) " GB"

# Свободное место на дисках
Write-Host "Свободное место на дисках:"
Get-WmiObject Win32_LogicalDisk | ForEach-Object {
    if ($_.DriveType -eq 3) {
        $freeSpace = [math]::Round($_.FreeSpace / 1GB, 2)
        $totalSpace = [math]::Round($_.Size / 1GB, 2)
        Write-Host "Диск:" $_.DeviceID "Свободно:" $freeSpace "GB из" $totalSpace "GB"
    }
}

# Операционная система
$os = Get-WmiObject Win32_OperatingSystem
Write-Host "Операционная система: " $os.Caption
Write-Host "Версия: " $os.Version
Write-Host "Архитектура системы: " $os.OSArchitecture

# Информация о сетевых интерфейсах
Write-Host "Сетевые интерфейсы:"
$networkAdapters = Get-WmiObject Win32_NetworkAdapterConfiguration | Where-Object { $_.IPEnabled -eq $true }
foreach ($adapter in $networkAdapters) {
    Write-Host "Имя адаптера: " $adapter.Description
    Write-Host "MAC адрес: " $adapter.MACAddress
    Write-Host "IP адреса: " $adapter.IPAddress
}

# Установка программы (пример для MSI файла)
function Install-Program {
    param (
        [string]$msiPath
    )

    if (Test-Path $msiPath) {
        Write-Host "Установка программы..."
        Start-Process "msiexec.exe" -ArgumentList "/i", "`"$msiPath`"", "/quiet", "/norestart" -Wait
        Write-Host "Программа установлена."
    } else {
        Write-Host "Файл MSI не найден. Проверьте путь: $msiPath" -ForegroundColor Red
    }
}

# Пример использования функции для установки программы
$msiFilePath = "C:\Path\To\Program.msi"
Install-Program -msiPath $msiFilePath
```

### Описание скрипта:
1. **Информация о системе**:
   - Выводится информация о процессоре (название, количество ядер, логических процессоров и частота).
   - Выводится информация о RAM (общий объём).
   - Выводится информация о дисках (свободное и общее пространство).
   - Печатается информация об операционной системе и версии.
   - Выводится информация о сетевых адаптерах (IP-адреса, MAC-адрес).

2. **Установка программы**:
   - В скрипте используется функция `Install-Program` для установки программы из MSI-файла.
   - Пример пути к файлу MSI задаётся переменной `$msiFilePath`. Если файл существует, запускается его установка через `msiexec`.

### Как использовать:
1. Сохраните скрипт в файл, например, `SystemInfoAndInstall.ps1`.
2. Измените путь к MSI-файлу в переменной `$msiFilePath` на актуальный путь к программе, которую вы хотите установить.
3. Запустите скрипт через PowerShell с администраторскими правами, так как для установки программ могут понадобиться права администратора.

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Path\To\SystemInfoAndInstall.ps1"
```