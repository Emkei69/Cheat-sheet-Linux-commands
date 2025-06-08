==========NetProvider==========  
  
---------Настройка сервера времени chrony на машине NetProvider---------  
  
---Настройка chrony на Alt---  
  
```apt-get update && apt-get install chrony nginx```  
  
---Вносим изменения в /etc/chrony.conf---  
```vim /etc/chrony.conf```  
  
```  
~~~  
######  
server ntp1.vniiftri.ru iburst  
local stratum 5  
allow all    
######  
#Record the rate at which the system... <--- (Чтобы определить нужную строку)  
~~~  
```  
  
---Где---  
| Конфигурация                     | Описание                      |
|----------------------------------|-------------------------------|
| `server ntp1.vniiftri.ru iburst` | эталонный сервер времени      |
| `local stratum 5`                | локальный стратум             |
| `allow all`                      | разрешение для всех клиентов  |

  
---Запустим chrony и поставим в автозагрузку---  
```systemctl enable –now chronyd```  
  
---на клиентах необходимо будет установить chrony, и прописать в  chrony.conf строку---  
```на MainServer, MainClient:```  
```server 172.16.4.1 iburst```  
  
```на BranchRouter, BranchServer:```  
```server 172.16.5.1 iburst```  
    
---------Настройте веб-сервер nginx как обратный прокси-сервер---------  
  
---Настройка обратного прокси Nginx---  

```apt-get install nginx```  
  
---Создаём новый конфигурационный файл proxy---  
```/etc/nginx/sites-available.d/proxy.conf```  

```  
~~~  
server {   
listen 80;   
server_name moodle.au-team.irpo;   
location / {   
proxy_pass http://192.168.1.10:80;   
proxy_set_header Host $host;  
proxy_set_header X-Real-IP  $remote_addr;  
proxy_set_header X-Forwarded-For $remote_addr;  
}  
}  
  
server {  
listen 80;  
server_name wiki.au-team.irpo;    
location / {  
proxy_pass http://192.168.3.10:8080;  
proxy_set_header Host $host;  
proxy_set_header X-Real-IP  $remote_addr;  
proxy_set_header X-Forwarded-For $remote_addr;    
}   
}  
~~~  
```  
  
---Включаем созданную нами (proxy.conf), путём создания  ссылки---  
```ln /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/proxy.conf```  
```ls -la /etc/nginx/sites-enabled```  
  
---Проверьте конфигурацию на наличие ошибок---  
  
```nginx -t```  

---Затем запускаем службу nginx---  

```systemctl enable --now nginx```  

---------BranchServer ---------  

---------Настройка доменного контроллера Samba на машине BranchServer---------  
  
---SambaAD на Alt---    
  
---удаляем файл /etc/samba/smb.conf ДО НАЧАЛА УСТАНОВКИ!!!---  
  
```apt-get update && apt-get install task-samba-dc -y```  
  
```rm -f /etc/samba/smb.conf```  
  
---Начинаем установку---  
  
```samba-tool domain provision -–interactive```  
  
```   
Realm [AU-TeAM.IrPO] :  
Server Role (dc, member, standalone) [dc] :  
DNS backend (SAMBA_intERnal, Bind9_flAtfile, bind9_dlz, none) [SAMBA_intERnaL] :  
DNS forwarder IP address (write 'none' to disable forwarding) [192.168.1.10] :  
Administrator password:  
Retype password:  
```  
  
---! В качестве пароля указываем P@ssw0rd---  
  
---Запускаем сервис---  
  
```systemctl enable --now samba.service```  
  
```  
Synchronizing state of samba.service with SysV service script with /lib/sys  
Executing: /lib/systemd/systemd-sysv-install enable samba  
Created symlink /etc/systemd/system/multi-user.target.wants/samba.service -  
```  
---------Меняем адрес DNS сервера на свой локальный---------  

```vim /etc/net/ifaces/ens19/resolv.conf```  

```  
~~~    
nameserver 127.0.0.1  
search au-team.irpo  
~~~  
```  

---Перезапускаем сетевую службу---  

```systemctl restart network```

---Перемещаем сгенерированный конфиг krb5.conf---  

```mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf```  

---Проверяем состояние домена---  

```samba-tool domain info 127.0.0.1```  

---Домен работает---  

---------Создадим пользователей---------  
  
---Создаем пользователей, создаем группу---  
  
