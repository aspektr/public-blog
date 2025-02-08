Репликация в ClickHouse осуществляется асинхронно и управляется с помощью ZooKeeper. Она обеспечивает высокую доступность, создавая несколько реплик одних и тех же данных на разных узлах.

### Ключевые особенности репликации в ClickHouse

✅ Движок **ReplicatedMergeTree** обеспечивает автоматическую синхронизацию данных.  
✅ **ZooKeeper** управляет репликами и предотвращает конфликты.  
✅ Автоматическое переключение на резервные реплики в случае сбоя.

### 1. Архитектура репликации в ClickHouse

### 2. Шаги для настройки репликации в ClickHouse

✅ **Шаг 1: Установить и настроить ZooKeeper**  
ZooKeeper необходим для управления реплицированными таблицами.

Установите ZooKeeper на каждом узле:

```bash
sudo apt-get install -y zookeeper
```

Конфигурация ZooKeeper (/etc/clickhouse-server/config.xml):

```xml
<zookeeper>
    <node index="1">
        <host>zoo1</host>
        <port>2181</port>
    </node>
    <node index="2">
        <host>zoo2</host>
        <port>2181</port>
    </node>
    <node index="3">
        <host>zoo3</host>
        <port>2181</port>
    </node>
</zookeeper>
```

Перезапустите ClickHouse:

```bash
sudo systemctl restart clickhouse-server
```

✅ **Шаг 2: Определите узлы кластера в ClickHouse (config.xml)**  
На всех узлах ClickHouse обновите /etc/clickhouse-server/config.xml:

```xml
<remote_servers>
    <my_cluster>
        <shard>
            <replica>
                <host>clickhouse1</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>clickhouse2</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <host>clickhouse3</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>clickhouse4</host>
                <port>9000</port>
            </replica>
        </shard>
    </my_cluster>
</remote_servers>

```

Перезапустите ClickHouse:

```bash
sudo systemctl restart clickhouse-server
```

✅ **Шаг 3: Создайте реплицированные таблицы на каждом узле**  
Каждая реплика должна содержать таблицу **ReplicatedMergeTree** с уникальным путем ZooKeeper.

На **clickhouse1** (Первая реплика):

```sql
CREATE TABLE events (
    event_id UInt32,
    event_time DateTime,
    user_id UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/shard1/events', 'replica_1')
ORDER BY event_time;

```

На **clickhouse2** (Вторая реплика для Шарда 1):

```sql
CREATE TABLE events (
    event_id UInt32,
    event_time DateTime,
    user_id UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/shard1/events', 'replica_2')
ORDER BY event_time;
```

✅ Каждая реплика использует один и тот же путь в ZooKeeper (`/clickhouse/tables/shard1/events`).  
✅ Имя реплики (`replica_1`, `replica_2`) должно быть уникальным для каждого узла.

✅ **Шаг 4: Создайте распределенную таблицу для маршрутизации запросов**  
Распределенная таблица направляет запросы на правильные шарды.

На всех узлах:

```sql
CREATE TABLE events_distributed ON CLUSTER my_cluster AS events
ENGINE = Distributed(my_cluster, default, events, rand());
```

✅ Запросы теперь автоматически маршрутизируются к доступным репликам.

✅ **Шаг 5: Проверьте статус репликации**  
Проверьте статус репликации:

```sql
SELECT * FROM system.replicas;
```

Для ручной синхронизации реплик:

```sql
SYSTEM SYNC REPLICA events;
```

✅ Если одна реплика выходит из строя, кластер продолжает работать.

✅ **Шаг 6: Тестирование репликации**  
Вставьте данные в распределенную таблицу:

```sql
INSERT INTO events_distributed VALUES (1, now(), 1001);
```

Проверьте, что данные реплицированы на узлы:

На **clickhouse1**:

```sql
SELECT * FROM events;
```

На **clickhouse2**:

```sql
SELECT * FROM events;
```

✅ Данные автоматически копируются между репликами!

### 3. Как ClickHouse обрабатывает отказ реплики

Если **clickhouse1** выходит из строя:

1. **ZooKeeper** обнаруживает сбой.
2. Запросы автоматически перенаправляются на **clickhouse2**.
3. Когда **clickhouse1** восстанавливается, выполните:

```sl
SYSTEM SYNC REPLICA events;
```

✅ Репликация обеспечивает нулевой простой!

### 4. Резюме: Шаги для настройки репликации

|Шаг|Действие|
|---|---|
|✅ Установите ZooKeeper|На всех узлах|
|✅ Настройте ZooKeeper в ClickHouse|Измените /etc/clickhouse-server/config.xml|
|✅ Определите узлы кластера|Обновите remote_servers в ClickHouse|
|✅ Создайте реплицированные таблицы|Используйте ReplicatedMergeTree на каждой реплике|
|✅ Создайте распределенную таблицу|Маршрутизация запросов между репликами|
|✅ Проверьте репликацию|Выполните SELECT * FROM system.replicas|
|✅ Обработайте отказ|Запросы автоматически перенаправляются|

🚀 **Рекомендация**:  
Используйте **ReplicatedMergeTree** для отказоустойчивости.  
Используйте **Distributed Table** для балансировки запросов между репликами.  
Используйте **SYSTEM SYNC REPLICA** для ручного восстановления узлов после сбоев.