ДОКУМЕНТАЦИЯ - ГИА ДЭ RED OS и Eltex 1.37.4


[Введение](#intro)[IP-план](#ipplan)
[М1. База](#m1-1)[М1. ISP](#m1-3)[М1. SSH](#m1-5)
[М1. GRE](#m1-6)[М1. OSPF](#m1-7)[М1. NAT/DHCP](#m1-8)[М1. DNS](#m1-9)
[М2. Samba](#m2-1)[М2. RAID](#m2-2)[М2. NFS](#m2-3)[М2. Chrony](#m2-4)
[М2. Docker](#m2-5)[М2. Apache](#m2-6)[М2. DNAT](#m2-7)[М2. Nginx](#m2-8)
[Вариативная часть](#variant)[Источники](#sources)
[Аудит критериев](#audit)

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## ДОКУМЕНТАЦИЯ

Последовательная инструкция по всем заданиям из загруженного файла Word.
Оформление повторяет стиль учебного PDF «Модуль-2»: белые листы, верхний колонтитул, красный логотип,
голубые таблицы и светло-голубые блоки команд.

| Устройство | ОС / версия | Роль |
| --- | --- | --- |
| ISP | РЕД ОС сервер минимальный, без GUI | Выход в интернет, маршрутизация, NAT, chrony, nginx |
| HQ-SRV | РЕД ОС сервер минимальный, без GUI | DNS, RAID0, NFS, Apache, MariaDB |
| BR-SRV | РЕД ОС сервер минимальный, без GUI | Samba, Docker testapp, Nextcloud |
| HQ-RTR | Eltex ESR/vESR 1.37.4 | Маршрутизация HQ, GRE, OSPF, NAT, DHCP, DNAT |
| BR-RTR | Eltex ESR/vESR 1.37.4 | Маршрутизация BR, GRE, OSPF, NAT, DNAT, NTP-клиент |
| HQ-CLI | РЕД ОС рабочая станция, GUI | DHCP-клиент, NFS-клиент, Samba-клиент, браузер, проверки вариативной части |

**Проверка достоверности.** Команды Eltex сверены со справочником CLI ESR-Series версии 1.37. Команды РЕД ОС собраны по официальной Базе знаний РЕД ОС и стандартным конфигурационным файлам Linux. Файл не заменяет контрольный прогон на вашем стенде: перед экзаменом выполните разделы проверки.

**Без VLAN.** В Word-топологии HQ-RTR подключается к HQ-SRV и HQ-CLI отдельными физическими интерфейсами. Поэтому VLAN-команды не используются. Сеть управления /29 заносится в IP-план, но отдельный интерфейс для неё в топологии не указан.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

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

На ваших текущих VM имена интерфейсов ISP уже наблюдались как **enp7s1 / enp7s2 / enp7s3**. Перед вводом команд всё равно выполните `ip -br a` и `nmcli con show`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задания 1-2. Имена устройств и IPv4

### 1. Настроить FQDN

#### ISP

ISP**BASH**

```
hostnamectl hostname isp.sirius-exam.org
hostnamectl
```

#### HQ-SRV

HQ-SRV**BASH**

```
hostnamectl hostname hq-srv.sirius-exam.org
hostnamectl
```

#### BR-SRV

BR-SRV**BASH**

```
hostnamectl hostname br-srv.sirius-exam.org
hostnamectl
```

#### HQ-CLI

HQ-CLI**BASH**

```
hostnamectl hostname hq-cli.sirius-exam.org
hostnamectl
```

#### HQ-RTR

HQ-RTR**ELTEX**

```
configure
hostname hq-rtr.sirius-exam.org
commit
confirm
save
```

#### BR-RTR

BR-RTR**ELTEX**

```
configure
hostname br-rtr.sirius-exam.org
commit
confirm
save
```

### 2. Назначить IPv4

#### ISP

ISP: переименовать профили и назначить адреса**BASH**

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

HQ-SRV**BASH**

```
nmcli con show
nmcli con mod to-hq connection.id HQ-SRV
nmcli con mod HQ-SRV ipv4.method manual ipv4.addresses 192.168.100.2/27 ipv4.gateway 192.168.100.1
nmcli con up HQ-SRV
ip -br a
ip r
```

#### BR-SRV

BR-SRV**BASH**

```
nmcli con show
nmcli con mod to-br connection.id BR-SRV
nmcli con mod BR-SRV ipv4.method manual ipv4.addresses 192.168.30.2/28 ipv4.gateway 192.168.30.1
nmcli con up BR-SRV
ip -br a
ip r
```

#### HQ-RTR

HQ-RTR**ELTEX**

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

BR-RTR**ELTEX**

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

**Проверка:** с ISP должны отвечать `ping -c 3 172.16.1.2` и `ping -c 3 172.16.2.2`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задание 3. Интернет на ISP и учётные записи

### 3.1-3.2. ISP: маршруты, forwarding и PAT

#### ISP

ISP: постоянные маршруты к офисам**BASH**

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

ISP: forwarding и nftables masquerade**BASH**

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

Одинаково на обоих серверах**BASH**

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

### 3.4-3.7. net\_admin на Eltex

#### HQ-RTR и BR-RTR

Одинаково на обоих маршрутизаторах**ELTEX**

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

На Eltex уровень **15** - максимальный. Команда `privilege 15` предусмотрена режимом CONFIG-USER справочника CLI 1.37.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задания 4-5. Маршрутизация HQ и безопасный SSH

### 4. Маршрутизация трафика HQ-RTR

#### HQ-RTR

Проверка connected-маршрутов**ELTEX**

```
show ip route
# Должны быть connected:
# 192.168.100.0/27
# 192.168.200.0/27
# 172.16.1.0/28
```

Отдельные статические маршруты между HQ-SRV-Net и HQ-CLI-Net не нужны: обе сети directly connected к HQ-RTR.

### 5. SSH: порт 2026, только sshuser, 2 попытки, баннер

#### HQ-SRV и BR-SRV

Одинаково на обоих серверах**BASH**

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

Если на сервере включён firewalld, откройте порт: `firewall-cmd --permanent --add-port=2026/tcp && firewall-cmd --reload`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задание 6. GRE-туннель между офисами

#### HQ-RTR

HQ-RTR: GRE 1**ELTEX**

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

BR-RTR: GRE 1**ELTEX**

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

**Проверка:** HQ-RTR: `ping 10.10.10.2`; BR-RTR: `ping 10.10.10.1`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задание 7. OSPF только через GRE

Используется link-state протокол OSPF. Соседство разрешается только на интерфейсе `tunnel gre 1`.
Пароль `P@ssw0rd` состоит из 8 символов и укладывается в ограничение cleartext-аутентификации Eltex.

#### HQ-RTR

HQ-RTR: OSPF**ELTEX**

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

BR-RTR: OSPF**ELTEX**

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

**Проверка:** на каждом маршрутизаторе должен отображаться один OSPF-сосед. HQ-SRV должен пинговать 192.168.30.2, BR-SRV - 192.168.100.2.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задание 8. NAT офисов и DHCP для HQ-CLI

### 8.1. Динамический NAT офисов в сторону ISP

#### HQ-RTR

HQ-RTR: SNAT**ELTEX**

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

BR-RTR: SNAT**ELTEX**

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

HQ-RTR: DHCP-пул**ELTEX**

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

HQ-CLI: получить адрес**BASH**

```
nmcli con show
nmcli con mod "Проводное подключение 1" ipv4.method auto
nmcli con down "Проводное подключение 1"
nmcli con up "Проводное подключение 1"
ip -br a
ip r
cat /etc/resolv.conf
```

Пул содержит только **192.168.200.10**, чтобы A-запись DNS для HQ-CLI оставалась стабильной.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 1. Задание 9. DNS и часовой пояс

### 9.1-9.3. DNS-сервер BIND на HQ-SRV

#### HQ-SRV

Установить BIND**BASH**

```
dnf install -y bind bind-utils
mkdir -p /var/named/master
nano /etc/named.conf
```

/etc/named.conf**CONF**

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

Прямая зона**BASH**

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

PTR-зоны**BASH**

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

Проверить и запустить BIND**BASH**

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

RED OS**BASH**

```
timedatectl set-timezone Europe/Moscow
timedatectl
```

#### HQ-RTR и BR-RTR

Eltex**ELTEX**

```
configure
clock timezone gmt +3
commit
confirm
save
show date
```

Если площадка находится не в часовом поясе Москвы, замените Europe/Moscow и gmt +3 на актуальный часовой пояс.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 1. Samba на BR-SRV

Word-задание требует службу Samba, FQDN `br-srv.sirius-exam.org`, пять пользователей, группу `hq`
и возможность аутентификации с HQ-CLI. Поэтому используется файловый сервер Samba с локальными Samba-учётными записями.
Контроллер домена `au-team.irpo` из учебного PDF не добавляется: в Word-задании он не требуется.

#### BR-SRV

Установить и подготовить Samba**BASH**

```
hostnamectl hostname br-srv.sirius-exam.org
dnf install -y samba samba-client
groupadd hq
mkdir -p /srv/samba/hq
chown root:hq /srv/samba/hq
chmod 2770 /srv/samba/hq
```

Создать пользователей Linux и Samba**BASH**

```
for n in 1 2 3 4 5; do
  useradd -m -G hq hquser$n
  echo "P@ssw0rd" | passwd --stdin hquser$n
  (echo "P@ssw0rd"; echo "P@ssw0rd") | smbpasswd -a -s hquser$n
done

getent group hq
pdbedit -L
```

/etc/samba/smb.conf**CONF**

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

Запуск и проверка Samba**BASH**

```
testparm
systemctl enable --now smb
systemctl restart smb
smbclient -L //127.0.0.1 -U hquser1
```

#### HQ-CLI

Проверить аутентификацию с HQ-CLI**BASH**

```
dnf install -y samba-client
smbclient //192.168.30.2/hq -U hquser1

# Внутри smbclient:
ls
put /etc/hosts hosts-test.txt
ls
exit
```

Проверка выполняется именно с HQ-CLI: пользователь группы hq вводит пароль P@ssw0rd и получает доступ к ресурсу BR-SRV.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 2. RAID0 на HQ-SRV

#### HQ-SRV

Найти дополнительные диски**BASH**

```
dnf install -y mdadm parted
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
# Ниже предполагаются два дополнительных диска: /dev/sdb и /dev/sdc
```

Создать md0 и сохранить конфигурацию**BASH**

```
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat
mdadm --detail /dev/md0
mdadm --detail --scan > /etc/mdadm.conf
cat /etc/mdadm.conf
```

Создать раздел md0p1**BASH**

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

ext4 и автомонтирование /raid**BASH**

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

Официальная База знаний РЕД ОС рекомендует mdadm для программного RAID. В этом задании после создания массива дополнительно создаётся раздел md0p1, как требует Word-файл.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 3. NFS

#### HQ-SRV

Экспортировать /raid/nfs только для HQ-CLI-Net**BASH**

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

Автомонтирование /mnt/nfs при обращении через autofs**BASH**

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

В отчёт занесите: экспорт **/raid/nfs**, разрешённая сеть **192.168.200.0/27**, режим **rw**, клиентская точка **/mnt/nfs**. Использование autofs обеспечивает монтирование именно при обращении к каталогу.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 4. Chrony на ISP

#### ISP

ISP: NTP-сервер stratum 5**BASH**

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

Клиенты HQ**BASH**

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

Клиент BR-SRV**BASH**

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

Клиент BR-RTR**ELTEX**

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

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 5. Docker testapp на BR-SRV

#### BR-SRV

Установить Docker и Compose**BASH**

```
dnf install -y docker-ce docker-ce-cli docker-compose
systemctl enable docker --now
systemctl status docker --no-pager
docker --version
docker-compose --version
```

Подключить Additional.iso и загрузить образы**BASH**

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

После `docker image ls` проверьте фактические теги. В примере ниже используются `site:latest` и `mariadb:10.11`, как в учебном PDF.

/root/web.yaml**YAML**

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

Запуск и проверка**BASH**

```
cd /root
docker-compose -f web.yaml up -d
docker ps
docker logs tespapp --tail 50
curl http://127.0.0.1:8080
```

Публикация `192.168.30.2:8080:8000` сохраняет внешний порт testapp **8080**, но освобождает loopback-адрес `127.0.0.1:8080` для Nextcloud. Word-файл требует имя контейнера **tespapp** - именно с буквой **s** после **te**. В Compose оно задано через `container_name: tespapp`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 6. Apache и MariaDB на HQ-SRV

#### HQ-SRV

Установить и запустить службы**BASH**

```
dnf install -y httpd php php-mysqlnd mariadb-server mariadb
systemctl enable --now mariadb
systemctl enable --now httpd
systemctl status mariadb httpd --no-pager
```

Подключить Additional.iso**BASH**

```
lsblk
mkdir -p /mnt/additional
mount /dev/sr0 /mnt/additional
find /mnt/additional/web -maxdepth 2 -type f | sort
```

Создать webdb и пользователя web**SQL**

```
mysql -u root

CREATE DATABASE webdb;
CREATE USER 'web'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Импортировать приложение**BASH**

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

В отчёт занесите: Apache на порту 80, БД webdb, пользователь web, импорт dump.sql, каталог приложения /var/www/html.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задание 7. DNAT на Eltex

### HQ-RTR: 8080 → HQ-SRV:80 и 2026 → HQ-SRV:2026

#### HQ-RTR

HQ-RTR**ELTEX**

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

BR-RTR**ELTEX**

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

Проверка с ISP**BASH**

```
curl http://172.16.1.2:8080
curl http://172.16.2.2:8080
ssh -p 2026 sshuser@172.16.1.2
ssh -p 2026 sshuser@172.16.2.2
```

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Модуль 2. Задания 8-10. Nginx, Basic Auth, Яндекс Браузер

### 8. Reverse proxy на ISP

#### ISP

Установить Nginx**BASH**

```
dnf install -y nginx httpd-tools
nano /etc/nginx/conf.d/exam.conf
```

/etc/nginx/conf.d/exam.conf**NGINX**

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

Проверить Nginx**BASH**

```
nginx -t
systemctl enable --now nginx
systemctl restart nginx
curl -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1
```

### 9. Basic Auth только для web.sirius-exam.org

#### ISP

Создать /etc/nginx/.htpasswd**BASH**

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

HQ-CLI**BASH**

```
sudo dnf install -y yandex-browser-stable

# Если пакет не найден:
sudo dnf install -y yandex-browser-release
sudo dnf install -y yandex-browser-stable

rpm -qa | grep -i yandex
```

Официальная инструкция РЕД ОС указывает пакет `yandex-browser-stable`; при отсутствии репозитория сначала устанавливается `yandex-browser-release`.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Вариативная часть. Nextcloud и права доступа

### Вариативный модуль 1. Nextcloud на BR-SRV

**testapp не останавливается.** Основное приложение остаётся доступным на `192.168.30.2:8080`. Для Nextcloud используется обязательный порт `127.0.0.1:8080`. Конфликта нет, потому что сервисы слушают разные локальные IP-адреса.

#### BR-SRV

Проверить статический IP Сервера 1**BASH**

```
ip -br a
ip r
# Должно быть:
# 192.168.30.2/28, шлюз 192.168.30.1
```

Установить совместимый Docker CLI и Docker Compose**BASH**

```
# Если Docker уже установлен для testapp, повторная установка не нужна.
command -v docker >/dev/null || sudo yum install -y podman-docker
command -v docker-compose >/dev/null || sudo yum install -y docker-compose

# Проверка установки
docker --version
docker-compose --version
```

Команды основаны на приложенной инструкции Nextcloud. Условная установка не пытается поставить `podman-docker` поверх уже работающего Docker Engine из основного Модуля 2.

Создать рабочий каталог**BASH**

```
sudo mkdir -p /opt/nextcloud
cd /opt/nextcloud
sudo nano docker-compose.yml
```

/opt/nextcloud/docker-compose.yml**YAML**

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

Запустить контейнеры Nextcloud**BASH**

```
cd /opt/nextcloud
sudo docker-compose up -d
sudo docker ps
curl -I http://127.0.0.1:8080
```

Установить Nginx и создать самоподписанный сертификат**BASH**

```
sudo yum install -y nginx openssl
sudo mkdir -p /etc/ssl/certs /etc/ssl/private

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nextcloud-selfsigned.key \
    -out /etc/ssl/certs/nextcloud-selfsigned.crt \
    -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=127.0.0.1"
```

Создать конфигурацию Nginx**BASH**

```
sudo mkdir -p /etc/nginx/sites-available
sudo nano /etc/nginx/sites-available/nextcloud
```

/etc/nginx/sites-available/nextcloud**NGINX**

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

Создать ссылку sites-enabled и подключить каталог**BASH**

```
sudo mkdir -p /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/nextcloud

sudo nano /etc/nginx/nginx.conf
# Внутри блока http { ... } добавить строку:
include /etc/nginx/sites-enabled/*;
```

Проверить и запустить Nginx**BASH**

```
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl restart nginx
curl -kI https://127.0.0.1
```

Локальная проверка по условию: на BR-SRV откройте **https://127.0.0.1** или выполните `curl -kI https://127.0.0.1`. Должна открыться страница начальной настройки Nextcloud.

### Вариативный модуль 2. Пользователи и каталоги на HQ-CLI

#### HQ-CLI

Создать группы, пользователей и каталоги**BASH**

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

Открыть страницу установки Nextcloud**BASH**

```
# В браузере HQ-CLI открыть:
https://192.168.30.2

# Подтвердить переход по самоподписанному сертификату.
# Должна открыться страница установки Nextcloud.
```

Проверка прав**BASH**

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

Просмотр access.log и поиск IP Сервера 2**BASH**

```
tail -f /var/log/nginx/access.log

# После запроса с HQ-CLI остановить tail сочетанием Ctrl+C и проверить:
grep '192.168.200.10' /var/log/nginx/access.log | tail
```

#### HQ-CLI

Запрос к BR-SRV и проверка HSTS**BASH**

```
curl -k https://192.168.30.2 > /dev/null
curl -kI https://192.168.30.2 | grep -i strict-transport-security

# Ожидается:
# Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

После первого HTTPS-перехода браузер HQ-CLI получает HSTS-заголовок. Для проверки оцениваемого результата сохраните вывод команды `curl -kI` и строку из access.log с IP HQ-CLI.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Финальная проверка после каждого модуля

### После Модуля 1

#### ISP

ISP**BASH**

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

Eltex**ELTEX**

```
show ip interfaces
show ip route
show tunnels status gre 1
show ip ospf neighbors
show ip nat source rulesets
show ip dhcp binding   # только HQ-RTR
```

#### HQ-CLI

HQ-CLI**BASH**

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

HQ-SRV**BASH**

```
cat /proc/mdstat
mdadm --detail /dev/md0
df -h /raid
exportfs -v
curl http://127.0.0.1
```

#### BR-SRV

BR-SRV**BASH**

```
testparm
pdbedit -L
docker ps
curl http://127.0.0.1:8080
chronyc sources -v
```

#### ISP

ISP**BASH**

```
nginx -t
curl -I -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -u WEB:P@ssw0rd -H 'Host: web.sirius-exam.org' http://127.0.0.1
curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1
```

#### HQ-CLI

HQ-CLI**BASH**

```
ls -la /mnt/nfs
mount | grep /mnt/nfs
smbclient //192.168.30.2/hq -U hquser1
rpm -qa | grep -i yandex
```

### После вариативной части

#### BR-SRV

BR-SRV**BASH**

```
docker ps
nginx -t
curl -kI https://127.0.0.1
ls -l /etc/nginx/sites-enabled/
```

#### HQ-CLI

HQ-CLI**BASH**

```
curl -kI https://192.168.30.2 | grep -i strict-transport-security
grep '192.168.200.10' /var/log/nginx/access.log | tail   # выполнить на BR-SRV
ls -ld /home/Folder/work_shared /home/Folder/job_readonly
```

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Аудит соответствия критериям

Ниже приведён статический аудит HTML и Markdown-копии по критериям оценивания. Статус означает, что в документации присутствуют необходимые команды и проверки. Реальное выполнение подтверждается только прогоном на стенде.

### Основная часть: 49 подкритериев

| № | Проверяемое действие | Раздел | Статус |
| --- | --- | --- | --- |
| 1 | FQDN всем устройствам | [m1-1](#m1-1) | Покрыто командами; требуется стендовый прогон |
| 2 | Расчёт RFC1918-подсетей | [ipplan](#ipplan) | Покрыто командами; требуется стендовый прогон |
| 3 | IPv4 на HQ-RTR, BR-RTR, HQ-SRV, BR-SRV, HQ-CLI | [m1-1](#m1-1) | Покрыто командами; требуется стендовый прогон |
| 4 | DHCP WAN ISP и default route | [m1-1](#m1-1) | Покрыто командами; требуется стендовый прогон |
| 5 | ISP-HQ и ISP-BR /28 | [m1-1](#m1-1) | Покрыто командами; требуется стендовый прогон |
| 6 | PAT на ISP | [m1-3](#m1-3) | Покрыто командами; требуется стендовый прогон |
| 7 | sshuser UID 2026, пароль, sudo NOPASSWD | [m1-3](#m1-3) | Покрыто командами; требуется стендовый прогон |
| 8 | net\_admin с максимальными привилегиями | [m1-3](#m1-3) | Покрыто командами; требуется стендовый прогон |
| 9 | SSH 2026 только sshuser | [m1-5](#m1-5) | Покрыто командами; требуется стендовый прогон |
| 10 | MaxAuthTries 2 | [m1-5](#m1-5) | Покрыто командами; требуется стендовый прогон |
| 11 | Баннер Authorized access only | [m1-5](#m1-5) | Покрыто командами; требуется стендовый прогон |
| 12 | GRE/IPIP между HQ-RTR и BR-RTR | [m1-6](#m1-6) | Покрыто командами; требуется стендовый прогон |
| 13 | OSPF только на GRE | [m1-7](#m1-7) | Покрыто командами; требуется стендовый прогон |
| 14 | Пароль OSPF | [m1-7](#m1-7) | Покрыто командами; требуется стендовый прогон |
| 15 | Обмен офисными маршрутами | [m1-7](#m1-7) | Покрыто командами; требуется стендовый прогон |
| 16 | DHCP HQ-CLI и исключение адреса роутера | [m1-8](#m1-8) | Покрыто командами; требуется стендовый прогон |
| 17 | Шлюз HQ-RTR для HQ-CLI | [m1-8](#m1-8) | Покрыто командами; требуется стендовый прогон |
| 18 | DNS HQ-SRV и суффикс sirius-exam.org | [m1-8](#m1-8) | Покрыто командами; требуется стендовый прогон |
| 19 | Прямая и обратные DNS-зоны | [m1-9](#m1-9) | Покрыто командами; требуется стендовый прогон |
| 20 | DNS forwarder | [m1-9](#m1-9) | Покрыто командами; требуется стендовый прогон |
| 21 | Chrony ISP stratum 5 | [m2-4](#m2-4) | Покрыто командами; требуется стендовый прогон |
| 22 | Клиенты chrony HQ-SRV, HQ-CLI, BR-RTR, BR-SRV | [m2-4](#m2-4) | Покрыто командами; требуется стендовый прогон |
| 23 | FQDN BR-SRV | [m2-1](#m2-1) | Покрыто командами; требуется стендовый прогон |
| 24 | Доступ HQ-CLI к BR-SRV | [m2-1](#m2-1) | Покрыто командами; требуется стендовый прогон |
| 25 | hquser1-hquser5 | [m2-1](#m2-1) | Покрыто командами; требуется стендовый прогон |
| 26 | Группа hq | [m2-1](#m2-1) | Покрыто командами; требуется стендовый прогон |
| 27 | Парольная Samba-аутентификация с HQ-CLI | [m2-1](#m2-1) | Покрыто literal-требование: Samba-вход по паролю с HQ-CLI; см. примечание ниже |
| 28 | RAID0 md0 из двух дисков | [m2-2](#m2-2) | Покрыто командами; требуется стендовый прогон |
| 29 | /etc/mdadm.conf | [m2-2](#m2-2) | Покрыто командами; требуется стендовый прогон |
| 30 | Раздел, ext4, автомонтирование /raid | [m2-2](#m2-2) | Покрыто командами; требуется стендовый прогон |
| 31 | Экспорт /raid/nfs только HQ-CLI-Net | [m2-3](#m2-3) | Покрыто командами; требуется стендовый прогон |
| 32 | Автомонтирование /mnt/nfs при обращении | [m2-3](#m2-3) | Покрыто командами; требуется стендовый прогон |
| 33 | Docker testapp + db из ISO | [m2-5](#m2-5) | Покрыто командами; требуется стендовый прогон |
| 34 | testdb, test, P@ssw0rd | [m2-5](#m2-5) | Покрыто командами; требуется стендовый прогон |
| 35 | testapp доступен извне на 8080 | [m2-5](#m2-5) | Покрыто командами; требуется стендовый прогон |
| 36 | Импорт dump.sql в webdb | [m2-6](#m2-6) | Покрыто командами; требуется стендовый прогон |
| 37 | web / P@ssw0rd и права webdb | [m2-6](#m2-6) | Покрыто командами; требуется стендовый прогон |
| 38 | index.php и images | [m2-6](#m2-6) | Покрыто командами; требуется стендовый прогон |
| 39 | Корректные реквизиты index.php | [m2-6](#m2-6) | Покрыто командами; требуется стендовый прогон |
| 40 | BR-RTR DNAT 8080 → testapp | [m2-7](#m2-7) | Покрыто командами; требуется стендовый прогон |
| 41 | HQ-RTR DNAT 8080 → Apache | [m2-7](#m2-7) | Покрыто командами; требуется стендовый прогон |
| 42 | HQ-RTR DNAT 2026 → HQ-SRV | [m2-7](#m2-7) | Покрыто командами; требуется стендовый прогон |
| 43 | BR-RTR DNAT 2026 → BR-SRV | [m2-7](#m2-7) | Покрыто командами; требуется стендовый прогон |
| 44 | web.sirius-exam.org → HQ-SRV | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |
| 45 | docker.sirius-exam.org → testapp | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |
| 46 | Basic Auth для web | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |
| 47 | WEB / P@ssw0rd в .htpasswd | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |
| 48 | Переход на сайт после авторизации | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |
| 49 | Яндекс Браузер HQ-CLI | [m2-8](#m2-8) | Покрыто командами; требуется стендовый прогон |

### Вариативная часть: 25 действий

| № | Проверяемое действие | Раздел | Статус |
| --- | --- | --- | --- |
| V1 | Docker и Docker Compose на BR-SRV | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V2 | Каталог /opt/nextcloud | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V3 | docker-compose.yml | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V4 | Volume БД | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V5 | Volume Nextcloud | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V6 | docker-compose up -d | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V7 | Nginx | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V8 | Файл конфигурации Nginx | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V9 | Символическая ссылка sites-enabled | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V10 | Самоподписанный SSL | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V11 | Прокси на обязательный порт 8080 Nextcloud | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V12 | Локальная проверка https://127.0.0.1 | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V13 | Группы Work, Job, labor | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V14 | User1, User2 в Work | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V15 | User3, User4 в Job | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V16 | User5 в labor | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V17 | Каталоги /home/Folder/... | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V18 | Права 770 root:Work | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V19 | Права 750 root:Job | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V20 | Зафиксировать IP Сервера 1 | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V21 | Браузер HQ-CLI → Nextcloud, принять сертификат | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V22 | Проверка User1/User3/User5 | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V23 | Просмотр access.log Nginx | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V24 | Поиск IP HQ-CLI в логах | [variant](#variant) | Покрыто командами; требуется стендовый прогон |
| V25 | Проверка точного HSTS-заголовка | [variant](#variant) | Покрыто командами; требуется стендовый прогон |

**Примечание по Samba:** текущий Word-файл требует службу Samba и парольную аутентификацию пользователей группы hq с HQ-CLI, но не требует явно разворачивать Samba AD DC и вводить HQ-CLI в домен. Документация реализует буквальный вариант через smbclient. Если на вашей площадке эксперт требует вход доменным пользователем в саму ОС HQ-CLI, используйте отдельный сценарий Samba AD DC и join-to-domain.

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

**РЕДОС**

## Официальные источники и границы проверки

Команды в runbook сверены с официальными документами и официальной Базой знаний.
Генерация файла не выполняет команды на вашем реальном стенде, поэтому финальная гарантия достигается контрольным прогоном на VM.

- [Eltex ESR-Series. Справочник команд CLI. Версия 1.37](https://api.prod.eltex-co.ru/storage/upload_center/files/53/ESR-Series_CLI_1.37.pdf)
- [РЕД ОС: установка и настройка Docker](https://redos.red-soft.ru/base/redos-7_3/7_3-administation/7_3-containers/7_3-docker-install/)
- [РЕД ОС: создание программного RAID0 и RAID1](https://redos.red-soft.ru/base/redos-7_3/7_3-install/7_3-alter-install/7_3-install-on-raid/7_3-program-raid0-and-raid1/)
- [РЕД ОС: установка Яндекс Браузера](https://redos.red-soft.ru/base/redos-7_3/7_3-users-tasks/7_3-browser/7_3-yandex-browser-install/)
- [Nginx: proxy\_pass и proxy\_set\_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Nginx: auth\_basic и auth\_basic\_user\_file](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)
- [Docker Compose documentation](https://docs.docker.com/compose/)
- [Official MySQL Docker image](https://hub.docker.com/_/mysql)
- [Official Nextcloud Docker image](https://hub.docker.com/_/nextcloud)

**Особенно важные проверки перед сдачей:** интерфейсы ISP, наличие Additional.iso как /dev/sr0, фактические Docker-теги после docker load, а также доступность репозиториев Docker на BR-SRV.

КОД-09.02.06-1-2026 · Сетевой и системный администратор · RED OS / Eltex ESR 1.37.4