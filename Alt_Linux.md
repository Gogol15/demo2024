# ALT - LINUX    

## Модуль 1 задание 1

<details>
<summary>Базовая настройка всех устройств</summary>
    
a. Собрать топологию согласно рисунку. Все устройства работают на OC Linux - ALT
- ISP - Альт Сервер 10.2 (CLI)
- CLI - Альт Рабочая станция 10.2 (GUI)
- HQ-R - Альт Сервер 10.2 (CLI)
- HQ-SRV - Альт Сервер 10.2 (GUI)
- BR-R - Альт Сервер 10.2 (CLI)
- BR-SRV - Альт Сервер 10.2 (CLI)
b. Присвоить имена в соответствии с топологией
c. Рассчитать IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактировать таблицу.
d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустить этот пункт.
e. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустить этот пункт.
## Топология сети:
![image](https://github.com/Gogol15/demo2024/assets/79337104/f9e240bb-93ec-4c99-a504-37c2a01528f2)
## Таблица адресации:
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
## Настройка статической маршрутизации:
Для начала узнаем, какие папки интерфейсов отображаются на `ISP`
~~~
ls /etc/net/ifaces/
~~~
После того, как нам известны существующие интерфейсы, в случае недостающих, добавляем их:
~~~
mkdir /etc/net/ifaces/ensxxx
~~~
Далее с помощью службы `vim` или `nano` открываем файл `options` в папке с нашим интефейсом `(vim /etc/net/ifaces/ensxxx/options)` и в него вводим:
~~~
TYPE=eth
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
~~~
Затем добавляем новый IP-адрес на интерфейс:
~~~
echo 192.168.0.170/30 > /etc/net/ifaces/ens192/ipv4address
~~~
В случае, если нужен шлюз, то пишмем:
~~~
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/ensxxx/ipv4route
~~~
И перезагружаем сетевой сервис
~~~
systemctl restart network
~~~
Проверяем результат командой `ip a`

**Если на одном интерфейсе отображается 2 разных ip aдреса, то пишем команду:**
~~~
systemctl disable network.service NetworkManager
~~~

</details>

<details>
<summary>Настройка тунеля на роутерах</summary> 
    
Создаём директорию, ведущую к новому интерфейсу `tun1`
~~~
mkdir /etc/net/ifaces/tun1
~~~
Открываем файл `options` с использованием `vim` или `nano` (я пользуюсь vim)
~~~
vim /etc/net/ifaces/tun1/options
~~~
В нём указываем следующее:

![image](https://github.com/Gogol15/demo2024/assets/79337104/5790c202-103a-4648-a5db-a39851baa503)

**Где `TUNLOCAL` – ip адрес внешнего интефейса настраиваемой машины (HQ-R)**

**А `TUNREMOTE` – это ip адрес внешнего интерфейса второй машины (BR-R)**
## Настройка NAT на ISP, HQ-R, BR-R.
Для начала, на устройстве ISP установим сервис Firewalld:
~~~
apt-get install firewalld
~~~
Включаем автозагрузку
~~~
systemctl enable --now firewalld
~~~
Добавляем правила к исходящим и входящим пакетам:
~~~
firewall-cmd --permanent --zone=public --add-interface=ens33
firewall-cmd --permanent --zone=trusted --add-interface=ens34
~~~
Включаем сам NAT:
~~~
firewall-cmd --permanent --zone=public --add-masquerade
~~~
И сохраняем правила:
~~~
firewall-cmd --reload
~~~
**Всё то же самое выполняем на HQ-R и BR-R**

---

## Настройка динамической маршрутизации FRR
Для начала установим FRR и включим его автозагрузку:
```
apt-get -y install frr
systemctl enable --now frr
```
Далее включение демона:  
```
nano /etc/frr/daemons
```
Меняем `ospfd=no`  
На `ospfd =yes`

После заходим в среду роутера через `vtysh`
И прописываем:  
```
conf t
    router ospf
    net 192.168.0.160/30 area 0
    net 192.168.0.164/30 area 0
    exit
ip forwarding
do w
```    
Иногда настройки vtysh слетают, и чтобы такого не происходило, заходим в:
```
nano /etc/frr/frr.conf
```
И добавляем после `ipv6 forwarding` такую строчку:
```
ip forwarding
```
**Все это проделывается на HQ-R и BR-R**

</details>

## Модуль 1 задание 2

<details>
    <summary>NAT с помощью iptables</summary>

Включить ip-адресацию `/etc/net/sysctl.conf`
```
net.ipv4.ip_forward = 1
```

Приминить изменения
```
sudo sysctl -p
```

Интерфейсы:
- `eth0` - внешний интерфейс
- `eth1` - внутрений интерфейс

Интерфейс с раздачей интернета:
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Разрешения на передачу адресации:
внутри
```
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```
снаружи
```
iptables -A FORWARD -i eth0 -o eth1 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Сохранить настройку:
```
iptables-save
```

</details>

<details>
    <summary>NAT с помощью firewalld</summary>
    
Отключить NetworkManager:
```
systemctl disable network.service NetworkManager
```
Настройки интерфейсов должны быть такими:
```
NM_CONTROLLED=no
DISABLED=no
```
Установка firewalld:
```
apt-get -y install firewalld
```
Автозагрузка:
```
systemctl enable --now firewalld
```
Правила к исходящим пакетам:
```
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входящим пакетам:
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --reload
```

</details>

## Модуль 1 задание 3

<details><summary>Маршрутизация через frr</summary>

Настройте внутреннюю динамическую маршрутизацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутизации из расчёта, что в дальнейшем сеть будет масштабироваться.  
a. Составьте топологию сети L3.  

Установка пакета:
```
apt-get -y install frr
```
Автозагрузка:
```
systemctl enable --now frr
```
Включение демона службы ospf:
```
nano /etc/frr/daemons
```
```
ospfd=yes
```
```
systemctl restart frr
```
Вход в среду роутера:
```
vtysh
```
Показать интерфейсы:
```
sh in br
```
|Interface|Status|VRF|Adresses|
|:----:|:-:|:------:|:--------------:|
|ens224|up |default |192.168.0.162/30|
|ens192|up |default |192.168.0.129/27|
|lo    |up |default |                |

Активировать ospf:
```
router ospf
```
Вводим СЕТИ:
```
net 192.168.0.160/30 area 0
net 192.168.0.128/27 area 0
```
Показать соседей:
```
do sh ip ospf neighbor
```
СОХРАНИТЬ КОНФИГИ:
```
do w
```

![image](https://github.com/abdurrah1m/DEMO2024/assets/148451230/a39631c1-a683-47d2-a63a-4bbb93d7556a)
</details>

## Модуль 1 задание 4

<details><summary>Раздача ip-адресов через dhcp</summary>

Настройте автоматическое распределение IP-адресов на роутере HQ-R.  
a. Учтите, что у сервера должен быть зарезервирован адрес.

Установка пакета:
```
apt-get -y install dhcp-server
```
`/etc/sysconfig/dhcpd`, указываю интерфейс внутренней сети:
```
DHCPDARGS=ens19
```
Копирую образец:
```
cp /etc/dhcp/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
```
`/etc/dhcp/dhcpd.conf` параметры раздачи:
```
ddns-update-style-none;

subnet 192.168.0.0 netmask 255.255.255.128 {
        option routers                  192.168.0.1;
        option subnet-mask              255.255.255.128;
        option domain-name-servers      8.8.8.8, 8.8.4.4;

        range dynamic-bootp 192.168.0.20 192.168.0.50;
        default-lease-time 21600;
        max-lease-time 43200;
}
```
```
systemctl restart dhcpd
```
```
systemctl status dhcpd.service
```
Автозагрузка:
```
chkconfig dhcpd on
service dhcpd start
```
HQ-SRV (клиент):
```
nano /etc/net/ifaces/ens18/ipv4address
```
```
#192.168.0.40
```
```
nano /etc/net/ifaces/ens18/options
```
```
BOOTROTO=dhcp
TYPE=eth
NM_CONTROLLED=yes
DISABLED=no
CONFIG_IPV4=yes
```
```
service network restart
```
```
ens18:
    inet 192.168.0.38/25 brd 192.168.0.127
```
</details>

## Модуль 1 задание 5

<details><summary>Добавление пользователей на виртуальные машины</summary>

Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

|Учётная запись|Пароль|Примечание|
|:--------------:|:------:|:----------------:|
|Admin           |P@ssw0rd|CLI, HQ-SRV       |
|Branch admin    |P@ssw0rd|BR-SRV, BR-R      |
|Network admin   |P@ssw0rd|HQ-R, BR-R, HQ-SRV|

Пользователь `admin` на `HQ-SRV`
```
adduser admin
```
```
usermod -aG root admin
```
```
passwd admin
P@ssw0rd
P@ssw0rd
```
```
nano /etc/passwd
```
```
admin:x:0:501::/home/admin:/bin/bash
```
</details>

## Модуль 1 задание 6

<details><summary>Измерьте пропускную способность сети между двумя узлами</summary>


Измерьте пропускную способность сети между двумя узлами HQ-R-ISP по средствам утилиты iperf 3. Предоставьте описание пропускной способности канала со скриншотами.

```
apt-get -y install iperf3
```
ISP как сервер:
если надо открыть порт
```
iptables -A INPUT -p tcp --dport 5201 -j ACCEPT
```
```
iperf3 -s
```
HQ-R:
```
iperf3 -c 192.168.0.161 -f M
```
```
[ID] Interval      Transfer   Bitrate        Retr Cwnd
[ 5] 0.00-1.00 sec 345 MBytes 344 MBytes/sec    0 538 KBytes
[ 5] 1.00-2.00 sec 338 MBytes 338 MBytes/sec    0 676 KBytes
[ 5] 3.00-4.00 sec 341 MBytes 341 MBytes/sec    0 749 KBytes
```
</details>

## Модуль 1 задание 7

<details><summary>backup скрипты для сохранения конфигурации сетевых устройств</summary>

Составьте backup скрипты для сохранения конфигурации сетевых устройств, а именно HQ-R BR-R. Продемонстрируйте их работу.

Заход в планировщик заданий:
```
EDITOR=nano crontab -e
```
минута | час | день | месяц | день недели | "команда, например `reboot`":
```
9 15 * * * cp /etc/frr/frr.conf /etc/networkbackup
```
```
ls /etc/networkbackup
```
```
frr.conf
```
</details>

## Модуль 1 задание 8

<details><summary>подключение по SSH для удалённого конфигурирования устройства</summary>

Настройте подключение по SSH для удалённого конфигурирования устройства HQ-SRV по порту 2222. Учтите, что вам необходимо перенаправить трафик на этот порт по средствам контролирования трафика.

HQ-SRV:
```
apt-get -y install openssh-server
```
```
systemctl enable --now sshd
```
```
nano /etc/openssh/sshd_config
```
```
Port 2222
PermitRootLogin no
PasswordAuthentication yes
```
Подключение
```
ssh student@192.168.0.40 -p 2222
```

</details>

## Модуль 1 задание 9

<details><summary>контроль доступа до HQ-SRV по SSH</summary>


Настройте контроль доступа до HQ-SRV по SSH со всех устройств, кроме CLI.

HQ-SRV:
```
nano /etc/openssh/sshd_config
```
Выбор пользователей
```
AllowUsers student@192.168.0.1 student@192.168.0.140 student@192.168.0.129 student@10.10.201.174
```
</details>
