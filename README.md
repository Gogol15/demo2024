# Demo2024
## Задание 1:
__Выполните базовую настройку всех устройств:__
- Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
- Присвоить имена в соответствии с топологией
- Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
- Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
- Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
### Топология:
![image](https://github.com/Gogol15/demo2024/assets/79337104/51b36e26-b2b1-4eb3-9f1c-9eb4fe1913c5)

### Таблица адресов сети:
| Имя устройства | Интерфейс |   IPv4/IPv6   | Маска/Префикс |      Шлюз      |
| -------------- | --------- | ------------- | ------------- | -------------- |
|                | gi1       | 192.168.0.126 | /30           |                |
| ISP            | gi2       | 192.168.0.119 | /30           |                |
|                | gi3       | 10.12.26.13   | /24           | 10.12.26.254   |
| HQ-R           | gi1       | 192.168.0.5   | /25           |                |
|                | gi2       | 192.168.0.125 | /30           | 192.168.0.126  |
| BR-R           | gi1       | 192.168.0.89  | /27           |                |
|                | gi2       | 192.168.0.118 | /30           | 192.168.0.119  |
| HQ-SRV         | gi1       | 192.168.0.6   | /25           | 192.168.0.5    |
| BR-SRV         | gi1       | 192.168.0.88  | /27           | 192.168.0.89   |

## Процесс настройки интерфейсов.
### Для просмотра существующих интерфейсов пропишем следующую команду: 
~~~
ip a
~~~
### Далее зайдём в файл конфигурации наших интерфейсов:
~~~
nano /etc/network/intefaces
~~~
### Затем нужно будет настоить IP адресацию в соответствии с нашей таблицей. Выглядеть это должно следующим способом:
~~~
auto ens192
iface ens192 inet static
address 192.168.0.126
netmask 255.255.255.0

auto ens224
iface ens224 inet static
address 192.168.0.119
netmask 255.255.255.252

auto ens256
iface ens256 inet static
address 10.12.26.13
netmask 255.255.255.224
gateway 10.12.26.254
~~~
### Сохраним конфигурацию и выйдем из настройки сочетанием клавиш:
~~~
ctrl + S
ctrl + X
~~~
### Теперь нужно перезагрузить сеть:
~~~
systemctl restart networking
~~~
### И нужно проделать те же самые действия с остальными виртуальными машинами.
## Задание 1.2
Настроить внутренюю динамическую маршрутицацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутицации из расчёта, что в дальнейшем сеть будет масштабиоваться.
### Изначально нужно установить пакет frr:
~~~
apt-get install frr
~~~
### После завершения установки проверим состояние пакета
~~~
systemctl status frr
~~~
