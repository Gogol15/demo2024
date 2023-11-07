# Demo2024
## Задание 1:
__Выполните базовую настройку всех устройств:__
- Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
- Присвоить имена в соответствии с топологией
- Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
- Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
- Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
### Топология:
![image](https://github.com/Gogol15/demo2024/assets/79337104/78665b36-aed6-402f-8ac7-0b6b818d018c)

### Таблица адресов сети:
| Имя устройства | Интерфейс |   IPv4/IPv6   | Маска/Префикс |      Шлюз      |
| -------------- | --------- | ------------- | ------------- | -------------- |
|                | ens192    | 192.168.0.126 | /30           |                |
| ISP            | ens224    | 192.168.0.121 | /30           |                |
|                | ens256    | 10.12.26.13   | /24           |                |
| HQ-R           | ens192    | 192.168.0.5   | /25           |                |
|                | ens224    | 192.168.0.125 | /30           |                |             
| BR-R           | ens192    | 192.168.0.89  | /27           |                |
|                | ens224    | 192.168.0.120 | /30           |                |
| HQ-SRV         | ens192    | 192.168.0.6   | /25           |  192.168.0.5   |
| BR-SRV         | ens192    | 192.168.0.88  | /27           |  192.168.0.89  |

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
address 192.168.0.126
netmask 255.255.255.0

auto ens224
iface ens224 inet static
address 192.168.0.121
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
address 192.168.0.5
netmask 255.255.255.128

auto ens224
iface ens224 inet static
address 192.168.0.125
netmask 255.255.255.252
gateway 192.168.0.126
~~~
#### BR-R
~~~
auto ens192
iface ens192 inet static
address 192.168.0.89
netmask 255.255.255.224

auto ens224
iface ens224 inet static
address 192.168.0.120
netmask 255.255.255.252
gateway 192.168.0.121
~~~
#### HQ-SRV
~~~
auto ens192
iface ens192 inet static
address 192.168.0.6
netmask 255.255.255.128
gateway 190.168.0.5
~~~
#### BR-SRV
~~~
auto ens192
iface ens192 inet static
address 192.168.0.88
netmask 255.255.255.224
gateway 192.168.0.89
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
ip tables -A POSTROUTING -t nat -j MAS QUERADE
~~~
Затем нужно создать файл для автоматического запуска NAT после перезагрузки:
~~~
nano /etc/network/if-pre-up.d/nat
~~~
В файл нужно вписать следующее:
~~~
#!/bin/sh
/sbin/iptables -A POSTROUTING -t nat -j MASQERADE
~~~
И даём соответствующие права файлу:
~~~
chmod +x /etc/network/if-pre-up.d/nat
~~~
### То же самое проделываем и на других устройствах

### И нужно проделать те же самые действия с остальными виртуальными машинами.
## Задание 1.2
Настроить внутренюю динамическую маршрутицацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутицации из расчёта, что в дальнейшем сеть будет масштабиоваться.
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
### Введём команду `vtysh` для входа в среду роутера и работы со следующими командами:
~~~
sh int br
~~~
![image](https://github.com/Gogol15/demo2024/assets/79337104/32c4e0dd-996b-4e9f-b430-43bd8a62f4f8)
### Настроим ospf, прописав IP-адреса BR-R и HQ-R
~~~
conf t
router ospf
net 192.168.0.89/30 area 0
net 192.168.0.5/30 area 0
~~~
__И далее нужно натроить __frr__ на BR-R и HQ-R__
### Затем нужно будет пропинговать машины, дабы убедиться в правильности настройки:

`HQ-SRV` __-__ `BR-SRV`

`BR-SRV`__-__ `HQ-SRV`
