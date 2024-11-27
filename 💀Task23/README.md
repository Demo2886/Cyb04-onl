Чтобы установить Elasticsearch на Ubuntu 22.04 с использованием Docker Compose, выполните следующие шаги:

### Шаг 1: Установите Docker и Docker Compose

Если у вас еще не установлен Docker и Docker Compose, выполните следующие команды:

1. **Обновите пакеты и установите необходимые зависимости:**
   ```bash
   sudo apt update
   ```

2. **Установите Docker:**
   ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo usermod -aG docker $USER
   ```

### Шаг 2: Создайте файл `docker-compose.yml`

Создайте новый каталог для вашего проекта и перейдите в него:

```bash
$ mkdir ~/efk
$ cd ~/efk
```

Создайте файл `docker-compose.yml` с содержимым:

```yaml
services:
  # Deploy using the custom image automatically be created during the build process.
  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links: # Sends incoming logs to the elasticsearch container.
      - elasticsearch
    depends_on:
      - elasticsearch
    ports: # Exposes the port 24224 on both TCP and UDP protocol for log aggregation
      - 24224:24224
      - 24224:24224/udp

  elasticsearch:
    image: elasticsearch:8.7.1
    expose:
      - 9200
    environment:
      - discovery.type=single-node # Runs as a single-node
      - xpack.security.enabled=false
    volumes: # Stores elasticsearch data locally on the esdata Docker volume
      - esdata:/usr/share/elasticsearch/data

  kibana:
    image: kibana:8.7.1
    links: # Links kibana service to the elasticsearch container
      - elasticsearch
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    environment: # Defined host configuration
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

# Define the Docker volume named esdata for the Elasticsearch container.
volumes:
  esdata:
```

### Шаг 3: Настройка файлов сборки Fluentd

```bash
    $ mkdir fluentd/conf -p
    $ cd fluentd
    $ nano Dockerfile
```

#### Структура каталогов.
```bash
tree
- efk
    docker-compose.yml 
    - fluentd
        - conf 
            fluent.conf 
    Dockerfile
2 directories, 3 files
```

#### Шаг 3.1: Этот код извлекает образ Fluentd Debian Docker и устанавливает плагин Fluentd для Elasticsearch.

```bash
# fluentd/Dockerfile
FROM fluent/fluentd:v1.16-debian-1
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "5.3.0"]
USER fluent
```
#### Шаг 3.2: Настройте конфигурацию плагина Fluentd для Elasticsearch.

```bash
    $ cd conf
    nano fluent.conf
```

#### Добавьте следующие строки в конфигурацию:

```bash
    # bind fluentd on IP 0.0.0.0
    # port 24224
    <source>
      @type forward
      port 24224
      bind 0.0.0.0
    </source>

    # sendlog to the elasticsearch
    # the host must match to the elasticsearch
    # container service
    <match *.**>
      @type copy
      <store>
        @type elasticsearch_dynamic
        hosts elasticsearch:9200
        logstash_format true
        logstash_prefix fluentd
        logstash_dateformat %Y%m%d
        include_tag_key true
        tag_key @log_name
        include_timestamp true
        flush_interval 30s
      </store>
         <store>
        @type stdout
      </store>
    </match>
```

### Шаг 3: Запустите Elasticsearch

Теперь вы можете запустить Elasticsearch с помощью Docker Compose:

```bash
cd ~/efk
sudo docker-compose up -d
```

### Шаг 4: Проверьте установку

После того как контейнер будет запущен, вы можете проверить, работает ли Elasticsearch, выполнив следующий запрос:

```bash
curl -X GET "localhost:9200/"
```

![img](/💀Task23/img/elk.png)

Вы должны увидеть информацию о версии Elasticsearch и другую информацию.

### Шаг 5: Остановка и удаление контейнера

Чтобы остановить контейнер, выполните:

```bash
sudo docker-compose down
```

### Заключение

Теперь успешно установлeн Elasticsearch на Ubuntu 22.04 с использованием Docker Compose. Вы можете настроить его в соответствии с вашими требованиями, изменив файл `docker-compose.yml`.