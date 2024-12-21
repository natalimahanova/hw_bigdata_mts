# ДЗ 4: Apache Spark

Для успешного выполнения должны быть выполнены пункты по настройке кластера из предыдущих ДЗ. 


### Скачиваем и настраиваем дистрибутив

 Подключаемся к Jump node
```bash
ssh team-11-nn
```
Скачиваем Spark 
```bash
wget https://dlcdn.apache.org/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz
``` 

Разархивируем
```bash
tar -xvzf spark-3.5.3-bin-hadoop3.tgz
``` 

Открываем конфиг ~/.profile
```bash
nano ~/.profile
``` 

Добавляем переменные окружения
```
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_HOME=/home/hadoop/spark-3.5.3-bin-hadoop3
export PATH=$PATH:$SPARK_HOME/bin
``` 

Активируем переменные окружения
```bash
source ~/.profile
```

В корневую директорию на Jump node скачиваем данные

```bash
wget https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv
```

Создадим в HDFS папку \input для хранения датасета, выполнив на Jump node:
```bash
hdfs dfs -mkdir /input
hdfs dfs -chmod g+w /input
hdfs dfs -put titanic.csv /input
```

В отдельном терминале на Jump node запускаем
```bash
 hive
 --hiveconf hive.server2.enable.doAs=false
 --hiveconf hive.security.authorization.enabled=false
 --service metastore
 1>> /tmp/metastore.log
 2>> /tmp/metastore.log
 ```

На Jump node выполняем
```bash
ssh team-11-nn
su team 
```

На Name node выполним установку pyspark:
```bash
sudo apt install python3.12-venv
su hadoop
cd ~
source ~/.profile
python3 -m venv .venv
source .venv/bin/activate
pip install pyspark
pip install onetl
python
 ```


Выполним код:

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import IntegerType, StringType, DoubleType
from onetl.file import FileDFReader
from onetl.connection import SparkHDFS
from onetl.file.format import CSV

# Создание spark session с yarn
spark = SparkSession.builder \
.master("yarn") \
.appName("spark-with-yarn") \
.config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
.config("spark.hive.metastore.uris", "thrift://team-11-jn:9083") \
.enableHiveSupport() \
.getOrCreate()
 
# Создание соединения hdfs
hdfs = SparkHDFS(host="team-11-nn", port=9000, spark=spark, cluster="test")

# Проверка статуса соединения
print(hdfs.check())

# Чтение данных из HDFS
reader = FileDFReader(connection=hdfs, format=CSV(delimiter=",", header=True), source_path="/input")
df = reader.run(["titanic.csv"])

# Проверка загруженных данных
df.printSchema()

# Получение текущего количества партиций
print(f"Количество партиций: {df.rdd.getNumPartitions()}")

# Замена пропусков "Age" на среднее значение
mean_age = df.select(F.round(F.avg(F.col("Age")))).first()[0]
df = df.na.fill({"Age": mean_age})
# Удаление строк если есть пропущенные значения в столбцах "Survived", "Pclass"
df = df.dropna(subset=["Survived", "Pclass"])

# Преобразование типов данных в 
df = df.withColumn("Age", df["Age"].cast(IntegerType()))
df = df.withColumn("SibSp", df["SibSp"].cast(IntegerType()))
df = df.withColumn("Parch", df["Parch"].cast(IntegerType()))
df = df.withColumn("Fare", df["Fare"].cast(DoubleType()))
df = df.withColumn("Pclass", df["Pclass"].cast(IntegerType()))
df = df.withColumn("Sex", df["Sex"].cast(StringType()))
df = df.withColumn("Embarked", df["Embarked"].cast(StringType())) 
# Добавление нового столбца с фамилией
df = df.withColumn("LastName", F.split(F.col("Name"), ",").getItem(0))
#Добавление нового столбца с категориями Fare: Low, Medium, High
df = df.withColumn("FareCategory", 
                   F.when(df["Fare"] < 10, "Low")
                    .when((df["Fare"] >= 10) & (df["Fare"] < 30), "Medium")
                    .otherwise("High"))


# Репартиционирование данных
df = df.repartition(3, "Pclass")

# Получение текущего количества партиций
print(f"Количество партиций: {df.rdd.getNumPartitions()}")

# Сохранение партиционированных данных в формате parquet
df.write.parquet("/input/titanic_parquet")

# Сохранение таблицы
df.write.saveAsTable("titanic_table", partitionBy="Pclass")

# Остановка Spark
spark.stop()
```

Проверим сохраненные файлы 
```bash
# в формате parquet 
hdfs dfs -ls /input/titanic_parquet
# в формате таблицы
hdfs dfs -ls /user/hive/warehouse
```