# ДОКУМЕНТАЦИЯ — ТОЛЬКО КОМАНДЫ

## Таблица адресации

| Сегмент | Сеть | Устройство и интерфейс | IP-адрес | Шлюз |
| --- | --- | --- | --- | --- |
| Internet | DHCP | ISP enp7s1 | DHCP | DHCP |
| ISP-HQ | 172.16.1.0/28 | ISP enp7s3 HQ-RTR gi1/0/2 | 172.16.1.1/28 172.16.1.2/28 | - 172.16.1.1 |
| ISP-BR | 172.16.2.0/28 | ISP enp7s2 BR-RTR gi1/0/2 | 172.16.2.1/28 172.16.2.2/28 | - 172.16.2.1 |
| HQ-SRV-Net | 192.168.100.0/27 | HQ-RTR gi1/0/3 HQ-SRV enp7s1 | 192.168.100.1/27 192.168.100.2/27 | - 192.168.100.1 |
| HQ-CLI-Net | 192.168.200.0/27 | HQ-RTR gi1/0/4 HQ-CLI enp7s1 | 192.168.200.1/27 DHCP: 192.168.200.10 | - 192.168.200.1 |
| Management | 192.168.99.0/29 | Резерв в отчёте | Не назначается без отдельного интерфейса | - |
| BR-Net | 192.168.30.0/28 | BR-RTR gi1/0/3 BR-SRV enp7s1 | 192.168.30.1/28 192.168.30.2/28 | - 192.168.30.1 |
| GRE | 10.10.10.0/30 | HQ-RTR gre 1 BR-RTR gre 1 | 10.10.10.1/30 10.10.10.2/30 | - |

## Модуль 1. Задания 1-2. Имена устройств и IPv4

### 1. Настроить FQDN

#### ISP

##### ISP

```
hostnamectl hostname isp.sirius-exam.org
hostnamectl
```

#### HQ-SRV

##### HQ-SRV

```
hostnamectl hostname hq-srv.sirius-exam.org
hostnamectl
```

#### BR-SRV

##### BR-SRV

```
hostnamectl hostname br-srv.sirius-exam.org
hostnamectl
```

#### HQ-CLI

##### HQ-CLI

```
hostnamectl hostname hq-cli.sirius-exam.org
hostnamectl
```

#### HQ-RTR

##### HQ-RTR

```
configure
hostname hq-rtr.sirius-exam.org
commit
confirm
save
```

#### BR-RTR

##### BR-RTR

```
configure
hostname br-rtr.sirius-exam.org
commit
confirm
save
```

### 2. Назначить IPv4

#### ISP

##### ISP: переименовать профили и назначить адреса

```
ip -br a
nmcli con show

# Подставьте фактические имена трёх активных профилей:
nmcli con mod "Проводное подключение 1" connection.id WAN
nmcli con mod "Проводное подключение 2" connection.id ISP-BR
nmcli con mod "Проводное подключение 3" connection.id ISP-HQ

nmcli con mod WAN ipv4.method auto ipv4.never-default no
nmcli con mod ISP-BR ipv4.method manual ipv4.addresses 172.16.2.1/28 ipv4.never-default yes
nmcli con mod ISP-HQ ipv4.method manual ipv4.addresses 172.16.1.1/28 ipv4.never-default yes

nmcli con up WAN
nmcli con up ISP-BR
nmcli con up ISP-HQ
ip -br a
ip r
```

#### HQ-SRV

##### HQ-SRV

```
nmcli con show
nmcli con mod to-hq connection.id HQ-SRV
nmcli con mod HQ-SRV ipv4.method manual ipv4.addresses 192.168.100.2/27 ipv4.gateway 192.168.100.1
nmcli con up HQ-SRV
ip -br a
ip r
```

#### BR-SRV

##### BR-SRV

```
nmcli con show
nmcli con mod to-br connection.id BR-SRV
nmcli con mod BR-SRV ipv4.method manual ipv4.addresses 192.168.30.2/28 ipv4.gateway 192.168.30.1
nmcli con up BR-SRV
ip -br a
ip r
```

#### HQ-RTR

##### HQ-RTR

