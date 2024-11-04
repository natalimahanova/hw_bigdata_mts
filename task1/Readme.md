# Разберем установку и настройку Apache Hadoop.

## ▎Шаг 1: Настройка SSH

Перед тем как перейти к установке Hadoop, важно настроить SSH для удаленного доступа между узлами. Это необходимо для того, чтобы Hadoop мог управлять своими компонентами.

1. Установите SSH:

   На большинстве дистрибутивов Linux SSH уже установлен. Если нет, вы можете установить его с помощью следующей команды:

      sudo apt update
   sudo apt install openssh-server
   

2. Создайте SSH-ключи:

   Войдите под пользователем, который будет использоваться для работы с Hadoop (например, hadoopuser):

      su - hadoopuser
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   

3. Добавьте публичный ключ в authorized_keys:

   Выполните следующую команду, чтобы добавить ваш публичный ключ в файл authorized_keys, что позволит вам подключаться к вашему узлу без пароля:

      cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   

4. Проверьте SSH-соединение:

   Попробуйте подключиться к localhost:

      ssh localhost
   

   Если вы сможете подключиться без запроса пароля, настройка SSH выполнена успешно.

## ▎Шаг 2: Установка Hadoop

Теперь, когда SSH настроен, мы можем перейти к установке Hadoop. В этом примере мы будем рассматривать конфигурацию с одним узлом (jump node) и несколькими data nodes.

## ▎2.1: Jump Node (Master Node)

1. Скачайте и установите Hadoop на вашем jump node (например, hadoopuser):

      wget https://downloads.apache.org/hadoop/common/hadoop-3.x.x/hadoop-3.x.x.tar.gz
   tar -xzf hadoop-3.x.x.tar.gz
   sudo mv hadoop-3.x.x /usr/local/hadoop
   

2. Настройте переменные окружения в .bashrc:

      export HADOOP_HOME=/usr/local/hadoop
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64  # Убедитесь, что путь к Java правильный
   

3. Настройте конфигурационные файлы Hadoop в /usr/local/hadoop/etc/hadoop:

   • core-site.xml:
          <configuration>
       <property>
         <name>fs.defaultFS</name>
         <value>hdfs://localhost:9000</value>
       </property>
     </configuration>
     

   • hdfs-site.xml:
          <configuration>
       <property>
         <name>dfs.replication</name>
         <value>1</value>
       </property>
     </configuration>
     

   • mapred-site.xml:
          <configuration>
       <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
       </property>
     </configuration>
     

   • yarn-site.xml:
          <configuration>
       <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
       </property>
       <property>
         <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
         <value>org.apache.hadoop.mapred.ShuffleHandler</value>
       </property>
     </configuration>
     

4. Форматируйте HDFS:

      hdfs namenode -format
   

5. Запустите Hadoop:

      start-dfs.sh
   start-yarn.sh
   

## ▎2.2: Data Nodes

Теперь, если у вас есть дополнительные узлы (data nodes), выполните следующие шаги на каждом из них:

1. Установите Java и Hadoop аналогично jump node.

2. Настройте SSH так же, как вы сделали это на jump node.

3. Скопируйте конфигурационные файлы с jump node на data nodes:

   Вы можете использовать SCP для копирования конфигурационных файлов:

      scp -r /usr/local/hadoop/etc/hadoop hadoopuser@data-node-ip:/usr/local/hadoop/etc/
   

4. Настройте slaves файл на jump node:

   В файле /usr/local/hadoop/etc/hadoop/slaves добавьте IP-адреса или имена всех ваших data nodes:

      data-node-ip1
   data-node-ip2
   

5. Запустите data nodes:

   На jump node выполните команду:

      start-dfs.sh
   

## ▎Шаг 3: Проверка установки
