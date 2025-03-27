## Инструкция участника
- Задания написаны не в хронологическом порядке.
- Не требуется выполнять все задания.
- Везде устанавливать пароль P@ssw0rd, если не указано иного.
- При использовании своих учетных данных/адресов/DNS имен, это заносится в черновик который остаётся на рабочем месте.
- Если устройство недоступно для подключения - баллы не даются.
## Легенда конкурсного задания
Организация -  Холдинг "CoolCompany", занимается аудитом систем ИБ и разрабатывает продукт - "Afisha Today". Это платформа для малых предприятий и частных лиц в индустрии развлечений и досуга. Платформа за фиксированную комиссию размещает объявление на открытом ресурсе. Платформа помогает с регистрацией слушателей и информационным сопровождением мероприятий. 

Компания имеет серверную стойку в ЦОД, офис в Москве, аренду виртуальной машины в публичном облаке "Cloud Happy" а также завершила покупку  ГК "Минералы Урала"  - вся ИТ-инфраструктура горнодобывающией компании и её процессы перешли под ответственность "CoolCompany". Ранее в организации "Минералы Урала" работала другая ИТ-компания и не оставила документации после себя. Необходимо исправить текущие процессы и проверить работу.

| Наименование виртуальной машины | Версия операционной системы | Режим работы                               |
| ------------------------------- | --------------------------- | ------------------------------------------ |
| DC-K8S                          | Ubuntu 24.04                | Неграфический сервер                       |
| DC-MAILSERVER                   | Debian 12                   | Неграфический сервер                       |
| DC-STORAGE                      | Debian 12                   | Неграфический сервер                       |
| DC-RTR-2                        | Debian 12                   | Неграфический сервер                       |
| DC-RTR-1                        | Debian 12                   | Неграфический сервер                       |
| REMOTE-TERMINAL                 | Astra Linux 1.7             | Графический клиент, режим работы “Воронеж” |
| MOOGLE                          | Участнику недоступен        | Участнику недоступен                       |
| AZAZON                          | Участнику недоступен        | Участнику недоступен                       |
| REPO-MIRROR                     | Участнику недоступен        | Участнику недоступен                       |
| CLOUD-VM                        | Debian 12                   | Неграфический сервер                       |
| MSK-RTR                         | Debian 12                   | Неграфический сервер                       |
| MSK-DC                          | Astra Linux 1.7             | Графический сервер, режим работы “Воронеж” |
| MSK-ADMINPC                     | Astra Linux 1.7             | Графический клиент, режим работы “Воронеж” |
| MSK-WORKER                      | Astra Linux 1.7             | Графический клиент, режим работы “Воронеж” |
| MSK-GITLAB                      | Debian 12                   | Неграфический сервер                       |
| YEKT-RTR                        | Debian 12                   | Неграфический сервер                       |
| YEKT-BILLING                    | Astra Linux 1.7             | Графический сервер, режим работы “Воронеж” |
| YEKT-DB                         | Astra Linux 1.7             | Графический сервер, режим работы “Воронеж” |
| YEKT-WORKER                     | Astra Linux 1.7             | Графический клиент, режим работы “Воронеж” |
REPO-MIRROR это прокси-сервер, транслирующий запросы на официальные репозитории: 
- Debian 12 (mirror.yandex.ru), 
- Ubuntu 24.04 (mirror.yandex.ru), 
- Astra Linux 1.7 (dl.astralinux.ru).

В задании выход в “настоящий” интернет не предусмотрен, система эмулирует работу реального Интернета, но напрямую выход недоступен.
## **Работы в ЦОД**

### 1. На всех устройствах, указанных на топологии, создайте и настройте L3-адреса.
![[Топология L3.png]]
#### Что требуется? 
- Настроить IP адреса и маски устройств (квадраты с числами означают последнюю цифру адреса).
#### Как это сделать?
Если у устройства один ip адрес, то выполнить команду:
```
sudo ip addr add АДРЕС/МАСКА dev eth0
```
После чего перезапустить сеть:
```
sudo systemctl restart networking
```

Если адресов несколько то, выполнить команду выше меняя адрес, маску и интерфейс и перезапустить сеть. 

Если интерфейс один, то выполнить команду с добавлением VLAN-интерфейсов и перезапустить сеть, пример:
```
ip addr add 31.18.10.10/24 dev eth0.10             #VLAN 10 
ip addr add 100.200.100.10/24 dev eth0.20          #VLAN 20 
ip addr add 188.121.90.1/30 dev eth0.30            #VLAN 30
```
### 2. Все устройства (за исключением DC-STORAGE) должны быть доступны по протоколу SSH исключительно с внутренней сети по логину - cod_admin.
#### Что требуется? 
- Настроить доступ по SSH ко всем устройствам кроме DC-STORAGE.
- Аутентификация должна происходить **по SSH-ключу** (без пароля) с логином **cod_admin**.
- Доступ должен быть **только с внутренней сети**.
- SSH-ключ должен быть сохранен на сервере **DC-STORAGE** в каталоге `/ssh_keys`. Доступа к DC-STORAGE через OpenConnect с логином cod_admin и паролем At0mSk1lls.
#### Как это сделать?
Подключаемся к DC-STORAGE и проверить установлен ли пакет `openssh-server`:
```
dpkg -l | grep openssh-server
```

Если не установлен, то установить:
```
sudo apt update
sudo apt install openssh-server
```

Запустить службу:
```
sudo systemctl start ssh
```

И включить автозапуск:
```
sudo systemctl enable ssh
```

Если всё установлено, выполняем[[Создание ssh ключа | команду создания ключа]]:
```
mkdir -p /ssh_keys
ssh-keygen -t ed25519 -f /ssh_keys/cod_admin_key -N ""
```

Создаём пользователя `cod_admin` на всех устройствах ([[Создание пользователя | тык]])
```
sudo useradd -m -s /bin/bash cod_admin
sudo mkdir -p /home/cod_admin/.ssh
sudo touch /home/cod_admin/.ssh/authorized_keys
sudo chmod 700 /home/cod_admin/.ssh
sudo chmod 600 /home/cod_admin/.ssh/authorized_keys
sudo chown -R cod_admin:cod_admin /home/cod_admin/.ssh
```

Создаём скрипт для передачи всем устройствам приватного и публичного ключа:
```
for server in 192.168.1.2 192.168.1.3 10.15.10.200; do
	# Создаём .ssh директорию (если нет)
    ssh cod_admin@$server "mkdir -p /home/cod_admin/.ssh"

    # Копируем приватный ключ
    scp /ssh_keys/cod_admin_key cod_admin@$server:/home/cod_admin/.ssh/id_rsa

    # Копируем публичный ключ (рекомендуется для удобства, но не обязателен для работы)
	scp /ssh_keys/cod_admin_key.pub cod_admin@$server:/home/cod_admin/.ssh/id_rsa.pub

    # Настраиваем права
    ssh cod_admin@$server "
      chmod 700 /home/cod_admin/.ssh && \
      chmod 600 /home/cod_admin/.ssh/id_rsa && \
      chmod 644 /home/cod_admin/.ssh/id_rsa.pub && \
      chown -R cod_admin:cod_admin /home/cod_admin/.ssh
    "
done
```

На машинах открываем конфиг SSH:
```
sudo nano /etc/ssh/sshd_config
```
Добавляем:
```
# Глобальные настройки (применяются ко всем подключениям)
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers cod_admin

# Переопределение настроек для конкретных IP-адресов
Match Address 192.168.1.0/24,10.15.10.0/24
```

Перезапускаем SSH:
```
sudo systemctl restart ssh
```

Настраиваем `/etc/ssh/sshd_config` на DC-STORAGE:
```
# Сбрасываем любые предыдущие настройки Match
Match all
# Общие настройки: запретить рут, etc.
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication yes

# Разрешить доступ только из VPN
Match Address 10.8.0.0/24 User cod_admin
    PasswordAuthentication yes
    PubkeyAuthentication no
    # Здесь мы говорим: вход по паролю разрешён, по ключу — нет.
    
# Запретить доступ всем остальным
Match all
    PasswordAuthentication no
```