```
configure
interface gigabitethernet 1/0/2
  description TO-ISP
  ip firewall disable
  ip address 172.16.1.2/28
  no shutdown
exit
interface gigabitethernet 1/0/3
  description TO-HQ-SRV
  ip firewall disable
  ip address 192.168.100.1/27
  no shutdown
exit
interface gigabitethernet 1/0/4
  description TO-HQ-CLI
  ip firewall disable
  ip address 192.168.200.1/27
  no shutdown
exit
ip route 0.0.0.0/0 172.16.1.1
commit
confirm
save
show ip interfaces
```

#### BR-RTR

##### BR-RTR

```
configure
interface gigabitethernet 1/0/2
  description TO-ISP
  ip firewall disable
  ip address 172.16.2.2/28
  no shutdown
exit
interface gigabitethernet 1/0/3
  description TO-BR-SRV
  ip firewall disable
  ip address 192.168.30.1/28
  no shutdown
exit
ip route 0.0.0.0/0 172.16.2.1
commit
confirm
save
show ip interfaces
```

## Модуль 1. Задание 3. Интернет на ISP и учётные записи

### 3.1-3.2. ISP: маршруты, forwarding и PAT

#### ISP

##### ISP: постоянные маршруты к офисам

```
ip route replace 192.168.100.0/27 via 172.16.1.2
ip route replace 192.168.200.0/27 via 172.16.1.2
ip route replace 192.168.30.0/28 via 172.16.2.2

nmcli con mod ISP-HQ +ipv4.routes "192.168.100.0/27 172.16.1.2"
nmcli con mod ISP-HQ +ipv4.routes "192.168.200.0/27 172.16.1.2"
nmcli con mod ISP-BR +ipv4.routes "192.168.30.0/28 172.16.2.2"
nmcli con up ISP-HQ
nmcli con up ISP-BR
ip r
```

##### ISP: forwarding и nftables masquerade

```
dnf install -y nftables
nano /etc/sysctl.d/99-ip-forward.conf

# Вставить:
net.ipv4.ip_forward = 1

sysctl -p /etc/sysctl.d/99-ip-forward.conf
nano /etc/nftables.conf

# Вставить:
flush ruleset
table ip nat {
  chain postrouting {
    type nat hook postrouting priority srcnat;
    oifname "enp7s1" masquerade
  }
}

systemctl enable --now nftables
systemctl restart nftables
nft list ruleset
ping -c 3 8.8.8.8
```

### 3.3. sshuser на серверах

#### HQ-SRV и BR-SRV

##### Одинаково на обоих серверах

```
dnf install -y sudo
useradd -u 2026 sshuser
passwd sshuser
# Дважды ввести: P@ssw0rd

usermod -aG wheel sshuser
nano /etc/sudoers.d/sshuser

# Вставить:
sshuser ALL=(ALL) NOPASSWD: ALL

chmod 440 /etc/sudoers.d/sshuser
id sshuser
sudo -l -U sshuser
```

### 3.4-3.7. net_admin на Eltex

#### HQ-RTR и BR-RTR

##### Одинаково на обоих маршрутизаторах

```
configure
username net_admin
  password ascii-text P@ssw0rd
  privilege 15
exit
ip ssh server
commit
confirm
save
```

## Модуль 1. Задания 4-5. Маршрутизация HQ и безопасный SSH

### 4. Маршрутизация трафика HQ-RTR

#### HQ-RTR

##### Проверка connected-маршрутов

```
show ip route
# Должны быть connected:
# 192.168.100.0/27
# 192.168.200.0/27
# 172.16.1.0/28
```

### 5. SSH: порт 2026, только sshuser, 2 попытки, баннер

#### HQ-SRV и BR-SRV

##### Одинаково на обоих серверах

```
dnf install -y openssh-server
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
nano /etc/ssh/sshd_config

# Добавить в конец:
Port 2026
MaxAuthTries 2
AllowUsers sshuser
PermitRootLogin no
Banner /root/banner

nano /root/banner
# Вставить:
Authorized access only

sshd -t
systemctl enable --now sshd
systemctl restart sshd
ss -tulpn | grep 2026
```

