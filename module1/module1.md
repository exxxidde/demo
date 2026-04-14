<p align="center">
  <img src="https://github.com/exxxidde/demo/blob/main/%D0%BC%D0%B0%D1%81%D0%BA%D0%B0%20%D0%BF%D0%BE%D0%B4%D1%81%D0%B5%D1%82%D0%B8.png" >
</p>

## Содержание (ссылки на задания)

- **[1. Базовая настройка устройств](#1-базовая-настройка-устройств)**
- **[2. Настройка ISP](#2-настройка-isp)**
- **[3. Создание локальных учетных записей](#3-создание-локальных-учетных-записей)**
- **[4. Настройка виртуального коммутатора на HQ-RTR](#4-настройка-виртуального-коммутатора-на-hq-rtr)**
- **[5. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV (SSH)](#5-настройка-безопасного-удаленного-доступа-на-серверах-hq-srv-и-br-srv-ssh)**
- **[6. Настройка IP-туннеля между офисами HQ и BR (GRE)](#6-настройка-ip-туннеля-между-офисами-hq-и-br-gre)**
- **[7. Обеспечение динамической маршрутизации (FRR)](#7-обеспечение-динамической-маршрутизации-frr)**
- **[8. Настройка динамической трансляции адресов (NAT)](#8-настройка-динамической-трансляции-адресов-nat)**
- **[9. Настройка DHCP](#9-настройка-dhcp)**
- **[10. Настройка DNS для офисов HQ и BR](#10-настройка-dns-для-офисов-hq-и-br)**
- **[11. Настройка часового пояса](#11-настройка-часового-пояса)**

#

<p align="center">
  <img src="https://github.com/exxxidde/demo/blob/main/%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F.png" height="600">
</p>

<br>

## 1. Базовая настройка устройств

- Настройте имена устройств согласно топологии. Используйте полное доменное имя.
- На всех устройствах необходимо сконфигурировать IPv4.
- IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918.
- Локальная сеть в сторону HQ-SRV (VLAN100) должна вмещать не более 64 адресов.
- Локальная сеть в сторону HQ-CLI (VLAN200) должна вмещать не более 16 адресов.
- Локальная сеть в сторону BR-SRV должна вмещать не более 32 адресов.
- Локальная сеть для управления (VLAN999) должна вмещать не более 8 адресов.
- Сведения об адресах занесите в таблицу.

**Таблица 1**


<table>
  <thead>
    <tr>
      <th>Имя устройства</th>
      <th>Интерфейс</th>
      <th>IPv4/IPv6</th>
      <th>Маска/Префикс</th>
      <th>Шлюз</th>
    </tr>
  </thead>
  <tbody>
    <!-- ISP -->
    <tr>
      <td rowspan="3"><b>ISP</b></td>
      <td>ens33</td>
      <td>DHCP</td>
      <td>—</td>
      <td>—</td>
    </tr>
    <tr>
      <td>ens37 (ISP-HQ)</td>
      <td>172.16.4.1</td>
      <td>/28</td>
      <td></td>
    </tr>
    <tr>
      <td>ens38 (ISP-BR)</td>
      <td>172.16.5.1</td>
      <td>/28</td>
      <td></td>
    </tr>
    <!-- HQ-RTR -->
    <tr>
      <td rowspan="4"><b>HQ-RTR</b></td>
      <td>ens33 (ISP-HQ)</td>
      <td>172.16.4.2</td>
      <td>/28</td>
      <td>172.16.4.1</td>
    </tr>
    <tr>
      <td>ens37 (HQ-SRV)</td>
      <td>192.168.6.1 (VLAN100)</td>
      <td>/26</td>
      <td></td>
    </tr>
    <tr>
      <td>ens38 (HQ-CLI)</td>
      <td>192.168.5.1 (VLAN200)</td>
      <td>/28</td>
      <td></td>
    </tr>
    <tr>
      <td>gre</td>
      <td>10.0.1.1</td>
      <td>/30</td>
      <td></td>
    </tr>
    <!-- BR-RTR -->
    <tr>
      <td rowspan="3"><b>BR-RTR</b></td>
      <td>ens33 (ISP-BR)</td>
      <td>172.16.5.2</td>
      <td>/28</td>
      <td>172.16.5.1</td>
    </tr>
    <tr>
      <td>ens37 (BR-SRV)</td>
      <td>192.168.7.1</td>
      <td>/27</td>
      <td></td>
    </tr>
    <tr>
      <td>gre</td>
      <td>10.0.1.2</td>
      <td>/30</td>
      <td></td>
    </tr>
    <!-- HQ-SRV -->
    <tr>
      <td><b>HQ-SRV</b></td>
      <td>ens33 (HQ-SRV)</td>
      <td>192.168.6.2</td>
      <td>/26</td>
      <td>192.168.6.1</td>
    </tr>
    <!-- BR-SRV -->
    <tr>
      <td><b>BR-SRV</b></td>
      <td>ens33 (BR-SRV)</td>
      <td>192.168.7.2</td>
      <td>/27</td>
      <td>192.168.7.1</td>
    </tr>
    <!-- HQ-CLI -->
    <tr>
      <td><b>HQ-CLI</b></td>
      <td>ens33 (HQ-CLI)</td>
      <td>192.168.5.3</td>
      <td>/28</td>
      <td>192.168.5.1</td>
    </tr>
  </tbody>
</table>


<details>
<summary>Решение</summary>
<br>

**Настройка имен устройств на ALT Linux**

```bash
hostnamectl set-hostname <FQDN>; exec bash
```
FQDN (Fully Qualified Domain Name) — полное доменное имя ([Таблица 2](#table2))

`exec bash` — обновление оболочки

#

**Настройка сетевых интерфейсов**

Создаем папку для интерфейса
```bash
mkdir /etc/net/ifaces/*имя интерфейса*
```
Настраиваем файл `options` (вариант для статичного ip-адреса)
```bash
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
```
Настраиваем файл `ipv4address` 
```bash
172.16.4.2/28
```

Настраиваем файл `ipv4route` 
```bash
default via 172.16.4.1/28
```

#

**Настройка DNS сервера**

Добавляем запись в `/etc/resolv.conf`
```bash
nameserver 8.8.8.8
```

</details>

<br>


## 2. Настройка ISP

- Настройте адресацию на интерфейсах:
  - Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP.
  - Настройте маршруты по умолчанию там, где это необходимо.
  - Интерфейс, к которому подключен HQ-RTR, подключен к сети `172.16.4.0/28`.
  - Интерфейс, к которому подключен BR-RTR, подключен к сети `172.16.5.0/28`.
- На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR для доступа к сети Интернет.

<details>
<summary>Решение</summary>
<br>
  
Включение маршрутизации **(обязательно сделать на всех роутерах)**

В файле `/etc/net/sysctl.conf` изменяем строку:

```bash
net.ipv4.ip_forward = 1
```

Изменения в файле `sysctl.conf` применяем следующей командой:

```bash
sysctl -p /etc/net/sysctl.conf
```

#

Настройка NAT экрана

Включение NAT

```bash
iptables -P FORWARD ACCEPT # Разрешить весь форвардинг (без ограничений по интерфейсам)
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE #Включить маскарадинг для всего исходящего трафика через ens33
```

Сохранение правил 

```bash
iptables-save > /etc/sysconfig/iptables
```

Включение iptables (по умолчанию отключен)

```bash
systemctl enable --now iptables
```

</details>

<br>

## 3. Создание локальных учетных записей

- Создайте пользователя `sshuser` на серверах HQ-SRV и BR-SRV:
  - Пароль пользователя `sshuser` — `P@ssw0rd`.
  - Идентификатор пользователя `1010`.
  - Пользователь `sshuser` должен иметь возможность запускать `sudo` без дополнительной аутентификации.
- Создайте пользователя `net_admin` на маршрутизаторах HQ-RTR и BR-RTR:
  - Пароль пользователя `net_admin` — `P@$$word`.
  - При настройке на EcoRouter пользователь `net_admin` должен обладать максимальными привилегиями.
  - При настройке ОС на базе Linux — запускать `sudo` без дополнительной аутентификации.

<details>
<summary>Решение</summary>
<br>
  
**Создание пользователя `sshuser` с идентификатором `1010` на серверах**

```bash
useradd sshuser -u 1010
```
Задаем пароль:

```bash
passwd sshuser
```

Добавляем в группу wheel:

```bash
usermod -aG wheel sshuser
```

Добавляем строку в `/etc/sudoers`

```bash
sshuser ALL=(ALL) NOPASSWD:ALL
```

#

**Создание пользователя `net_admin` на маршрутизаторах**

```bash
useradd net_admin
```
Задаем пароль:

```bash
passwd net_admin
```

Добавляем в группу wheel:

```bash
usermod -aG wheel net_admin
```

Добавляем строку в `/etc/sudoers`

```bash
net_admin ALL=(ALL) NOPASSWD:ALL
```

</details>

<br>

## 4. Настройка виртуального коммутатора на HQ-RTR

На интерфейсе HQ-RTR в сторону офиса HQ настройте виртуальный коммутатор:

- Сервер HQ-SRV должен находиться в **VLAN 100**.
- Клиент HQ-CLI — в **VLAN 200**.
- Создайте подсеть управления с **VLAN 999**.
- Основные сведения о настройке коммутатора и выборе реализации разделения на VLAN занесите в отчёт.

<details>
<summary>Решение</summary>
<br>

</details>

<br>

## 5. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV (SSH)

- Для подключения используйте порт `2024`.
- Разрешите подключения только пользователю `sshuser`.
- Ограничьте количество попыток входа до двух.
- Настройте баннер **"Authorized access only"**.

<details>
<summary>Решение</summary>
<br>

Настраиваем `/etc/openssh/sshd_config`

```bash
Port 2024
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/openssh/bannermotd
AllowUsers  sshuser
```
**`В параметре AllowUsers вместо пробела используется Tab`**


Создаем файл `/etc/openssh/bannermotd`

```bash
----------------------
Authorized access only
----------------------
```

Перезагружаем службу:

```bash
systemctl restart sshd
```

</details>

<br>

## 6. Настройка IP-туннеля между офисами HQ и BR (GRE)

- Сведения о туннеле занесите в отчёт.
- На выбор технологии: **GRE** или **IP in IP**.

<details>
<summary>Решение</summary>
<br>

**Создание интерфейса на HQ-RTR**

```bash
mkdir /etc/net/ifaces/gre1
```

Настраиваем файл `options`
```bash
TYPE=iptun
TUNTYPE=gre         
TUNLOCAL=172.16.4.2
TUNREMOTE=172.16.5.2
TUNOPTIONS='ttl 64'
DISABLED=no
```
Настраиваем файл `ipv4address` 
```bash
10.0.1.1/30 peer 10.0.1.2
```

Включение интерфейса

```bash
ifup gre1
```

#

**Создание интерфейса на BR-RTR**

```bash
mkdir /etc/net/ifaces/gre1
```

Настраиваем файл `options`
```bash
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.5.2
TUNREMOTE=172.16.4.2
TUNOPTIONS='ttl 64'
DISABLED=no
```
Настраиваем файл `ipv4address` 

```bash
10.0.1.2/30 peer 10.0.1.1
```

Включение интерфейса

```bash
ifup gre1
```

</details>

<br>

## 7. Обеспечение динамической маршрутизации (FRR)

Ресурсы одного офиса должны быть доступны из другого офиса. Для динамической маршрутизации используйте **link state** протокол на ваше усмотрение.

- Разрешите выбранный протокол только на интерфейсах в IP-туннеле.
- Маршрутизаторы должны делиться маршрутами только друг с другом.
- Обеспечьте защиту выбранного протокола посредством парольной защиты.
- Сведения о настройке и защите протокола занесите в отчёт.

<details>
<summary>Решение</summary>
<br>

Установка frr
```bash
apt-get install frr -y
```

Включение `ospf` (изменяем файл `/etc/frr/daemons`)

```bash
ospfd=yes
```
```bash
systemctl enable --now frr
```

## HQ-RTR

Настройка конфигурации `ospf`

```bash
vtysh
```

```bash
conf ter
router ospf
router-id 1.1.1.1
network 192.168.5.0/28 area 0
network 192.168.6.0/26 area 0
network 10.0.1.0/30 area 0
exit
interface gre1
ip ospf network point-to-point
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
end
write memory
```

## BR-RTR

Настройка конфигурации `ospf`

```bash
vtysh
```

```bash
conf ter
router ospf
router-id 2.2.2.2
network 192.168.7.0/27 area 0
network 10.0.1.0/30 area 0
exit
interface gre1
ip ospf network point-to-point
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
end
write memory
```

#

**Проверка**
```bash
show ip ospf neighbor # должен показать id соседнего роутера
show ip route ospf # должен показать ip адреса подсетей соседнего роутера
```
</details>

<br>

## 8. Настройка динамической трансляции адресов (NAT)

- Настройте динамическую трансляцию адресов для обоих офисов.
- Все устройства в офисах должны иметь доступ к сети Интернет.

<details>
<summary>Решение</summary>
<br>

Включение маршрутизации

В файле `/etc/net/sysctl.conf` изменяем строку:

```bash
net.ipv4.ip_forward = 1
```

Изменения в файле `sysctl.conf` применяем следующей командой:

```bash
sysctl -p /etc/net/sysctl.conf
```

#

Настройка NAT экрана

Включение NAT

```bash
iptables -P FORWARD ACCEPT # Разрешить весь форвардинг (без ограничений по интерфейсам)
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE #Включить маскарадинг для всего исходящего трафика через ens33
```

Сохранение правил 

```bash
iptables-save > /etc/sysconfig/iptables
```

Включение iptables (по умолчанию отключен)

```bash
systemctl enable --now iptables
```

</details>

<br>

## 9. Настройка DHCP

- Настройте нужную подсеть.
- Для офиса HQ в качестве сервера DHCP выступает маршрутизатор **HQ-RTR**.
- Клиентом является машина **HQ-CLI**.
- Исключите из выдачи адрес маршрутизатора.
- Адрес шлюза по умолчанию — адрес маршрутизатора HQ-RTR.
- Адрес DNS-сервера для HQ-CLI — адрес сервера HQ-SRV.
- DNS-суффикс для офиса HQ — `au-team.irpo`.
- Сведения о настройке протокола занесите в отчёт.

<details>
<summary>Решение</summary>
<br>

</details>

<br>

## 10. Настройка DNS для офисов HQ и BR

- Основной DNS-сервер реализован на **HQ-SRV**.
- Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с **Таблицей 2**.
- В качестве DNS-сервера пересылки используйте любой общедоступный DNS-сервер.

<details>
<summary>Решение</summary>
<br>

</details>

<br>

## 11. Настройка часового пояса

Настройте часовой пояс на всех устройствах согласно месту проведения экзамена.

<details>
<summary>Решение</summary>
<br>

</details>

#

<a id="table2"></a>
**Таблица 2**

| Устройство                                     | Запись                          | Тип      |
|------------------------------------------------|---------------------------------|----------|
| HQ-RTR                                         | hq-rtr.au-team.irpo            | A, PTR   |
| BR-RTR                                         | br-rtr.au-team.irpo            | A        |
| HQ-SRV                                         | hq-srv.au-team.irpo             | A, PTR   |
| HQ-CLI                                         | hq-cli.au-team.irpo             | A, PTR   |
| BR-SRV                                         | br-srv.au-team.irpo             | A        |
| ISP (интерфейс в сторону HQ-RTR)               | moodle.au-team.irpo             | A        |
| ISP (интерфейс в сторону BR-RTR)               | wiki.au-team.irpo               | A        |
