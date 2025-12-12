# Аналитическая платформа для мониторинга COVID-19 на основе рентгеновских снимков
   
  Проект представляет собой аналитическую платформу для эпидемиологического мониторинга COVID-19 на основе рентгеновских снимков. Платформа использует стек Hadoop/Spark для хранения, обработки и анализа больших объемов данных.
## Структура проекта

```sh
my_project/
    ├── clean_metadata.ipynb               # Скрипт предобработки данных
    ├── analysis.ipynb                  # Jupyter Notebook с SQL, PySpark и визуализацией
    ├── README.md                       # Инструкции по развертыванию и запуску
    ├── docker-compose.yml              # Файл для Docker Compose
    └── data/                           # Директория для вспомогательных файлов данных
        ├── metadata_cleaned.csv        # Очищенный файл метаданных (CSV)
        └── metadata.parquet            # Очищенный файл метаданных (Parquet)
```

## Развертывание

### Предварительные требования

*   Docker и Docker Compose (если используется Docker)
*   Python 3.6+

### 1. Развертывание HDFS и Spark (с использованием Docker Compose)

Этот проект использует Docker Compose для упрощения развертывания HDFS и Spark. Если вы не используете Docker, пропустите этот раздел и перейдите к ручной установке.

1.  **Установите Docker и Docker Compose:**

    *   [Инструкции по установке Docker](https://docs.docker.com/get-docker/)
    *   [Инструкции по установке Docker Compose](https://docs.docker.com/compose/install/)

2.  **Запустите инфраструктуру:**

    ```bash
    docker-compose up -d
    ```

    Эта команда запустит контейнеры для:

    *   `namenode` (HDFS NameNode)
    *   `datanode` (HDFS DataNode)
    *   `spark-master` (Spark Master)
    *   `spark-worker` (Spark Worker)
    *   `jupyter` (Jupyter Notebook)

3.  **Дождитесь запуска контейнеров:**

    Убедитесь, что все контейнеры успешно запущены, выполнив команду:

    ```bash
    docker-compose ps
    ```

    Все контейнеры должны иметь статус `Up`.

4.  **Web UI:**

    *   **HDFS NameNode:**  Доступен по адресу `http://localhost:9870` (или `http://<IP-адрес_контейнера>:9870`, если Docker запущен удаленно).
    *   **Spark Master:**  Доступен по адресу `http://localhost:8080` (или `http://<IP-адрес_контейнера>:8080`).
    *   **Spark History Server:** Доступен по адресу `http://localhost:18080` (или `http://<IP-адрес_контейнера>:18080`).
    *   **Jupyter Notebook:** Доступен по адресу `http://localhost:8888` (или `http://<IP-адрес_контейнера>:8888`).  Вам потребуется токен, который можно найти в логах контейнера `jupyter`.

### 2. Ручная установка HDFS и Spark (если не используете Docker)

1.  **Установите Java:**
    Убедитесь, что у вас установлена Java 8 или выше.
2.  **Установите Hadoop:**
    *   Скачайте дистрибутив Hadoop с [официального сайта Apache Hadoop](https://hadoop.apache.org/releases.html).
    *   Настройте переменные окружения `JAVA_HOME` и `HADOOP_HOME`.
    *   Настройте файлы `hdfs-site.xml`, `core-site.xml` и `mapred-site.xml` (см. документацию Hadoop).
    *   Отформатируйте NameNode: `hdfs namenode -format`
    *   Запустите HDFS: `start-dfs.sh`
3.  **Установите Spark:**
    *   Скачайте дистрибутив Spark с [официального сайта Apache Spark](https://spark.apache.org/downloads.html).
    *   Настройте переменные окружения `SPARK_HOME`.
    *   Запустите Spark Master и Worker (см. документацию Spark).

### 3. Загрузка данных в HDFS

1.  **Запустите контейнер NameNode (если используете Docker):**

    ```bash
    docker exec -it namenode bash
    ```

2.  **Создайте директории в HDFS:**

    ```bash
    hdfs dfs -mkdir -p /tmp/covid_dataset/metadata
    hdfs dfs -mkdir -p /tmp/covid_dataset/images
    hdfs dfs -mkdir -p /tmp/covid_dataset/processed
    ```

3.  **Загрузите файлы метаданных и изображения в HDFS:**

    ```bash
    hdfs dfs -put metadata_cleaned.csv /tmp/covid_dataset/metadata/
    hdfs dfs -put metadata.parquet /tmp/covid_dataset/metadata/
    # Загрузите изображения в соответствующие поддиректории в /tmp/covid_dataset/images
    ```

    **Примечание:** Если вы выполняете эти команды внутри Docker-контейнера, убедитесь, что файлы `metadata_cleaned.csv`, `metadata.parquet` и директория `images` находятся в доступном месте. Если вы запускаете команды вне контейнера, замените `localhost` на IP-адрес вашей HDFS NameNode.

### 4. Запуск анализа

1.  **Запустите Jupyter Notebook (если используете Docker):**
      Откройте Jupyter Notebook в браузере (обычно `http://localhost:8888`) и перейдите в директорию проекта.
2.  **Откройте и запустите `analysis.ipynb`:**
      Запустите все ячейки в ноутбуке, чтобы выполнить SQL-запросы, PySpark-анализ и визуализацию.
3.  **Запуск скрипта предобработки (опционально):**
    При необходимости, запустите скрипт `clean_metadata.py` для предобработки данных:

    ```bash
    python clean_metadata.py
    ```
    Убедитесь, что у вас установлены необходимые библиотеки (pandas, numpy).
### 5. Выполнение SQL-запросов

1.  **Инициализируйте SparkSession:**
    Убедитесь, что SparkSession настроен для работы с HDFS. В вашем Jupyter Notebook это должно выглядеть примерно так:

    ```python
    from pyspark.sql import SparkSession

    spark = SparkSession.builder \
        .appName("COVID-19 Analysis") \
        .config("spark.hadoop.fs.defaultFS", "hdfs://namenode:9000") \
        .getOrCreate()
    ```

2.  **Выполняйте SQL-запросы:**
    Используйте `spark.sql()` для выполнения SQL-запросов. Например:

    ```python
    result = spark.sql("SELECT * FROM covid_data LIMIT 10")
    result.show()
    ```

    **Примечание:** Замените `hdfs://namenode:9000` на фактический адрес вашей HDFS NameNode, если необходимо.  Убедитесь, что временное представление `covid_data` (или другое имя) создано перед выполнением запросов.

## Справочная информация

### Docker Compose Configuration (docker-compose.yml)

```yaml
version: "3.7"
services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop2.7.4-java8
    container_name: namenode
    ports:
      - "9870:50070" # Web UI
      - "9000:9000"  # HDFS port
    environment:
      - CLUSTER_NAME=test
    volumes:
      - namenode_data:/data
  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop2.7.4-java8
    container_name: datanode
    ports:
      - "50075:50075"
    environment:
      - CLUSTER_NAME=test
    volumes:
      - datanode_data:/data
    depends_on:
      - namenode
  spark-master:
    image: bitnami/spark:3.3
    container_name: spark-master
    ports:
      - "8080:8080"
      - "7077:7077"
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_DIR_IN_IMAGE=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - namenode
      - datanode
  spark-worker:
    image: bitnami/spark:3.3
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_DIR_IN_IMAGE=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - spark-master
  jupyter:
    image: bitnami/spark:3.3
    container_name: jupyter
    ports:
      - "8888:8888"
    volumes:
      - .:/home/jovyan/work
    environment:
      - JUPYTER_ENABLE_LAB=yes
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_DIR_IN_IMAGE=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - spark-master
      - spark-worker
volumes:
  namenode_data:
    driver: local
  datanode_data:
    driver: local