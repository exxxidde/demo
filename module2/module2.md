### Содержание
1. **[Настройка доменного контроллера Samba](#task1)**
2. **[Конфигурация файлового хранилища](#task2)**
3. **[Настройка службы сетевого времени](#task3)**
4. **[Конфигурация Ansible](#task4)**
5. **[Развертывание приложений в Docker](#task5)**
6. **[Статическая трансляция портов](#task6)**
7. **[Запуск сервиса Moodle](#task7)**
8. **[Nginx как обратный прокси-сервер](#task8)**
9. **[Установка Яндекс Браузера](#task9)**

---

<a name="task1"></a>
## 1. Настройка доменного контроллера Samba на машине BR-SRV
* Создайте **5 пользователей** для офиса HQ: имена пользователей формата `user№.hq`. Создайте группу `hq`, введите в эту группу созданных пользователей.
* Введите в домен машину **HQ-CLI**.
* Пользователи группы `hq` имеют право аутентифицироваться на клиентском ПК.
* Пользователи группы `hq` должны иметь возможность повышать привилегии для выполнения ограниченного набора команд: `cat`, `grep`, `id`. Запускать другие команды с повышенными привилегиями пользователи группы не имеют права.
* Выполните импорт пользователей из файла `users.csv`. Файл будет располагаться на виртуальной машине BR-SRV в папке `/opt`.

<details>
<summary>Решение</summary>
<br>

## BR-SRV

Подготовка и установка

```bash
apt-get install -y samba-dc samba-client
```
```bash
rm -f /etc/samba/smb.conf
```

Инициализация домена

```bash
samba-tool domain provision --use-rfc2307 --realm=AU-TEAM.IRPO --domain=AU-TEAM --server-role=dc --dns-backend=SAMBA_INTERNAL --adminpass='P@ssw0rd123'
```
ИЛИ
```bash
# Инициализация домена (интерактивно)
samba-tool domain provision --use-rfc2307 --interactive
# Realm: AU-TEAM.IRPO
# Domain: AU-TEAM
# Role: DC
```

```bash
systemctl enable --now samba.service
```

Создание группы и 5 пользователей

```bash
# Создаем группу hq
samba-tool group add hq
```

```bash
# Создаем 5 пользователей user1.hq - user5.hq с паролем P@ssw0rd
for i in {1..5}; do
  samba-tool user create user$i.hq 'P@ssw0rd'
  samba-tool group addmembers hq user$i.hq
done
```

Импорт пользователей из users.csv

```bash
while IFS=, read -r name pass; do
  samba-tool user create "$name" "$pass"
  samba-tool group addmembers hq "$name"
done < /opt/users.csv
```
Или
```bash
samba-tool user add --csv-file=/opt/users.csv
```

#

## HQ-CLI

Установка необходимых пакетов

```bash
apt-get install realmd sssd adcli
```

Редактирование `/etc/resolv.conf`

```bash
nameserver 192.168.7.2
search au-team.irpo
```

Вход в домен 

```bash
realm join --user=Administrator au-team.irpo # отсутсвие ошибок = успешный вход
# команда для проверки: realm list
```

Настройка привилегий (Sudo)

Отредактируйте файл `/etc/sudoers.d/hq_users`
```bash
%hq ALL=(ALL) NOPASSWD: /usr/bin/cat, /usr/bin/grep, /usr/bin/id
```
```bash
chmod 440 /etc/sudoers.d/hq_users
```
```bash
chmod 4755 /usr/bin/sudo
```

Настройка автоматическог создание домашней директории (необходимо добивать запись в файл `/etc/pam.d/system-auth`)

```bash
session optional pam_mkhomedir.so skel=/etc/skel umask=0022 
```

Вход в пользователя и проверка

```bash
su - user1.hq@au-team.irpo
```
```bash
sudo id
```

</details>

<a name="task2"></a>
## 2. Сконфигурируйте файловое хранилище
* При помощи трёх дополнительных дисков, размером 1Гб каждый, на **HQ-SRV** сконфигурируйте дисковый массив уровня **5**.
* Имя устройства – `md0`, конфигурация массива размещается в файле `/etc/mdadm.conf`.
* Обеспечьте автоматическое монтирование в папку `/raid5`.
* Создайте раздел, отформатируйте раздел, в качестве файловой системы используйте **ext4**.
* Настройте сервер сетевой файловой системы (**NFS**), в качестве папки общего доступа выберите `/raid5/nfs`, доступ для чтения и записи для всей сети в сторону HQ-CLI.
* На **HQ-CLI** настройте автомонтирование в папку `/mnt/nfs`. 
* Основные параметры сервера отметьте в отчёте.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task3"></a>
## 3. Настройте службу сетевого времени на базе сервиса chrony
* В качестве сервера выступает **HQ-RTR**.
* На HQ-RTR настройте сервер chrony, выберите стратум 5.
* В качестве клиентов настройте HQ-SRV, HQ-CLI, BR-RTR, BR-SRV.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task4"></a>
## 4. Сконфигурируйте ansible на сервере BR-SRV
* Сформируйте файл инвентаря, в инвентарь должны входить **HQ-SRV, HQ-CLI, HQ-RTR и BR-RTR**.
* Рабочий каталог ansible должен располагаться в `/etc/ansible`.
* Все указанные машины должны без предупреждений и ошибок отвечать `pong` на команду `ping` в ansible, посланную с BR-SRV.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task5"></a>
## 5. Развертывание приложений в Docker на сервере BR-SRV
* Создайте в домашней директории пользователя файл `wiki.yml` для приложения MediaWiki.
* Средствами **docker compose** должен создаваться стек контейнеров с приложением MediaWiki и базой данных.
* Используйте два сервиса:
    * Основной контейнер MediaWiki должен называться `wiki` и использовать образ `mediawiki`.
    * Контейнер с базой данных должен называться `mariadb` и использовать образ `mariadb`.
* Файл `LocalSettings.php` с корректными настройками должен находиться в домашней папке пользователя и автоматически монтироваться в образ.
* Настройки БД: база с названием `mediawiki`, пользователь `wiki` с паролем `WikiP@ssw0rd`.
* MediaWiki должна быть доступна извне через порт **8080**.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task6"></a>
## 6. На маршрутизаторах сконфигурируйте статическую трансляцию портов
* Пробросьте порт `80` в порт `8080` на **BR-SRV** на маршрутизаторе **BR-RTR** (сервис wiki).
* Пробросьте порт `80` в порт `80` на **HQ-SRV** на маршрутизаторе **HQ-RTR** (сервис moodle).
* Пробросьте порт `2024` в порт `2024` на **HQ-SRV** на маршрутизаторе **HQ-RTR**.
* Пробросьте порт `2024` в порт `2024` на **BR-SRV** на маршрутизаторе **BR-RTR**.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task7"></a>
## 7. Запустите сервис moodle на сервере HQ-SRV
* Используйте веб-сервер **Apache**.
* В качестве системы управления базами данных используйте **MariaDB** (база `moodledb`).
* Создайте пользователя `moodle` с паролем `P@ssw0rd` и предоставьте ему права к базе.
* У пользователя `admin` в системе обучения задайте пароль `P@ssw0rd`.
* На главной странице должен отражаться номер рабочего места в виде арабской цифры, других подписей делать не надо.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task8"></a>
## 8. Настройте веб-сервер nginx как обратный прокси-сервер на ISP
* При обращении по доменному имени `moodle.au-team.irpo` должен открываться сервис **moodle**.
* При обращении по доменному имени `wiki.au-team.irpo` должен открываться сервис **mediawiki**.

<details>
<summary>Решение</summary>
<br>

```bash

```

</details>

<a name="task9"></a>
## 9. Удобным способом установите приложение Яндекс Браузер для организаций на HQ-CLI
* Установку браузера отметьте в отчёте.
