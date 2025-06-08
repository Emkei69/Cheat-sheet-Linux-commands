========== Сетевой провайдер==========  
  
---------Настрой сервера изменить время на другого провайдера сети---------  
  
---Настрой времени на другое---  
  
``apt-получить обновление && apt-получить установку chrony nginx``  
  
---Внесем изменения в /etc/chrony.conf---  
``vim /etc/chrony.conf``  
  
```  
~~~  
######  
сервер ntp1.vniiftri.ru я
запускаю локальный уровень 5
, разрешаю все    
######  
#Записываю скорость, с которой система... <--- (Чтобы ограничить нужную скорость)  
~~~  
```  
  
---Где---  
| Конфигурация                     | Описание                      |
|----------------------------------|-------------------------------|
| `сервер ntp1.vniiftri.ru iburst` | локальный сервер изменений |
| `локальный слой 5` | локальный страт |
| `разрешить все" | изменение для всех терминов |

  
---Включить хронику и установить в автозагрузку---  
``systemctl включить –теперь хронид``  
  
---возможно, необходимо установить хронику, и написать в хроник.конф статью---  
``на главном сервере, главный клиент:``  
``сервер 172.16.4.1 загружен``  
  
``на маршрутизаторе филиала, сервере-филиале:``  
``сервер 172.16.5.1 загружен``  
    
---------Установите веб-сервер nginx как общий браузер-сервер---------  
  
---Настройка общего доступа к Nginx---  

``apt-get install nginx``  
  
---Создаем новый конфигурационный файл прокси---  
``/etc/nginx/sites-available.d/proxy.conf``  

```  
~~~  
сервер {
listen 80;
имя_сервера moodle.au-team.irpo;
местоположение / {
proxy_pass http://192.168.1.10:80 ;
proxy_set_header Host $хост;
proxy_set_header X-Реальный IP $удаленный адрес;  
proxy_set_header X-Переадресован-Для $remote_addr;  
}  
}  
  
сервер {  
прослушивание 80;  
имя_сервера wiki.au-team.irpo;
местоположение / {  
прокси-сервер_pass http://192.168.3.10:8080;  
proxy_set_header Host $хост;
proxy_set_header X-Реальный IP $удаленный адрес;  
proxy_set_header X-Перенаправленный-Для $remote_addr;    
}   
}  
~~~  
```  
  
---Включаем информацию нами (proxy.conf), возможно, это ссылки---  
``ln /etc/nginx/сайты-доступны.d/proxy.conf /etc/nginx/сайты-включены.d/proxy.conf``  
`ls -la /etc/nginx/сайты-включены``  
  
---Проверьте конфигурацию на наличие ошибок---  
  
``nginx -t``  

---Теперь подключаем службу nginx---  

``systemctl включить --теперь nginx``  

---------Сервер филиалов ---------  

---------Настрой отдельного контроллера Самбы на новый сервер филиалов---------  
  
---СамбаАД на альтернативный---    
  
---спасибо /etc/samba/smb.conf ЗА ТО, ЧТО ВЫ ЕСТЬ!!!---  
  
``apt-получить обновление && apt-получить задачу установки-samba-dc -y``  
  
``rm -f /etc/samba/smb.conf``  
  
---Начинаем установку---  
  
``предоставление домена samba-инструмента - интерактивно""  
  
```   
Область [AU-TeAM.IrPO] :  
Роль сервера (dc, участник, автономный сервер) [dc] :  
Серверная часть DNS (SAMBA_intERnal, Bind9_flAtfile, bind9_dlz, нет) [SAMBA_intERnaL] :  
IP-адрес службы переадресации DNS (введите "нет", чтобы отключить переадресацию) [192.168.1.10] :  
Пароль администратора:  
Введите пароль повторно:  
```  
  
---! В качестве партнера обращаемся к@ssw0rd---  
  
---Запускаем сервис---  
  
``systemctl включить -теперь samba.service``  
  
