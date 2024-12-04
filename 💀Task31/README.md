
# Рассмотрим содержимое YAML-файла описывающего GitHub Actions Workflow, состоящий из двух задач (`jobs`): **SAST (Static Application Security Testing)** для проверки безопасности кода с помощью инструмента Bandit и **Image Scan** для проверки Docker-образа на уязвимости с помощью Docker Scout. Рассмотрим каждую секцию подробнее:

## Намеренная уязвимость безопасности веб-приложений в Django [ссылка на репозиторий](https://github.com/Demo2886/devsecops-pygoat).

```yaml
name: CI

on: [push]

jobs:
 sast_scan:
   name: Run Bandit Scan
   runs-on: ubuntu-latest

   steps:
   - name: Checkout code
     uses: actions/checkout@v2

   - name: Set up Python
     uses: actions/setup-python@v2
     with:
       python-version: 3.8

   - name: Install Bandit
     run: pip install bandit

   - name: Run Bandit Scan
     run: bandit -ll -ii -r . -f json -o bandit-report.json

   - name: Upload Artifact
     uses: actions/upload-artifact@v3
     if: always()
     with:
      name: bandit-findings
      path: bandit-report.json

 image_scan:
   name: Build Image and Run Image Scan
   runs-on: ubuntu-latest

   steps:
   - name: Checkout code
     uses: actions/checkout@v2

   - name: Set up Docker
     uses: docker-practice/actions-setup-docker@v1
     with:
      docker_version: '20.10.7'

   - name: Build Docker Image
     run: docker build -f Dockerfile -t myapp:latest .


   - name: Docker Scout Scan
     uses: docker/scout-action@v1.15.1
     with:
       dockerhub-user: ${{ secrets.REPO_USER }}
       dockerhub-password: ${{ secrets.REPO_PWD }}
       command: quickview,cves
       only-severities: critical,high
       sarif-file: scout-report.sarif

   - name: Upload Artifact
     uses: actions/upload-artifact@v3
     if: always()
     with:
       name: docker-scout-findings
       path: scout-report.sarif


```

---

### **1. Основная информация о Workflow**
- `name: CI`  
  Имя Workflow. Название удобно для отображения в интерфейсе GitHub.
  
- `on: [push]`  
  Указывает, что Workflow будет запускаться при каждом push в репозиторий.

---

### **2. Задача: `sast_scan`**  
**Цель**: Анализ исходного Python-кода на уязвимости с использованием Bandit.

#### **Параметры**
- `runs-on: ubuntu-latest`: Используется виртуальная машина с последней версией Ubuntu для выполнения задачи.

#### **Шаги**
1. **Checkout code**: 
   ```yaml
   uses: actions/checkout@v2
   ```
   Скачивает код из репозитория для выполнения анализа.

2. **Set up Python**: 
   ```yaml
   uses: actions/setup-python@v2
   with:
     python-version: 3.8
   ```
   Устанавливает Python версии 3.8 для выполнения Python-скриптов.

3. **Install Bandit**: 
   ```yaml
   run: pip install bandit
   ```
   Устанавливает инструмент **Bandit**, предназначенный для статического анализа безопасности Python-кода.

4. **Run Bandit Scan**: 
   ```yaml
   run: bandit -ll -ii -r . -f json -o bandit-report.json
   ```
   Запускает сканирование:
   - `-ll`: Показывает все предупреждения, включая низкий уровень риска.
   - `-ii`: Выводит дополнительную информацию о проблемах.
   - `-r .`: Сканирует текущий каталог рекурсивно.
   - `-f json`: Форматирует результат в JSON.
   - `-o bandit-report.json`: Сохраняет отчет в файл `bandit-report.json`.

[Артефакт сканирования Bandit](/💀Task31/artifacts/bandit-report.json)

---

![img](/💀Task31/img/bandit.png)

---

5. **Upload Artifact**: 
   ```yaml
   uses: actions/upload-artifact@v3
   with:
     name: bandit-findings
     path: bandit-report.json
   ```
   Сохраняет отчет о сканировании как артефакт в GitHub, чтобы он был доступен для скачивания.

---

### **3. Задача: `image_scan`**  
**Цель**: Создание Docker-образа и проверка его на уязвимости с помощью Docker Scout.

#### **Параметры**
- `runs-on: ubuntu-latest`: Выполняется на виртуальной машине с Ubuntu.

#### **Шаги**
1. **Checkout code**: 
   ```yaml
   uses: actions/checkout@v2
   ```
   Скачивает исходный код репозитория.

2. **Set up Docker**: 
   ```yaml
   uses: docker-practice/actions-setup-docker@v1
   with:
     docker_version: '20.10.7'
   ```
   Устанавливает Docker версии 20.10.7 для выполнения задач.

3. **Build Docker Image**: 
   ```yaml
   run: docker build -f Dockerfile -t myapp:latest .
   ```
   Собирает Docker-образ:
   - `-f Dockerfile`: Указывает Dockerfile для сборки.
   - `-t myapp:latest`: Присваивает образу тег `myapp:latest`.

4. **Docker Scout Scan**:  
   Выполняет анализ Docker-образа с помощью Docker Scout. Этот шаг представлен двумя альтернативами:
   - Закомментированный пример ручной установки Docker Scout через скрипт.
   - Используемая версия с Docker Scout Action:
     ```yaml
     uses: docker/scout-action@v1.15.1
     with:
       dockerhub-user: ${{ secrets.REPO_USER }}
       dockerhub-password: ${{ secrets.REPO_PWD }}
       command: quickview,cves
       only-severities: critical,high
       sarif-file: scout-report.sarif
     ```
     - `dockerhub-user` и `dockerhub-password`: Используются для аутентификации в Docker Hub. Секреты должны быть заранее настроены в репозитории.
     - `command: quickview,cves`: Выполняет команду для анализа образа и обнаружения уязвимостей (CVEs).
     - `only-severities: critical,high`: Показывает только уязвимости высокого или критического уровня.
     - `sarif-file`: Сохраняет отчет в формате SARIF в файл `scout-report.sarif`.

5. **Upload Artifact**: 
   ```yaml
   uses: actions/upload-artifact@v3
   with:
     name: docker-scout-findings
     path: scout-report.sarif
   ```
   Сохраняет отчет об анализе Docker-образа как артефакт.

---

[Артефакт сканирования Docker Scout](/💀Task31/artifacts/scout-report.sarif)

---

![img](/💀Task31/img/docker-scout-sh.png)
![img](/💀Task31/img/doc-scout-ui.png)

---

### **Общие моменты**
- Использование **`upload-artifact`** позволяет сохранять результаты сканирования, чтобы они были доступны для дальнейшего анализа или загрузки.
- Аутентификация в Docker Hub осуществляется с использованием секретов `REPO_USER` и `REPO_PWD`. Эти данные необходимо добавить в настройки репозитория.

### **Итог**
Данный Workflow охватывает две ключевые области DevSecOps:
1. Статический анализ кода (SAST) для поиска уязвимостей в Python-коде.
2. Анализ безопасности Docker-образа для обнаружения уязвимостей в используемых компонентах.