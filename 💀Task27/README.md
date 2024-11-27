# Создание интерактивного Python-скрипта, который автоматизирует часто повторяемые действия или выполняет парсинг сайта, включает несколько ключевых компонентов. В качестве примера, давайте создадим скрипт, который будет парсить сайт погоды и предоставлять пользователю текущую информацию о погоде в указанном городе.

## Для этого скрипта мы будем использовать библиотеку `requests` для получения данных с сайта и `BeautifulSoup` для их парсинга. Также добавим интерактивность через библиотеку `inquirer`, которая позволяет более удобно получать ввод от пользователя.

### Установка необходимых библиотек

Прежде чем начать, убедитесь, что у вас установлены необходимые библиотеки. Вы можете установить их с помощью `pip`:

```bash
pip install requests beautifulsoup4 inquirer
```

### Скрипт для получения информации о телефонах с сайта `onliner.by`.

```python
import requests
from bs4 import BeautifulSoup

def get_phones():
    URL = "https://catalog.onliner.by/mobile"
    HEADERS = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36"
    }

    response = requests.get(URL, headers=HEADERS)
    response.raise_for_status()

    soup = BeautifulSoup(response.text, 'html.parser')

    phones = []
    
    # Находим контейнеры с информацией о телефонах
    phone_items = soup.find_all('div', class_='catalog-form__offers-flex')

    for item in phone_items:
        # Извлекаем название телефона
        title_tag = item.find('div', class_='catalog-form__description catalog-form__description_primary catalog-form__description_base-additional catalog-form__description_font-weight_semibold catalog-form__description_condensed-other')
        title = title_tag.get_text(strip=True) if title_tag else 'Unknown'

        # Извлекаем цену телефона
        price_tag = item.find('div', class_='catalog-form__description catalog-form__description_huge-additional catalog-form__description_font-weight_bold catalog-form__description_condensed-other catalog-form__description_primary')
        price = price_tag.get_text(strip=True) if price_tag else 'Price not available'

        # Добавляем информацию о телефоне в список
        phones.append({
            'title': title,
            'price': price
        })

    return phones

def main():
    phones = get_phones()
    for phone in phones:
        print(f"Title: {phone['title']}, Price: {phone['price']}")

if __name__ == "__main__":
    main()
```

### Запуск скрипта

```bash
    python script.py
```
![img](/💀Task27/img/req.png)

### Описание

1. **Запрос к сайту:** Мы используем `requests.get` с заголовком `User-Agent` для имитации запроса от браузера. Это важно, так как некоторые сайты могут блокировать запросы от скриптов, если они не содержат заголовка `User-Agent`.

2. **Парсинг HTML:** С помощью `BeautifulSoup` мы находим все элементы, содержащие информацию о телефонах. Мы ищем элементы с классом `catalog-form__offers-flex`, который содержит информацию о каждом телефоне.

3. **Извлечение данных:** Для каждого телефона мы извлекаем название и цену. Если элемент не найден, устанавливаем значение по умолчанию.

4. **Вывод данных:** Выводим список телефонов с их названиями и ценами.

### Примечания

- **Структура HTML:** Структура сайта может изменяться, и в этом случае необходимо будет обновить селекторы, используемые для парсинга.
- **Задержки между запросами:** Если вы планируете парсить много данных, добавьте задержку между запросами, чтобы не перегружать сервер. Это можно сделать с помощью `time.sleep()`.

Помните, что при работе с веб-скрапингом важно соблюдать правила сайта, чтобы избежать блокировки и юридических последствий.