## Модуль 1. Задание 6. GRE-туннель между офисами

#### HQ-RTR

##### HQ-RTR: GRE 1

```
configure
tunnel gre 1
  description GRE-TO-BR
  local address 172.16.1.2
  remote address 172.16.2.2
  ip address 10.10.10.1/30
  enable
exit
commit
confirm
save
show tunnels configuration gre 1
show tunnels status gre 1
```

#### BR-RTR

##### BR-RTR: GRE 1

```
configure
tunnel gre 1
  description GRE-TO-HQ
  local address 172.16.2.2
  remote address 172.16.1.2
  ip address 10.10.10.2/30
  enable
exit
commit
confirm
save
show tunnels configuration gre 1
show tunnels status gre 1
```

## Модуль 1. Задание 7. OSPF только через GRE

#### HQ-RTR

##### HQ-RTR: OSPF

```
configure
router ospf 1
  router-id 1.1.1.1
  area 0.0.0.0
    network 10.10.10.0/30
  exit
  redistribute connected
  enable
exit
tunnel gre 1
  ip ospf area 0.0.0.0
  ip ospf instance 1
  ip ospf network point-to-point
  ip ospf authentication algorithm cleartext
  ip ospf authentication key ascii-text P@ssw0rd
exit
commit
confirm
save
show ip ospf neighbors
show ip route ospf
```

#### BR-RTR

##### BR-RTR: OSPF

```
configure
router ospf 1
  router-id 2.2.2.2
  area 0.0.0.0
    network 10.10.10.0/30
  exit
  redistribute connected
  enable
exit
tunnel gre 1
  ip ospf area 0.0.0.0
  ip ospf instance 1
  ip ospf network point-to-point
  ip ospf authentication algorithm cleartext
  ip ospf authentication key ascii-text P@ssw0rd
exit
commit
confirm
save
show ip ospf neighbors
show ip route ospf
```

## Модуль 1. Задание 8. NAT офисов и DHCP для HQ-CLI

### 8.1. Динамический NAT офисов в сторону ISP

#### HQ-RTR

##### HQ-RTR: SNAT

```
configure
nat source
  ruleset HQ-SNAT
    to interface gigabitethernet 1/0/2
    rule 10
      match source-address prefix 192.168.100.0/27
      action source-nat interface
      enable
    exit
    rule 20
      match source-address prefix 192.168.200.0/27
      action source-nat interface
      enable
    exit
  exit
exit
commit
confirm
save
show ip nat source rulesets
```

#### BR-RTR

##### BR-RTR: SNAT

```
configure
nat source
  ruleset BR-SNAT
    to interface gigabitethernet 1/0/2
    rule 10
      match source-address prefix 192.168.30.0/28
      action source-nat interface
      enable
    exit
  exit
exit
commit
confirm
save
show ip nat source rulesets
```

### 8.2. DHCP для HQ-CLI

#### HQ-RTR

##### HQ-RTR: DHCP-пул

```
configure
ip dhcp-server
ip dhcp-server pool HQ-CLI
  network 192.168.200.0/27
  address-range 192.168.200.10-192.168.200.10
  default-router 192.168.200.1
  dns-server 192.168.100.2
  domain-name sirius-exam.org
exit
commit
confirm
save
show ip dhcp server pool HQ-CLI
show ip dhcp binding
```

#### HQ-CLI

##### HQ-CLI: получить адрес

```
nmcli con show
nmcli con mod "Проводное подключение 1" ipv4.method auto
nmcli con down "Проводное подключение 1"
nmcli con up "Проводное подключение 1"
ip -br a
ip r
cat /etc/resolv.conf
```

## Модуль 1. Задание 9. DNS и часовой пояс

### 9.1-9.3. DNS-сервер BIND на HQ-SRV

#### HQ-SRV

##### Установить BIND

```
dnf install -y bind bind-utils
mkdir -p /var/named/master
nano /etc/named.conf
```

##### /etc/named.conf