```samba-tool user add user1.hq P@ssw0rd```  
```samba-tool user add user2.hq P@ssw0rd```  
```samba-tool user add user3.hq P@ssw0rd```  
```samba-tool user add user4.hq P@ssw0rd```  
```samba-tool user add user5.hq P@ssw0rd```  

---Добавляем пользователей в группу---  
```samba-tool group addmembers hq user1.hq,user2.hq,user3.hq,user4.hq,user5.hq```  

---проверим---  

```samba-tool group listmembers hq```  

```  
user5.hq  
user4.hq  
user3.hq  
uesr2.hq  
user1.hq
```  

---Смотрим настройки пользователя---  

```samba-tool user show user1.hq```

---Из соображений безопасности все создаваемые УЗ блокированы---  

---Параметр accountExpires---

```  
accountExpires: 92223372036854775807  
logonCount: 0  
```  

---Любое значение отличное от 0 обозначает, что учетная запись пользователя заблокирована---

---Разблокируем УЗ---  

```samba-tool user setexpiry user1.hq --noexpiry```  

---Повторная проверка показывает, что мы разблокировали УЗ---  

```
userAccountConrol: 66048  
accountExpires: 0  
logonCount: 20250111103606.0z  
```

---Выполните импорт пользователей из файла users.csv---  

```Создаём файл import.sh```  

~~~  
```  
#!/bin/bash  
#  
csv_file="/opt/users.csv"  
while IFS=";" read -r firstName lastName role phone ou street zip city  
country password; do  
if [ "$firstName"=="First Name"]; then  
	     continue  
fi  
username="${firstName,,}.${LastName,,}"  
	samba−tool user add "$username" P@ssw0rd1  
done < "$csv_file"  
```  
~~~  

---даем право на исполнение---  

```chmod +x import.sh```

---и запускаем---  

```./import.sh```  

---После выполнения скрипта проверяем---  

```samba-tool user list```  

---------Сконфигурируйте файловое хранилище---------  

---Проверяем наличие дисков---  

```lsblk```  

```  
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT  
sda    8:0    0  10G  0 disk /  
sdb    8:16   0   1G  0 disk /  
sdc    8:32   0   1G  0 disk /  
sdd    8:48   0   1G  0 disk /  
```  

---Программный RAID в ALT---  

---Создадим дисковый массив уровня 5 из трёх дополнительных дисков следующей командой---  

```mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[b-d]```  

---Посмотрим статус raid-массива---  

```cat /proc/mdstat```  

---Сохраним конфигурацию массива в файл /etc/mdadm.conf следующей командой---  

```mdadm --detail -scan --verbose > /etc/mdadm.conf```  

---теперь необходимо создать раздел---  

```parted /dev/md0```  

---1) Необходимо создать таблицу разделов. Используем самый простой и распространённый тип MBR (msdos)---  
```  
parted /dev/md0  
GNU Parted 3.2.46-e4ae  
Using /dev/md0  
Welcome to GNU Parted! Type 'help' to view a list of command.  
(parted) mktable msdos
```

---2) Посмотрим таблицу разделов, чтобы выяснить размер свободного пространства---  
```(parted) print```  
```  
Model: Linux Software RAID Array (md)  
Disk /dev/md0: 2143MB  
Sector size (logical/physical): 512B/512B  
Partition Table: msdos  
Disk Flags:
  
Number  Start	End	Size	Type	File	system	Flags
```  

---3) Создадим раздел.---  
```(parted) mkpart primary ext4 1 2143MB```  

---4) Снова посмотрим таблицу разделов---  
```(parted) print```
```  
Model: Linux Software RAID Array (md)  
Disk /dev/md0: 2143MB  
Sector size (logical/physical): 512B/512B  
Partition Table: msdos  
Disk Flags:  

Number	Start	End	Size	Туре	File system	Flags  
		1049kB	2143MB	2142MB	primary	ext4	lba  
```  
---5) Выходим из parted. Команда quit---  

---Проверим lsblk---  

```lsblk /dev/md0```  

---Теперь создадим файловую систему, по заданию требуется ext4, создаём её следующей командой---  
```mkfs.ext4 /dev/md0p1```  

---Настроим автоматическое монтирование в /raid5---  

---Создаём каталог /raid5---  
```mkdir /raid5```  

---Обеспечим автоматическое монтирование в папку /raid5. Для этого добавим в конец файла /etc/fstab---
```/dev/md0p1 /raid5 ext4 defaults 0 0```  

---Выполним команду монтирования---  

```mount -a```  

---Посмотрим точки монтирования командой df---  

---------Настройте сервер сетевой файловой системы(nfs)---------  

