## Задание 1:
__Выполните базовую настройку всех устройств:__
- Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
- Присвоить имена в соответствии с топологией
- Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
- Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
- Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
### Топология:
<img width="670" alt="Снимок экрана 2023-11-20 в 15 32 52" src="https://github.com/Gogol15/demo2024/assets/79337104/d5e4866b-73c8-4335-88fe-2bb9571ab731">

### Таблица адресов сети:
| Имя устройства | Интерфейс |   IPv4/IPv6   | Маска/Префикс |      Шлюз      |
| -------------- | --------- | ------------- | ------------- | -------------- |
|                | ens192    | 192.168.0.162 | /30           |                |
| ISP            | ens224    | 192.168.0.166 | /30           |                |
|                | ens256    | 10.12.26.13   | /24           |                |
| HQ-R           | ens192    | 192.168.0.2   | /25           |                |
|                | ens224    | 192.168.0.161 | /30           |                |             
| BR-R           | ens192    | 192.168.0.130 | /27           |                |
|                | ens224    | 192.168.0.165 | /30           |                |
| HQ-SRV         | ens192    | 192.168.0.1   | /25           |  192.168.0.2   |
| BR-SRV         | ens192    | 192.168.0.129 | /27           |  192.168.0.130 |

## Задание 1.1 
## Процесс настройки интерфейсов.
### Для просмотра существующих интерфейсов пропишем следующую команду: 
~~~
ip a
~~~
### Далее зайдём в файл конфигурации наших интерфейсов на всех машинах в порядке очереди:
~~~
nano /etc/network/intefaces
~~~
### Затем нужно будет настоить IP адресацию в соответствии с нашей таблицей. Выглядеть это должно следующим способом (после установки IP адресации, не забудьте перезагрузить сеть):
#### ISP
~~~
auto ens192
iface ens192 inet static
address 192.168.0.162
netmask 255.255.255.0

auto ens224
iface ens224 inet static
address 192.168.0.166
netmask 255.255.255.252

auto ens256
iface ens256 inet static
address 10.12.26.13
netmask 255.255.255.224
gateway 10.12.26.254
~~~
#### HQ-R
~~~
auto ens192
iface ens192 inet static
address 192.168.0.2
netmask 255.255.255.128

auto ens224
iface ens224 inet static
address 192.168.0.161
netmask 255.255.255.252
gateway 192.168.0.162
~~~
#### BR-R
~~~
auto ens192
iface ens192 inet static
address 192.168.0.130
netmask 255.255.255.224

auto ens224
iface ens224 inet static
address 192.168.0.165
netmask 255.255.255.252
gateway 192.168.0.166
~~~
#### HQ-SRV
~~~
auto ens192
iface ens192 inet static
address 192.168.0.1
netmask 255.255.255.128
gateway 190.168.0.2
~~~
#### BR-SRV
~~~
auto ens192
iface ens192 inet static
address 192.168.0.129
netmask 255.255.255.224
gateway 192.168.0.130
~~~
### Сохранить конфигурацию и выйти из настройки сочетанием клавиш:
`ctrl + S` и `ctrl + X`
### Не забудьте перезагрузить сеть во всех машинах:
~~~
systemctl restart networking
~~~
## Насройка NAT
На ISP прописываем следующую команду:
~~~
apt install iptables
~~~
Далее зайдём в конфигурацию файла `sysctl.conf`
~~~
nano /etc/sysctl.conf
~~~
В файле убираем знак решётки со следующей строки:
~~~
#net.ipv4.ip_forward=1
~~~
__Сохраняем конфигурацию и выходим из неё__

`CTRL + S`

`CTRL + X`

Проверяем выполнение с помощью команды `sysctl -p`