Изменим пароль пользователя на DC-STORAGE::
```
echo "cod_admin:At0mSk1lls" | sudo chpasswd
```
### 3. Между DC-RTR-2,DC-RTR-1,MSK-RTR,YEKT-RTR настройте удалённый защищённый туннель.
#### Что требуется? 
- Настроить защитный туннель между четырьмя точками:
	- **DC-RTR-1 ↔ MSK-RTR** (Основной канал для Москвы)
	- **DC-RTR-2 ↔ YEKT-RTR** (Основной канал для Екатеринбурга)
	- **DC-RTR-1 ↔ YEKT-RTR** (Резерв для Екатеринбурга)
	- **DC-RTR-2 ↔ MSK-RTR** (Резерв для Москвы)
- Если нет связи Московского офиса до ЦОД, то замена на Екатеринбургский офис и наоборот. 
- Должна быть бесперебойная работа сервера связи для всех филиалов.
- Основное и резервное соединения должны быть зашифрованы.
![[Топология VPN.png]]
#### Как это сделать?
Установить WireGuard на каждом из 4-х роутеров:
```
sudo apt update
sudo apt install -y wireguard

# Проверка версии
wg --version
```

Если WireGuard нет в списке пакетов, то нужно подключить репозитории с REPO-MIRROR.
Проверяем есть ли подключение к прокси серверу одной из команд:
```
ping REPO-MIRROR
curl -I http://REPO-MIRROR
wget --spider http://REPO-MIRROR
curl -I http://REPO-MIRROR/debian/dists/bookworm/Release
```
Редактируем список репозиториев:
```
sudo nano /etc/apt/sources.list
```
Добавить следующие строки для Debian:
```
deb http://REPO-MIRROR/debian bookworm main contrib non-free
deb http://REPO-MIRROR/debian bookworm-updates main contrib non-free
deb http://REPO-MIRROR/debian-security bookworm-security main contrib non-free
```
Добавить следующие строки для Astra:
```
deb http://REPO-MIRROR/astra stable main contrib non-free
```
После чего выполнить
```
sudo apt update
```

Закончив с установкой, создаём на каждом роутере публичный и приватный ключ:
```
wg genkey | tee /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key

sudo chmod 600 /etc/wireguard/private.key
```

Передаём публичные ключи и создаём конфигурации `/etc/wireguard/wg0.conf` и `/etc/wireguard/wg1.conf`
Конфигурации: 
**DC-RTR-1**
wg0
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-1>
Address = 10.100.0.3/24
ListenPort = 51820

# Основной туннель: DC-RTR-1 <-> YEKT-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_YEKT-RTR>
Endpoint = <IP_YEKT-RTR>:51820
AllowedIPs = 10.100.0.2/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
wg1
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-1>
Address = 10.200.0.3/24
ListenPort = 51821

# Резервный туннель: DC-RTR-1 <-> MSK-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_MSK-RTR>
Endpoint = <IP_MSK-RTR>:51820
AllowedIPs = 10.200.0.1/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
**DC-RTR-2**
wg0
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-2>
Address = 10.100.0.4/24
ListenPort = 51820

# Основной туннель: DC-RTR-2 <-> MSK-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_MSK-RTR>
Endpoint = <IP_MSK-RTR>:51820
AllowedIPs = 10.100.0.1/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
wg1
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-2>
Address = 10.200.0.4/24
ListenPort = 51821

# Резервный туннель: DC-RTR-2 <-> YEKT-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_YEKT-RTR>
Endpoint = <IP_YEKT-RTR>:51820
AllowedIPs = 10.200.0.2/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
**MSK-RTR**
wg0
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_MSK-RTR>
Address = 10.100.0.1/24
ListenPort = 51820

# Основной туннель: DC-RTR-2 <-> MSK-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_DC-RTR-2>
Endpoint = <IP_DC-RTR-2>:51820
AllowedIPs = 10.100.0.4/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
wg1
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_MSK-RTR>
Address = 10.200.0.1/24
ListenPort = 51821

# Резервный туннель: DC-RTR-2 <-> YEKT-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_YEKT-RTR>
Endpoint = <IP_YEKT-RTR>:51820
AllowedIPs = 10.200.0.2/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
**YEKT-RTR**
wg0
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_YEKT-RTR>
Address = 10.100.0.2/24
ListenPort = 51820

# Основной туннель: DC-RTR-1 <-> YEKT-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_DC-RTR-1>
Endpoint = <IP_DC-RTR-1>:51820
AllowedIPs = 10.100.0.3/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```
wg1
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_YEKT-RTR>
Address = 10.200.0.2/24
ListenPort = 51821

# Резервный туннель: DC-RTR-2 <-> MSK-RTR
[Peer]
PublicKey = <ПУБЛИЧНЫЙ_КЛЮЧ_MSK-RTR>
Endpoint = <IP_MSK-RTR>:51820
AllowedIPs = 10.200.0.1/32, 192.168.1.0/24 # Подключение к тунелю и ЦОД
PersistentKeepalive = 25
```

После **запускаем WireGuard** на каждом роутере:
```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

sudo systemctl enable wg-quick@wg1
sudo systemctl start wg-quick@wg1
```

Проверяем соединение:
```
wg show
```

Настройка OSPF для автоматического переключения:
```
sudo apt update
sudo apt install -y frr
# Во время установки выбрать zebra и ospfd
```

**Включим демоны**:
```
sudo nano /etc/frr/daemons
# zebra=yes
# ospfd=yes

sudo systemctl restart frr
```

**Настроить /etc/frr/frr.conf**. 
Конфигурации (объяснение - [[Конфигурация FRR |тык]]):
!!!Важно, проверить на каком eth действительно LAN.
**DC-RTR-1**
```
frr defaults traditional
hostname DC-RTR-1
log file /var/log/frr.log

router ospf
  ospf router-id 1.1.1.1
  network 10.100.0.0/24 area 0  # Резервный туннель (MSK)
  network 10.200.0.0/24 area 0  # Основной туннель (YEKT)
  network 192.168.1.0/24 area 0 # Сеть ЦОД с сервером печати
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

interface wg0
  ip ospf cost 10             # YEKT приоритетный маршрут
  ip ospf hello-interval 10   # Кол-во сек. до отправки Hello пакета
  ip ospf dead-interval 40    # Кол-во сек. до признания маршрута недоступным
  ip ospf retransmit-interval 5 # Через сколько сек. повторно отправит пакет

interface wg1
  ip ospf priority 0          # Защита от хаотичного переключения
  ip ospf cost 50             # MSK резервный маршрут
  ip ospf hello-interval 10   # Кол-во сек. до отправки Hello пакета
  ip ospf dead-interval 40    # Кол-во сек. до признания маршрута недоступным
  ip ospf retransmit-interval 5 # Через сколько сек. повторно отправит пакет
  
line vty
```
DC-RTR-2
```
frr defaults traditional
hostname DC-RTR-2
log file /var/log/frr.log

router ospf
  ospf router-id 1.1.1.2
  network 10.100.0.0/24 area 0  # Основной туннель (MSK)
  network 10.200.0.0/24 area 0  # Резервный туннель (YEKT)
  network 192.168.1.0/24 area 0 # Сеть ЦОД с сервером печати
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

interface wg0
  ip ospf cost 10            # Основной маршрут к MSK-RTR
  ip ospf hello-interval 10
  ip ospf dead-interval 40

interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

line vty
```
MSK-RTR
```
frr defaults traditional
hostname MSK-RTR
log file /var/log/frr.log

router ospf
  ospf router-id 2.2.2.2
  network 10.100.0.0/24 area 0  # Основной туннель (MSK)
  network 10.200.0.0/24 area 0  # Резервный туннель (YEKT)
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

interface wg0
  ip ospf cost 10            # Основной маршрут в ЦОД через DC-RTR-2
  ip ospf hello-interval 10
  ip ospf dead-interval 40
    
interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

line vty
```
YEKT-RTR
```
frr defaults traditional
hostname YEKT-RTR
log file /var/log/frr.log

router ospf
  ospf router-id 3.3.3.3
  network 10.100.0.0/24 area 0  # Резервный туннель (MSK)
  network 10.200.0.0/24 area 0  # Основной туннель (YEKT)
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

interface wg0
  ip ospf cost 10            # Основной маршрут в ЦОД через DC-RTR-1
  ip ospf hello-interval 10
  ip ospf dead-interval 40
  
interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

line vty
```
**Перезапускаем**:
```
sudo systemctl restart frr
```
**Проверка**:
1. **Проверка синтаксиса** после редактирования:    
```
    sudo frr-reload.py --test
```
2. **Проверка OSPF-соседей**:   
```
    sudo vtysh -c "show ip ospf neighbor"
```
    Ожидаемый вывод:
    Neighbor ID     Pri  State      Dead Time  Address      Interface
    1.1.1.1          1  Full/DR    00:00:35   10.100.0.1   wg0
    4.4.4.4          1  Full/Backup 00:00:37  10.100.0.4   wg0
    
