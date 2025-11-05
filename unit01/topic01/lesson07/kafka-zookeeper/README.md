# Kafka-кластер с Zookeeper и Kafka UI

Этот проект поднимает Kafka-кластер из трёх брокеров с Zookeeper и веб-интерфейсом Kafka UI.

## 1. Как развернуть Kafka-кластер

Требования: Docker и Docker Compose.

Из каталога с `docker-compose.yml`:

```bash
    docker compose up -d
    # или
    # docker-compose up -d
```

Поднимаются контейнеры: `zookeeper`, `kafka1`, `kafka2`, `kafka3`, `ui`.
Данные хранятся в именованных volume: `zk-data`, `zk-log`, `kafka1-data`, `kafka2-data`, `kafka3-data`.

## 2. Как проверить, что кластер работает

### Контейнеры и логи

```bash
    docker compose ps
    docker compose logs zookeeper
    docker compose logs kafka1
    docker compose logs kafka2
    docker compose logs kafka3
```

Состояние - `Up`, в логах не должно быть ошибок (`ERROR`).

### Быстрый тест через CLI

Зайти в первый брокер:

```bash
  docker compose exec kafka1 bash
```

Создать и посмотреть топик:

```bash
    kafka-topics --bootstrap-server kafka1:9092 --create --topic test-topic --partitions 3 --replication-factor 3
    
    kafka-topics --bootstrap-server kafka1:9092 --list
```

Появление `kafka-topic` означает, что кластер работает и брокеры видят друг друга.

## 3. Конфигурация и её смысл

### Zookeeper

- `ZOOKEEPER_CLIENT_PORT=2181` - порт для клиентов (Kafka, Kafka UI).
- `ZOOKEEPER_TICK-TIME=2000` -  базовый тайм-аут в мс.

### Kafka (общие параметры)

Для `kafka1`, `kafka2`, `kafka3`:

- `KAFKA_BROKER_ID` - уникальный ID брокера (1, 2, 3).
- `KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181` - адрес Zookeeper.
- `KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:<порт>` - на каком порту слушает брокер.
- `KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafkaN:<порт>` - как к брокеру обращаются другие контейнеры.
- `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT` - протокол безопасности (без шифрования)
- `KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT` - listener, который используют брокеры между собой.
- `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1` - репликация служебного топика `__consumer_offsets` (для локального стенда).

Данные брокеров лежат в `/var/lib/kafka/data`, смонтированных в volume `kafka1-data`, `kafka2-data`, `kafka3-data`.

## 4. Как проверить работу Kafka через Kafka UI

1. Убедиться, что кластер запущен:

```bash
  docker compose up -d
```

2. Открыть в браузере:

```text
http://localhost:8080
```

3. В интерфейсе выбрать кластер `local` и:

    - посмотреть список брокеров и топиков;
    - при желании создать топик `test-topic`;
    - отправить и прочитать сообщения.

Если Kafka UI видит все три брокера и корректно показывает топики и сообщения - кластер работает.

## 5. Остановка

```bash
  # остановить, сохранив данные
  docker compose down
    
  # остановить и удалить данные (тома)
  docker compose down -v
```