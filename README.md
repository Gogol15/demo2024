# demo2024
# Задание:
__Выполните базовую настройку всех устройств:__
- Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
- Присвоить имена в соответствии с топологией
- Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
- Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
- Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
# Топология:
![image](https://github.com/Gogol15/demo2024/assets/79337104/51a1ae43-f86c-4716-b54a-25dd8118b08a)

# Таблица адресов сети:
| Имя устройства | Интерфейс |   IPv4/IPv6   | Маска/Префикс |      Шлюз      |
| -------------- | --------- | ------------- | ------------- | -------------- |
|                | g1        | 192.168.0.126 | /30           |                |
| ISP            | g2        | 192.168.0.119 | /30           |                |
|                | g3        | 10.12.26.     | /24           |                |
| HQ-R           | g1        | 192.168.0.5   | /25           |                |
|                | g2        | 192.168.0.125 | /30           | 192.168.0.126  |
| BR-R           | g1        | 192.168.1.89  | /27           |                |
|                | g2        | 192.168.0.118 | /30           | 192.168.0.119  |
| HQ-SRV         | g1        | 192.168.0.6   | /25           | 192.168.0.5    |
| BR-SRV         | g1        | 192.168.0.88  | /27           | 192.168.0.89   |