3. **Проверка маршрутов**:
```
    sudo vtysh -c "show ip route ospf"
```
     Должны отображаться маршруты ко всем локальным сетям филиалов:
    O   10.15.10.0/24 [110/100] via 10.100.0.1, wg0
    O   192.168.1.0/24 [110/200] via 10.100.0.2, wg0
    O   172.16.1.0/24 [110/200] via 10.100.0.4, wg0

### 4. Для обеспечения сетевого взаимодействия настройте протокол динамической маршрутизации EIGRP на роутерах.
#### Что требуется? 
   - Обеспечьте работу EIGRP только на туннельных интерфейсах;
   - Обеспечьте аутентификацию через MD5 в EIGRP по ключу - C00lCompanY;
#### Как это сделать?
Установка FRR (если не установлен).
Заходим в `/etc/frr/daemons` и устанавливаем `eigrpd=yes`.
Применяем изменения
```
sudo systemctl restart frr
```

Создаём `/etc/wireguard/wg2.conf` и настраиваем конфигурацию на устройствах, меняя порт на 51822 и адрес сети:
DC-RTR-1 
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-1>
Address = 10.300.0.3/24                 # wg2 (EIGRP)
ListenPort = 51822

[Peer]
PublicKey = <КЛЮЧ_YEKT-RTR>
Endpoint = <IP_YEKT-RTR>:51822
AllowedIPs = 10.300.0.0/24
PersistentKeepalive = 25
```
DC-RTR-2
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_DC-RTR-2>
Address = 10.300.0.4/24                 # wg2 (EIGRP)
ListenPort = 51822

[Peer]
PublicKey = <КЛЮЧ_MSK-RTR>
Endpoint = <IP_MSK-RTR>:51822
AllowedIPs = 10.300.0.0/24
PersistentKeepalive = 25
```
MSK-RTR
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_MSK-RTR>
Address = 10.300.0.1/24                 # wg2 (EIGRP)
ListenPort = 51822

[Peer]
PublicKey = <КЛЮЧ_DC-RTR-2>
Endpoint = <IP_DC-RTR-2>:51822
AllowedIPs = 10.300.0.0/24
PersistentKeepalive = 25
```
YEKT-RTR
```
[Interface]
PrivateKey = <ПРИВАТНЫЙ_КЛЮЧ_YEKT-RTR>
Address = 10.300.0.2/24                  # wg2 (EIGRP)
ListenPort = 51822

[Peer]
PublicKey = <КЛЮЧ_DC-RTR-1>
Endpoint = <IP_DC-RTR-1>:51822
AllowedIPs = 10.300.0.0/24
PersistentKeepalive = 25
```

Запускаем второй интерфейс:
```
sudo systemctl enable wg-quick@wg2
sudo systemctl start wg-quick@wg2
```

Открываем конфигурацию FRR в  `/etc/frr/frr.conf` и добавляем изменения:
Конфигурации:
**DC-RTR-1**
```
frr defaults traditional
hostname DC-RTR-1
log file /var/log/frr.log

router ospf
  ospf router-id 1.1.1.1
  network 10.100.0.0/24 area 0  # Резервный туннель (MSK)
  network 10.200.0.0/24 area 0  # Основной туннель (YEKT)
  network 192.168.1.0/24 area 0 # Сеть ЦОД с сервером печати
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

router eigrp 100
  network 10.300.0.0/24
  passive-interface default
  no passive-interface wg2

key chain EIGRP-KEY
  key 1
    key-string C00lCompanY

interface wg0
  ip ospf cost 10             # YEKT приоритетный маршрут
  ip ospf hello-interval 10   # Кол-во сек. до отправки Hello пакета
  ip ospf dead-interval 40    # Кол-во сек. до признания маршрута недоступным
  ip ospf retransmit-interval 5 # Через сколько сек. повторно отправит пакет

interface wg1
  ip ospf priority 0          # Защита от хаотичного переключения
  ip ospf cost 50             # MSK резервный маршрут
  ip ospf hello-interval 10   # Кол-во сек. до отправки Hello пакета
  ip ospf dead-interval 40    # Кол-во сек. до признания маршрута недоступным
  ip ospf retransmit-interval 5 # Через сколько сек. повторно отправит пакет

interface wg2
  ip eigrp 100 authentication mode md5
  ip eigrp 100 authentication key-chain wg2 EIGRP-KEY

line vty
```
DC-RTR-2
```
frr defaults traditional
hostname DC-RTR-2
log file /var/log/frr.log

router ospf
  ospf router-id 1.1.1.2
  network 10.100.0.0/24 area 0  # Основной туннель (MSK)
  network 10.200.0.0/24 area 0  # Резервный туннель (YEKT)
  network 192.168.1.0/24 area 0 # Сеть ЦОД с сервером печати
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

router eigrp 100
  network 10.300.0.0/24
  passive-interface default
  no passive-interface wg2

key chain EIGRP-KEY
  key 1
    key-string C00lCompanY

interface wg0
  ip ospf cost 10            # Основной маршрут к MSK-RTR
  ip ospf hello-interval 10
  ip ospf dead-interval 40

interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

interface wg2
  ip eigrp 100 authentication mode md5
  ip eigrp 100 authentication key-chain wg2 EIGRP-KEY

line vty
```
MSK-RTR
```
frr defaults traditional
hostname MSK-RTR
log file /var/log/frr.log

router ospf
  ospf router-id 2.2.2.2
  network 10.100.0.0/24 area 0  # Основной туннель (MSK)
  network 10.200.0.0/24 area 0  # Резервный туннель (YEKT)
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

router eigrp 100
  network 10.300.0.0/24
  passive-interface default
  no passive-interface wg2

key chain EIGRP-KEY
  key 1
    key-string C00lCompanY

interface wg0
  ip ospf cost 10            # Основной маршрут в ЦОД через DC-RTR-2
  ip ospf hello-interval 10
  ip ospf dead-interval 40
    
interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

interface wg2
  ip eigrp 100 authentication mode md5
  ip eigrp 100 authentication key-chain wg2 EIGRP-KEY

line vty
```
YEKT-RTR
```
frr defaults traditional
hostname YEKT-RTR
log file /var/log/frr.log

router ospf
  ospf router-id 3.3.3.3
  network 10.100.0.0/24 area 0  # Резервный туннель (MSK)
  network 10.200.0.0/24 area 0  # Основной туннель (YEKT)
  passive-interface eth0        # Отключает отправку Hello пакетов в LAN

router eigrp 100
  network 10.300.0.0/24
  passive-interface default
  no passive-interface wg2

key chain EIGRP-KEY
  key 1
    key-string C00lCompanY

