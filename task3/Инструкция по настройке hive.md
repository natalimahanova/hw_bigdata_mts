# Установка Apache HIVE

Заходим на сервер
``` bash
ssh team@176.109.91.13
```
Переходим в jump node (по умолчанию уже на ней)
``` bash
ssh 192.168.1.46
```
Качаем Apache Hive
``` bash
wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz
```
Разворачиваем Hive
``` bash
tar -xzvf apache-hive-4.0.1-bin.tar.gz 
```
Перейдем внутрь папки:
``` bash
cd apache-hive-4.0.1-bin
```
Установим переменную окружения
``` bash
export HIVE_HOME=/home/hadoop/apache-hive-4.0.1-bin 
```
Добавим исполняемые файлы в переменную окружения PATH
``` bash
export HIVE_HOME=/home/hadoop/apache-hive-4.0.1-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin
```
Создадим папки для работы:
подключимся к name node
``` bash
ssh 192.168.1.47
```
Установим Postgres
``` bash
sudo apt install postgresql
```
Переключимся на пользователя:
``` bash
sudo -i -u postgres
```
Подключимся к консоли:
``` bash
psql
```
Создадим бд, пользователя и дадим ему права:
```
CREATE DATABASE metastore;
CREATE USER hive WITH PASSWORD '123456';
GRANT ALL PRIVILEGES ON DATABASE "metastore" TO hive;
ALTER DATABASE metastore OWNER TO hive;
```
выход из консоли и из пользователя на Name node
```
\q
exit
```
Правим конфиг постгреса
``` bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```
Добавляем в файл в рздел connection settings:
```
listen_addresses = 'team-11-nn'
```
Правим файл pg_hba.conf:
``` bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```
Добавляем в IPv4 local connections:
host    metastore       hive            <ip_jumpnode>/32         password
и удаляем
host    all             all             127.0.0.1/32            scram-sha-256
Перезапускаем Postgres и проверяем статус:
``` bash
sudo systemctl restart postgresql
sudo systemctl status postgresql
```

создадим папку
``` bash
hdfs dfs -mkdir -p /user/hive/warehouse
```
Выдадим права:
``` bash
hdfs dfs -chmod g+w /user/hive/warehouse
hdfs dfs -chmod g+w /tmp
```
Вернемся назад:
``` bash
exit
```
Установим Postgres Client
``` bash
sudo apt install postgresql-client-16
```
Подключимся к бд
```
psql -h team-11-nn -p 5432 -U hive -W -d metastore
```
Выходим из консоли
```
\q
```
Скачаем драйвер jdbc
``` bash
wget https://jdbc.postgresql.org/download/postgresql-42.7.4.jar
```
Перейдем в папку с конфигом
``` bash
cd ../apache-hive-4.0.1-bin/conf
```
Подправим базовый конфиг HIVE (если он есть)
``` bash
nano hive-site.xml
```
Если его нет:
``` bash
cp hive-default.xml.template hive-site.xml
nano hive-site.xml
```
Удалем содержимое и вставляем:
``` xml
<configuration>
   <property>
       <name>hive.server2.authentication</name>
       <value>NONE</value>
   </property>
   <property>
       <name>hive.metastore.warehouse.dir</name>
       <value>/user/hive/warehouse</value>
   </property>
   <property>
       <name>hive.server2.thrift.port</name>
       <value>5433</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionURL</name>
       <value>jdbc:postgresql://team-11-nn:5432/metastore</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionDriverName</name>
       <value>org.postgresql.Driver</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionUserName</name>
       <value>hive</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionPassword</name>
       <value><postgre password></value>
   </property>
</configuration>
```
Отредактируем файл .profile в Hive
``` bash
nano ~/.profile
```
Активируем окружение
``` bash
source ~/.profile
```
Инициализируем Hive
``` bash
cd ../
bin/schematool -dbType postgres -initSchema
```
Запуск Hive
```
hive --hiveconf hive.server2.enable.doAs=false --hiveconf hive.security.authorization.enabled=false --service hiveserver2 1>> /tmp/hs2.log 2>> /tmp/hs2.log &
```
Проверка запуска
```
jps
```