```  
Синхронизация состояния samba.service со служебным скриптом SysV с помощью /lib/sys  
Выполнение: /lib/systemd/systemd-sysv-install включить samba  
Созданная символическая ссылка /etc/systemd/system/multi-user.target.wants/samba.service -  
```  
---------Я знаю, где находится сервер на своей локальной---------  

``vim /etc/net/ifaces/ens19/resolv.conf``  

```  
~~~
сервер имен 127.0.0.1
поиск au-team.irpo  
~~~  
```  

---Перезапускаем сетевую службу---  

``systemctl перезапускает сеть``

---Добавляем сгенерированный файл krb5.conf---  

``mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf``  

---Проверяем состояние домена---  

``информация о домене samba-tool 127.0.0.1``  

---Домен работает---  

---------Создадим пользователей---------  
  
---Создаем пользователей, создаем группу---  
  
``пользователь samba-tool добавляет пользователя 1.hq P@ssw0rd``  
``пользователь samba-tool добавляет пользователя 2.hq P@ssw0rd``  
`пользователь samba-tool добавляет пользователя 3.hq P@ssw0rd``  
``пользователь samba-tool добавляет пользователя 4.hq P@ssw0rd``  
`пользователь samba-tool добавляет пользователя 5.hq P@ssw0rd``  

---Добавляем пользователей в группу---  
``в группу добавлений samba-tool входят пользователи: user1.hq,user2.hq,user3.hq,user4.hq,user5.hq``  

---проверим---  

``штаб-квартира группы samba-tool со списком участников``  

```  
пользователь5.hq  
пользователь4.hq  
пользователь3.hq  
uesr2.hq
пользователь1.hq
```  

---Смотрим настройки пользователя---  

``пользователь samba-инструмента показывает user1.hq``

---Из соображений безопасности все создаваемые УЗ блокированы---  

---Срок действия учетной записи пользователя истек---

``
Срок действия учетной записи: 92223372036854775807  
Количество зарегистрированных пользователей: 0  
```  

---Любое значение отличное от 0 обозначает, что учетная запись пользователя заблокирована---

---Разблокируем УЗ---  

``пользователь samba-tool установил expiry user1.hq --noexpiry``  

---Повторная проверка показывает, что мы разблокировали УЗ---  

```
Учетная запись пользователя: 66048
Срок действия учетной записи: 0
Учетная запись для входа в систему: 20250111103606.0z  
```

---Предоставить доступ пользователям сайта.csv---  

---Создать сайт import.sh---  

~~~  
```  
#!/bin/bash  
#
csv_file="/opt/users.csv"
, в то время как IFS=";" для чтения - имя, фамилия, должность, телефон, улица, почтовый индекс, город
, страна, пароль;  
если [ "$FirstName"=="Первое имя"]; то  
	     продолжить

: username="${Имя пользователя,,}.${фамилия,,}"
пользователь samba−tool добавляет "$имя пользователя" в@ssw0rd1
готово < "$csv_file"  
```  
~~~  

---даем право на исполнение---  

``chmod +x import.sh`, - ответил я.

---и запускаем---  

``./import.sh``  

---После выполнения скрипта проверяем---  

``список пользователей samba-tool``  

---------Сконфигурируйте файловое хранилище---------  

---Проверяем наличие дисков---  

``lsblk``  

```  
ИМЯ MAJ:МИНИМАЛЬНЫЙ РАЗМЕР RM ТИП RO ТОЧКА МОНТИРОВАНИЯ
sda 8:0 0 10G 0 диск /
sdb 8:16 0 1G 0 диск /  
sdc 8:32 0 1G 0 диск /
sdd 8:48 0 1G 0 диск /  
```  

---Программный РЕЙД в АЛЬТ---  

---Создадим дисковый массив уровня 5 из трёх дополнительных дисков следующей командой---  

``mdadm --create /dev/md0 --level=5 --raid-устройства=3 /dev/sd[b-d]``  

