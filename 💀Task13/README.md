# Учетные записи и их настройки. 

### 1. Создать учетную запись с административными правами
```powershell
# Создать учетную запись с административными правами
$adminUsername = "AdminUser"
$adminPassword = "P@ssw0rd123!" | ConvertTo-SecureString -AsPlainText -Force
New-LocalUser -Name $adminUsername -Password $adminPassword -FullName "Admin User" -Description "Administrator account" -PasswordNeverExpires $true
Add-LocalGroupMember -Group "Administrators" -Member $adminUsername
```

### 2. Создать учетную запись с уникальным паролем и записать в хранилище
```powershell
# Создать уникальный пароль для учетной записи
$employeeUsername = "Ivan.Ivanov"
$employeePassword = [System.Web.Security.Membership]::GeneratePassword(12, 2) | ConvertTo-SecureString -AsPlainText -Force

# Сохранить пароль в зашифрованном виде (например, в файл)
$employeePassword | ConvertFrom-SecureString | Set-Content "C:\secure_password_$employeeUsername.txt"

# Создать учетную запись без административных прав
New-LocalUser -Name $employeeUsername -Password $employeePassword -FullName "Ivan Ivanov" -Description "Employee account"
Add-LocalGroupMember -Group "Users" -Member $employeeUsername

# Требовать смену пароля при следующем входе
Set-LocalUser -Name $employeeUsername -PasswordExpired $true
```

### 3. Отключить учетную запись «Гость»
```powershell
# Отключить учетную запись «Гость»
Set-LocalUser -Name "Guest" -AccountDisabled $true
```

### 4. Включить контроль учетных записей (UAC)
Для настройки уровня уведомлений UAC можно изменить реестр через PowerShell:
```powershell
# Включить UAC и установить уровень уведомления
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 5
```

### Дополнительные настройки:

1. **Создать учетные записи через PowerShell**: `New-LocalUser`.
2. **Добавление пользователей в группы**: `Add-LocalGroupMember`.
3. **Отключение пользователей**: `Set-LocalUser`.

## Сделаем ещё круче, используя PowerShell-скрипт для автоматизации этого процесса, так же дальнейшего декларирования и использования на других устройствах windows:

### Скрипт: `setup_users_and_security.ps1`
```powershell
# Скрипт для создания учетных записей и настройки безопасности

# Функция для создания учетной записи администратора
function Create-AdminUser {
    param (
        [string]$AdminUsername,
        [string]$AdminPassword
    )
    # Преобразование пароля в SecureString
    $securePassword = $AdminPassword | ConvertTo-SecureString -AsPlainText -Force

    # Создать учетную запись администратора
    New-LocalUser -Name $AdminUsername -Password $securePassword -FullName "Admin User" -Description "Administrator account" -PasswordNeverExpires $true
    Add-LocalGroupMember -Group "Administrators" -Member $AdminUsername

    Write-Host "Создана учетная запись администратора: $AdminUsername"
}

# Функция для создания учетной записи сотрудника
function Create-EmployeeUser {
    param (
        [string]$EmployeeUsername
    )
    # Генерация уникального пароля
    $EmployeePassword = [System.Web.Security.Membership]::GeneratePassword(12, 2) | ConvertTo-SecureString -AsPlainText -Force

    # Сохранить пароль в файл
    $PasswordPath = "C:\secure_password_$EmployeeUsername.txt"
    $EmployeePassword | ConvertFrom-SecureString | Set-Content $PasswordPath

    # Создать учетную запись без административных прав
    New-LocalUser -Name $EmployeeUsername -Password $EmployeePassword -FullName $EmployeeUsername -Description "Employee account"
    Add-LocalGroupMember -Group "Users" -Member $EmployeeUsername

    # Требовать смену пароля при следующем входе
    Set-LocalUser -Name $EmployeeUsername -PasswordExpired $true

    Write-Host "Создана учетная запись сотрудника: $EmployeeUsername"
    Write-Host "Пароль сохранен в файле: $PasswordPath"
}

# Функция для отключения учетной записи Гость
function Disable-GuestAccount {
    # Отключить учетную запись Гость
    Set-LocalUser -Name "Guest" -AccountDisabled $true
    Write-Host "Учетная запись Гость отключена"
}

# Функция для включения UAC
function Enable-UAC {
    # Включить UAC и установить уровень уведомления
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 5
    Write-Host "UAC включен"
}

# Основной скрипт

# 1. Создать учетную запись с административными правами
$AdminUsername = "AdminUser"
$AdminPassword = "P@ssw0rd123!" # Замените на необходимый пароль
Create-AdminUser -AdminUsername $AdminUsername -AdminPassword $AdminPassword

# 2. Создать учетную запись сотрудника с уникальным паролем
$EmployeeUsername = "Ivan.Ivanov"
Create-EmployeeUser -EmployeeUsername $EmployeeUsername

# 3. Отключить учетную запись Гость
Disable-GuestAccount

# 4. Включить контроль учетных записей (UAC)
Enable-UAC

Write-Host "Настройка учетных записей и безопасности завершена"
```