```
options {
    listen-on port 53 { any; };
    listen-on-v6 port 53 { none; };
    directory "/var/named";
    allow-query { any; };
    recursion yes;
    dnssec-validation no;
    forwarders { 77.88.8.7; 77.88.8.3; };
};

zone "sirius-exam.org" IN {
    type master;
    file "master/sirius-exam.org.zone";
};

zone "100.168.192.in-addr.arpa" IN {
    type master;
    file "master/192.168.100.rev";
};

zone "200.168.192.in-addr.arpa" IN {
    type master;
    file "master/192.168.200.rev";
};
```

##### Прямая зона

```
nano /var/named/master/sirius-exam.org.zone

$TTL 604800
@ IN SOA hq-srv.sirius-exam.org. root.sirius-exam.org. (
  2026010101 600 3600 604800 360 )
@       IN NS hq-srv.sirius-exam.org.
hq-rtr  IN A 192.168.100.1
br-rtr  IN A 192.168.30.1
hq-srv  IN A 192.168.100.2
hq-cli  IN A 192.168.200.10
br-srv  IN A 192.168.30.2
docker  IN A 172.16.1.1
web     IN A 172.16.2.1
```

##### PTR-зоны

```
nano /var/named/master/192.168.100.rev

$TTL 604800
@ IN SOA hq-srv.sirius-exam.org. root.sirius-exam.org. (
  2026010101 600 3600 604800 360 )
@ IN NS hq-srv.sirius-exam.org.
1 IN PTR hq-rtr.sirius-exam.org.
2 IN PTR hq-srv.sirius-exam.org.

nano /var/named/master/192.168.200.rev

$TTL 604800
@ IN SOA hq-srv.sirius-exam.org. root.sirius-exam.org. (
  2026010101 600 3600 604800 360 )
@ IN NS hq-srv.sirius-exam.org.
10 IN PTR hq-cli.sirius-exam.org.
```

##### Проверить и запустить BIND

```
chown -R root:named /var/named/master
chmod 640 /var/named/master/*
named-checkconf
named-checkzone sirius-exam.org /var/named/master/sirius-exam.org.zone
named-checkzone 100.168.192.in-addr.arpa /var/named/master/192.168.100.rev
named-checkzone 200.168.192.in-addr.arpa /var/named/master/192.168.200.rev
systemctl enable --now named
systemctl restart named

nslookup hq-srv.sirius-exam.org 127.0.0.1
nslookup 192.168.100.2 127.0.0.1
nslookup ya.ru 127.0.0.1
```

### 9.4. Часовой пояс

#### ISP, HQ-SRV, BR-SRV, HQ-CLI

##### RED OS

```
timedatectl set-timezone Europe/Moscow
timedatectl
```

#### HQ-RTR и BR-RTR

##### Eltex

```
configure
clock timezone gmt +3
commit
confirm
save
show date
```

## Модуль 2. Задание 1. Samba на BR-SRV

#### BR-SRV

##### Установить и подготовить Samba

```
hostnamectl hostname br-srv.sirius-exam.org
dnf install -y samba samba-client
groupadd hq
mkdir -p /srv/samba/hq
chown root:hq /srv/samba/hq
chmod 2770 /srv/samba/hq
```

##### Создать пользователей Linux и Samba

```
for n in 1 2 3 4 5; do
  useradd -m -G hq hquser$n
  echo "P@ssw0rd" | passwd --stdin hquser$n
  (echo "P@ssw0rd"; echo "P@ssw0rd") | smbpasswd -a -s hquser$n
done

getent group hq
pdbedit -L
```

##### /etc/samba/smb.conf

```
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak 2>/dev/null || true
nano /etc/samba/smb.conf

[global]
   workgroup = WORKGROUP
   security = user
   map to guest = never

[hq]
   path = /srv/samba/hq
   browseable = yes
   read only = no
   valid users = @hq
   force group = hq
   create mask = 0660
   directory mask = 2770
```

##### Запуск и проверка Samba

```
testparm
systemctl enable --now smb
systemctl restart smb
smbclient -L //127.0.0.1 -U hquser1
```

#### HQ-CLI

##### Проверить аутентификацию с HQ-CLI

```
dnf install -y samba-client
smbclient //192.168.30.2/hq -U hquser1

# Внутри smbclient:
ls
put /etc/hosts hosts-test.txt
ls
exit
```

