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

#Cеть	#Название	#Подсеть		#Hostmin	#Hostmax	#Broadcast	#Hosts
office1	dev		192.168.2.0/26		192.168.2.1	192.168.2.62	192.168.2.63	62
	test servers	192.168.2.64/26		192.168.2.65	192.168.2.126	192.168.2.127	62
	managers	192.168.2.128/26	192.168.2.129	192.168.2.190	192.168.2.191	62
	office hardware	192.168.2.192/26	192.168.2.193	192.168.2.254	192.168.2.255	62
office2	dev		192.168.1.0/25		192.168.1.1	192.168.1.126	192.168.1.127	126
	test servers	192.168.1.128/26  	192.168.1.129	192.168.1.190	192.168.1.191	62
	office hardware	192.168.1.192/26 	192.168.1.193	192.168.1.254	192.168.1.255	62
central	directors	192.168.0.0/28 		192.168.0.1	192.168.0.14	192.168.0.15	14
	office hardware	192.168.0.32/28		192.168.0.33	192.168.0.46	192.168.0.47	14
	wifi		192.168.0.64/26		192.168.0.65	192.168.0.126	192.168.0.127	62

<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8">
  <title>Тег table</title>
  <style>
   table {
    width: 100%; /* Ширина таблицы */
    background: white; /* Цвет фона таблицы */
    color: white; /* Цвет текста */
    border-spacing: 1px; /* Расстояние между ячейками */
   }
   td, th {
    background: maroon; /* Цвет фона ячеек */
    padding: 5px; /* Поля вокруг текста */
   }
  </style>
 </head> 
 <body>
  <table>
   <tr><th>Заголовок 1</th><th>Заголовок 2</th></tr>
   <tr><td>Ячейка 3</td><td>Ячейка 4</td></tr>
  </table>
 </body>
</html>
