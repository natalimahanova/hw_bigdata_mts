# Разберем установку и настройку Apache Hadoop.

Сразу продемострируем результат работы
![image](https://github.com/user-attachments/assets/2e4295d5-b8de-41b2-b8c5-0b133caf4b22)

Продолжение
![image](https://github.com/user-attachments/assets/0729b22e-75d1-4825-ba7d-c23e0a658168)



## ▎Шаг 1: Настройка SSH

Перед тем как перейти к установке Hadoop, важно настроить SSH для удаленного доступа между узлами. Это необходимо для того, чтобы Hadoop мог управлять своими компонентами.

1. Установите SSH:

   На большинстве дистрибутивов Linux SSH уже установлен. Если нет, вы можете установить его с помощью следующей команды:
```bash
sudo apt update
sudo apt install openssh-server
```   

2. Создайте SSH-ключи:

   Войдите под пользователем, который будет использоваться для работы с Hadoop (например, hadoopuser):
```bash
   su - hadoopuser
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```  

3. Добавьте публичный ключ в authorized_keys:

   Выполните следующую команду, чтобы добавить ваш публичный ключ в файл authorized_keys, что позволит вам подключаться к вашему узлу без пароля (также происходит и добавление внешних ключей сокомандников):
```bash
      cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```   

4. Проверьте SSH-соединение:

   Попробуйте подключиться к localhost:
```bash
      ssh localhost
```   

   Если вы сможете подключиться без запроса пароля, настройка SSH выполнена успешно.

## ▎Шаг 2: Установка Hadoop

Теперь, когда SSH настроен, мы можем перейти к установке Hadoop. 

## ▎2.1: Jump Node (Master Node)

1. В первую очередь нужно войти, а значит чтобы подключиться к jump node, нужно сделать следующее:
```bash
ssh team@ip
```
2. Необходимо обеспечить взаимодействие между всеми узлами, и для безопасности создаем специального пользователя:
```bash
sudo adduser hadoop
```
3. А дальше производим операции для нового пользователя:
   во-первых, мы входим в пользовтаеля hadoop
```bash
sudo -i -u hadoop
```
4. во-вторых, мы генерируем ключ для нашего нового пользователя:
```bash
ssh-keygen
```
5. в-третьих, мы сохраняем наш ключ на внешний носитель, а для этого выводим его на экран:
```bash
cat .ssh/id_ed25519.pub
```
Подготовительный этап завершился, поэтому непосредственно сама установка. Из-за необходимости ожидания скачивания, воспрользуемся сессионным менеджером:
```bash
tmux
```
Зайдя в котороый мы пишем команду по скачиванию дитрибутива
```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.x.x/hadoop-3.x.x.tar.gz
```
где .x.x - это версия дистрибутива на текущий день, для октября 2024 года это .4.0

И после этого выходим из сессионного менеджера.

6.Теперь надо удалить локальную адресацию, чтобы хосты знали друг друга по именам:
```bash
sudo nano /etc/hosts 
```
7.Перед нами открылся файл, в котором уже присутствуют записи, нам их нужно закомментировать, и вставляем адреса наших 4х узлов:
```bash
192.168.1.nn team-k-jn # jump node
192.168.1.nn team-k-nn # name node
192.168.1.nn team-k-dn-0 # data node
192.168.1.nn team-k-dn-1 # data node
```
Где nn - это наши хосты, а k - это номер команды (в нашем случае - это 11)

Проверяем по именам, узнала ли "машинка" как и кого зовут:
```bash
ping team-k-jn
```
```bash
ping team-k-nn
```
```bash
ping team-k-dn-0
```
```bash
ping team-k-dn-1
```
После объявления этих команд, должны быть записи с информацией по нодам, значит их имена были успешно внесены.

## ▎2.2: Name Node

После описанных выше действий переключаемся на Name Node:

```bash
ssh team-k-nn
```

Вводим пароль, потому что пока что не разложены авторизационные ключи.
И воспроизовдим эатпы под номерами 2-7 из раздела 2.1

## ▎2.3: Оставшиеся Data Node

Проделываем то же самое с как в пункте 2.2 и для двух наших data node.

## ▎2.4: После этих операций сложив все ключи мы их добавляем так чтобы можно было не вводим пароль каждый раз:

Вернуться на jump node, зайти в пользователя hadoop и сложить в файлик ключи:
```bash
nano .ssh/authorized_keys
```
Туда вписываем то, что выводили и сохраняли на каждой ноже после команды cat .ssh/id_ed25519.pub
Должно быть 4 штуки.

Теперь распространим их на все ноды.
```bash
scp .ssh/authorized_keys team-k-nn:/home/hadoop/.ssh/
scp .ssh/authorized_keys team-k-dn-0:/home/hadoop/.ssh/
scp .ssh/authorized_keys team-k-dn-1:/home/hadoop/.ssh/
```
Выше написаны три команды для каждой оставшейся ноды, после каждой нужно ввести пароль.
Теперь ноды могут взаимодействовать между собой.

## ▎2.5: Вовзаращемся к скачанному дистрибутиву

Вспоминаем, что у нас скачивался дистрибутив, и предположив, что к этому моменту он уже скачан, передаем его на все ноды.
Но перед этим выходим:
```bash
exit
```
Потом возвращаемся в сессионный менеджер для проверки того, скачалось или нет:
```bash
tmux attach -t 0
```
Ну и теперь уже передаем на ноды:
```bash
scp hadoop-3.x.x.tar.gz team-k-nn:/home/hadoop
scp hadoop-3.x.x.tar.gz team-k-dn-0:/home/hadoop
scp hadoop-3.x.x.tar.gz team-k-dn-1:/home/hadoop
```
Заходим поочередно на name node и data node и распакуем архив:
```bash
ssh team-k-nn 
tar -xvzf hadoop-3.x.x.tar.gz 
exit
```
и так же потом для data node
```bash
ssh team-k-dn-0 
tar -xvzf hadoop-3.x.x.tar.gz 
exit
```
и наконец
```bash
ssh team-k-dn-1 
tar -xvzf hadoop-3.x.x.tar.gz 
exit
```

## ▎Шаг 3: Настраиваем Hadoop

Теперь когда создана единая учетка, от имени которой будут выполняться все операции, и наконец обеспечили взаимодействие между всеми узлами: ssh-ключ и имена хостов.

А настройка начнется с name node и проверяем что версия 11.
```bash
ssh team-k-nn
java -version
```
Добалвяем переменные окружения, но для этого нужно найти местоположение:
```bash
which java # здесь у нас вышло  /usr/bin/java
readlink -f /usr/bin/java
```
Теперь открываеем файл:
```bash
nano ~/.profile
```
Добавляем переменные в файл
```bash
export HADOOP_HOME=/home/hadoop/hadoop-x.x.0 # где развернут дистрибутив
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 # где лежит java
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin # путь для выполнения исполняемых файлов hadoop
```

То же самое делаем на data nodes
```bash
scp ~/.profile team-k-dn-0:/home/hadoop
scp ~/.profile team-k-dn-1:/home/hadoop
```

Переходим в папку дистрибутива
```bash
cd hadoop-3.4.0/etc/hadoop
```

JAVA_HOME надо добавить в hadoop-env.sh
```bash
nano hadoop-env.sh # открыть файл
```
Вставить в него 
```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```
Сохранить и выйти

Копируем hadoop-env.sh на data nodes
```bash
scp hadoop-env.sh team-k-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hadoop-env.sh team-k-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

Откроем файл core-site.xml
```bash
nano core-site.xml
```

Добавим внутрь файла следующие параметры:
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://team-k-nn:9000</value>
    </property>
</configuration>
```

Откроем файл hdfs-site.xml
```bash
nano hdfs-site.xml
```

Добавим внутрь файла следующие параметры (фактор репликации 3)
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>
```

Откроем файл workers
```bash
nano workers
```

Меняем localhost на имена нод кластера

- team-k-nn
- team-k-dn-0
- team-k-dn-1

Копируем все файлы на data nodes
```bash
scp core-site.xml team-k-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp core-site.xml team-k-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-k-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-k-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-k-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-k-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

Переходим на 2 папки назад
```bash
cd ../../
```

Форматируем файловую систему
```bash
bin/hdfs namenode -format
```

Запускаем hadoop
```bash
sbin/start-dfs.sh
```

Проверяем что все поднялось 
```bash
jps
```

## ▎Шаг 4: Настройка nginx

Переходим на jump node
```bash
exit
exit
```

Копируем конфиг для nginx
```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn
```
Открываем конфиг в nano
```bash
sudo nano /etc/nginx/sites-available/nn
```

Заменяем внутренности файла на следующие и сохраняем:
```bash
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 9870 default_server;
        #listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                proxy_pass http://team-k-nn:9870;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

Сделаем ссылку
```bash
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn
```

Перезапускаем nginx
```bash
sudo systemctl reload nginx
```

Отключаемся от сервера
```bash
exit
```

Подключаемся к серверу с учетом проброшеннего порта
```bash
ssh -L 9870:team-k-nn:9870 team@ip
```

Проверка hadoop интерфейса (в браузере ввести)
```bash
127.0.0.1:9870
```
