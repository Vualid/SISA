# SISA
ho-ho-ho
![image](https://github.com/user-attachments/assets/61a266a5-224f-48f6-b944-a24eb18f210b)


SW-DT


![image](https://github.com/user-attachments/assets/e2b7e48e-099a-454b-ac04-cd5b24b8b86b)


1.      Настроить хостнеймы
Настраивать хостнейм вместе с доменом (как в таблице)
На Линукс:
```
 hostnamectl set-hostname имя.домен
```

```
На vESR:
hostname *имя*
domain name *домен*
На EcoRouter
hostname *имя*
ip domain-name *домен*
```


Ecorouter
Интерфейсы:
```
enable
configure terminal
interface ISP
ip address 172.16.4.14/28
ip nat outside
exit
ip route 0.0.0.0/0 172.16.4.1
ip nat pool NAT 192.168.33.1-192.168.33.254
ip nat source dynamic inside pool NAT overload 172.16.4.14
exit
port ge0
service-instance Internet
encapsulation untagged
connect ip interface ISP
exit
exit
interface v110
ip address 192.168.33.1/26
ip nat inside
exit
interface v330
ip address 192.168.33.65/29
ip nat inside
exit
port ge1
service-instance ge1/vlan110
encapsulation dot1q 110 exact
rewrite pop 1
connect ip interface v110
exit
service-instance ge1/vlan330
encapsulation dot1q 330 exact
rewrite pop 1
connect ip interface v330
exit
exit
write
```

Туннель:
R-DT - R-BR:
```
interface tunnel.1
ip add 10.10.10.1/30
ip mtu 1400
ip tunnel 172.16.4.14 172.16.6.2 mode gre
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
R-DT - R-HQ:
interface tunnel.2
ip add 10.10.11.1/30
ip mtu 1400
ip tunnel ip tunnel 172.16.4.14 172.16.5.14 mode gre
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
R-HQ - R-BR:
interface tunnel.3
ip add 10.10.12.1/30
ip mtu 1400
ip tunnel 172.16.5.14 172.16.6.2 mode gre
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
```

R-HQ - R-DT:


```
interface tunnel.2
ip add 10.10.11.2/30
ip mtu 1400
ip tunnel 172.16.5.14 172.16.4.14 mode gre
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
```


OSPF:
R-DT:

```
r-dt(config)#router ospf 1
r-dt(config-router)#network 10.10.10.0/30 area 0.0.0.0
r-dt(config-router)#network 10.10.11.0/30 area 0.0.0.0
r-dt(config-router)#network 192.168.33.0/26 area 0.0.0.0
r-dt(config-router)#network 192.168.33.64/29 area 0.0.0.0
r-dt(config-router)#passive-interface default
r-dt(config-router)#no passive-interface tunnel.1
r-dt(config-router)#no passive-interface tunnel.2
r-dt(config-router)#exit
```

R-HQ:
```
r-dt(config)#router ospf 1
r-dt(config-router)#network 10.10.11.0/30 area 0.0.0.0
r-dt(config-router)#network 10.10.12.0/30 area 0.0.0.0
r-dt(config-router)#network 192.168.11.0/26 area 0.0.0.0
r-dt(config-router)#network 192.168.11.64/28 area 0.0.0.0
r-dt(config-router)#network 192.168.11.80/29 area 0.0.0.0
r-dt(config-router)#passive-interface default
r-dt(config-router)#no passive-interface tunnel.2
r-dt(config-router)#no passive-interface tunnel.3
r-dt(config-router)#exit
```

NTP:
```
ntp server 192.168.11.2
ntp timezone utc+3
DHCP:
ТОЛЬКО НА R-HQ
ip pool CLI 192.168.11.66-192.168.11.79
dhcp-server 1
pool CLI 1
gateway 192.168.11.65 (который настроен на порту!!!)
domain-name au.team
dns <АДРЕС DNS СЕРВЕРА>
exit
exit
interface v220
dhcp-server 1
exit
```



vESR
Интерфейсы:

```
configure
interface gi1/0/1
ip address dhcp
ip firewall disable
no shutdown
exit
interface gi1/0/2
no shutdown
exit
interface gi1/0/2.330
ip address 192.168.22.1/29
ip firewall disable
exit
object-group network LAN
ip prefix 192.168.22.0/24
exit
nat source
ruleset MASQUERADE
to interface gi1/0/1
rule 10
match source-address LAN
action source-nat interface
enable
exit
exit
exit
do commit
do confirm
```
Туннель:
to R-DT:

```
tunnel gre 1
ip add 10.10.10.2/30
local interface local interface gi1/0/1
remote address 172.16.4.14
mtu 1400
ttl 64
ip firewall disable
enable
exit
```
to R-HQ:
```
tunnel gre 3
ip add 10.10.12.2/30
local interface local interface gi1/0/1
remote address 172.16.5.14
mtu 1400
ttl 64
ip firewall disable
enable
exit
```



OSPF:
```
r-br(config)#router ospf 1
r-br(config-ospf)#area 0.0.0.0
r-br(config-ospf-area)#network 10.10.10.0/30
r-br(config-ospf-area)#network 10.10.12.0/30
r-br(config-ospf-area)#network 192.168.22.0/29
r-br(config-ospf-area)#enable
r-br(config-ospf-area)#exit
r-br(config-ospf)#enable
r-br(config-ospf)#exit
r-br(config)#
r-br(config-gre)#tunnel gre 1
r-br(config-gre)#ip ospf instance 1
r-br(config-gre)#ip ospf
r-br(config-gre)#exit
r-br(config)#
r-br(config)#do commit
r-br(config)#do confirm
r-br(config)#tunnel gre 1
r-br(config-gre)#ip ospf authentication key ascii-text P@ssw0rd
r-br(config-gre)#ip ospf authentication algorithm cleartext
tr-br(config-gre)#exit
r-br(config)#
r-br(config)#do commit
r-br(config)#do confirm
```


DHCP:
```
ip dhcp-server pool <ИМЯ ПУЛА>
network <IP АДРЕС СЕТИ>
address-range <НАЧАЛЬНЫЙ АДРЕС> - <КОНЕЧНЫЙ АДРЕС>
excluded-address-range <АДРЕС> - <АДРЕС> (уточнить у экспертов)
default-router <АДРЕС ШЛЮЗА> (который настроен на порту!!!)
domain-name "<ДОМЕННОЕ ИМЯ>"
dns-server <АДРЕС DNS 1>, <АДРЕС DNS 2>
exit
ip dhcp-server
```

NTP:

```
ntp enable
ntp server 192.168.11.2
prefer
minpoll 4
exit
clock timezone gmt +3
do commit
do confirm
```

6.      Настроить коммутаторы
 

7.      AD (Samba)
Было:
```
echo nameserver 77.88.8.8 > /etc/net/ifaces/enp6s18/resolv.conf
```
Нужно:
```
echo nameserver 192.168.11.2 > /etc/net/ifaces/enp6s18/resolv.conf
echo search au.team >> /etc/net/ifaces/enp6s18/resolv.conf
cat /etc/net/ifaces/enp6s18/resolv.conf
```

#Обновляем репозитории
```
apt-get update
```


#Устанавливаем требуемые пакеты
```
apt-get install bind
apt-get install bind-utils
apt-get install task-samba-dc
 ```
 
 #Меняем имя на короткое

``` 
hostnamectl set-hostname srv1-hq
#Отключаем chroot:
control bind-chroot disabled
```

#Отключаем KRB5RCACHETYPE

```
vim /etc/sysconfig/bind
```


1.      NTP
 NTP:

```
vim /etc/chrony.con
```


![image](https://github.com/user-attachments/assets/be350bdf-15d5-4305-9ccd-368212b392c4)


```
systemctl restart chronyd
systemctl status chronyd
#Проверяем с каким сервером
синхронизировалось время
chronyc tracking
#Проверяем часовой пояс
timedatectl
#Если часовой пояс отличается от московского
timedatectl set-timezone Europe/Moscow
```

Настройка клиента сервера времени на ALT на базе chrony


#Редактируем конфигурационный файл

```
vim /etc/chrony.conf
```


![image](https://github.com/user-attachments/assets/a8a33d2b-5541-4c15-8f19-e4a244451c63)


#Перезапускаем службу chronyd
systemctl restart chronyd
#Проверяем статус служб chronyd
systemctl status chronyd
#Проверяем с каким сервером
синхронизировалось время
```
chronyc tracking
```

#Проверяем часовой пояс
```
timedatectl
```

#Если часовой пояс отличается от московского
```
timedatectl set-timezone Europe/Moscow
```

RedOS
#Редактируем конфигурационный файл
```
vim /etc/chrony.conf
```

![image](https://github.com/user-attachments/assets/326d7ca2-d519-47e4-b838-0407914e1a91)



#Перезапускаем службу chronyd
```
systemctl restart chronyd
```
#Проверяем статус служб chronyd

```
systemctl status chronyd
```
#Проверяем с каким сервером
синхронизировалось время

```
chronyc tracking
```
#Проверяем часовой пояс
```
timedatectl
```
#Если часовой пояс отличается от московского

```
timedatectl set-timezone Europe/Moscow
```



Настройка OpenvSwitch

```
systemctl enable --now openvswitch
```
Включаем и добавляем OpenvSwitch в автозагрузку
```
systemctl status openvswitch
```
Проверяем работоспособность OpenvSwitch
```
ovs-vsctl show
```
Вывод информации о системе коммутации
```
ovs-vsctl add-br <ИМЯ SW>
```
Добавление виртуального коммутатора. Автоматически
создается интерфейс с аналогичным именем
```
ovs-vsctl add-port <ИМЯ SW> <ИМЯ ПОРТА>
```

Добавление порта в коммутатор (порт должен быть
включен)
```
ovs-vsctl add-port <ИМЯ SW> <ИМЯ ВИРТУАЛЬНОГО ПОРТА>
```
Добавление виртуального порта в коммутатор (выйдет
ошибка, что нет такого порта)
```
ovs-vsctl set Interface <ИМЯ ВИРТУАЛЬНОГО ПОРТА> type=internal
```
Устанавливаем виртуальному порту тип internal

```
ovs-vsctl set Port <ИМЯ ПОРТА> tag=<НОМЕР VLAN>
```

Установить на данном порту номер VLAN
```
ovs-vsctl set Port <ИМЯ ПОРТА> trunks=<VLAN1>,<VLAN2>
```
Установить на данном порту параметры trunk
```
ovs-appctl stp/show
```
Вывод информации о службе STP
```
ovs-vsctl set Bridge <ИМЯ SW> stp_enable=yes

ovs-vsctl set Bridge ${IFACE} rstp_enable=true other_config:rstp-priority=
```
Активация (R)STP





DOCKER
#Установка Docker и Docker-compose
```
dnf install docker-ce
dnf install docker-ce-cli
dnf install docker-compose
```
#Включаем и добавляем службу в автозагрузки
```
systemctl enable docker --now
```
#Прверяем статус службу
```
systemctl status docker
usermod -aG docker <ИМЯ_ПОЛЬЗОВАТЕЛЯ>

docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2
```
#Пишем Dockerfile для приложения web.

```
vi Dockerfile
```
#Развертываем приложение NGINX на базе Alpine
```
FROM nginx:alpine
#Заменяем дефолтную страницу nginx своей
RUN rm -rf /usr/share/nginx/html/*
COPY ./index.html /usr/share/nginx/html
#Добавляем конфигурационный файл для нашего web приложения
COPY ./web.conf /etc/nginx/conf.d/web.conf
#Указываем на необходимость открыть порт
EXPOSE 80
#Переводит Nginx «на передний план» (если этого не сделать, контейнер остановится сразу после запуска)
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

```
vi index.html
```

![image](https://github.com/user-attachments/assets/29fe4a10-b963-4f3f-a62a-d98fedab56e3)
```
vi web.conf
```

![image](https://github.com/user-attachments/assets/20acf607-6828-46c6-a96b-f3f005ccec23)


#Выполняем сборку образа
```
docker build -t web .


#Проверяем наличие собранного образа

docker images

#Присваиваем образу тег для размещения его в локальном Docker Registry
docker tag web localhost:5000/web:1.0
#Загружаем образ в локальный Docker Registry:
docker push localhost:5000/web:1.0
#Проверяем наличие образа локальном Docker Registry
docker images
```

```
#Удаляем образы localhost:5000/web:1.0 и web
docker rmi web
docker rmi localhost:5000/web:1.0
#Проверяем наличие образов
docker images

```
```

#Загружаем образ приложения web из локального Docker Registry:
docker pull localhost:5000/web:1.0
#Проверяем наличие образов
docker images

```

```
#Запускаем приложение web из скаченного образа из локального репозитория
docker run -d --name web -p 80:80 --restart=always localhost:5000/web:1.0
#Проверяем, что контейнер запустился
docker ps -a

```


```

Т.к. наше web приложение работает за NAT, необходимо настроить проброс портов на маршрутизаторе (R-DT)
Задаём правило для проброса порта на R-DT (EcoRouter)
При обращении на внешний адрес маршрутизатора (172.16.4.14) на порт 80 должен происходить проброс на
адрес 192.168.33.3 (SRV2-DT) на порт 80 по протоколу tcp
R-DT(config)#ip nat source static tcp 192.168.33.3 80 172.16.4.14 80

```

ВНИМАНИЕ!
На CLI необходимо прописать имя www.au.team.ru в файл /etc/hosts


![image](https://github.com/user-attachments/assets/79574a9a-4f43-41dd-9dfb-664e933c067b)



```
#В домашней директории пользователя создаём файл monitoring.yml
vim ~/monitoring.yml
#Содержимое файла monitoring.yml


#Выполняем сборку и запуск контейнеров описанных в файле monitoring.yml
docker-compose -f monitoring.yml up -d
```


```
#Переходим в браузер http://<IP адрес SRV2-DT>:3000
#Для доступа в веб-интерфейс Grafana - стандартный логин и пароль "admin"
#Задаём новый пароль (P@ssw0rd) и подтверждаем его
```

```
#Добавляем в Grafana – Prometheus
1) на главном меню нажимаем Add your first data source
2) выбираем Prometheus
3) вводим адрес контейнера с Prometheus
4) внизу на этой же странице нажимаем Save and Test
```

Конечно! Вот полная инструкция для настройки мониторинга с использованием NodeExporter, Prometheus и Grafana с командами:

Шаг 1: Создание файла monitoring.yml для Docker Compose

Откройте терминал на вашей Linux-машине.

Перейдите в домашнюю директорию пользователя:

cd ~
Создайте файл monitoring.yml с помощью текстового редактора (например, nano):

nano monitoring.yml
Вставьте следующий код в файл monitoring.yml:
```
version: '3'
services:
  node-exporter:
    image: prom/node-exporter
    ports:
      - 9100:9100
    restart: always

  prometheus:
    image: prom/prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    restart: always

  grafana:
    image: grafana/grafana
    ports:
      - 3000:3000
    restart: always
```

vim /var/lib/docker/volumes/root_prom-configs/_data/prometheus.yml

![image](https://github.com/user-attachments/assets/cbd057c2-4061-40c0-9b91-9c29d1ead043)


Сохраните и закройте файл.

Шаг 2: Запуск контейнеров NodeExporter, Prometheus и Grafana

В терминале выполните следующую команду для запуска контейнеров:
```
docker-compose -f monitoring.yml up -d
```
Эта команда запустит контейнеры NodeExporter, Prometheus и Grafana в фоновом режиме.

Дождитесь, пока контейнеры полностью запустятся. Вы можете проверить их статус с помощью команды:
```
docker-compose -f monitoring.yml ps
Убедитесь, что все контейнеры имеют статус "Up".
```
Шаг 3: Настройка Dashboard в Grafana

Откройте веб-браузер и перейдите по адресу http://monitoring.au.team:3000 (убедитесь, что у вас настроено соответствующее DNS-разрешение).

Войдите в Grafana с помощью учетных данных по умолчанию (логин: admin, пароль: admin).

Создайте новый Dashboard, нажав на значок плюса (+) в левой панели навигации и выбрав "Dashboard".

Добавьте панели для отображения метрик CPU, памяти и диска, нажав на значок плюса (+) в верхней панели и выбрав "Add Panel".

Настройте запросы Prometheus для получения данных о загрузке CPU, памяти и диска с серверов SRV1-DT, SRV2-DT, SRV3-DT. Для этого в каждой панели выберите "Edit" и настройте соответствующий запрос.

Шаг 4: Доступ к интерфейсу Grafana по имени monitoring.au.team

Откройте файл /etc/hosts с помощью текстового редактора:

sudo nano /etc/hosts
Добавьте следующую строку в файл, указав IP-адрес вашей Linux-машины:
```
<IP-адрес> monitoring.au.team
Замените <IP-адрес> на фактический IP-адрес вашей машины.
```
Сохраните и закройте файл.

Теперь вы должны иметь полностью настроенный мониторинг с использованием NodeExporter, Prometheus и Grafana. Вы сможете просматривать метрики и отображать их на Dashboard в Grafana, а также получать доступ к интерфейсу Grafana по имени monitoring.au.team.


https://docs.google.com/document/d/1rmdDhMv-Ufn_6jkP2p3azbprRBSh5zXlTSKYQoOymeY/edit?pli=1

https://disk.yandex.ru/d/14gvyGRWrfulfw


BACKUP


На SRV1-HQ в директории /etc/systemd/system создаём два файла:
```
backup.timer (определяет, когда наш сервис будет запускаться)
backup.service (описание сервиса).
где backup – название согласно задания
touch /etc/systemd/system/backup.timer
touch /etc/systemd/system/backup.service

```

Содержимое файла backup.timer
```
[Unit]
Description=Backup SHARE timer
#Описание сервиса (любое)
Requires=backup.service
#Указание строгой зависимости с нашим сервисом (backup.service)
[Timer]
Unit=backup.service
#Ссылка на сервис
OnBootSec=0
#Срабатывает через указанное время после запуска системы
OnUnitActiveSec=8h
#Срабатывает через указанное время после активации целевого юнита
[Install]
WantedBy=timers.target
#Уровень запуска сервиса
```


Содержимое файла backup.service

```
[Unit]
Description=Backup SHARE
#Описание сервиса (любое)
Wants=backup.timer
#Зависимость (отсылает к нашему таймеру)
[Service]
Type=simple
#Тип службы
ExecStart=<СКРИПТ или КОМАНДА>
#Указывается полный путь к исполняемому файлу программы
[Install]
WantedBy=multi-user.target
#указывает на запуск в многопользовательском режиме
```


```
systemctl daemon-reload
#Добавляем в автозагрузку и запускаем созданный сервис и таймер
systemctl enable --now backup.service
systemctl enable --now backup.timer
#Проверяем работоспособность сервиса и таймера
systemctl status backup.service
systemctl status backup.timer
```




SAMBA

#Обновляем репозитории
```
apt-get update
```
#Устанавливаем требуемые пакеты
```
apt-get install bind
apt-get install bind-utils
apt-get install task-samba-dc

```


#Меняем имя на короткое
hostnamectl set-hostname srv1-hq


#Отключаем chroot:
control bind-chroot disabled
 
#Отключаем KRB5RCACHETYPE
vim /etc/sysconfig/bind


![image](https://github.com/user-attachments/assets/e4e18ece-5152-4948-9c9d-1e24e6739712)


#Подключаем плагин BIND_DLZ:
vim /etc/bind/named


![image](https://github.com/user-attachments/assets/602dd284-9519-49d8-9591-2ea05cd4689f)


![image](https://github.com/user-attachments/assets/f0a166c3-c255-40b4-8b41-3d9668351d85)


![image](https://github.com/user-attachments/assets/355f4c7f-127b-48e8-acc6-92fa9881d575)



#Очищаем базы и конфигурацию Samba
 
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba

mkdir -p /var/lib/samba/sysvol



#Пример команды создания контроллера домена au.team в пакетном режиме:
 
samba-tool domain provision --realm=au.team --domain=au --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc --use-rfc2307
 


#Включаем и добавляем в автозагрузку службы samba и bind

systemctl enable --now samba
systemctl enable --now bind
 
#Проверяем статус служб

systemctl status samba
systemctl status bind




#Заменяем файл krb5.conf, находящийся в каталоге /etc/ на файл, созданный в момент создания домена Samba
 
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
 
#Меняем адрес DNS сервера на 127.0.0.1
 
echo nameserver 127.0.0.1 > /etc/net/ifaces/enp6s18/resolv.conf
 
#Перезагружаем сервер
 
reboot



samba-tool domain info 127.0.0.1


smbclient -L localhost -Uadministrator


#Проверка имён хостов
 
host -t SRV _kerberos._udp.au.team.
host -t SRV _ldap._tcp.au.team.
host -t A srv1-hq.au.team



#Проверка Kerberos (имя домена должно быть в верхнем регистре):
 
kinit administrator@AU.TEAM
klist



При использовании команды samba-tool dns указание аутентифицирующей информации (имени пользователя и пароля) обязательно!



#Вывести список зон DNS

samba-tool dns zonelist <СЕРВЕР>

#На SRV1-HQ

samba-tool dns zonelist 192.168.11.2 -Uadministrator

#Создать зону DNS

samba-tool dns zonecreate <СЕРВЕР> <ЗОНА>

#На SRV1-HQ

samba-tool dns zonecreate 192.168.11.2 11.168.192.in-addr.arpa -Uadministrator



Согласно нашей схемы, необходимо создать 3 обратные зоны для каждой сети

1. Для офиса HQ выделена сеть 192.168.11.0/24 (создали в примере предыдущего слайда)
2. Для офиса BR выделена сеть 192.168.22.0/24
3. Для офиса DT выделена сеть 192.168.33.0/24


#Добавить новую запись

samba-tool dns add <СЕРВЕР> <ЗОНА> <ИМЯ> <A|AAAA|PTR|CNAME|NS|MX|SRV|TXT> <ДАННЫЕ>

#На SRV1-HQ добавить запись типа A

samba-tool dns add 192.168.11.2 au.team sw1-hq A 192.168.11.82 –Uadministrator

#На SRV1-HQ добавить запись типа PTR для обратной зоны 192.168.11.0/24

samba-tool dns add 192.168.11.2 11.168.192.in-addr.arpa 82 PTR sw1-hq.au.team -Uadministrator



#Вывести информацию о DNS-записях

samba-tool dns query <сервер> <зона> <имя> <A|AAAA|PTR|CNAME|NS|MX|SOA|SRV|TXT|ALL>

#На SRV1-HQ вывести всю информацию о DNS-записях зоны au.team

samba-tool dns query 192.168.11.2 au.team @ ALL -Uadministrator

#На SRV1-HQ вывести всю информацию о DNS-записях зоны 11.168.192.in-addr.arpa

samba-tool dns query 192.168.11.2 11.168.192.in-addr.arpa @ ALL -U administrator



На всех внутренних устройствах (ADMIN, SW, CLI) меняем адрес DNS сервера на адрес SRV1-HQ (в нашем случае 192.168.11.2) и  указываем имя поискового домена (в нашем случае au.team)



Ввод ALT в домен SAMBA
https://docs.altlinux.org/ru-RU/alt-workstation/10.0/html/alt-workstation/activedirectory-login--chapter.html
https://rutube.ru/video/4f626a67f23e22a8a3e5ad65bff7a257/




Ввод RED OS в домен SAMBA
https://redos.red-soft.ru/base/arm/arm-domen/redos-in-samba/



Обновляем репозитории
apt-get update

#Устанавливаем пакет task-auth-ad-sssd:
apt-get install -y task-auth-ad-sssd

#Задаем имя компьютера

#Указываем в поле DNS-серверы DNS-сервер домена и в поле Домены поиска — домен для поиска

#Для ввода компьютера в домен в Центре управления системой (Меню → Центр управления → Центр управления системой) необходимо выбрать пункт Пользователи → Аутентификация.

#В окне модуля Аутентификация следует выбрать пункт Домен Active Directory, заполнить поля (Домен, Рабочая группа, Имя компьютера), выбрать пункт SSSD (в единственном домене) и нажать кнопку Применить.



![image](https://github.com/user-attachments/assets/9ebd29d7-422b-4430-a77b-135180523737)



#Создать пользователя user1 с паролем P@ssw0rd
samba-tool user add user1 P@ssw0rd 

#Просмотреть доступных пользователей:
samba-tool user list

#Создать группу group1
samba-tool group add group1

#Добавить пользователя user1 в группу group1
samba-tool group addmembers group1 user1

#Просмотреть пользователей в группе group1
samba-tool group listmembers group1


Для управления пользователями и группами в AD можно использовать модуль удалённого управления базой данных конфигурации (ADMC)

На рабочей станции администратора с граф. интерфейсом или клиенте необходимо установить пакет admc

apt-get install admc

ADMC позволяет:
- создавать и администрировать учётные записи пользователей, компьютеров и групп;
- менять пароли пользователя;
- создавать организационные подразделения, для структурирования и выстраивания иерархической системы -  распределения учётных записей в AD;
- просматривать и редактировать атрибуты объектов;
- создавать и просматривать объекты групповых политик;
- выполнять поиск объектов по разным критериям;
- сохранять поисковые запросы;
- переносить поисковые запросы между компьютерами (выполнять экспорт и импорт поисковых запросов).



В ADMC реализована функция поиска объектов групповых политик.

Для использования ADMC необходимо предварительно получить ключ Kerberos для администратора домена. Получить ключ Kerberos можно, например, выполнив следующую команду:

kinit administrator


![image](https://github.com/user-attachments/assets/642dfb8c-9c92-4b4c-bdb7-8a7dc5fc52a9)


![image](https://github.com/user-attachments/assets/5efee88e-468c-45e0-961a-15a1ef2a04b1)


![image](https://github.com/user-attachments/assets/50f6713a-3b6f-472e-a2eb-7fe76e1e2b03)


![image](https://github.com/user-attachments/assets/37f0950c-678a-44bf-9e17-c1e53a7f9d4c)


Samba AD на RED OS
#На время настройки переведите SELinux в режим уведомлений
setenforce 0#Проверяем права на доступ к файлу /etc/krb5.conf (Пользователем-владельцем файла должен быть root, а группой-
владельцем – named)
ls -l /etc/krb5.conf
#При необходимости меняем владельца
chown root:named /etc/krb5.conf
#Устанавливаем необходимые пакеты
dnf install samba*
dnf install krb5*
dnf install bind





vi /etc/krb5.conf


![image](https://github.com/user-attachments/assets/e008f51e-02c2-491e-be5d-f9fec897bac5)


#Редактируем файл /etc/krb5.conf.d/crypto-policies
 vi /etc/krb5.conf.d/crypto-policies


![image](https://github.com/user-attachments/assets/2a009986-00a2-4f48-a35d-36c059da4492)


![image](https://github.com/user-attachments/assets/bf15be4e-2a1b-4497-b9fd-74bea0147d0f)



#Очищаем базы и конфигурацию Samba
 
rm -rf /etc/samba/smb.conf


#Присоединение сервера в качестве вторичного контроллера домена

samba-tool domain join au.team DC -U Administrator --dns-backend=BIND9_DLZ --realm=au.team
 
vi /etc/samba/smb.conf


![image](https://github.com/user-attachments/assets/a3ac664d-871f-48d4-976d-f536fec30b09)


 testparm


#Включаем и добавляем в автозагрузку службы samba и bind (named)
systemctl enable --now samba
systemctl enable --now named
 
#Проверяем статус служб
systemctl status samba
systemctl status named

#Проверьте работу динамического обновления DNS
samba_dnsupdate --verbose --all-names



Samba можно настроить как файловый сервер. Samba также можно настроить как сервер печати для совместного доступа к принтеру.

Создаём директорию

mkdir /opt/data

Задаём права на созданную директорию

chmod 777 /opt/data

Описываем общие папки для публикации в конфигурационном файле /etc/samba/smb.conf:

vim /etc/samba/smb.conf


![image](https://github.com/user-attachments/assets/f0bfd0cb-117a-4bfc-8c45-123e28f64627)



где:

[SAMBA] - имя общей папки, для публикации требуемое по заданию;

path — указывает каталог, к которому должен быть предоставлен доступ;

writable — инвертированный синоним для read only (по умолчанию: writeable = no);

read only — если для этого параметра задано значение «yes», то пользователи службы не могут создавать или изменять файлы в каталоге (по умолчанию: read only = yes);

valid users - это список пользователей, которым должно быть разрешено входить в эту службу (по заданию не ТРЕБУЕТСЯ)


Перезапускаем службу samba

systemctl restart samba

Проверяем

smbclient -L localhost -Uadministrator


Настройте синхронизацию времени между сетевыми устройствами по протоколу NTP. 
a) В качестве сервера должен выступать SRV1-HQ 
1. Используйте стратум 5 
2. Используйте ntp2.vniiftri.ru в качестве внешнего сервера синхронизации времени
b) Все устройства должны синхронизировать своё время с SRV1-HQ. 
c) Используйте на всех устройствах московский часовой пояс.
d) Используйте chrony, где это возможно



Настройка сервера времени на SRV1-HQ на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd



![image](https://gitНастройка клиента сервера времени на ALT на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd

hub.com/user-attachments/assets/fd959953-476b-42c4-bf5c-1fabd5c859a1)


Настройка сервера времени на SRV1-HQ на базе chrony

#Проверяем с каким сервером 
синхронизировалось время
chronyc tracking

#Проверяем часовой пояс
timedatectl  

#Если часовой пояс отличается от московского
timedatectl  set-timezone Europe/Moscow



![image](https://github.com/user-attachments/assets/c9967b1b-7ffe-4ba7-a54c-29c8faca0182)



Настройка клиента сервера времени на ALT на базе chrony

#Проверяем с каким сервером 
синхронизировалось время
chronyc tracking

#Проверяем часовой пояс
timedatectl  

#Если часовой пояс отличается от московского
timedatectl  set-timezone Europe/Moscow


![image](https://github.com/user-attachments/assets/ba932857-9d96-4ed7-9d44-ffaa58847477)


Настройка клиента сервера времени на RED OS на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd




![image](https://github.com/user-attachments/assets/45195146-ad1b-4393-bd6d-b4689471aa68)


Настройка OpenvSwitch с помощью Etcnet
ip add
cp -r /etc/net/ifaces/enp6s18 /etc/net/ifaces/enp6s19
cp -r /etc/net/ifaces/enp6s18 /etc/net/ifaces/enp6s20
mkdir /etc/net/ifaces/SW1-HQ
mkdir /etc/net/ifaces/mgmt
vim /etc/net/ifaces/SW1-HQ/options
TYPE=ovsbr
HOST='enp6s18 enp6s19 enp6s20'
OVS_OPTIONS=’stp_enable=yes other_config:stp-
priority=4096’
OVS_EXTRA=’set port enp6s18 trunk=110,220,330 -- set port enp6s19 trunk=110,220,330 -- set port enp6s20 trunk=110,220,330’
mcedit /etc/net/ifaces/mgmt/options
TYPE=ovsport
BOOTPROTO=static
CONFIG_IPV4=yes
BRIDGE=SW1-HQ
VID=330
echo 192.168.11.82/29 > /etc/net/ifaces/mgmt/ipv4address
echo default via 192.168.11.81 > /etc/net/ifaces/mgmt/ipv4route
echo nameserver 77.88.8.8 > /etc/net/ifaces/mgmt/resolv.conf
systemctl restart network
ovs-vsctl show
ovs-appctl stp/show



![image](https://github.com/user-attachments/assets/68fbdd5d-70ca-4cdc-b7a8-7fc10ac9ad20)