interface wg0
  ip ospf cost 10            # Основной маршрут в ЦОД через DC-RTR-1
  ip ospf hello-interval 10
  ip ospf dead-interval 40
  
interface wg1
  ip ospf priority 0
  ip ospf cost 50            # YEKT резервный маршрут
  ip ospf hello-interval 10
  ip ospf dead-interval 40

interface wg2
  ip eigrp 100 authentication mode md5
  ip eigrp 100 authentication key-chain wg2 EIGRP-KEY

line vty
```
**Перезапускаем**:
```
sudo systemctl restart frr
```

Убедится что включена пересылка пакетов между интерфейсами:
```
sudo sysctl -w net.ipv4.ip_forward=1
```

Убедитесь, что порт `51822` открыт на публичных интерфейсах роутеров (например, на `eth0`).
```
sudo ufw allow 51822/udp
```

Проверяем, что **EIGRP-соседи появились**:
```
vtysh -c "show ip eigrp neighbors"
```

Проверяем **маршруты EIGRP**:
```
vtysh -c "show ip route eigrp"
```

### 5. Для выхода в сеть «Интернет» используйте PAT, настроенный на приграничных роутерах соответственно.
#### Что требуется?
#### Как это сделать?
Проверить, какие интерфейсы есть на роутере (в первом задании мы настраивали адреса):
```
ip link show
```

Определить, какой интерфейс подключен к Интернету (нужно определить внешний интерфейс!):
```
ip route show
```

Включите пересылку пакетов (чтобы роутер мог передавать трафик между сетями):
```
sudo sysctl -w net.ipv4.ip_forward=1
```

Установить, если не установлен:
```
sudo apt install netfilter-persistent -y
```

**Настройте PAT** (чтобы трафик из внутренней сети выходил через внешний интерфейс):
```
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```
- -o eth1 — трафик выходит через внешний интерфейс.
- -j MASQUERADE — подменяет внутренние IP на внешний IP роутера.

**Сохраните настройки** (чтобы они не сбросились после перезагрузки):
```
sudo iptables-save > /etc/iptables/rules.v4
# Или
sudo netfilter-persistent save
```

Повторить на всех роутерах.
Проверить работу:
```
ping 8.8.8.8
```

### 6. На базе сервера CLOUD-VM1 необходимо реализовать OpenConnect.
#### Что требуется?
#### Как это сделать?
Установите OpenConnect на **CLOUD-VM1**:
```
sudo apt update 
sudo apt install openconnect
```
Создайте файл **vpn-failover.sh**. В скрипте у вас указаны «публичные» IP (например, `188.121.90.2`). Убедитесь, что это **реальные адреса** ваших DC-RTR-1/2, и что **CLOUD-VM1** может их пинговать:
```
#!/bin/bash

# Публичные IP-адреса роутеров (примерные, нужно заменить на настоящие)
PRIMARY_ROUTER="188.121.90.2"  # DC-RTR-1
SECONDARY_ROUTER="200.100.100.20"  # DC-RTR-2
VPN_USER="vpnuser"
VPN_PASSWORD="vpnpassword"
LOG_FILE="/var/log/vpn-failover.log"
TIMEOUT=60  # Время ожидания перед переключением (1 минута)

# Функция проверки доступности роутера
check_router() {
    ping -c 3 -W 2 $1 > /dev/null 2>&1
    echo $?
}

# Функция подключения к VPN
connect_vpn() {
    echo "$(date): Подключение к VPN через $1..." | tee -a $LOG_FILE
    pkill -f "openconnect"  # Отключаем старое соединение
    echo $VPN_PASSWORD | openconnect --user=$VPN_USER --passwd-on-stdin $1 >> $LOG_FILE 2>&1 &
    sleep 5  # Даём время на установку соединения
}

# Основной цикл
current_router=""
while true; do
    # Проверяем основной роутер
    if [ $(check_router $PRIMARY_ROUTER) -eq 0 ]; then
        if [ "$current_router" != "$PRIMARY_ROUTER" ]; then
            echo "$(date): Основной роутер доступен, подключаемся..." | tee -a $LOG_FILE
            connect_vpn $PRIMARY_ROUTER
            current_router=$PRIMARY_ROUTER
        fi
    else
        # Основной роутер недоступен, ждём минуту перед переключением
        echo "$(date): Основной роутер недоступен, ждём $TIMEOUT секунд..." | tee -a $LOG_FILE
        sleep $TIMEOUT
        if [ $(check_router $PRIMARY_ROUTER) -ne 0 ] && [ $(check_router $SECONDARY_ROUTER) -eq 0 ]; then
            if [ "$current_router" != "$SECONDARY_ROUTER" ]; then
                echo "$(date): Переключаемся на резервный роутер..." | tee -a $LOG_FILE
                connect_vpn $SECONDARY_ROUTER
                current_router=$SECONDARY_ROUTER
            fi
        else
            echo "$(date): Оба роутера недоступны или основной восстановился..." | tee -a $LOG_FILE
        fi
    fi
    sleep 10  # Проверяем каждые 10 секунд
done
```
Дать права на выполнение и запустить:
```
chmod +x vpn-failover.sh
./vpn-failover.sh
```
Чтобы скрипт запускался автоматически при загрузке:
```
sudo cp vpn-failover.sh /usr/local/bin/
sudo nano /etc/systemd/system/vpn-failover.service
```
Добавьте в открывшийся файл:
```
[Unit]
Description=VPN Failover Service
After=network.target

[Service]
ExecStart=/usr/local/bin/vpn-failover.sh
Restart=always

[Install]
WantedBy=multi-user.target
```
Запустите сервис:
```
sudo systemctl daemon-reload
sudo systemctl enable vpn-failover
sudo systemctl start vpn-failover
```
Проверьте запущенные VPN-соединения:
```
ps aux | grep openconnect
```
### 8. На машине DC-STORAGE реализуйте файловый сервер на базе NFS
#### Что требуется?
- **На DC-STORAGE** поднять **NFS-сервер**, который будет раздавать сетевую папку.
- **На MSK-ADMINPC и MSK-WORKER** сделать **автоматическое монтирование** (через **pam_mount**) **только** для пользователей, состоящих в определённой **группе**.
- Смонтированная папка должна появляться **на рабочем столе** пользователя.
- **Настройка прав**: даже если у пользователя есть «максимальный» доступ (RW), он **не должен удалять** чужие файлы/директории, владельцем которых не является. (Подсказка: это достигается установкой «sticky bit» на каталог.)
#### Как это сделать?
Установка NFS-сервера на **DC-STORAGE**:
```
sudo apt update
sudo apt install nfs-kernel-server
```
Убедитесь, что служба запущена:
```
systemctl status nfs-kernel-server
```
Создайте директорию:
```
sudo mkdir -p /storage/it
sudo mkdir -p /storage/office
```
**Установите нужные права и владельцев:
```
sudo chown root:IT /storage/it
sudo chown root:office /storage/office
sudo chmod 1770 /storage/it
sudo chmod 1770 /storage/office
```
Откройте `/etc/exports`:
```
sudo nano /etc/exports
```
Добавьте, например:
```
/storage/it X.X.X.X/24(rw,sync,no_subtree_check)
/storage/office X.X.X.X/24(rw,sync,no_subtree_check)
```
Замените `X.X.X.X/24` на сеть, откуда будут подключаться MSK-ADMINPC и MSK-WORKER.
> Опции:
> - `rw` — чтение/запись,
> - `sync` — записывать изменения сразу,
> - `no_subtree_check` — улучшает стабильность.

Примените настройки:
```
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```
Настройка pam_mount на MSK-ADMINPC и MSK-WORKER. На **каждом клиенте**:
```
sudo apt update
sudo apt install libpam-mount nfs-common
```
- `libpam-mount` — это PAM-модуль, который монтирует ресурсы при входе пользователя.
- `nfs-common` — клиентские утилиты для NFS.
Настройка /etc/security/pam_mount.conf.xml
```
sudo nano /etc/security/pam_mount.conf.xml
```
Добавьте примерно такой блок (внутри тега `<pam_mount>`):
```
<volume fstype="nfs" server="dc-storage" path="/storage/it" mountpoint="~/Desktop/IT_Folder" options="nosuid,nodev" sgrp="IT" />
<volume fstype="nfs" server="dc-storage" path="/storage/office" mountpoint="~/Desktop/Office_Folder" options="nosuid,nodev" sgrp="office" />
```
- **`sgrp=""`** означает «только для пользователей, которые состоят в группе».
- **`server="dc-storage"`** — имя (или IP) NFS-сервера.
- **`path="`** — какая папка расшарена на сервере.
- **`mountpoint=""`** — монтируем **на рабочий стол** пользователя).
- 
Убедитесь, что директория `~/Desktop/company_shared` существует на клиентах (по умолчанию должны создаются через PAM). Если её нет, создайте её:
```
mkdir -p ~/Desktop/IT_Folder
mkdir -p ~/Desktop/Office_Folder
```
Подключение PAM
Убедитесь, что в `/etc/pam.d/common-session` (или `common-auth` / `login` в зависимости от системы) есть строчка:
```
auth     optional pam_mount.so
session  optional pam_mount.so
```

