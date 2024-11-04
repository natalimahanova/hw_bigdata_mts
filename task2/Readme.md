
# Разберем настройку Yarn:

#### Срузу можно продемонстровать результат работы:
![image](https://github.com/user-attachments/assets/f437ae46-40c1-4d1a-8950-4a9417a3d5ef)

Используем уже известные нам команды для первоначальной настройки и входа:

шаг 1
```bash
ssh team@host
```
шаг 2
```bash
sudo -i -u hadoop
```
шаг 3
```bash
ssh team-k-nn
```
шаг 4
```bash
cd hadoop-x.x.0/etc/hadoop
```

 
## Настройка конфига MapReduce

Маленький ликбез к тому, что это и зачем оно нам надо вообще:

Конфигурация MapReduce в Hadoop необходима для настройки и управления процессами обработки данных, которые выполняются в рамках модели MapReduce. Эта модель делит задачи обработки данных на два основных этапа: Map и Reduce. Каждый из этих этапов требует определенных настроек, которые могут включать в себя:

1. Определение фреймворка:

   • В конфигурационном файле mapred-site.xml указывается, какой фреймворк будет использоваться для выполнения задач MapReduce. Например, это может быть YARN (Yet Another Resource Negotiator), который управляет ресурсами кластера.

2. Настройки ресурсов:

   • Конфигурация позволяет задать, сколько ресурсов (CPU, память) будет выделено для каждого экземпляра задачи Map или Reduce. Это важно для оптимизации производительности и эффективного использования ресурсов кластера.

3. Настройки ввода/вывода:

   • В конфигурации можно указать форматы ввода и вывода данных, а также пути к данным, которые будут обрабатываться. Это включает в себя настройки для работы с HDFS (Hadoop Distributed File System) и другими источниками данных.

4. Настройки для специфичных задач:

   • Можно задать параметры, специфичные для конкретных задач, такие как количество мапперов и редьюсеров, настройки для шифрования, управление ошибками и т.д.

5. Управление зависимостями:

   • Конфигурация может также включать информацию о необходимых библиотеках и зависимостях, которые должны быть доступны во время выполнения задач.

6. Мониторинг и логирование:

   • Настройки могут включать параметры для управления логированием и мониторингом выполнения задач, что помогает в отладке и анализе производительности.

Таким образом, файл конфигурации mapred-site.xml является ключевым элементом, который позволяет настроить поведение системы обработки данных в Hadoop и адаптировать её под конкретные требования вашего приложения или рабочего процесса.

```bash
nano mapred-site.xml # Заходим в конфиг MapReduce
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

Опять же немного о том, что такое конфиг yarn и зачем оно нам надо.
Конфигурация YARN (Yet Another Resource Negotiator) в Hadoop необходима для управления ресурсами кластера и обеспечения эффективного выполнения распределённых вычислений. YARN является основным компонентом экосистемы Hadoop, который отвечает за управление ресурсами и планирование задач. Вот несколько ключевых причин, почему конфигурация YARN важна:

1. Управление ресурсами: 

   • YARN позволяет эффективно распределять ресурсы (CPU, память) между различными приложениями и задачами, работающими в кластере. Конфигурация помогает задать лимиты на использование ресурсов для каждого приложения.

2. Планирование задач:

   • YARN включает в себя планировщик, который управляет очередями задач и определяет порядок их выполнения. Конфигурация позволяет настроить различные параметры планирования, такие как приоритеты задач и количество одновременно выполняемых задач.

3. Масштабируемость:

   • YARN поддерживает масштабируемость, позволяя добавлять новые узлы в кластер без необходимости изменения конфигурации приложений. Это важно для обработки больших объёмов данных и увеличения производительности.

4. Поддержка различных типов приложений:

   • YARN не ограничивается только задачами MapReduce; он поддерживает выполнение различных типов приложений, таких как Spark, Tez и другие. Конфигурация позволяет интегрировать эти приложения в экосистему Hadoop.

5. Мониторинг и управление:

   • С помощью конфигурации YARN можно настроить параметры мониторинга и управления, что позволяет отслеживать состояние задач, использование ресурсов и производительность кластера.

6. Настройки безопасности:

   • YARN поддерживает различные механизмы безопасности, такие как Kerberos для аутентификации. Конфигурация позволяет задать параметры безопасности, чтобы защитить данные и ресурсы кластера.

7. Оптимизация производительности:

   • Правильная конфигурация YARN может значительно повысить производительность обработки данных за счёт оптимального распределения ресурсов и эффективного планирования задач.

В целом, конфигурация YARN играет критическую роль в обеспечении стабильной работы кластера Hadoop, управлении ресурсами и оптимизации выполнения вычислительных задач.

```bash
nano yarn-site.xml # заходим в кофин YARN
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
Теперь немного о том, зачем будет происзодить то, что в команде дальше.
History Server в YARN (Yet Another Resource Negotiator) — это компонент, который отвечает за хранение и отображение информации о завершённых заданиях. Он позволяет пользователям и администраторам получать доступ к данным о выполнении предыдущих приложений, что полезно для анализа производительности, отладки и мониторинга.

▎Основные функции History Server:

1. Хранение информации о заданиях:

   • History Server сохраняет метаданные о завершённых заданиях, таких как параметры конфигурации, информация о ресурсах, использованных задачами, время выполнения и статус каждой задачи.

2. Предоставление интерфейса для просмотра:

   • History Server предоставляет веб-интерфейс, через который пользователи могут просматривать историю выполненных приложений, их статус, логи и другие детали. Это позволяет анализировать производительность и выявлять возможные проблемы.

3. Поддержка различных приложений:

   • Хотя History Server часто ассоциируется с приложениями MapReduce, он также может хранить данные для других фреймворков, работающих на YARN, таких как Apache Spark.

4. Упрощение отладки:

   • Благодаря доступу к логам и метаданным завершённых заданий, History Server упрощает процесс отладки и позволяет разработчикам анализировать проблемы, которые возникли во время выполнения приложений.

5. Конфигурация:

   • History Server требует настройки в конфигурационных файлах YARN (например, yarn-site.xml), чтобы указать, где будут храниться файлы истории и как долго они будут сохраняться.

▎Как работает History Server:

• Когда приложение завершается, его метаданные записываются в файловую систему (например, HDFS) в формате, который может быть прочитан History Server.

• History Server периодически сканирует эти файлы и обновляет свою базу данных с информацией о завершённых заданиях.

• Пользователи могут получить доступ к этой информации через веб-интерфейс, обычно доступный по адресу http://<history-server-host>:<port>/.

В целом, History Server является важным инструментом для управления и мониторинга выполненных приложений в экосистеме Hadoop, позволяя пользователям получать ценную информацию о прошлых задачах и оптимизировать будущие вычисления.

```bash
mapred --daemon start historyserver # Запускаем historyserver
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
шаг 1
```bash
ssh team@host
sudo -i -u hadoop
ssh team-k-nn
cd hadoop-x.x.0/
```
шаг 2
```bash
mapred --daemon stop historyserver # Останавливаем historyserver
```
шаг 3
```bash
sbin/stop-yarn.sh # Останавливаем YARN
```
шаг 4
```bash
sbin/stop-dfs.sh # Останавливаем dfs
```
шаг 5
```bash
jps # Проверка nn
```
шаг 6
```bash
ssh team-k-dn-0 # Проверка dn-0
jps
exit
```
шаг 7
```bash
ssh team-k-dn-1 # Проверка dn-1
jps
exit
```