---Посмотрим на состояние raid-системы---  

``cat /proc/mdstat``  

---Сохраним конфигурацию массива в файл /и т. д./ - адреса.конф следующей командой---  

``mdadm -детальное сканирование -подробное описание > /etc/mdadm.conf``  

---теперь необходимо создать раздел---  

``разделенный /dev/md0``  

---1) Необходимо создать таблицу разделов. Используем обычный и расширенный тип MBR (msdos)---  
```  
разделенный /dev/md0  
GNU Parted 3.2.46-e4ae  
Используется файл /dev/md0  
Добро пожаловать в GNU Parted! Введите "справка", чтобы просмотреть список команд.  
(parted) mktable msdos
```

---2) Посмотрим таблицу разделов, чтобы выяснить размер свободного пространства---  
``(разделенный) отпечаток``  
```  
Модель: Программный RAID-массив Linux (md)  
Размер диска /dev/md0: 2143 МБ  
Размер сектора (логический/физический): 512 Б/512 В  
Таблица разделов: msdos  
Дисковые флаги:
  
Число Начальный конечный размер Тип Флаги файловой системы
```  

---3) Создадим раздел.---  
``(разделенный) первичный файл mkpart объемом 4 12143 МБ``  

---4) Снова посмотрим таблицу разделов---  
``(разделенный) отпечаток``
```  
Модель: Программный RAID-массив Linux (md)  
Размер диска /dev/md0: 2143 МБ  
Размер сектора (логический/физический): 512B/5122B  
Таблица разделов: msdos  
Флаги диска:  

Количество флагов файловой системы, начиная с конечного размера  
		1049 КБАЙТ 2143 МБ 2142 МБ основного объема 4 фунта  
```  
---5) Мы расстались. Команда уволилась---  

---Проверим lsblk---  

``lsblk /dev/md0``  

---Теперь создадим файловую систему, по заданию требуется в ext4, создаём её следующей командой---  
``mkfs.ext4 /dev/md0p1``  

---Установленное автоматическое конфигурирование в /raid5---  

---Создаем каталог /raid5---  
``mkdir /raid5``  

---Наблюдаем автоматическое монтирование в папку /raid5. Для этого добавим в файл /etc/fstab---
`'`dev/md0p1 /raid5 ext4 по умолчанию 0 0"`  

---Выполним команду монтирования---  

``гора -а``  

---Посмотрим на точки монтирования камеры дф---  

---------Настройте сервер сетевой системы(nfs)---------  

---Создадим файловые ресурсы и настроим права к ним---  

``mkdir /raid5/nfs``  
``chmod 777 /raid5/nfs``  

---Установим необходимое ПО---  

``apt-get установит nfs-сервер``  

---подпишем новый NFS---  
  
``systemctl включить --теперь nfs-сервер``  
  
---предлагаем перейти к каталогу /etc/exports---  

``vim /etc/exports``  
  
```##########```  
```/raid5/nfs 192.168.2.0/28(rw,no_subtree_check)```  
```##########```  
   
  
---Применим наши настройки---  
  
``exportfs -vra``  
  
---------Ansible на сервере-филиале---------  
  
---Предварительные настройки---  
  
---Включаем SSH на главном клиенте---  
  
``systemctl включить -теперь sshd.service``  
  
---Создаем ключевое слово на сервере-филиале---  
  
`ssh-keygen``
  
---Устанавливаем быстрый доступ на главный сервер и главного клиента---  
  
``ssh-copy-id user@192.168.1.10``  
``ssh-copy-id user@192.168.2.10``  
  
---Устанавливаем ansible---  
``apt-получаем установку ansible``  

---редактируем конфиг---  
``vim /etc/ansible/ansible.cfg``  

```
~~~  
[по умолчанию]  
  
# некоторые базовые значения по умолчанию...  
  