Проверка работы.
Проверяем есть ли группы:
```
getent group IT
getent group office
```

**Создайте группу** (если ещё нет):
```
sudo groupadd IT
sudo groupadd office
```
**Добавьте пользователя** (например, `bober`) в эту группу:
```
sudo usermod -aG IT bober
```
- **Перезайдите** под этим пользователем на MSK-ADMINPC или MSK-WORKER (GUI или SSH).
- **Проверьте**, что на рабочем столе (`~/Desktop/company_shared`) появился ресурс.
- Попробуйте создать файл, затем попробуйте удалить файл **другого** пользователя. Должно не давать удалять чужой файл благодаря «sticky bit» (1770).
### 9. Реализуйте систему резервного копирования важных данных
#### Что требуется?
#### Как это сделать?
На сервере **DC-STORAGE** установим:
```
sudo apt update
sudo apt install inotify-tools rsync
```
Создадим файл `/usr/local/bin/backup_save.sh`:
```
sudo nano /usr/local/bin/backup_save.sh
```
Вставляем код:
```
#!/bin/bash

WATCH_DIR="/storage/office"
BACKUP_DIR="/crypto-folder"
LOG_FILE="/var/log/backup_save.log"
LOCK_FILE="/var/run/backup_save.lock"
TIMEOUT=60  # Максимальное время ожидания разблокировки файла (в секундах)

# ✅ 1. Проверка на повторный запуск через LOCK_FILE
exec 200>"$LOCK_FILE"
if ! flock -n 200; then
    echo "$(date): ⚠ Скрипт уже запущен! Второй экземпляр не будет выполнен." | tee -a "$LOG_FILE"
    exit 1
fi

# ✅ 2. Проверка, смонтирован ли /crypto-folder (Crypto-LVM)
if ! mountpoint -q "$BACKUP_DIR"; then
    echo "$(date): ❌ Ошибка: $BACKUP_DIR не смонтирован!" | tee -a "$LOG_FILE"
    exit 1
fi

# ✅ 3. Следим за файлами с "SAVE" в названии (в любом регистре)
inotifywait -m -r -e create,modify,moved_to --format "%w%f" "$WATCH_DIR" | while IFS= read -r FILE
do
    # ✅ 4. Проверяем, содержит ли имя файла "SAVE" (в любом регистре)
    if [[ "$(basename "$FILE")" =~ [Ss][Aa][Vv][Ee] ]]; then
        echo "$(date): 🔍 Найден файл $FILE, ожидаем завершения записи..." | tee -a "$LOG_FILE"

        elapsed=0
        while lsof "$FILE" &>/dev/null; do
            sleep 2
            (( elapsed += 2 ))
            if [[ $elapsed -ge $TIMEOUT ]]; then
                echo "$(date): ⚠ ВНИМАНИЕ! Файл $FILE заблокирован дольше $TIMEOUT секунд, пропускаем." | tee -a "$LOG_FILE"
                break
            fi
        done
        
        # ✅ 5. Проверяем, что файл существует и не является директорией
        if [[ -f "$FILE" ]]; then
            rsync -a "$FILE" "$BACKUP_DIR/"
            if [ $? -ne 0 ]; then
                echo "$(date): ❌ Ошибка копирования $FILE" | tee -a "$LOG_FILE"
            else
                echo "$(date): ✅ Файл $FILE успешно скопирован в $BACKUP_DIR" | tee -a "$LOG_FILE"
            fi
        else
            echo "$(date): ⚠ Файл $FILE был удалён до копирования или является директорией, пропускаем." | tee -a "$LOG_FILE"
        fi
    fi
done
```
Дать права на выполнение:
```
sudo chmod +x /usr/local/bin/backup_save.sh
```
Чтобы скрипт запускался автоматически, создаём сервис:
```
sudo nano /etc/systemd/system/backup_save.service
```
Вставляем код:
```
[Unit]
Description=Мониторинг файлов с "SAVE" в /storage/office и резервное копирование
After=network.target

[Service]
ExecStart=/usr/local/bin/backup_save.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
Запускаем сервис:
```
sudo systemctl daemon-reload
sudo systemctl enable backup_save
sudo systemctl start backup_save
```
Проверяем статус:
```
sudo systemctl status backup_save
```
Проверка работы.
**Создайте файл** в `/storage/office` с "SAVE" в названии, например:
```
touch /storage/office/my_SAVE_document.txt
```
Проверьте, что файл **скопировался в `/crypto-folder/`**.
```
ls -l /crypto-folder/
```
**Удалите файл из `/storage/office`** и убедитесь, что в `/crypto-folder` он остался.
### 10. Реализуйте LVM-массив на DC-STORAGE
#### Что требуется?
#### Как это сделать?
Предположим, что на сервере DC-STORAGE есть три доступных диска: /dev/sdb, /dev/sdc и /dev/sdd. Убедитесь, что эти диски не содержат важных данных, так как они будут отформатированы. Если диски ещё не подключены, подключите их через интерфейс управления сервером или виртуальной машиной.

Создание физических томов (Physical Volumes)
```
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc
sudo pvcreate /dev/sdd
```

Объединим три физических тома в одну группу томов, например, с именем crypto_vg:
```
sudo vgcreate crypto_vg /dev/sdb /dev/sdc /dev/sdd
```

Создадим логический том внутри группы crypto_vg размером 1 ГБ с именем crypto_lv:
```
sudo lvcreate -L 1G -n crypto_lv crypto_vg
```
- -L 1G задаёт размер 1 ГБ.
- -n crypto_lv задаёт имя логического тома.
- crypto_vg — имя группы томов.

Теперь зашифруем логический том с помощью cryptsetup, который использует dm-crypt:
```
sudo cryptsetup luksFormat /dev/crypto_vg/crypto_lv
```
	Система запросит пароль для шифрования. Введите и подтвердите его.

Откроем зашифрованный том, присвоив ему имя crypto_mas:
```
sudo cryptsetup luksOpen /dev/crypto_vg/crypto_lv crypto_mas
```
После этого том будет доступен как /dev/mapper/crypto_mas.

Создадим файловую систему ext4 на зашифрованном томе:
```
sudo mkfs.ext4 /dev/mapper/crypto_mas
```

Смонтируем том в каталог /crypto-folder.
```
sudo mount /dev/mapper/crypto_mas /crypto-folder
```

Настройка автоматического монтирования при загрузке.
Создадим случайный ключевой файл, например, /etc/crypto_keyfile:
```
sudo dd if=/dev/urandom of=/etc/crypto_keyfile bs=1024 count=4
sudo chmod 400 /etc/crypto_keyfile
```

Добавим этот ключевой файл в LUKS для разблокировки тома:
```
sudo cryptsetup luksAddKey /dev/crypto_vg/crypto_lv /etc/crypto_keyfile
```

Отредактируем файл /etc/crypttab, чтобы указать системе, как открывать зашифрованный том при загрузке:
```
sudo nano /etc/crypttab
```

Добавьте следующую строку:
```
crypto_mas /dev/crypto_vg/crypto_lv /etc/crypto_keyfile luks
```
- crypto_mas — имя устройства в /dev/mapper/.
- /dev/crypto_vg/crypto_lv — путь к логическому тому.
- /etc/crypto_keyfile — путь к ключевому файлу.
- luks — указывает тип шифрования.

Отредактируем файл /etc/fstab для автоматического монтирования:
```
sudo nano /etc/fstab
```
Добавьте строку:
```
/dev/mapper/crypto_mas /crypto-folder ext4 defaults 0 2
```
- /dev/mapper/crypto_mas — устройство.
- /crypto-folder — точка монтирования.
- ext4 — тип файловой системы.
- defaults — стандартные параметры монтирования.
- 0 2 — параметры для утилит dump и fsck.

Перезагрузите сервер, чтобы убедиться, что всё работает.
После перезагрузки проверьте, смонтирован ли том:
```
mount | grep /crypto-folder
```
Если всё настроено правильно, вы увидите строку, показывающую, что /dev/mapper/crypto_mas смонтирован на /crypto-folder.

## **Настройка офиса в Москве**
### 1. Убедитесь, что домен OpenLDAP на MSK-DC1 развернут корректно
Подключитесь к серверу MSK-DC1, на котором установлен OpenLDAP.

Конфигурация OpenLDAP обычно находится в файле /etc/ldap/slapd.conf или в каталоге /etc/ldap/slapd.d/.

Проверьте, какая система используется:
- Если существует каталог /etc/ldap/slapd.d/ с файлами конфигурации, используется динамическая конфигурация (cn=config).
- Если используется только /etc/ldap/slapd.conf, это статическая конфигурация.

- Для домена "company.cool" базовый DN должен быть dc=company,dc=cool.
- **Для slapd.conf:**
    - Откройте файл и найдите строку:
```
suffix "dc=company,dc=cool"
```
**Для динамической конфигурации:**
Выполните команду:
```
ldapsearch -H ldapi:/// -Y EXTERNAL -b "cn=config" "(objectClass=olcDatabaseConfig)" olcSuffix
```
- - Убедитесь, что в выводе присутствует olcSuffix: dc=company,dc=cool.

Проверка Root DN. Root DN — это административный пользователь, обычно cn=admin,dc=company,dc=cool
**Для slapd.conf:**
- Найдите строку:
```
rootdn "cn=admin,dc=company,dc=cool"
```
**Для динамической конфигурации:**
Выполните:
```
ldapsearch -H ldapi:/// -Y EXTERNAL -b "cn=config" "(objectClass=olcDatabaseConfig)" olcRootDN
```
Ожидаемый результат:
```
dn: dc=company,dc=cool
objectClass: dcObject
objectClass: organization
o: Company Cool
dc: company
```
 
 **Проверка статуса службы LDAP**
 Убедитесь, что служба slapd запущена:
```
 systemctl status slapd
