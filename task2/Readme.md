
# Разберем настройку Yarn:

#### Срузу можно продемонстровать результат работы:
![image](https://github.com/user-attachments/assets/f437ae46-40c1-4d1a-8950-4a9417a3d5ef)

Подключаемся
```bash
ssh team@host
```

Переходим на пользователя hadoop
```bash
sudo -i -u hadoop
```

Переходим на name node
```bash
ssh team-19-nn
```

Переходим в папку дистрибутива
```bash
cd hadoop-x.x.0/etc/hadoop
```

 
## Настройка конфига MapReduce

Заходим в конфиг MapReduce
```bash
nano mapred-site.xml
```

Вставляем в конфиг MapReduce
```bash
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.application.classpath</name>
                <value>$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
        </property>
</configuration>
```
## Настройка конфига YARN

Заходим в конфиг YARN
```bash
nano yarn-site.xml
```
Вставляем в конфиг YARN 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```
## Копируем конфиги на data nodes

Прописываем проброску на data nodes и для YARN и для MapReduce
```bash
scp mapred-site.xml team-k-dn-0:/home/hadoop/hadoop-x.x.0/etc/hadoop
scp mapred-site.xml team-k-dn-1:/home/hadoop/hadoop-x.x.0/etc/hadoop

scp yarn-site.xml team-k-dn-0:/home/hadoop/hadoop-x.x.0/etc/hadoop
scp yarn-site.xml team-k-dn-1:/home/hadoop/hadoop-x.x.0/etc/hadoop
```

## Запускаем YARN
```bash
cd ../../
sbin/start-yarn.sh
```

Запускаем historyserver
```bash
mapred --daemon start historyserver
```

## Конфиги для веб-интерфейсов YARN и historyserver

Переходим на jn
```bash
ssh team-k-jn
```

Выходим из учетки hadoop
```bash
exit
```

Выполняем
```bash
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/ya
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/dh
```

YARN. Открываем конфиг и заменяем порт в listen и proxy_pass на порт 8088
```bash
 sudo nano /etc/nginx/sites-available/ya
```

historyserver. Открываем конфиг и заменяем порт в listen и proxy_pass на порт 19888
```bash
 sudo nano /etc/nginx/sites-available/dh
```

Включаем хосты
```bash
 sudo ln -s /etc/nginx/sites-available/ya /etc/nginx/sites-enabled/ya
 sudo ln -s /etc/nginx/sites-available/dh /etc/nginx/sites-enabled/dh
```

Запуск nginx
```bash
 sudo systemctl reload nginx
 exit
```
 
## Проверка работы UI YARN
Подключаемся к серверу с учетом проброшеннего порта
```bash
ssh -L 8088:team-k-nn:8088 team@host
```
Переходим в браузере по адресу
```bash
127.0.0.1:8088
```

## Проверка работы UI historyserver
Подключаемся к серверу с учетом проброшеннего порта
```bash
ssh -L 19888:team-k-nn:19888 team@host
```
Переходим в браузере по адресу
```bash
127.0.0.1:19888
```

## Настройка веб-интерфейсов NodeManager

Подключаемся
```bash
ssh team@host
sudo -i -u hadoop
```

Переключаемся на dn-0 и открываем yarn-site.xml
```bash
ssh team-k-dn-0
nano /home/hadoop/hadoop-x.x.0/etc/hadoop/yarn-site.xml
```

Вставляем в конфиг 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
       <property>
      		<name>yarn.nodemanager.webapp.address</name>
      		<value>0.0.0.0:8042</value>
  	</property>
       <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```

Переключаемся на dn-1 и открываем yarn-site.xml
```bash
ssh team-k-dn-1
nano /home/hadoop/hadoop-x.x.0/etc/hadoop/yarn-site.xml
```

Вставляем в конфиг 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
       <property>
      		<name>yarn.nodemanager.webapp.address</name>
      		<value>0.0.0.0:8042</value>
  	</property>
       <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```

Переходим на jn
```bash
ssh team-k-jn
```

Выходим из учетки hadoop
```bash
exit
```

Копируем конфигурации nginx для каждого интерфейса NodeManager
```bash
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/nm-0
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/nm-1
```

Открываем файл и заменяем 
Открываем 
```bash
sudo nano /etc/nginx/sites-available/nm-0
```
Заменяем строку с listen на:
```bash
listen 8043 default_server;
```
Заменяем строку с proxy_pass на:
```bash
proxy_pass http://team-19-nn:8042;
```

Открываем файл и заменяем 
Открываем 
```bash
sudo nano /etc/nginx/sites-available/nm-1
```
Заменяем строку с listen на:
```bash
listen 8044 default_server;
```
Заменяем строку с proxy_pass на:
```bash
proxy_pass http://team-k-nn:8042;
```

Включаем NodeManager в nginx
```bash
sudo ln -s /etc/nginx/sites-available/nm-0 /etc/nginx/sites-enabled/nm-0
sudo ln -s /etc/nginx/sites-available/nm-1 /etc/nginx/sites-enabled/nm-1
```

Перезапускаем nginx
```bash
sudo systemctl reload nginx
exit
```

## Проверка работы UI nm-0
Подключаемся к серверу с учетом проброшеннего порта
```bash
ssh -L 8043:team-k-dn-0:8042 team@host
```
Переходим в браузере по адресу
```bash
127.0.0.1:8043
```

Проверка работы UI nm-1
Подключаемся к серверу с учетом проброшеннего порта
```bash
ssh -L 8044:team-k-dn-1:8042 team@host
```
Переходим в браузере по адресу
```bash
127.0.0.1:8044
```


## Остановка сервисов

##### Выполняем
```bash
ssh team@host
sudo -i -u hadoop
ssh team-k-nn
cd hadoop-x.x.0/
```

Останавливаем historyserver
```bash
mapred --daemon stop historyserver
```

Останавливаем YARN
```bash
sbin/stop-yarn.sh
```

Останавливаем dfs
```bash
sbin/stop-dfs.sh
```

Проверка nn
```bash
jps
```

Проверка dn-0
```bash
ssh team-k-dn-0
jps
exit
```

Проверка dn-1
```bash
ssh team-k-dn-1
jps
exit
```
