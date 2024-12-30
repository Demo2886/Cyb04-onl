# Установить SIEM систему.
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