Следующим шагом нам нужно прописать следующее:
~~~
iptables -A POSTROUTING -t nat -j MASQUERADE
~~~
Затем нужно создать файл для автоматического запуска NAT после перезагрузки:
~~~
nano /etc/network/if-pre-up.d/nat
~~~
В файл нужно вписать следующее:
~~~
#!/bin/sh
/sbin/iptables -A POSTROUTING -t nat -j MASQUERADE
~~~
И даём соответствующие права файлу:
~~~
chmod +x /etc/network/if-pre-up.d/nat
~~~
### То же самое проделываем и на HQ-R и BR-R
## Задание 1.2
#### Настроить внутренюю динамическую маршрутицацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутицации из расчёта, что в дальнейшем сеть будет масштабиоваться.
### Изначально нужно установить пакет frr:
~~~
apt update & apt upgrade
apt install frr
~~~
### После завершения установки проверим состояние пакета
~~~
systemctl status frr
~~~
### Теперь нужно следующую команду:
~~~
nano /etc/frr/daemons
~~~
### Меняем параметр `ospfd` с `=no` на `=yes`
~~~
ospfd=yes
~~~
### Перезагружаем frr:
~~~
systemctl restart frr
~~~
### Введём команду `vtysh` для входа в среду роутера
### Настроим ospf, прописав IP-адреса BR-R и HQ-R
~~~
conf t
router ospf
net 192.168.0.160/30 area 0
net 192.168.0.164/30 area 0
ip forwarding
~~~
__И далее нужно натроить __frr__ на BR-R и HQ-R__
### Затем нужно будет пропинговать машины, дабы убедиться в правильности настройки:

`HQ-SRV` __-__ `BR-SRV`

`BR-SRV`__-__ `HQ-SRV`

## Задание 1.3
#### Настроить DHCP IP-адресов на HQ-R

### Загружаем нужный пакет, после чего заходим в конфиг
~~~
apt install isc-dhcp-server
nano /etc/default/isc-dhcp-server
~~~
### В конфиге указываем интерфейс
~~~
INTERFACE="ens192"
~~~
### Затем открываем другую конфигурацию
~~~
nano /etc/dhcp/dhcpd.conf
~~~
### Уже в ней настраиваем IP-адреса
~~~
subnet 192.168.0.0 netmask 255.255.255.0 {
 range 192.168.0.4 192.168.0.168;
 option routers 192.168.0.1;
}
~~~
### Далее нужно перейти на HQ-SRV и открыть конфигурацию IP-адресов. В ней написать следующее:
~~~
auto ens192
iface ens192 inet dhcp
#address 192.168.0.6
#netmask 255.255.255.128
#gateway 192.168.0.5
~~~
### Перезагружаем сам DHCP и сервер
~~~
systemctl restart isc-dhcp-server.service
systemctl restart networking
~~~
## Задание 1.4
#### Настройка локальных учетных записей на наших машинах 
| Учетная запись |	 Пароль 	|   Примечание    |
|:--------------:|:--------:|:---------------:|
|      Admin     |	P@ssw0rd	| CLI HQ-SRV HQ-R |
|  Branch admin  |	P@ssw0rd	|   BR-SRV BR-R   |
|  Network admin | P@ssw0rd	| HQ-R BR-R BR-SRV|
### Создаём нового пользователя командой 
~~~
useradd (имя пользователя без скобок)
~~~
### Задаём пароль для пользователей
~~~
passwd (имя пользователя без скобок)
~~~
### Проверяем пользователей 
~~~
nano /etc/passwd
~~~
### Эту работу проделываем на всех вышеуказанных устройствах

## Задание 1.5
#### Измерить пропускную способность сети между HQ-SRV и BR-SRV при помощи iperf3
### Для начала установим утилиту на обеих машинах
~~~
apt install iperf3
~~~
### Следкющим шагом на устройстве, что будет выполнять роль сервера, а в нашем случае, `BR-SRV` пишем следующее:
```
iperf3 -s
```
(Поскольку по умолчанию у сервера открыт порт 5021, мы можем его поменять, по желанию):
```
iperf3 -s -p 5024
```
### Не забудем про то, что на клиенте (у нас это `HQ-SRV`) нам нужно написать IP-адрес принимающего сервера с его портом:
```
iperf3 -c 192.168.0.129 -p 5024
```
### Выглядеть это должно подобным образом:
`BR-SRV`

<img width="623" alt="Снимок экрана 2023-11-20 в 18 03 39" src="https://github.com/Gogol15/demo2024/assets/79337104/d0919470-aeae-4708-bcae-7ef2bce1a8b8">

`HQ-SRV`


<img width="573" alt="Снимок экрана 2023-11-20 в 18 05 38" src="https://github.com/Gogol15/demo2024/assets/79337104/6fe8f4a1-4670-48fc-907b-72af675bf82f">

## Задание 1.6
Составление backup скриптов для сохранения конфигурации сетевых устройств HQ-R и BR-R
### Для начала установим утилиты на обеих машинах
~~~
apt install rsync -y
~~~
### Затем создадим каталог для сохранения бэкапов
~~~
nano /etc/networkbackup
~~~
### Далее зайём в каталог crontab (каталог для выполнения задач по расписанию)
~~~
crontab -e
~~~
