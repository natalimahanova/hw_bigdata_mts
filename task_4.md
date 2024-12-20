# ДЗ 4: Apache Spark

Для успешного выполнения должны быть выполнены пункты по настройке кластера из предыдущих ДЗ.

### Скачиваем данные

```bash
wget https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv
```


### Создадим в HDFS папку \input для хранения датасета
```bash
hdfs dfs -mkdir /input
hdfs dfs -chmod g+w /input
hdfs dfs -put titanic.csv /input
```

###   Создадим файл spark_processing.py 

```bash
vim spark_processing.py
```

###  Добавим в файл код:

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import IntegerType, StringType, DoubleType
from onetl.file import FileDFReader
from onetl.file.format import CSV
from onetl.connection import SparkHDFS

# Создание spark session с yarn
spark = SparkSession.builder \
   .master("yarn") \
   .appName("spark-with-yarn") \
   .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
   .enableHiveSupport() \
   .getOrCreate()

# Создание соединения hdfs
hdfs = SparkHDFS(host="team-11-nn", port=9000, spark=spark, cluster="test")

# Проверка статуса соединения
print(hdfs.check())

# Создание файла-ридера
reader = FileDFReader(connection=hdfs, format=CSV(delimiter=",", header=True), source_path="/input")

# Загрузка данных
df = reader.run(["titanic.csv"])

# Проверка загруженных данных
print(df.count())
df.printSchema()

# Получение текущего количества партиций
print(f"Количество партиций: {df.rdd.getNumPartitions()}")

# Преобразование данных: конвертация
df = df.withColumn("Age", df["Age"].cast(IntegerType()))
df = df.withColumn("SibSp", df["SibSp"].cast(IntegerType()))
df = df.withColumn("Parch", df["Parch"].cast(IntegerType()))
df = df.withColumn("Fare", df["Fare"].cast(DoubleType()))
df = df.withColumn("Pclass", df["Pclass"].cast(IntegerType()))
df = df.withColumn("Sex", df["Sex"].cast(StringType()))
df = df.withColumn("Embarked", df["Embarked"].cast(StringType())) 

# Репартиционирование данных
df = df.repartition(3, "Pclass")

# Получение текущего количества партиций
print(f"Количество партиций: {df.rdd.getNumPartitions()}")

# Сохранение партиционированных данных в формате parquet
df.write.parquet("/input/titanic_parqut")

# Сохранение таблицы
df.write.saveAsTable("titanic_table", partitionBy="Pclass")

# Остановка Spark
spark.stop()
```
###  Запускаем файл

```bash
 python3 spark_processing.py
```
### Проверим сохраненные файлы 
```bash
# в формате parquet 
 hdfs dfs -ls /input/titanic_parquet
 # в формате таблицы
 hdfs dfs -ls /user/hive/warehouse
```