---Создадим файловые ресурсы и настроим права к ним---  

```mkdir /raid5/nfs```  
```chmod 777 /raid5/nfs```  

---Установим необходимое ПО---  

```apt-get install nfs-server```  

---запустим сервер NFS---  
  
```systemctl enable --now nfs-server```  
  
---пропишем доступ к каталогу в файле /etc/exports---  

```vim /etc/exports```  
  
```  
##########  
/raid5/nfs 192.168.2.0/28(rw,no_subtree_check)  
##########  
```  
  
---Применим наши настройки---  
  
```exportfs -vra```  
  
---------Ansible на сервере BranchServer---------  
  
---Предварительные настройки---  
  
---Включаем SSH на MainClient---  
  
```systemctl enable --now sshd.service```  
  
---Создаем ключевую пару на BranchServer---  
  
```ssh-keygen```
  
---Настраиваем бесключевой доступ на MainServer и MainClient---  
  
```ssh-copy-id user@192.168.1.10```  
```ssh-copy-id user@192.168.2.10```  
  
---Устанавливаем ansible---  
```apt-get install ansible```  

---редактируем конфиг---  
```vim /etc/ansible/ansible.cfg```  

```
~~~  
[defaults]  
  
# some basic default values...  
  
###############  
inventory = /etc/ansible/hosts  
interpreter_python = auto_silent  
###############  
  
#library	= /usr/share/my_modules/  
~~~  
```  

---Создаем инвентарный файл---  

```vim /etc/ansible/hosts```  
  
```
~~~  
[Linux]  
mainserver ansible_host=user@192.168.1.10  
mainclient ansible_host=user@192.168.2.10  

[ECO_ROUTERS]  
mainrouter ansible_host=192.168.5.1  
branchrouter ansible_host=192.168.5.2  

[ECO_ROUTERS:vars]  

ansible_connection=network_cli  
ansible_network_os=ios  
ansible_user-admin  
ansible_password-admin
~~~  
```  

---------Выполняем проверку доступности---------

```ansible all -m ping```

---------Развертывание приложений в Docker на сервере BranchServer---------

---------Установить docker-engine и docker-compose---------

```apt-get install docker-engine docker-compose -y```

---------запустим службу docker---------

```systemctl enable --now docker```

---------добавляем пользователя sshuser в группу doсker, что он имел возможность работать с контейнерами---------

```usermod sshuser -aG docker```
```grep docker /etc/group```

---------Переходим в контекст пользователя sshuser---------

```su -l sshuser```

---------Загружаем образы следующей командой---------

```docker pull mediawiki```
```docker pull mariadb```

---------Создаем в домашней директории пользователя файл wiki.yml для приложения MediaWiki.---------

```vim wiki.yml```  

~~~  
```  
services  
 wiki:  
 image: mediawiki  
 container_name: wiki  
 environment:  
 MEDIAWIKI_DB_HOST: mariadb  
 MEDIAWIKI_DB_USER: wiki  
 MEDIAWIKI_PASSWORD: WikiP@ssw0rd  
 MEDIAWIKI_DB_NAME: mediawiki  
 ports:  
 -8080:80  
 # volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]  
 db:  
 image: mariadb  
 container_name: mariadb  
 hostname: mariadb  
 environment:  
 MYSQL_DATABASE: mariadb  
 MYSQL_USER: wiki  
 MYSQL_PASSWORD: WikiP@ssw0rd  
 volumes:  
 - ./db:/var/lib/mysql  
volumes:  
 db:  
```  
~~~  

---------Проверяем конфигурацию---------

```docker-compose -f wiki.yml config```

---после проверки, запускаем---

```docker-compose -f wiki.yml up -d```

