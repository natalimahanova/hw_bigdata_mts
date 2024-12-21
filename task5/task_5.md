# ДЗ 5: Prefect
Для успешного выполнения должны быть выполнены пункты по настройке кластера из предыдущих ДЗ.

Подключаемся к Name Node
```bash
ssh team-11-nn
```
Заходим в пользователя hadoop
```bash
sudo -i -u hadoop
``` 
Активируем виртуальное окружение из предыдущего дз
```bash
source .venv/bin/activate
``` 
Установливаем prefect
```bash
pip install prefect
``` 
Создаем файл
```bash
nano prefect_flow_hw5.py
```
Заполняем кодом:

```python
from prefect import flow, task
from pyspark.sql import SparkSession, DataFrame
from pyspark.sql import functions as F
from pyspark.sql.types import IntegerType, StringType, DoubleType


@task(name="Create Spark Session")
def create_spark_session():
    spark = SparkSession.builder \
        .master("yarn") \
        .appName("spark-with-yarn") \
        .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
        .config("spark.hive.metastore.uris", "thrift://team-11-jn:9083") \
        .enableHiveSupport() \
        .getOrCreate()
    return spark

@task(name="Read csv-file")
def read_csv(spark: SparkSession, input_path: str = "/input/titanic.csv") -> DataFrame:
    return spark.read.csv(input_path, header=True, inferSchema=True)  

@task(name="Transform df")
def transform_df(df: DataFrame) -> DataFrame:
    mean_age_row = df.select(F.round(F.avg(F.col("Age")))).first()
    mean_age = mean_age_row[0] if mean_age_row is not None else 0 
    df = df.na.fill({"Age": mean_age})
    df = df.dropna(subset=["Survived", "Pclass"])
    df = df.withColumn("Age", df["Age"].cast(IntegerType()))
    df = df.withColumn("SibSp", df["SibSp"].cast(IntegerType()))
    df = df.withColumn("Parch", df["Parch"].cast(IntegerType()))
    df = df.withColumn("Fare", df["Fare"].cast(DoubleType()))
    df = df.withColumn("Pclass", df["Pclass"].cast(IntegerType()))
    df = df.withColumn("Sex", df["Sex"].cast(StringType()))
    df = df.withColumn("Embarked", df["Embarked"].cast(StringType())) 
    df = df.withColumn("LastName", F.when(F.col("Name").isNotNull(), F.split(F.col("Name"), ",").getItem(0)))
    df = df.withColumn("FareCategory", 
                    F.when(df["Fare"] < 10, "Low")
                        .when((df["Fare"] >= 10) & (df["Fare"] < 30), "Medium")
                        .otherwise("High"))
    return df

@task(name="Repartition df")
def repartition_df(df: DataFrame, n_partitions: int = 3) -> DataFrame:
    return df.repartition(n_partitions, "Pclass")

@task(name="Save parquet to HDFS")
def save_parquet(df: DataFrame, output_path: str = "/input/titanic_parquet"):
    df.write.parquet(output_path)

@task(name="Save table to Hive")
def save_table(df: DataFrame, hive_table: str = "titanic_table"):
    df.write.saveAsTable(hive_table, partitionBy="Pclass")

@flow()
def spark_flow(input_path: str, output_path: str, hive_table: str):
    spark = create_spark_session()
    raw_df = read_csv(spark, input_path)
    transformed_df = transform_df(raw_df)
    repartitioned_df = repartition_df(transformed_df)
    save_parquet(repartitioned_df, output_path)
    save_table(repartitioned_df, hive_table)

if __name__ == "__main__":
    spark_flow(
        input_path="/input/titanic.csv",
        output_path="/input/titanic_parquet",
        hive_table="titanic_table"
    )

   ``` 
Запускаем файл с кодом
   ```bash
   python3 prefect_flow_hw5.py
   ``` 