## Модуль 2. Задание 2. RAID0 на HQ-SRV

#### HQ-SRV

##### Найти дополнительные диски

```
dnf install -y mdadm parted
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
# Ниже предполагаются два дополнительных диска: /dev/sdb и /dev/sdc
```

##### Создать md0 и сохранить конфигурацию

```
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat
mdadm --detail /dev/md0
mdadm --detail --scan > /etc/mdadm.conf
cat /etc/mdadm.conf
```

##### Создать раздел md0p1

```
fdisk /dev/md0

# В fdisk нажать:
n
p
1


w

partprobe /dev/md0
lsblk
```

##### ext4 и автомонтирование /raid

```
mkfs.ext4 /dev/md0p1
mkdir -p /raid
blkid /dev/md0p1

nano /etc/fstab
# Добавить строку с UUID:
UUID=ВСТАВЬТЕ_UUID /raid ext4 defaults 0 0

systemctl daemon-reload
mount -a
df -h /raid
lsblk -f
```

## Модуль 2. Задание 3. NFS

#### HQ-SRV

##### Экспортировать /raid/nfs только для HQ-CLI-Net

```
dnf install -y nfs-utils
mkdir -p /raid/nfs
chmod 777 /raid/nfs
nano /etc/exports

# Вставить:
/raid/nfs 192.168.200.0/27(rw,sync,no_subtree_check)

systemctl enable --now nfs-server
exportfs -rav
exportfs -v
```

#### HQ-CLI

##### Автомонтирование /mnt/nfs при обращении через autofs

```
dnf install -y nfs-utils autofs
mkdir -p /mnt
nano /etc/auto.master

# Добавить строку:
/mnt /etc/auto.nfs --timeout=60

nano /etc/auto.nfs
# Добавить строку:
nfs -rw,soft,intr 192.168.100.2:/raid/nfs

systemctl enable --now autofs
systemctl restart autofs
ls -la /mnt/nfs
touch /mnt/nfs/hq-cli-test.txt
ls -l /mnt/nfs
```

## Модуль 2. Задание 4. Chrony на ISP

#### ISP

##### ISP: NTP-сервер stratum 5

```
dnf install -y chrony
nano /etc/chrony.conf

# Оставить или добавить:
pool pool.ntp.org iburst
local stratum 5
allow 192.168.100.0/27
allow 192.168.200.0/27
allow 192.168.30.0/28
allow 172.16.0.0/12

systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

#### HQ-SRV и HQ-CLI

##### Клиенты HQ

```
dnf install -y chrony
nano /etc/chrony.conf
# Добавить:
server 172.16.1.1 iburst

systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

#### BR-SRV

##### Клиент BR-SRV

```
dnf install -y chrony
nano /etc/chrony.conf
# Добавить:
server 172.16.2.1 iburst

systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

#### BR-RTR

##### Клиент BR-RTR

```
configure
ntp enable
ntp server 172.16.2.1
exit
commit
confirm
save
show ntp configuration
show ntp peers
```

## Модуль 2. Задание 5. Docker testapp на BR-SRV

#### BR-SRV

##### Установить Docker и Compose

```
dnf install -y docker-ce docker-ce-cli docker-compose
systemctl enable docker --now
systemctl status docker --no-pager
docker --version
docker-compose --version
```

##### Подключить Additional.iso и загрузить образы

```
lsblk
mkdir -p /mnt/additional
mount /dev/sr0 /mnt/additional
ls -l /mnt/additional/docker
cat /mnt/additional/docker/readme.txt

docker load < /mnt/additional/docker/site_latest.tar
docker load < /mnt/additional/docker/mariadb_latest.tar
docker image ls
```

##### /root/web.yaml

```
cd /root
nano web.yaml

services:
  db:
    image: mariadb:10.11
    container_name: db
    restart: always
    environment:
      MARIADB_DATABASE: testdb
      MARIADB_USER: test
      MARIADB_PASSWORD: P@ssw0rd
      MARIADB_ROOT_PASSWORD: P@ssw0rd
    volumes:
      - db_data:/var/lib/mysql

  testapp:
    image: site:latest
    container_name: tespapp
    restart: always
    ports:
      - "192.168.30.2:8080:8000"
    environment:
      DB_HOST: db
      DB_PORT: "3306"
      DB_NAME: testdb
      DB_USER: test
      DB_PASS: P@ssw0rd
      DB_TYPE: maria
    depends_on:
      - db

