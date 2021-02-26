# lan
азворачиваем сетевую лабораторию

# otus-linux
Vagrantfile - для стенда урока 9 - Network

# Дано
https://github.com/erlong15/otus-linux/tree/network
(ветка network)

Vagrantfile с начальным построением сети
- inetRouter
- centralRouter
- centralServer

тестировалось на virtualbox

# Планируемая архитектура
построить следующую архитектуру

Сеть office1
- 192.168.2.0/26 - dev
- 192.168.2.64/26 - test servers
- 192.168.2.128/26 - managers
- 192.168.2.192/26 - office hardware

Сеть office2
- 192.168.1.0/25 - dev
- 192.168.1.128/26 - test servers
- 192.168.1.192/26 - office hardware


Сеть central
- 192.168.0.0/28 - directors
- 192.168.0.32/28 - office hardware
- 192.168.0.64/26 - wifi

```
Office1 ---\
-----> Central --IRouter --> internet
Office2----/
```
Итого должны получится следующие сервера
- inetRouter
- centralRouter
- office1Router
- office2Router
- centralServer
- office1Server
- office2Server

# Теоретическая часть
- Найти свободные подсети
- Посчитать сколько узлов в каждой подсети, включая свободные
- Указать broadcast адрес для каждой подсети
- проверить нет ли ошибок при разбиении

# Практическая часть
- Соединить офисы в сеть согласно схеме и настроить роутинг
- Все сервера и роутеры должны ходить в инет черз inetRouter
- Все сервера должны видеть друг друга
- у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи
- при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс

# 1. Теория
Для начала подсчитаем сколько узлов в каждой подсети и нам сразу станет понятно сколько у нас свободных. воспользуемся утилитой ipcalc
 - Сеть office 1
192.168.2.0/26 - dev
```ruby
ipcalc 192.168.2.0/26                           
Network:        192.168.2.0/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.2.63

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.2.1
HostMax:        192.168.2.62
Hosts/Net:      62
```
- 192.168.2.64/26 - test servers
```ruby
ipcalc 192.168.2.64/26                                                                
Network:        192.168.2.64/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.2.127

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.2.65
HostMax:        192.168.2.126
Hosts/Net:      62
```
 - 192.168.2.128/26 - managers
```ruby
ipcalc 192.168.2.128/26          
Network:        192.168.2.128/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.2.191

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.2.129
HostMax:        192.168.2.190
Hosts/Net:      62
```
- 192.168.2.192/26 - office hardware
```ruby
ipcalc 192.168.2.192/26                                                               
Network:        192.168.2.192/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.2.255

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.2.193
HostMax:        192.168.2.254
Hosts/Net:      62
```
- Сеть office2

- 192.168.1.0/25 - dev
```ruby
ipcalc 192.168.1.0/25                                                                
Network:        192.168.1.0/25
Netmask:        255.255.255.128 = 25
Broadcast:      192.168.1.127

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.1.1
HostMax:        192.168.1.126
Hosts/Net:      126
```
- 192.168.1.128/26 - test servers
```ruby
ipcalc 192.168.1.128/26  
Network:        192.168.1.128/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.1.191

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.1.129
HostMax:        192.168.1.190
Hosts/Net:      62
```
- 192.168.1.192/26 - office hardware
```ruby
ipcalc 192.168.1.192/26                                                               
Network:        192.168.1.192/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.1.255

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.1.193
HostMax:        192.168.1.254
Hosts/Net:      62
```
- Сеть central

- 192.168.0.0/28 - directors
```ruby
ipcalc 192.168.0.0/28                                                           
Network:        192.168.0.0/28
Netmask:        255.255.255.240 = 28
Broadcast:      192.168.0.15

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.1
HostMax:        192.168.0.14
Hosts/Net:      14
```
- 192.168.0.32/28 - office hardware
```ruby
Network:        192.168.0.32/28
Netmask:        255.255.255.240 = 28
Broadcast:      192.168.0.47

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.33
HostMax:        192.168.0.46
Hosts/Net:      14
```
- 192.168.0.64/26 - wifi
```ruby
ipcalc 192.168.0.64/26                                                                
Network:        192.168.0.64/26
Netmask:        255.255.255.192 = 26
Broadcast:      192.168.0.127

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.65
HostMax:        192.168.0.126
Hosts/Net:      62
```
для наглядности и удобного анализа составим таблицу

<body link="#0563C1" vlink="#954F72">