### Как запустить скрипт:

1. Откройте PowerShell с правами администратора.
2. Сохраните этот код в файл, например, `setup_users_and_security.ps1`.
3. Выполните команду для запуска скрипта:
   ```powershell
   .\setup_users_and_security.ps1
   ```

### Что делает скрипт:

1. **Создает учетную запись администратора** с именем и паролем, указанным в переменных `$AdminUsername` и `$AdminPassword`.
2. **Создает учетную запись сотрудника** с уникальным паролем и записывает этот пароль в файл (например, `C:\secure_password_Ivan.Ivanov.txt`).
3. **Отключает учетную запись Гость**.
4. **Включает UAC** с уровнем уведомлений по умолчанию.

## Настройка параметров безопасности на Windows Server 2019, включая политики паролей и PIN-кодов, можно использовать командлеты PowerShell и утилиты `Secedit` и `LGPO`. Ниже приведен скрипт, который настраивает парольную политику и политику PIN (для Windows Hello).

### Скрипт настройки параметров безопасности пароля и PIN:

```powershell
# Скрипт для настройки параметров безопасности на Windows Server 2019

# 1. Настройка политики паролей
function Configure-PasswordPolicy {
    # Вести журнал паролей - 5 последних паролей
    secedit /export /cfg C:\secpol.cfg
    (Get-Content C:\secpol.cfg).replace('PasswordHistorySize = 24', 'PasswordHistorySize = 5') | Set-Content C:\secpol.cfg
    secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /quiet

    # Максимальный срок действия паролей - 90 дней
    net accounts /maxpwage:90

    # Минимальная длина пароля - 8 символов
    net accounts /minpwlen:8

    # Пароль должен отвечать требованиям сложности - Включен
    secedit /export /cfg C:\secpol.cfg
    (Get-Content C:\secpol.cfg).replace('PasswordComplexity = 0', 'PasswordComplexity = 1') | Set-Content C:\secpol.cfg
    secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /quiet

    Write-Host "Политика паролей настроена"
}

# 2. Настройка политики блокировки учетной записи
function Configure-LockoutPolicy {
    # Пороговое значение блокировки - 5 ошибок
    net accounts /lockoutthreshold:5

    # Продолжительность блокировки УЗ - 15 мин
    net accounts /lockoutduration:15

    # Разрешить блокировку УЗ администратора - Включено
    secedit /export /cfg C:\secpol.cfg
    (Get-Content C:\secpol.cfg).replace('LockoutBadCount = 0', 'LockoutBadCount = 1') | Set-Content C:\secpol.cfg
    secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /quiet

    Write-Host "Политика блокировки настроена"
}

# 3. Настройка политики PIN (для Windows Hello)
function Configure-PinPolicy {
    # Открыть групповую политику для редактирования
    $PolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork\PINComplexity"

    # Требовать использование цифр - Включена
    New-ItemProperty -Path $PolicyPath -Name "RequireDigits" -Value 1 -PropertyType DWord -Force

    # Требовать использование строчных букв - Включена
    New-ItemProperty -Path $PolicyPath -Name "RequireLowercase" -Value 1 -PropertyType DWord -Force

    # Минимальная длина PIN-кода - 8 символов
    New-ItemProperty -Path $PolicyPath -Name "MinimumPINLength" -Value 8 -PropertyType DWord -Force

    # Срок действия - 90 дней
    New-ItemProperty -Path $PolicyPath -Name "Expiration" -Value 90 -PropertyType DWord -Force

    # Журнал - 5 последних паролей
    New-ItemProperty -Path $PolicyPath -Name "HistorySize" -Value 5 -PropertyType DWord -Force

    # Требовать использование специальных символов - Включена
    New-ItemProperty -Path $PolicyPath -Name "RequireSpecialChar" -Value 1 -PropertyType DWord -Force

    # Требовать использование прописных букв - Включена
    New-ItemProperty -Path $PolicyPath -Name "RequireUppercase" -Value 1 -PropertyType DWord -Force

    Write-Host "Политика PIN настроена"
}

# Основной скрипт

# 1. Настроить политику паролей
Configure-PasswordPolicy

# 2. Настроить политику блокировки учетной записи
Configure-LockoutPolicy

# 3. Настроить политику PIN для Windows Hello
Configure-PinPolicy

Write-Host "Все параметры безопасности настроены успешно"
```