volumes:
  db_data:
```

##### Запуск и проверка

```
cd /root
docker-compose -f web.yaml up -d
docker ps
docker logs tespapp --tail 50
curl http://127.0.0.1:8080
```

## Модуль 2. Задание 6. Apache и MariaDB на HQ-SRV

#### HQ-SRV

##### Установить и запустить службы

```
dnf install -y httpd php php-mysqlnd mariadb-server mariadb
systemctl enable --now mariadb
systemctl enable --now httpd
systemctl status mariadb httpd --no-pager
```

##### Подключить Additional.iso

```
lsblk
mkdir -p /mnt/additional
mount /dev/sr0 /mnt/additional
find /mnt/additional/web -maxdepth 2 -type f | sort
```

##### Создать webdb и пользователя web

```
mysql -u root

CREATE DATABASE webdb;
CREATE USER 'web'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

##### Импортировать приложение

```
mysql -u root webdb < /mnt/additional/web/dump.sql
cp /mnt/additional/web/index.php /var/www/html/
cp -r /mnt/additional/web/images /var/www/html/

nano /var/www/html/index.php
# Указать:
# host     = localhost
# database = webdb
# user     = web
# password = P@ssw0rd

systemctl restart mariadb httpd
curl http://127.0.0.1
```

## Модуль 2. Задание 7. DNAT на Eltex

### HQ-RTR: 8080 → HQ-SRV:80 и 2026 → HQ-SRV:2026

#### HQ-RTR

##### HQ-RTR

```
configure
nat destination
  pool HQ-WEB
    ip address 192.168.100.2
    ip port 80
  exit
  pool HQ-SSH
    ip address 192.168.100.2
    ip port 2026
  exit
  ruleset HQ-DNAT
    from interface gigabitethernet 1/0/2
    rule 10
      match protocol tcp
      match destination-port port-range 8080
      action destination-nat pool HQ-WEB
      enable
    exit
    rule 20
      match protocol tcp
      match destination-port port-range 2026
      action destination-nat pool HQ-SSH
      enable
    exit
  exit
exit
commit
confirm
save
show ip nat destination pools
show ip nat destination rulesets
```

### BR-RTR: 8080 → BR-SRV:8080 и 2026 → BR-SRV:2026

#### BR-RTR

##### BR-RTR

```
configure
nat destination
  pool BR-WEB
    ip address 192.168.30.2
    ip port 8080
  exit
  pool BR-SSH
    ip address 192.168.30.2
    ip port 2026
  exit
  ruleset BR-DNAT
    from interface gigabitethernet 1/0/2
    rule 10
      match protocol tcp
      match destination-port port-range 8080
      action destination-nat pool BR-WEB
      enable
    exit
    rule 20
      match protocol tcp
      match destination-port port-range 2026
      action destination-nat pool BR-SSH
      enable
    exit
  exit
exit
commit
confirm
save
show ip nat destination pools
show ip nat destination rulesets
```

#### ISP

##### Проверка с ISP

```
curl http://172.16.1.2:8080
curl http://172.16.2.2:8080
ssh -p 2026 sshuser@172.16.1.2
ssh -p 2026 sshuser@172.16.2.2
```

## Модуль 2. Задания 8-10. Nginx, Basic Auth, Яндекс Браузер

### 8. Reverse proxy на ISP

#### ISP

##### Установить Nginx

```
dnf install -y nginx httpd-tools
nano /etc/nginx/conf.d/exam.conf
```

##### /etc/nginx/conf.d/exam.conf

```
server {
    listen 80;
    server_name web.sirius-exam.org;

    location / {
        proxy_pass http://172.16.1.2:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 80;
    server_name docker.sirius-exam.org;

    location / {
        proxy_pass http://172.16.2.2:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

##### Проверить Nginx

```
nginx -t
systemctl enable --now nginx
systemctl restart nginx
curl -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1
```

### 9. Basic Auth только для web.sirius-exam.org

#### ISP

##### Создать /etc/nginx/.htpasswd

```
htpasswd -c /etc/nginx/.htpasswd WEB
# Дважды ввести: P@ssw0rd