![image](https://github.com/user-attachments/assets/7139c3c2-ba46-4a0c-8240-adf657168f79)

---------!Переходим на MainClient, в браузер по адресу http://192.168.3.10:8080---------  

![image](https://github.com/user-attachments/assets/b98a70d4-3f7d-4fe1-b3fb-0233c1f35e37)  

Заполняем поля, Далее   

![image](https://github.com/user-attachments/assets/700808e9-6cba-4e79-82df-ce89a421feac)


```scp LocalSettings.php sshuser@192.168.3.10:/home/sshuser/```

```cat LocalSettings.php```

---------После этого останавливаем все контейнеры---------

```docker stop $(docker ps -a -q)```

---------удаляем все контейнеры---------

```docker rm $(docker ps -a -q)```

---------Раскомментируем строку wiki.yml---------

``` - 8080:80```
``` volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]```

---------и снова запускаем wiki.yml---------

```docker-compose -f wiki.yml up -d```  
```На MainClient, в браузер по адресу http://192.168.3.10:8080```  

![image](https://github.com/user-attachments/assets/c3403324-ee47-411e-b8c5-e0d552a59238)

```Wiki работает```  

==========MainServer==========  

---------Запустите сервис moodle на сервере MainServer---------  
---Устанавливаем для ряд пакетов, которые будут нам нужны для работы---

```apt-get update && apt-get install apache2 php8.2 apache2-mod_php8.2 mariadb-server php8.2-opcache php8.2-curl php8.2-gd php8.2-intl php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xmlreader php8.2-fileinfo php8.2-sodium```  

---Включаем службы httpd2 и mysqld---  

```systemctl enable –-now httpd2 mysqld```  

---Настроим безопасный доступ к нашей будущей базе данных с помощью команды---  

```mysql_secure_installation```  

---меняем пароль на P@ssw0rd, все остальное по умолчанию---  
---заходим в СУБД для создания и настройки базы данных---  

```mariadb -u root -p```  

```MariaDB [(none)]> CREATE DATABASE moodledb;```  
```Query OK, 1 row affected (0.002 sec)```  

```MariaDB [(none)]> CREATE USER moodle IDENTIFIED BY 'P@ssw0rd';```  
```Query K, 0 rows affected (0.004 sec)```  

```MariaDB [(none)]> GRANT ALL PRIVILEGES ON moodledb.* TO moodle;```  
```Query OK, rows affected (0.003 sec)```  

```MariaDB [(none)]> FLUSH PRIVILEGES;```  
```Query OK, 0 rows affected (0.002 sec)```  

```MariaDB [(none)]>```  

---скачаем MOODLE стабильной версии---  

```curl -L https://tinyurl.com/2z2btyrz > /root/moodle.zip```  

---Разархивируем его в /var/www/html/ для дальнейшей настройки---  

```unzip /root/moodle.zip -d /var/www/html```  
```mv /var/www/html/moodle-4.5.0/* /var/www/html/```  
```ls /var/www/html```  

---Создадим новый каталог moodledata, там будут храниться данные и изменим владельца на каталогах html и moodledata---  

```mkdir /var/www/moodledata```  
```chown apache2:apache2 /var/www/html```  
```chown apache2:apache2 /var/www/moodledata```  

---Поменяем значение параметра max_input_vars в файле php.ini---  

```vim /etc/php/8.2/apache2-mod_php/php.ini```  
```  
~~~  
```; How many GET/POST/COOKIE input variables may be accepted```  
```; max_input_vars = 1000```  

```; Maximum amount of memory a script may consume (128MB)```  
```; http://php.net/memory-limit```  
```memory_limit = 128M```  

```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  
```; Error handling and logging ;```  
```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  

```; This directive informs PHP of which errors, warnings and notices you would like```  
```; it to take action for. The recommended way of setting values for this```  
```/max_input_va```  
~~~
```  

--- !Раскоментируем строку, новое значение 5000---  

---Удаляем стандартную страницу **apache**---  

```rm -f /var/www/html/index.html```  

---------Перезапускаем службу httpd2---------  

```systemctl restart httpd2```  

---Подключаемся с клиента MainClient и начинаем настройку---  

```http://192.168.1.10/install.php```  

---Выбираем MariaDB в качестве драйвера базы данных---  

![image](https://github.com/user-attachments/assets/15196dbc-0181-4cc0-afc8-2117c6e2d10c)

---Просматриваем всё ли в статус **“OK”** или “Проверка” и прожимаем **“Продолжить”**---  

---Заполняем обязательные поля для создания основного администратора---  

![image](https://github.com/user-attachments/assets/9c6f2f25-0893-400a-a11d-dffaaa19ff85)
  
| Заполним ещё некоторые строки на следующем шаге:  |                                  |
|---------------------------------------------------|----------------------------------|
| Полное название сайта:                            | moodle                           |
| Краткое название сайта:                           | XX (номер рабочего места)        |
| Настройки местоположения:                         | Европа/Москва (согласно региону) |
| Контакты службы поддержки:                        | any@random.com (любое)           |
  
 
---------Введём клиентскую машину в домен---------  
 
---Необходимо настроить DNS на контроллер домена---  

![image](https://github.com/user-attachments/assets/4f20d78e-b2dd-4124-8564-f66e88731862)

---Для удобства, поменяем редактор по умолчанию на vim---  
```export EDITOR=/usr/bin/vim```
 
---------Повышение привилегий пользователей группы hq---------  
 
---Мы для простоты и скорости будем использовать механизм ролей---  
  
---Установим необходимый модуль---  
```apt-get install libnss-role```  
  
---создадим локальную группу hq_users---  
```groupadd hq_users```  
  
---настроим роль, чтобы члены доменной группы hq автоматически становились членами локальной группы hq_users---  
```vim /etc/role```  
  
```  
~~~  
Domain Users:users  
Domain Admins:localadmins  

hq:hq_users  
~~~  
```  
  
---Настроим права выполнения sudo всеми пользователями---  
```control sudo public```    
  
---Далее, требуется настроить файл конфигурации sodoers. Используем утилиту visudo, которая проверяет корректность синтаксиса---  
 
```Visudo```  

---Добавляем в файл (удобно в конце) следующий блок---   

```  
~~~  
###############  

Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id  

%hq_users ALL=(ALL) NOPASSWD: HQ  

###############  
~~~  
```  
  
---Где---  
```  
Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id – алиас HQ, в котором  
описан набор исполняемых файлов  
%hq_users ALL=(ALL) NOPASSWD: HQ – связывает группу hq_users с алиасом HQ и  
настраивает sudo на беспарольный доступ  
```  
  
---Для проверки регистрируемся любым пользователем из доменной группы hq---  
  
![image](https://github.com/user-attachments/assets/7fac3a25-6bcb-4298-add5-9df66bd9e809)
  
---Проверяем членство в локальных и доменных группах утилитой id---  
  
![image](https://github.com/user-attachments/assets/f2a07ebb-28ed-4534-9382-a6e308ec4bfa)
  
---Доменный пользователь входит в локальную группу---  
---Далее, проверяем доступ к утилите sudo---  
  
![image](https://github.com/user-attachments/assets/12ab551d-533d-4c68-9b18-142a2a831477)
  
---Пользователю можно выполнять sudo id без ввода пароля---  
---И одновременно не разрешено выполнять другие---  
  
![image](https://github.com/user-attachments/assets/ea83943a-f768-48ba-9b61-bdc826849ecd)
  
---Устанавливаем NFS сервер---  
  
```apt-get install nfs-client```  
```  
Reading Package Lists... Done  
Building Dependency Tree... Done  
nfs-clients is already the newest version.    
0 upgraded, 0 newly installed, 0 removed and 629 not upgraded.  
```  
  
---Проверяем доступность ресурсов---  
  
```showmount -e 192.168.3.10```  
```  
Export list for 192.168.3.10:  
/srv/public *  
/raid5/nfs 192.168.2.0/28  
```  
  
---Создаем каталог---  
  
```mkdir / mnt/nfs```  
```mount -t nfs 192.168.3.10:/raid5/nfs /mnt/nfs```  
```df /mnt/nfs```  
  
```  
Filesystem		Size Used Avail Use% Mounted on  
192.168.3.10:/raid5/nfs 2.0G	0  1.9G	  0% /mnt/nfs  
```  
  
```umount -a```  

```  
umount: /run/user/1080001104: target is busy.  
umount: /run/user/0: target is busy.  
umount: /tmp: target is busy.  
umount: /sys/fs/cgroup: target is busy.  
umount: /: target is busy.  
umount: /run: target is busy.  
umount: /dev: targt is busy.  
```  
  
---Добавляем следующую строку в конец файла /etc/fstab---  
  
---192.168.3.10:/raid5/nfs /mnt/nfs nfs intr,soft,_netdev 0  0---  
  
```  
intr/nointr — отк/вкл операции монтирования по ctrl+C  
soft — предотвращает зависание, в случае недоступности ресурса  
_netdev — Управление системой инициализации, монтирование после доступа к сети  
```  
  
---------Установите  приложение  Яндекс  Браузере  для организаций на MainClient---------  
---Установим Яндекс Браузер на MainClient через терминал командами---  
  
```apt-get install yandex-browser-stable```    
  
-------проброс портов---------  
---NAT port forwarding---
  
==========MainRouter==========  
  
```(config)#ip nat source static tcp 192.168.1.10 2024 172.16.4.4 2024```  
  
==========BranchRouter==========  
  
```(config)#ip nat source static tcp 192.168.3.10 8080 172.16.5.5 80```  
```(config)#ip nat source static tcp 192.168.3.10 2024 172.16.5.5 2024```  