```
Если служба не работает, запустите её:
```
systemctl start slapd
```

Тестирование запросов к LDAP. Выполните простой запрос для проверки доступности:
```
ldapsearch -x -b "dc=company,dc=cool" "(objectClass=*)"
```
Команда должна вернуть все записи под базовым DN.
### 2. Импортируйте пользователей в домен. Используйте для этого файл по пути - /root/users.ldif
Убедитесь, что файл /root/users.ldif существует и содержит корректные данные. Для этого:
```
ls -l /root/users.ldif
```
Просмотрите его содержимое:
```
cat /root/users.ldif
```
Пример корректного формата LDIF:
```
dn: uid=user1,ou=People,dc=company,dc=cool
objectClass: inetOrgPerson
uid: user1
cn: User One
sn: One
userPassword: {SSHA}hashedpassword

dn: uid=user2,ou=People,dc=company,dc=cool
objectClass: inetOrgPerson
uid: user2
cn: User Two
sn: Two
userPassword: {SSHA}hashedpassword
```
Убедитесь, что все записи соответствуют структуре вашего домена (например, dc=company,dc=cool).

Убедитесь, что служба OpenLDAP запущена.
```
systemctl status slapd
```

Для импорта потребуется учетная запись администратора LDAP (обычно это cn=admin,dc=company,dc=cool). Убедитесь, что у вас есть пароль администратора.

Используйте команду ldapadd для добавления пользователей из файла /root/users.ldif:
```
ldapadd -x -D "cn=admin,dc=company,dc=cool" -W -f /root/users.ldif
```
- -x: Используется простая аутентификация (без SASL).
- -D "cn=admin,dc=company,dc=cool": Указывает DN администратора.
- -W: Запрашивает ввод пароля администратора.
- -f /root/users.ldif: Указывает путь к файлу LDIF.
После ввода команды система запросит пароль администратора LDAP. Введите его.

Чтобы убедиться, что пользователи добавлены, выполните поиск:
```
ldapsearch -x -b "ou=People,dc=company,dc=cool" "(objectClass=inetOrgPerson)"
```

Если импорт не удался, проверьте следующее:
- **Формат файла LDIF**: Убедитесь, что DN и атрибуты в файле корректны.
- **Существование контейнера**: Контейнер ou=People должен существовать. Если его нет, то создайте файл ou_people.ldif со следующим содержимым:
```
dn: ou=People,dc=company,dc=cool
objectClass: organizationalUnit
ou: People
```
Импортируйте его:
```
ldapadd -x -D "cn=admin,dc=company,dc=cool" -W -f ou_people.ldif
```
**Права доступа**: Убедитесь, что у администратора есть права на добавление записей.
Повторите шаг 4: (Используйте команду ldapadd для добавления пользователей из файла /root/users.ldif).

### 3. Добавьте компьютеры MSK-ADMINPC и MSK-WORKER в домен OpenLDAP
На каждом компьютере установите пакеты для работы с LDAP. Выбор команд зависит от операционной системы:
```
sudo apt update
sudo apt install libnss-ldap libpam-ldap ldap-utils
```
После установки пакетов настройте клиентскую часть LDAP:

1. Запустите конфигурацию (если она не началась автоматически):
```
  sudo dpkg-reconfigure libnss-ldap
```
Укажите следующие параметры:
- **LDAP server URI**: ldap://<IP_MSK-DC1> (замените <IP_MSK-DC1> на реальный IP-адрес сервера MSK-DC1).
- **Search base**: dc=company,dc=cool (уточните у администратора, если база отличается).
- **LDAP version**: 3.
- **Make local root Database admin**: No.
- **Does the LDAP database require login?**: No.

Настройка PAM для аутентификации через LDAP
Настройте модуль PAM, чтобы компьютеры использовали LDAP для входа пользователей:
1. Откройте файл /etc/pam.d/common-auth:
```
sudo nano /etc/pam.d/common-auth
```
Добавьте или проверьте наличие строки:
```
auth sufficient pam_ldap.so
```
Повторите для файлов:
```
/etc/pam.d/common-account
/etc/pam.d/common-session
/etc/pam.d/common-password
```

Настройте систему так, чтобы она искала пользователей и группы в LDAP:
Откройте файл /etc/nsswitch.conf:
```
sudo nano /etc/nsswitch.conf
```
Убедитесь, что строки для passwd, group и shadow включают ldap:
```
passwd: files ldap
group: files ldap
shadow: files ldap
```
После внесения изменений перезапустите службы или компьютер:
Если установлен nscd (кэш имен)
```
sudo systemctl restart nscd
```
После перезагрузки проверьте, работает ли интеграция с LDAP:
1. Попробуйте войти на компьютер с учетной записью из OpenLDAP.
2. Выполните команду для проверки видимости пользователей LDAP:
```
getent passwd
```
### 4. На компьютере MSK-DC1 разверните Основной Центр сертификации
**Шаг 1: Установка OpenSSL.**
Для создания и управления сертификатами будем использовать OpenSSL. Если он еще не установлен на MSK-DC1, выполните следующие команды:
```
sudo apt update
sudo apt install openssl
```

**Шаг 2: Создание каталога для центра сертификации**
Создайте каталог /etc/ca для хранения всех файлов центра сертификации и установите необходимые права доступа:
```
sudo mkdir /etc/ca
sudo chmod 700 /etc/ca
```

**Шаг 3: Настройка конфигурационного файла OpenSSL**
Создайте файл /etc/ca/openssl.cnf для задания параметров сертификатов. Содержимое файла:
```
[ ca ]
default_ca = CA_default