nano /etc/nginx/conf.d/exam.conf
# Внутрь location / блока web.sirius-exam.org добавить:
auth_basic "Restricted area";
auth_basic_user_file /etc/nginx/.htpasswd;

nginx -t
systemctl restart nginx

curl -I -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -u WEB:P@ssw0rd -H 'Host: web.sirius-exam.org' http://127.0.0.1
```

### 10. Яндекс Браузер

#### HQ-CLI

##### HQ-CLI

```
sudo dnf install -y yandex-browser-stable

# Если пакет не найден:
sudo dnf install -y yandex-browser-release
sudo dnf install -y yandex-browser-stable

rpm -qa | grep -i yandex
```

## Вариативная часть. Nextcloud и права доступа

### Вариативный модуль 1. Nextcloud на BR-SRV

#### BR-SRV

##### Проверить статический IP Сервера 1

```
ip -br a
ip r
# Должно быть:
# 192.168.30.2/28, шлюз 192.168.30.1
```

##### Установить совместимый Docker CLI и Docker Compose

```
# Если Docker уже установлен для testapp, повторная установка не нужна.
command -v docker >/dev/null || sudo yum install -y podman-docker
command -v docker-compose >/dev/null || sudo yum install -y docker-compose

# Проверка установки
docker --version
docker-compose --version
```

##### Создать рабочий каталог

```
sudo mkdir -p /opt/nextcloud
cd /opt/nextcloud
sudo nano docker-compose.yml
```

##### /opt/nextcloud/docker-compose.yml

```
version: '3'
services:
  db:
    image: mariadb:10.6
    container_name: nextcloud-db
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=YourStrongRootPass123
      - MYSQL_PASSWORD=NextcloudDBpass2025
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud-app
    restart: always
    ports:
      - "127.0.0.1:8080:80"
    volumes:
      - ./nextcloud_data:/var/www/html
    environment:
      - MYSQL_HOST=db
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=NextcloudDBpass2025
      - MYSQL_DATABASE=nextcloud
    depends_on:
      - db
```

##### Запустить контейнеры Nextcloud

```
cd /opt/nextcloud
sudo docker-compose up -d
sudo docker ps
curl -I http://127.0.0.1:8080
```

##### Установить Nginx и создать самоподписанный сертификат

```
sudo yum install -y nginx openssl
sudo mkdir -p /etc/ssl/certs /etc/ssl/private

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nextcloud-selfsigned.key \
    -out /etc/ssl/certs/nextcloud-selfsigned.crt \
    -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=127.0.0.1"
```

##### Создать конфигурацию Nginx

```
sudo mkdir -p /etc/nginx/sites-available
sudo nano /etc/nginx/sites-available/nextcloud
```

##### /etc/nginx/sites-available/nextcloud

```
server {
    listen 443 ssl;
    server_name 127.0.0.1 192.168.30.2;

    ssl_certificate /etc/ssl/certs/nextcloud-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nextcloud-selfsigned.key;

    client_max_body_size 10G;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

        proxy_pass http://127.0.0.1:8080;
        proxy_redirect off;
    }
}
```

##### Создать ссылку sites-enabled и подключить каталог

```
sudo mkdir -p /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/nextcloud