### Описание настроек:

1. **Политика паролей**:
   - Вести журнал паролей (5 последних паролей) — это параметр, который ограничивает повторное использование старых паролей.
   - Максимальный срок действия паролей — 90 дней.
   - Минимальная длина пароля — 8 символов.
   - Требования к сложности пароля включены.

2. **Политика блокировки учетной записи**:
   - Порог блокировки после 5 неудачных попыток входа.
   - Продолжительность блокировки — 15 минут.
   - Блокировка учетной записи администратора включена.

3. **Политика PIN для Windows Hello**:
   - Требование использования цифр, прописных и строчных букв.
   - Минимальная длина PIN-кода — 8 символов.
   - Срок действия PIN-кода — 90 дней.
   - Журнал для хранения 5 последних PIN-кодов.
   - Требование использования специальных символов.

### Как запустить скрипт:

1. Сохраните скрипт в файл, например, `security_policy.ps1`.
2. Запустите PowerShell с правами администратора.
3. Выполните команду для запуска скрипта:
   ```powershell
   .\security_policy.ps1
   ```


## Cкрипт описанный выше можно использовать для Windows 8, 10, и 11 с некоторыми нюансами:

1. **Политики паролей и блокировки учетной записи**:
   - **Настройки паролей** и **блокировки учетных записей** применяются одинаково для всех версий Windows, начиная с Windows 7 и выше, включая Server 2019, Windows 8, 10, и 11. Использование команд `net accounts` и `secedit` подходит для этих систем.

2. **Политика PIN (Windows Hello)**:
   - Политика PIN через Windows Hello поддерживается на Windows 10 и Windows 11. Для Windows 8 функция Windows Hello не доступна. 
   - В скрипте используется настройка через реестр для Windows Hello, что актуально для Windows 10 и 11. Политики PIN работают через изменение реестра, который также совместим с этими версиями.

### Что стоит учитывать:

1. **Windows 8**:
   - Функциональность Windows Hello, в том числе политика PIN, не поддерживается в Windows 8, поэтому секцию конфигурации PIN можно пропустить при применении на этой версии.

2. **Windows 10/11**:
   - Политика PIN, описанная в скрипте, полностью поддерживается. Windows Hello и управление PIN-кодом — это встроенная функция в этих системах.

### Как адаптировать скрипт для Windows 8:

Для Windows 8 можно просто удалить или закомментировать раздел с конфигурацией PIN-кода, так как он не будет применяться:

```powershell
# Удалите или закомментируйте эту секцию для Windows 8:
# Configure-PinPolicy
```

Скрипт можно использовать без изменений для Windows 10 и Windows 11, так как все параметры, включая политику паролей, блокировки учетной записи и политику PIN-кода (Windows Hello), будут поддерживаться.

### Итог:

- **Windows 8**: Работает, но без политики PIN.
- **Windows 10/11**: Работает полностью.

## В скрипте `.\security_policy.ps1` используется утилита `Secedit` для настройки политики паролей и блокировки учетной записи. Однако утилита **LGPO** (Local Group Policy Object Utility) в скрипте не задействована, так как настройки производятся напрямую через PowerShell (в том числе через реестр для Windows Hello PIN). Теперь давайте рассмотрим, как работают утилиты `Secedit` и `LGPO`, и какие дополнительные настройки могут понадобиться для Windows Server 2019.

### Как используется утилита **Secedit**:

`Secedit` — это команда, которая позволяет управлять параметрами локальной политики безопасности на уровне системы.

1. **Экспорт текущих параметров безопасности**:
   В скрипте команда `secedit /export` используется для экспорта текущей конфигурации политики безопасности в файл (`C:\secpol.cfg`). Пример:
   ```powershell
   secedit /export /cfg C:\secpol.cfg
   ```

2. **Изменение параметров безопасности**:
   После экспорта скрипт изменяет содержимое файла конфигурации, чтобы настроить определённые параметры, такие как:
   - Ведение журнала паролей.
   - Включение требования сложности паролей.
   - Разрешение блокировки учетной записи администратора.

   Пример замены строки в конфигурации для включения требования к сложности паролей:
   ```powershell
   (Get-Content C:\secpol.cfg).replace('PasswordComplexity = 0', 'PasswordComplexity = 1') | Set-Content C:\secpol.cfg
   ```

3. **Применение новых настроек**:
   После изменения файла конфигурации командой `secedit /configure` скрипт загружает обновлённые настройки в систему:
   ```powershell
   secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /quiet
   ```

### Нужно ли использовать утилиту **LGPO**?

Утилита **LGPO** (Local Group Policy Object Utility) может быть полезна для автоматизации управления групповыми политиками, особенно если нужно изменять более сложные параметры, связанные с политикой безопасности или если настройки должны применяться через объект локальной политики.

**В данном скрипте** вместо использования LGPO для настроек PIN (Windows Hello), мы напрямую изменяем значения в реестре через PowerShell. Это нормально для управления PIN-политикой, однако можно также использовать **LGPO** для управления локальными групповыми политиками.

Если вам нужно гибко управлять групповыми политиками, то **LGPO** может быть полезна. Пример использования LGPO:
```powershell
lgpo.exe /t security-template.inf
```
Вы можете создать и применить шаблон локальных групповых политик, если потребуется более детальная настройка.

### Нужно ли производить дополнительные настройки на Server 2019?

Для Windows Server 2019 все представленные параметры скрипта должны работать корректно, так как:

- Политика паролей и блокировки учетной записи настраивается через `Secedit` и команду `net accounts`.
- Политика PIN-кодов через Windows Hello настраивается через реестр, что также подходит для Server 2019 с Windows Hello.

Однако **дополнительно** на Server 2019 вы можете учесть следующие моменты:

1. **Убедиться, что включена поддержка Windows Hello**:
   На серверных ОС (включая Server 2019), Windows Hello может быть отключена по умолчанию. Для её активации можно использовать групповую политику:
   - Пуск → Введите "gpedit.msc" → Конфигурация компьютера → Административные шаблоны → Система → Вход в систему → Разрешить использование Windows Hello для бизнесов.
   - Включите эту настройку.

2. **Синхронизация с Active Directory**:
   Если сервер находится в домене, синхронизация с политиками домена может перезаписать локальные настройки. В таких случаях лучше настраивать политики через GPO (групповые политики) на уровне домена.

### Заключение:

1. **Secedit** используется для управления политикой паролей и блокировки учетных записей, а изменения параметров Windows Hello (PIN) настраиваются через реестр.
2. **LGPO** не используется в скрипте, но может быть задействована для управления групповыми политиками, если это требуется.
3. **Дополнительные настройки** на Windows Server 2019 могут включать активацию Windows Hello, если это не было сделано ранее, а также возможную настройку через GPO для доменных серверов.