############### 
инвентарь = /и т. д./анзибль/хосты 
interpreter_python = auto_silent  
###############  
  
#библиотека = /usr/общий доступ/my_modules/  
~~~  
```  

---Создаем инвентарный файл---  

``vim /etc/ansible/hosts``  
  
```
~~~  
[Linux]  
главный сервер ansible_host=user@192.168.1.10  
главный клиент ansible_host=user@192.168.2.10  

[ECO_ROUTERS]  
главный маршрутизатор ansible_host=192.168.5.1, дочерний
маршрутизатор ansible_host=192.168.5.2  

[ECO_ROUTERS:переменные]  

ansible_connection=network_cli
, ansible_network_os=ios
, ansible_user-администратор
, ansible_password-администратор
~~~  
```  

---------Выполняем проверку доступности---------

``ansible all -m ping""

---------Создание приложений в Docker на сервере-филиале---------

---------Установить docker-движок и docker-compose---------

``apt-get устанавливает docker-движок docker-compose -y``

---------подключаем службу docker---------

``systemctl включить -теперь docker``

---------Performing an availability check---------

``ansible all -m ping""

---------Creating applications in Docker on a branch server---------

---------Install the docker engine and docker-compose---------

`apt-get installs the docker engine docker-compose -y`

---------enabling the docker service---------

`enable systemctl -now docker`

---------we provide an opportunity to combine users into a docker group so that it can work with containers---------

`usermod sshuser -aG configuration tool`
`grep /etc/group configuration tool'

---------We go into the context of the sshuser user---------

``su -l sshuser``

---------Upload images with the following command---------

`docker extracts mediawiki`
`docker extracts mariadb`

---------Creating a wiki.yml for the MediaWiki application in the user's home directory.---------

``vim wiki.yml``  

~~~  
```  

wiki services:  
 image: mediawiki_name of the
container: wiki
-Wednesday:
MEDIAWIKI_DB_HOST: mariadb  
 MEDIAWIKI_DB_USER: wiki  
 MEDIAWIKI PASSWORD: WikiP@ssw0rd  
 MEDIAWIKI_DB NAME:
mediawiki ports:
-8080:80  
 # volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]  
 database:
image: mariadb container name
: mariadb
host name:
mariadb environment
: MYSQL_DATABASE: mariadb  
 MYSQL_ USER: wiki  
 MYSQL_PASSWORD: volumes Wikipedia@ssw0rd
:
- ./db:volumes/var/lib/mysql
:
dB:  
```  
~~~  

---------Checking the configuration---------

`configuration of docker-compose -f wiki.yml`

---after checking, we launch---

``docker-compose -f wiki.yml up -d``