sudo nano /etc/nginx/nginx.conf
# Внутри блока http { ... } добавить строку:
include /etc/nginx/sites-enabled/*;
```

##### Проверить и запустить Nginx

```
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl restart nginx
curl -kI https://127.0.0.1
```

### Вариативный модуль 2. Пользователи и каталоги на HQ-CLI

#### HQ-CLI

##### Создать группы, пользователей и каталоги

```
sudo groupadd Work
sudo groupadd Job
sudo groupadd labor

sudo useradd -m -G Work User1
sudo useradd -m -G Work User2
sudo useradd -m -G Job User3
sudo useradd -m -G Job User4
sudo useradd -m -G labor User5

sudo mkdir -p /home/Folder/work_shared
sudo mkdir -p /home/Folder/job_readonly

sudo chown root:Work /home/Folder/work_shared
sudo chmod 770 /home/Folder/work_shared

sudo chown root:Job /home/Folder/job_readonly
sudo chmod 750 /home/Folder/job_readonly

# Зафиксировать в отчёте IP Сервера 1:
echo "BR-SRV (Сервер 1): 192.168.30.2/28"

ls -ld /home/Folder/work_shared /home/Folder/job_readonly
```

### Вариативный модуль 3. Эксплуатационные проверки

#### HQ-CLI

##### Открыть страницу установки Nextcloud

```
# В браузере HQ-CLI открыть:
https://192.168.30.2

# Подтвердить переход по самоподписанному сертификату.
# Должна открыться страница установки Nextcloud.
```

##### Проверка прав

```
sudo -u User1 touch /home/Folder/work_shared/user1.txt
sudo -u User1 rm /home/Folder/work_shared/user1.txt

echo "readonly" | sudo tee /home/Folder/job_readonly/readme.txt
sudo chown root:Job /home/Folder/job_readonly/readme.txt
sudo chmod 640 /home/Folder/job_readonly/readme.txt

sudo -u User3 cat /home/Folder/job_readonly/readme.txt
sudo -u User3 touch /home/Folder/job_readonly/user3.txt
# Ожидается: Permission denied

sudo -u User5 ls /home/Folder/work_shared
# Ожидается: Permission denied
sudo -u User5 ls /home/Folder/job_readonly
# Ожидается: Permission denied
```

#### BR-SRV

##### Просмотр access.log и поиск IP Сервера 2

```
tail -f /var/log/nginx/access.log

# После запроса с HQ-CLI остановить tail сочетанием Ctrl+C и проверить:
grep '192.168.200.10' /var/log/nginx/access.log | tail
```

#### HQ-CLI

##### Запрос к BR-SRV и проверка HSTS

```
curl -k https://192.168.30.2 > /dev/null
curl -kI https://192.168.30.2 | grep -i strict-transport-security

# Ожидается:
# Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

## Финальная проверка после каждого модуля

### После Модуля 1

#### ISP

##### ISP

```
ip -br a
ip r
sysctl net.ipv4.ip_forward
nft list ruleset
ping -c 3 172.16.1.2
ping -c 3 172.16.2.2
ping -c 3 192.168.100.2
ping -c 3 192.168.30.2
```

#### HQ-RTR и BR-RTR

##### Eltex

```
show ip interfaces
show ip route
show tunnels status gre 1
show ip ospf neighbors
show ip nat source rulesets
show ip dhcp binding   # только HQ-RTR
```

#### HQ-CLI

##### HQ-CLI

```
ip -br a
ip r
cat /etc/resolv.conf
ping -c 3 192.168.100.2
ping -c 3 192.168.30.2
nslookup hq-srv.sirius-exam.org
nslookup ya.ru
```

### После Модуля 2

#### HQ-SRV

##### HQ-SRV

```
cat /proc/mdstat
mdadm --detail /dev/md0
df -h /raid
exportfs -v
curl http://127.0.0.1
```

#### BR-SRV

##### BR-SRV

```
testparm
pdbedit -L
docker ps
curl http://127.0.0.1:8080
chronyc sources -v
```

#### ISP

##### ISP

```
nginx -t
curl -I -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -u WEB:P@ssw0rd -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1
```

#### HQ-CLI

##### HQ-CLI

```
ls -la /mnt/nfs
mount | grep /mnt/nfs
smbclient //192.168.30.2/hq -U hquser1
rpm -qa | grep -i yandex
```

### После вариативной части

#### BR-SRV

##### BR-SRV

```
docker ps
nginx -t
curl -kI https://127.0.0.1
ls -l /etc/nginx/sites-enabled/
```

#### HQ-CLI

##### HQ-CLI

```
curl -kI https://192.168.30.2 | grep -i strict-transport-security
grep '192.168.200.10' /var/log/nginx/access.log | tail   # выполнить на BR-SRV
ls -ld /home/Folder/work_shared /home/Folder/job_readonly
```
