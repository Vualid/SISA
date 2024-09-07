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