<table border=0 cellpadding=0 cellspacing=0 width=737 style='border-collapse:
 collapse;table-layout:fixed;width:554pt'>
 <col width=98 style='mso-width-source:userset;mso-width-alt:3584;width:74pt'>
 <col width=104 style='mso-width-source:userset;mso-width-alt:3803;width:78pt'>
 <col width=124 style='mso-width-source:userset;mso-width-alt:4534;width:93pt'>
 <col width=118 span=2 style='mso-width-source:userset;mso-width-alt:4315;
 width:89pt'>
 <col width=111 style='mso-width-source:userset;mso-width-alt:4059;width:83pt'>
 <col width=64 style='width:48pt'>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl67 width=98 style='height:15.0pt;width:74pt'>Cеть</td>
  <td class=xl67 width=104 style='border-left:none;width:78pt'>Название</td>
  <td class=xl67 width=124 style='border-left:none;width:93pt'>Подсеть</td>
  <td class=xl67 width=118 style='border-left:none;width:89pt'>Hostmin</td>
  <td class=xl67 width=118 style='border-left:none;width:89pt'>Hostmax</td>
  <td class=xl67 width=111 style='border-left:none;width:83pt'>Broadcast</td>
  <td class=xl67 width=64 style='border-left:none;width:48pt'>Hosts</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl66 style='height:15.0pt;border-top:none'>office1</td>
  <td class=xl65 style='border-top:none;border-left:none'>dev</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.0/26</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.1</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.62</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.63</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>test servers</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.64/26</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.65</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.126</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.127</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>managers</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.128/26</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.129</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.190</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.191</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>office hardware</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.192/26</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.193</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.254</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.2.255</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl66 style='height:15.0pt;border-top:none'>office2</td>
  <td class=xl65 style='border-top:none;border-left:none'>dev</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.0/25</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.1</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.126</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.127</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>126</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>test servers</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.128/26<span
  style='mso-spacerun:yes'>  </span></td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.129</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.190</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.191</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>office hardware</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.192/26<span
  style='mso-spacerun:yes'> </span></td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.193</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.254</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.1.255</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl66 style='height:15.0pt;border-top:none'>central</td>
  <td class=xl65 style='border-top:none;border-left:none'>directors</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.0/28<span
  style='mso-spacerun:yes'> </span></td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.1</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.14</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.15</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>14</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>office hardware</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.32/28</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.33</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.46</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.47</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>14</td>
 </tr>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 style='height:15.0pt;border-top:none'>&nbsp;</td>
  <td class=xl65 style='border-top:none;border-left:none'>wifi</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.64/26</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.65</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.126</td>
  <td class=xl65 style='border-top:none;border-left:none'>192.168.0.127</td>
  <td class=xl65 align=right style='border-top:none;border-left:none'>62</td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=98 style='width:74pt'></td>
  <td width=104 style='width:78pt'></td>
  <td width=124 style='width:93pt'></td>
  <td width=118 style='width:89pt'></td>
  <td width=118 style='width:89pt'></td>
  <td width=111 style='width:83pt'></td>
  <td width=64 style='width:48pt'></td>
 </tr>
 <![endif]>
</table>

</body>

</html>

На таблице видно что office1 и office2 заюзаны все подсети, свободные есть только в central
```ruby
 ipcalc 192.168.0.16/28                                                               
Netmask:        255.255.255.240 = 28
Broadcast:      192.168.0.31

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.17
HostMax:        192.168.0.30
Hosts/Net:      14
```
```ruby
ipcalc 192.168.0.48/28                                                                ─
Netmask:        255.255.255.240 = 28
Broadcast:      192.168.0.63

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.49
HostMax:        192.168.0.62
Hosts/Net:      14
```
```ruby
ipcalc 192.168.0.128/25                                                               
Netmask:        255.255.255.128 = 25
Broadcast:      192.168.0.255

Address space:  Private Use
Address class:  Class C
HostMin:        192.168.0.129
HostMax:        192.168.0.254
Hosts/Net:      126
```
Ошибок при разбение не нашел

# 2. Практика
В вагрант файлик пропишем некоторые особености 
1. пропишем сети в iptables.
```ruby
iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
ip route add 192.168.0.0/16 via 192.168.255.2
```
Удалим дефолтные маршруты
```ruby
echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
```
для удобства пользования внесем правки /etc/hosts
```ruby
sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
sudo echo "192.168.254.2 office1Router" >> /etc/hosts
sudo echo "192.168.253.2 office2Router" >> /etc/hosts
sudo echo "192.168.0.2 centralServer" >> /etc/hosts
sudo echo "192.168.2.2 office1Server" >> /etc/hosts
sudo echo "192.168.1.2 office2Server" >> /etc/hosts
```
Проверим доступность нашей сети
```ruby
[vagrant@centralServer ~]$ ping inetRouter
PING inetRouter (192.168.255.1) 56(84) bytes of data.
64 bytes from inetRouter (192.168.255.1): icmp_seq=1 ttl=63 time=0.810 ms
64 bytes from inetRouter (192.168.255.1): icmp_seq=2 ttl=63 time=0.843 ms
64 bytes from inetRouter (192.168.255.1): icmp_seq=3 ttl=63 time=0.813 ms
64 bytes from inetRouter (192.168.255.1): icmp_seq=4 ttl=63 time=0.765 ms
```
```ruby
[vagrant@office1Server ~]$ ping office1Router
PING office1Router (192.168.254.2) 56(84) bytes of data.
64 bytes from office1Router (192.168.254.2): icmp_seq=1 ttl=64 time=0.442 ms
64 bytes from office1Router (192.168.254.2): icmp_seq=2 ttl=64 time=0.417 ms
64 bytes from office1Router (192.168.254.2): icmp_seq=3 ttl=64 time=0.398 ms
```