![image](https://github.com/user-attachments/assets/7139c3c2-ba46-4a0c-8240-adf657168f79 )

---------!We go to the main client, we look at him `http://192.168.3.10:8080 `---------  

![image](https://github.com/user-attachments/assets/b98a70d4-3f7d-4fe1-b3fb-0233c1f35e37 )  

Fill in the fields, Then   

![image](https://github.com/user-attachments/assets/700808e9-6cba-4e79-82df-ce89a421feac )


``scp LocalSettings.php sshuser@192.168.3.10:/home/sshuser/``  

`the cat LocalSettings.php `  

---------After that, we stop all containers---------  

`stopping docker$(docker ps -a -q)``  

---------delete all containers---------  

`docker rm$(docker ps -a -q)`

---------Commenting on the wiki.yml page---------  

``` - 8080:80```  
` volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]`  

--------- I just downloaded wiki.yml---------  

``docker-compose -f wiki.yml up -d``  
`On the main server, in the browser at http://192.168.3.10:8080 `  

![image](https://github.com/user-attachments/assets/c3403324-ee47-411e-b8c5-e0d552a59238 )

`Wiki works`  

========== The main server==========  

---------Install moodle on the main server---------  
---We are installing a number of packages that we will need to work with---

`apt-get the update && apt-Get the installation of apache2 php8.2 apache2-mod_php8.2 mariadb-Server php8.2-opcache php8.2-curl php8.2-gd php8.2-International php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xml-reader php8.2-fileinfo php8.2-sodium`  

---Connecting httpd2 and mysqld services---  

`enable systemctl - now httpd2 mysqld`  

---Set up secure access to our future database using the command---  

``mysql_secure_installation``  

--- I'm sure you're right, I do the rest of the time.---  
---go to the DBMS to create and configure the database---  

``mariadb -u root -p``  

`MariaDB [(no)]> CREATE A moodledb DATABASE;`  
`The request is OK, 1 line is affected (0.002 seconds)``  

`MariaDB [(no)]> CREATE A CUSTOM moodle IDENTIFIED AS 'P@ssw0rd';"`  
`Request K, 0 rows affected (0.004 seconds)``  

`MariaDB [(no)]> GRANTS moodle ALL PRIVILEGES ON moodledb.*;"`  
`The query is OK, rows are affected (0.003 seconds)``  

`MariaDB [(no)]> RESET PRIVILEGES;`  
`The query is OK, 0 lines are affected (0.002 seconds)``  

`MariaDB [(no)]>``  

---download MOODLE in the online version---  

``curl -L https://tinyurl.com/2z2btyrz > /root/moodle.zip``  

---Configure it in /var/www/html/ for long-range navigation---  

`unpack /root/moodle.zip -d /var/www/html``  
``mv /var/www/html/moodle-4.5.0/* /var/www/html/``  
``ls /var/www/html``  

---Create a new moodledata directory, where you can find data and use html and moodledata---  

``mkdir /var/www/moodledata``  
`launching apache2:apache2 /var/www/html`  
`launch apache2:apache2 /var/www/moodledata`  

---Note the value of the max_input_vars parameter in the php.ini file---  

``vim /etc/php/8.2/apache2-mod_php/php.ini``  
```  
~~~  
`How many GET/POST/COOKIE input variables can be accepted`  
``; max_input_vars = 1000``  

`; The maximum amount of memory that a script can occupy (128 MB)``  
``; http://php.net/memory-limit``  
`memory_limit = 128 MB`  

```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  
`; Error handling and logging ;`  
```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  

`This directive informs PHP about what errors, warnings and notifications you would like to receive`  
`actions must be taken to achieve this. The recommended way to set values for this is`  
``/max_input_va``  
~~~
```  

--- !Uncomment the string, the new value is 5000---  

---We invite you to cooperate **apache**---  

``rm -f /var/www/html/index.html``  

---------Enabling the httpd2 service---------  

`systemctl restarting httpd2`  

---Join the client and get acquainted with the mood---  

``http://192.168.1.10/install.php``  

---Choosing MariaDB as a data base partner---  

![image](https://github.com/user-attachments/assets/15196dbc-0181-4cc0-afc8-2117c6e2d10c )

---View everything in the **“OK"** or “Check” and continue **“Continue”**---  

---Fill in the required fields to create a primary administrator---  

![image](https://github.com/user-attachments/assets/9c6f2f25-0893-400a-a11d-dffaaa19ff85 )

| Fill in some more lines in the next step: | |
|---------------------------------------------------|----------------------------------|
| Full name of the website: | moodle |
| Short name of the website: | XX (workplace number) |
| Location settings:                         | Europe/Moscow (according to region) |
| Support Service contacts: | any@random.com (more) |


--------- Let's add the client machine to the domain---------  
 
---It is impossible to install DNS on the user's computer---  

![image](https://github.com/user-attachments/assets/4f20d78e-b2dd-4124-8564-f66e88731862 )

---For convenience, we remember the export editor for use in vim---  
```Export EDITOR=/usr/bin/vim```
 
---------Management of the group leaders' headquarters---------  
 
---We will use the role mechanism for simplicity and speed.---  
  
---Install the required module---  
`apt-get installs the libnss role`  
  
---create a local hq_users group---  
```add the hq_users group```  
  
---configure the role, for which the members of the local hq_users group automatically become members of the local hq_users group---  
```vim /etc/role```  
  
```  
~~~  
Domain Users:users  
Domain Administrators:localadmins  

Headquarters:hq_users  
~~~  
```  
  
---The government's commitment to using the court by all users---  
```public control```    
  
--- Indeed, we need a commitment to fighting pederasts. Let's use this to improve our eyesight, which checks the sensitivity of the sensitivity.---  
 
```Visudo```  

---Add the following block to the file (conveniently at the end)---   

```  
~~~  
###############  

Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id  

%hq_users ALL=(ALL) NO ACCESS: HQ  

###############  
~~~  
```  
  
---Where---
```  
Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id is the Alias HEADQUARTERS in which  
a set of executable files is described  
%hq_users ALL=(ALL) NO access: HQ – connects the hq_users group with other HQ users and
sets up a win-win trial  
```  
  
---For verification, we register together with a user from a separate hq group---

![image](https://github.com/user-attachments/assets/7fac3a25-6bcb-4298-add5-9df66bd9e809 )
  
---We check the employees in the local and domain group utilitoy id---

![image](https://github.com/user-attachments/assets/f2a07ebb-28ed-4534-9382-a6e308ec4bfa )
  
---The domain user is a member of the local group---  
--- So we want to help you sudo---

![image](https://github.com/user-attachments/assets/12ab551d-533d-4c68-9b18-142a2a831477 )
  
---You can only use the court ID due to entering a password---  
---And it is not allowed to perform other tasks at the same time---

![image](https://github.com/user-attachments/assets/ea83943a-f768-48ba-9b61-bdc826849ecd )
  
---Install the NFS server---  
  
```apt-get install nfs client```  
```  
Reading the package lists... Made by  
Building a dependency tree... Made by  
nfs-clients already has the latest version.    
0 updated, 0 recently installed, 0 deleted, and 629 not updated.  
```  
  
---Checking the availability of resources---  
  
``showmount -e 192.168.3.10``  
```  
Export list for 192.168.3.10:
/srv/public *
/raid5/nfs 192.168.2.0/28  
```  
  
---Creating a catalog---  
  
``mkdir /mnt/nfs``  
`mount -t nfs 192.168.3.10:/raid5/nfs/mnt/nfs`  
``df /mnt/nfs``  
  
```  
The file system size used, %, is set to  
192.168.3.10:/raid5/nfs 2.0G 0 1.9G 0% /mnt/nfs  
```

`umount -a`  

```
umount: /run/user/1080001104: the target object is busy.  
umount: /run/user/0: The target object is busy.  
umount: /tmp: the target object is busy.  
umount: /sys/fs/cgroup: the target object is busy.  
umount: /: the target object is busy.  
umount: /run: The target object is busy.  
umount: /dev: The target object is busy.  
```

---Add the following line to the end of the /etc/fstab file---  
  
```192.168.3.10:/raid5/nfs /mnt/nfs nfs intr,soft,_netdev 0  0```
  
``
intr/nointr — turn off/on mounting operations by ctrl+C  
soft — prevents hanging, in case of resource unavailability  
_netdev — Initialization system management, mounting after network access  
```  
  
------------------Install the Yandex Browser application for organizations on the Main Client---------
------ Install the Yandex Browser on the Main Client via the terminal using the commands---  
  
```apt-get install yandex-browser-stable```    
  
-------port forwarding---------  
---NAT port forwarding---
  
====================Main Router==========  
  
```(config)#ip nat source static tcp 192.168.1.10 2024 172.16.4.4 2024```  
  
====================Branch Router==========  
  
```(config)#ip nat source static tcp 192.168.3.10 8080 172.16.5.5 80```  
```(config)#ip nat source static tcp 192.168.3.10 2024 172.16.5.5 2024```