[ CA_default ]
dir = /etc/ca
certs = $dir/certs
new_certs_dir = $dir/newcerts
database = $dir/index.txt
serial = $dir/serial
private_key = $dir/private/ca.key
certificate = $dir/certs/ca.crt
default_days = 365
default_md = sha256
policy = policy_match

[ policy_match ]
countryName = match
stateOrProvinceName = match
organizationName = optional
organizationalUnitName = optional
commonName = supplied
emailAddress = optional

[ req ]
default_bits = 2048
distinguished_name = req_distinguished_name
x509_extensions = v3_ca

[ req_distinguished_name ]
countryName = Country Name (2 letter code)
countryName_default = RU
stateOrProvinceName = State or Province Name (full name)
stateOrProvinceName_default = Moscow Oblast
localityName = Locality Name (eg, city)
localityName_default = Moscow
organizationName = Organization Name (eg, company)
organizationName_default = Cool CA
commonName = Common Name (eg, your name or your server\'s hostname)
commonName_max = 64

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
```

**Объяснение:**
- dir = /etc/ca — каталог центра сертификации.
- countryName_default = RU — страна "Россия".
- stateOrProvinceName_default = Moscow Oblast — область "Московская".
- localityName_default = Moscow — город "Москва".
- organizationName_default = Cool CA — имя центра сертификации "Cool CA".

**Шаг 4: Инициализация структуры каталогов и файлов**
Создайте подкаталоги и файлы в /etc/ca:
```
sudo mkdir -p /etc/ca/{certs,newcerts,private}
sudo touch /etc/ca/index.txt
sudo echo '01' > /etc/ca/serial
sudo chmod 600 /etc/ca/index.txt /etc/ca/serial
```

**Шаг 5: Создание корневого сертификата CA**
Создайте приватный ключ и самоподписанный корневой сертификат для центра сертификации:
```
sudo openssl req -x509 -newkey rsa:2048 -nodes -keyout /etc/ca/private/ca.key -out /etc/ca/certs/ca.crt -days 3650 -config /etc/ca/openssl.cnf
```
При выполнении команды введите:
- **Country Name**: RU (Россия)
- **State or Province Name**: Moscow Oblast (Московская область)
- **Locality Name**: Moscow (Москва)
- **Organization Name**: Cool CA
- **Common Name**: Cool CA

Сертификат будет действителен 10 лет (3650 дней).

**Шаг 6: Настройка веб-сервисов для HTTPS**
Все веб-сервисы должны работать только по HTTPS с сертификатами, выданными вашим CA. Пример настройки для Apache:
1. **Создание приватного ключа и запроса на сертификат (CSR):**
```
sudo openssl req -new -newkey rsa:2048 -nodes -keyout /etc/ssl/private/webserver.key -out /etc/ssl/certs/webserver.csr -config /etc/ca/openssl.cnf
```
Укажите доменное имя сервера в поле **Common Name**, например, webserver.company.cool.

Подписание CSR вашим CA:
```
sudo openssl ca -in /etc/ssl/certs/webserver.csr -out /etc/ssl/certs/webserver.crt -config /etc/ca/openssl.cnf
```

**Настройка Apache:** Откройте файл конфигурации, например, /etc/apache2/sites-enabled/default-ssl.conf, и добавьте:
```
SSLCertificateFile /etc/ssl/certs/webserver.crt
SSLCertificateKeyFile /etc/ssl/private/webserver.key
SSLCACertificateFile /etc/ca/certs/ca.crt

```
Перезапустите Apache:
```
sudo systemctl restart apache2
```
Повторите для каждого веб-сервера.

**Шаг 7: Настройка OpenConnect**
OpenConnect также должен использовать сертификаты от вашего CA.
1. **Создание сертификата для сервера OpenConnect:**
    - Создайте CSR и подпишите его, как в шаге 6.
    - Настройте OpenConnect для использования полученного сертификата (см. документацию OpenConnect для точных путей).
2. **Настройка клиентов OpenConnect:**
    - Клиенты должны доверять вашему CA. Для этого добавьте корневой сертификат на клиентские машины (см. шаг 8).

**Шаг 8: Добавление корневого сертификата в доверенные**
Чтобы все компьютеры в инфраструктуре доверяли сертификатам от вашего CA, добавьте файл /etc/ca/certs/ca.crt в хранилище доверенных сертификатов.

`sudo mkdir /usr/local/share/ca-certificates/extra sudo cp /etc/ca/certs/ca.crt /usr/local/share/ca-certificates/extra/cool_ca.crt sudo update-ca-certificates`

Выполните это на каждом компьютере инфраструктуры.

---

**Шаг 9: Проверка**
- Убедитесь, что веб-сервисы доступны только по HTTPS и сертификаты распознаются как доверенные.
- Проверьте работу OpenConnect с сертификатами от вашего CA.
- Удостоверьтесь, что корневой сертификат добавлен в доверенные на всех компьютерах.

### 5. Настройте ограничение на работу компьютера в нерабочее время
**Шаг 1: Редактирование файла /etc/security/time.conf**
Откройте файл конфигурации на компьютере MSK-WORKER с помощью команды:
```
sudo nano /etc/security/time.conf
```
Добавьте следующую строку, чтобы разрешить вход только с 09:00 до 18:00 по московскому времени (MSK):
```
login;*;*;!Al0000-0900|1800-2400
```
- login — применяется к сервису входа.
- Первый * — ко всем терминалам.
- Второй * — ко всем пользователям.
- !Al0000-0900|1800-2400 — запрещает вход с 00:00 до 09:00 и с 18:00 до 24:00.
**Объяснение:** Символ ! инвертирует условие, то есть вход запрещен в указанные промежутки времени.
Сохраните файл и выйдите

**Шаг 2: Активация модуля pam_time**
Откройте файл /etc/pam.d/login:
```
sudo nano /etc/pam.d/login
```
Добавьте в начало файла следующую строку:
```
account required pam_time.so
```
- Это активирует проверку временных ограничений при попытке входа.
Сохраните изменения и закройте файл.

**Шаг 3: Проверка часового пояса**
Убедитесь, что на компьютере установлен правильный часовой пояс (MSK):
```
timedatectl
```
Если часовой пояс неверный, установите его:
```
sudo timedatectl set-timezone Europe/Moscow
```

**Шаг4: Информирование пользователя**
Редактирование файла /etc/gdm3/greeter.dconf-defaults
```
sudo nano /etc/gdm3/greeter.dconf-defaults
```
Найдите или создайте секцию [org/gnome/login-screen] и добавьте следующие строки:
```
[org/gnome/login-screen]
banner-message-enable=true
banner-message-text='В случае необходимости доступа к рабочему месту вне регламентированных работ, напишите на почту - admin@company.cool'
```
Сохраните изменения и закройте файл.

**Шаг5: Применение изменений**
Перезапустите дисплейный менеджер GDM:
```
sudo dpkg-reconfigure gdm3
```
Или перезагрузите компьютер.
### 6. MSK-ADMINPC позволяет входить в систему только пользователям из группы IT. Остальным вход запрещен
Проверьте, существует ли группа IT, с помощью команды:
```
getent group IT
```
Если группа отсутствует, создайте её:
```
sudo groupadd IT
```
Добавьте пользователей, которым нужен доступ, в группу IT:
```
sudo usermod -aG IT username
```
Замените username на имя конкретного пользователя. Повторите команду для каждого пользователя, которому требуется доступ.

Настройте систему аутентификации, чтобы разрешить вход только членам группы IT:
```
sudo nano /etc/pam.d/common-auth
```

Добавьте в конец файла следующую строку:
```
auth required pam_succeed_if.so user ingroup IT
```
Эта строка указывает, что аутентификация успешна только для пользователей из группы IT.
Сохраните изменения и закройте файл.

Добавьте ту же строку в файл /etc/pam.d/sshd, чтобы ограничить удалённый вход.
```
auth required pam_succeed_if.so user ingroup IT
```

Предоставление прав sudo группе IT
Откройте файл /etc/sudoers для редактирования с помощью команды visudo (это обеспечивает безопасное изменение):
```
sudo visudo
```
Добавьте в конец файла следующую строку
```
%IT ALL=(ALL) ALL
```
- %IT — указывает на группу IT.
- ALL=(ALL) ALL — разрешает выполнение любых команд от имени любого пользователя на всех хостах.
Сохраните изменения и закройте файл.

Проверка прав sudo
- Войдите под учетной записью из группы IT.
- Выполните команду, требующую прав суперпользователя, например:
```
sudo apt update
```

## **Настройка офиса в Екатеринбурге**
### 1. На сервере YEKT-BILLING развернута самописная система контроля за добычей горных пород, система работает с базой данных на сервере YEKT-DB, необходимо исправить особенности платформы.
**1️⃣ Проверяем загрузку системы**  
Открываем терминал и выполняем команды:
- **Процессор** (если загрузка 100%, значит сервер перегружен):
```
top
```
    или
```
htop
```
    
- **Оперативная память** (если памяти мало, сервер может тормозить):
```
free -h
```

- **Диски (проверяем скорость чтения/записи)**
```
iostat -xz 1
```

- **Сеть (если есть задержки, значит проблема в соединении)**
```
iftop
```

**2️⃣ Перезагружаем сервер (если долго не перезагружался)**
```
sudo reboot
```

**3️⃣ Проверяем работу базы данных**  
Скорее всего, медленная работа связана с базой. Открываем базу:
```
sudo -u postgres psql
```

и смотрим **текущие запросы**:
```
SELECT * FROM pg_stat_activity;
```
Если есть длинные запросы, значит база перегружена.

**4️⃣ Увеличиваем кеш базы (если PostgreSQL)**  
Открываем конфигурацию:
```
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Ищем строку `shared_buffers`, увеличиваем (например, до `2GB`), сохраняем (CTRL+X → Y → ENTER):
```
shared_buffers = 2GB
```

Перезапускаем базу:
```
sudo systemctl restart postgresql
```

Часть 2. Очистка базы данных от старых данных
**1️⃣ Подключаемся к базе данных**
```
sudo -u postgres psql
```

**2️⃣ Проверяем, какие данные старше 15 лет**
```
SELECT * FROM mining_data WHERE extraction_date < NOW() - INTERVAL '15 years';
```

**3️⃣ Экспортируем эти данные в файл (чтобы не потерять)**
```
COPY (SELECT * FROM mining_data WHERE extraction_date < NOW() - INTERVAL '15 years')
TO '/tmp/mining_backup.csv' DELIMITER ',' CSV HEADER;
```

Теперь в файле `/tmp/mining_backup.csv` есть данные.

**4️⃣ Копируем файл на сервер DC-STORAGE**
Выходим из базы (`\q`) и выполняем в терминале:
```
scp /tmp/mining_backup.csv user@DC-STORAGE:/crypto-folder/
```

Замените `user` на вашего пользователя.

**5️⃣ Удаляем старые данные из базы**
Снова заходим в базу (`sudo -u postgres psql`) и выполняем:
```
DELETE FROM mining_data WHERE extraction_date < NOW() - INTERVAL '15 years';
```

**6️⃣ Оптимизируем базу после удаления данных**
```
VACUUM FULL;
```
Это очистит "мусорные" данные и ускорит работу.

**Часть 3. Автоматизация процесса (чтобы не делать вручную каждый раз)**

Чтобы всё это выполнялось **автоматически раз в неделю**, создаём скрипт.

**1️⃣ Открываем редактор:**
```
nano /usr/local/bin/db_cleanup.sh
```

**2️⃣ Вставляем скрипт (замените `postgres` на имя пользователя БД, если оно другое):**
```
#!/bin/bash
set -euo pipefail

# Конфигурация
DB_USER="postgres"
DB_NAME="mining_db"
BACKUP_DIR="/crypto-folder"
DATE=$(date +%Y-%m-%d)
EXPORT_FILE="/tmp/mining_backup_${DATE}.csv"
LOG_FILE="/var/log/db_cleanup.log"

# Функция логирования
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

# Проверка монтирования каталога для бэкапа
if ! mountpoint -q "$BACKUP_DIR"; then
    log "Ошибка: $BACKUP_DIR не смонтирован!"
    exit 1
fi

# Проверка свободного места на диске
REQUIRED_SPACE_MB=100  # пример: требуется минимум 100MB свободного места
AVAILABLE_SPACE_MB=$(df --output=avail /tmp | tail -1)
if [ "$AVAILABLE_SPACE_MB" -lt "$REQUIRED_SPACE_MB" ]; then
    log "Ошибка: недостаточно свободного места для экспорта данных."
    exit 1
fi

log "Начало процесса экспорта старых данных."

# Экспорт данных старше 15 лет
if ! psql -U "$DB_USER" -d "$DB_NAME" -c "\COPY (SELECT * FROM mining_data WHERE extraction_date < NOW() - INTERVAL '15 years') TO '$EXPORT_FILE' DELIMITER ',' CSV HEADER"; then
    log "Ошибка при экспорте данных."
    exit 1
fi

log "Экспорт завершён, файл: $EXPORT_FILE."

# Передача файла на сервер DC-STORAGE с использованием rsync
if ! rsync -av --progress "$EXPORT_FILE" "DC-STORAGE:$BACKUP_DIR/"; then
    log "Ошибка при передаче файла на сервер DC-STORAGE."
    exit 1
fi

log "Файл успешно передан на DC-STORAGE."

# Удаление старых данных из базы
if ! psql -U "$DB_USER" -d "$DB_NAME" -c "DELETE FROM mining_data WHERE extraction_date < NOW() - INTERVAL '15 years';"; then
    log "Ошибка при удалении старых данных из базы."
    exit 1
fi

log "Удаление старых данных завершено."

# Оптимизация базы данных
if ! psql -U "$DB_USER" -d "$DB_NAME" -c "VACUUM FULL;"; then
    log "Ошибка при выполнении VACUUM FULL."
    exit 1
fi

log "Оптимизация базы завершена."

# Удаление временного файла
rm -f "$EXPORT_FILE"
log "Временный файл удалён. Процесс завершён успешно."

```

**3️⃣ Сохраняем и выходим (CTRL+X → Y → ENTER)**

**4️⃣ Делаем скрипт исполняемым:**
```
chmod +x /usr/local/bin/db_cleanup.sh
```

**5️⃣ Добавляем в cron (автоматический запуск каждую неделю)**
```
crontab -e
```

Добавляем строку:
```
0 2 * * 1 /usr/local/bin/db_cleanup.sh >> /var/log/db_cleanup.log 2>&1
```
Это запустит скрипт **каждый понедельник в 2:00 ночи**.

### 2. На сервере YEKT-DB уже есть сервер Zabbix:
    - Подключите к серверу HTTPS;
    - Настройте метрики Zabbix так, что в случае перегрузки CPU больше чем на 80% и переполнения диска более чем на 90%, Zabbix автоматически писал уведомления на почту - **admin@company.cool**.
    - почтовый аккаунт admin@company.cool подключен к MSK-ADMINPC для доменного пользователя - admin_infra (импортируемый из LDIF-файла)

