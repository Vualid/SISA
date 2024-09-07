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
