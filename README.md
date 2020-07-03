# Урок 7.
В репозитори находятся файл вирт. машины [Vagrantfile](Vagrantfile).
## Задание 1. Написать service, который мониторит лог по ключевому слову. (/etc/sysconfig);
1. Создаем файл конфигурации для своего сервиса, создается в /etc/sysconfig/mylog. Плюс добавляем переменные WORD и LOG;

```
vi /etc/sysconfig/mylog
```
2. Создаем в /var/log файл лог и пишем туда слово ALERT(которое добавляли в пункте 1 ) и произвольное количество слов;

```
vi /var/log/mylog.log
```
3. Этап создания скрипта ,который отправляет лог в системный журнал и обрабатывает условия поиска. Изменяем права на запуск chmod 770; 

```
vi /opt/mylog.sh

#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`
if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi
```
4. Создаем юнит для сервисаи для и таймера. Создаем в каталоге /etc/systemd/system;

```
vi myservice_lesson7.service

[Unit]
Description=My service lesson7
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/mylog
ExecStart=/opt/mylog.sh $WORD $LOG
```

```
vi mytimer_lesson7.timer

[Unit]
Description=Run mylog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=myservice_lesson7.service
[Install]
WantedBy=multi-user.target
```
5. Стартуем наш свой таймер и убеждаемся, что в message появилось инфо найденых слов;

```
systemctl start  mytimer_lesson7.timer

tail -f /var/log/messages

Jul  3 15:30:29 localhost systemd: Starting Run mylog script every 30 second.
Jul  3 15:30:29 localhost systemd: Starting My service lesson7...
Jul  3 15:30:29 localhost root: Fri Jul  3 15:30:29 UTC 2020: I found word, Master!
```
## Задание 2 и 3. Из репозитория epel установить по и переписать init-скрипт на unit-файл;
1. Установка по командой, yum install epel-release -y && yum install spawn-fcgi php php-cli
mod_fcgid httpd -y;

2. Убираем комментарии в файле vi /etc/sysconfig/spawn-fcgi;

3. Создаем юнит для spawn-fcgi. vi /etc/systemd/system/spawn-fcgi.service;

4. Далее запускаем наш юнит и проверяем работу;

```
[root@lesson7 vagrant]# systemctl start spawn-fcgi.service 
[root@lesson7 vagrant]# systemctl status spawn-fcgi.service 
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-07-03 16:00:56 UTC; 7s ago
 Main PID: 31087 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─31087 /usr/bin/php-cgi
           ├─31088 /usr/bin/php-cgi
           ├─31089 /usr/bin/php-cgi
           ├─31090 /usr/bin/php-cgi
           ├─31091 /usr/bin/php-cgi
           ├─31092 /usr/bin/php-cgi
           ├─31093 /usr/bin/php-cgi
           ├─31094 /usr/bin/php-cgi
           ├─31095 /usr/bin/php-cgi
           ├─31096 /usr/bin/php-cgi
           ├─31097 /usr/bin/php-cgi
           ├─31098 /usr/bin/php-cgi
           ├─31099 /usr/bin/php-cgi
           ├─31100 /usr/bin/php-cgi
           ├─31101 /usr/bin/php-cgi
           ├─31102 /usr/bin/php-cgi
           ├─31103 /usr/bin/php-cgi
           ├─31104 /usr/bin/php-cgi
           ├─31105 /usr/bin/php-cgi
           ├─31106 /usr/bin/php-cgi
           ├─31107 /usr/bin/php-cgi
           ├─31108 /usr/bin/php-cgi
           ├─31109 /usr/bin/php-cgi
           ├─31110 /usr/bin/php-cgi
           ├─31111 /usr/bin/php-cgi
           ├─31112 /usr/bin/php-cgi
           ├─31113 /usr/bin/php-cgi
           ├─31114 /usr/bin/php-cgi
           ├─31115 /usr/bin/php-cgi
           ├─31116 /usr/bin/php-cgi
           ├─31117 /usr/bin/php-cgi
           ├─31118 /usr/bin/php-cgi
           └─31119 /usr/bin/php-cgi

Jul 03 16:00:56 lesson7 systemd[1]: Started Spawn-fcgi startup service by Otus.
Jul 03 16:00:56 lesson7 systemd[1]: Starting Spawn-fcgi startup service by Otus...
[root@lesson7 vagrant]# 
```
5. Создаем unit для httpd. vi /etc/systemd/system/httpd@.service;

```
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)
[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd-%I
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
KillSignal=SIGCONT
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

6. Создаем в каталоге /etc/sysconfig два файла конфигурации с опциями запускаемыми;

```
[root@lesson7 conf]# vi /etc/sysconfig/httpd-first
[root@lesson7 conf]# vi /etc/sysconfig/httpd-second
# /etc/sysconfig/httpd-first
OPTIONS=-f conf/first.conf
# /etc/sysconfig/httpd-second
OPTIONS=-f conf/second.conf
```
7. Создаем два файла конфигурации для httpd в /etc/httpd/conf;

```
cp httpd.conf second.conf
cp httpd.conf first.conf
```
```
Добавим в second.conf, чтоб было несколько инстансов с разными конфигурациями httpd
PidFile /var/run/httpd-second.pid
Listen 8080
```
8. Запускаем наши службы и проверяем ss -tnulp | grep httpd;

```
[root@lesson7 conf]# systemctl start httpd@first
[root@lesson7 conf]# systemctl start httpd@second
```

```
[root@lesson7 system]# systemctl status httpd@first.service 
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-07-03 18:51:54 UTC; 2s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 3191 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─3191 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─3192 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─3193 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─3194 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─3195 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           └─3196 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

Jul 03 18:51:53 lesson7 systemd[1]: Starting The Apache HTTP Server...
Jul 03 18:51:54 lesson7 httpd[3191]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set ...s message
Jul 03 18:51:54 lesson7 systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.


[root@lesson7 system]# systemctl status httpd@second.service 
● httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-07-03 18:52:55 UTC; 6s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 3232 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           ├─3232 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─3233 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─3234 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─3235 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─3236 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           └─3237 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

Jul 03 18:52:55 lesson7 systemd[1]: Starting The Apache HTTP Server...
Jul 03 18:52:55 lesson7 httpd[3232]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set ...s message
Jul 03 18:52:55 lesson7 systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.

```

```
[root@lesson7 system]# ss -tunlp | grep httpd
tcp    LISTEN     0      128      :::8080                 :::*                   users:(("httpd",pid=3237,fd=4),("httpd",pid=3236,fd=4),("httpd",pid=3235,fd=4),("httpd",pid=3234,fd=4),("httpd",pid=3233,fd=4),("httpd",pid=3232,fd=4))
tcp    LISTEN     0      128      :::80                   :::*                   users:(("httpd",pid=3196,fd=4),("httpd",pid=3195,fd=4),("httpd",pid=3194,fd=4),("httpd",pid=3193,fd=4),("httpd",pid=3192,fd=4),("httpd",pid=3191,fd=4))

```
