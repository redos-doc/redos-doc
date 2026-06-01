<div class="wrap">

[Введение](#intro)[IP-план](#ipplan) [М1. База](#m1-1)[М1. ISP](#m1-3)[М1. SSH](#m1-5) [М1. GRE](#m1-6)[М1. OSPF](#m1-7)[М1. NAT/DHCP](#m1-8)[М1. DNS](#m1-9) [М2. Samba](#m2-1)[М2. RAID](#m2-2)[М2. NFS](#m2-3)[М2. Chrony](#m2-4) [М2. Docker](#m2-5)[М2. Apache](#m2-6)[М2. DNAT](#m2-7)[М2. Nginx](#m2-8) [Вариативная часть](#variant)[Источники](#sources)

</div>

<div id="intro" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## ДОКУМЕНТАЦИЯ

</div>

Последовательная инструкция по всем заданиям из загруженного файла Word. Оформление повторяет стиль учебного PDF «Модуль-2»: белые листы, верхний колонтитул, красный логотип, голубые таблицы и светло-голубые блоки команд.

| Устройство | ОС / версия                        | Роль                                                                       |
|------------|------------------------------------|----------------------------------------------------------------------------|
| ISP        | РЕД ОС сервер минимальный, без GUI | Выход в интернет, маршрутизация, NAT, chrony, nginx                        |
| HQ-SRV     | РЕД ОС сервер минимальный, без GUI | DNS, RAID0, NFS, Apache, MariaDB                                           |
| BR-SRV     | РЕД ОС сервер минимальный, без GUI | Samba, Docker testapp, Nextcloud                                           |
| HQ-RTR     | Eltex ESR/vESR 1.37.4              | Маршрутизация HQ, GRE, OSPF, NAT, DHCP, DNAT                               |
| BR-RTR     | Eltex ESR/vESR 1.37.4              | Маршрутизация BR, GRE, OSPF, NAT, DNAT, NTP-клиент                         |
| HQ-CLI     | РЕД ОС рабочая станция, GUI        | DHCP-клиент, NFS-клиент, Samba-клиент, браузер, проверки вариативной части |

<div class="note ok">

**Проверка достоверности.** Команды Eltex сверены со справочником CLI ESR-Series версии 1.37. Команды РЕД ОС собраны по официальной Базе знаний РЕД ОС и стандартным конфигурационным файлам Linux. Файл не заменяет контрольный прогон на вашем стенде: перед экзаменом выполните разделы проверки.

</div>

<div class="note warn">

**Без VLAN.** В Word-топологии HQ-RTR подключается к HQ-SRV и HQ-CLI отдельными физическими интерфейсами. Поэтому VLAN-команды не используются. Сеть управления /29 заносится в IP-план, но отдельный интерфейс для неё в топологии не указан.

</div>

</div>

<div id="ipplan" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Таблица адресации

</div>

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th>Сегмент</th>
<th>Сеть</th>
<th>Устройство и интерфейс</th>
<th>IP-адрес</th>
<th>Шлюз</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Internet</td>
<td>DHCP</td>
<td>ISP enp7s1</td>
<td>DHCP</td>
<td>DHCP</td>
</tr>
<tr class="even">
<td>ISP-HQ</td>
<td>172.16.1.0/28</td>
<td>ISP enp7s3<br />
HQ-RTR gi1/0/2</td>
<td>172.16.1.1/28<br />
172.16.1.2/28</td>
<td>-<br />
172.16.1.1</td>
</tr>
<tr class="odd">
<td>ISP-BR</td>
<td>172.16.2.0/28</td>
<td>ISP enp7s2<br />
BR-RTR gi1/0/2</td>
<td>172.16.2.1/28<br />
172.16.2.2/28</td>
<td>-<br />
172.16.2.1</td>
</tr>
<tr class="even">
<td>HQ-SRV-Net</td>
<td>192.168.100.0/27</td>
<td>HQ-RTR gi1/0/3<br />
HQ-SRV enp7s1</td>
<td>192.168.100.1/27<br />
192.168.100.2/27</td>
<td>-<br />
192.168.100.1</td>
</tr>
<tr class="odd">
<td>HQ-CLI-Net</td>
<td>192.168.200.0/27</td>
<td>HQ-RTR gi1/0/4<br />
HQ-CLI enp7s1</td>
<td>192.168.200.1/27<br />
DHCP: 192.168.200.10</td>
<td>-<br />
192.168.200.1</td>
</tr>
<tr class="even">
<td>Management</td>
<td>192.168.99.0/29</td>
<td>Резерв в отчёте</td>
<td>Не назначается без отдельного интерфейса</td>
<td>-</td>
</tr>
<tr class="odd">
<td>BR-Net</td>
<td>192.168.30.0/28</td>
<td>BR-RTR gi1/0/3<br />
BR-SRV enp7s1</td>
<td>192.168.30.1/28<br />
192.168.30.2/28</td>
<td>-<br />
192.168.30.1</td>
</tr>
<tr class="even">
<td>GRE</td>
<td>10.10.10.0/30</td>
<td>HQ-RTR gre 1<br />
BR-RTR gre 1</td>
<td>10.10.10.1/30<br />
10.10.10.2/30</td>
<td>-</td>
</tr>
</tbody>
</table>

<div class="note info">

На ваших текущих VM имена интерфейсов ISP уже наблюдались как **enp7s1 / enp7s2 / enp7s3**. Перед вводом команд всё равно выполните `ip -br a` и `nmcli con show`.

</div>

</div>

<div id="m1-1" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задания 1-2. Имена устройств и IPv4

</div>

### 1. Настроить FQDN

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP**BASH**

</div>

    hostnamectl hostname isp.sirius-exam.org
    hostnamectl

</div>

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

HQ-SRV**BASH**

</div>

    hostnamectl hostname hq-srv.sirius-exam.org
    hostnamectl

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

BR-SRV**BASH**

</div>

    hostnamectl hostname br-srv.sirius-exam.org
    hostnamectl

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI**BASH**

</div>

    hostnamectl hostname hq-cli.sirius-exam.org
    hostnamectl

</div>

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR**ELTEX**

</div>

    configure
    hostname hq-rtr.sirius-exam.org
    commit
    confirm
    save

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR**ELTEX**

</div>

    configure
    hostname br-rtr.sirius-exam.org
    commit
    confirm
    save

</div>

### 2. Назначить IPv4

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP: переименовать профили и назначить адреса**BASH**

</div>

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

</div>

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

HQ-SRV**BASH**

</div>

    nmcli con show
    nmcli con mod to-hq connection.id HQ-SRV
    nmcli con mod HQ-SRV ipv4.method manual ipv4.addresses 192.168.100.2/27 ipv4.gateway 192.168.100.1
    nmcli con up HQ-SRV
    ip -br a
    ip r

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

BR-SRV**BASH**

</div>

    nmcli con show
    nmcli con mod to-br connection.id BR-SRV
    nmcli con mod BR-SRV ipv4.method manual ipv4.addresses 192.168.30.2/28 ipv4.gateway 192.168.30.1
    nmcli con up BR-SRV
    ip -br a
    ip r

</div>

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR**ELTEX**

</div>

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

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR**ELTEX**

</div>

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

</div>

<div class="note ok">

**Проверка:** с ISP должны отвечать `ping -c 3 172.16.1.2` и `ping -c 3 172.16.2.2`.

</div>

</div>

<div id="m1-3" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задание 3. Интернет на ISP и учётные записи

</div>

### 3.1-3.2. ISP: маршруты, forwarding и PAT

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP: постоянные маршруты к офисам**BASH**

</div>

    ip route replace 192.168.100.0/27 via 172.16.1.2
    ip route replace 192.168.200.0/27 via 172.16.1.2
    ip route replace 192.168.30.0/28 via 172.16.2.2

    nmcli con mod ISP-HQ +ipv4.routes "192.168.100.0/27 172.16.1.2"
    nmcli con mod ISP-HQ +ipv4.routes "192.168.200.0/27 172.16.1.2"
    nmcli con mod ISP-BR +ipv4.routes "192.168.30.0/28 172.16.2.2"
    nmcli con up ISP-HQ
    nmcli con up ISP-BR
    ip r

</div>

<div class="cmd">

<div class="cmd-title">

ISP: forwarding и nftables masquerade**BASH**

</div>

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

</div>

### 3.3. sshuser на серверах

#### HQ-SRV и BR-SRV

<div class="cmd">

<div class="cmd-title">

Одинаково на обоих серверах**BASH**

</div>

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

</div>

### 3.4-3.7. net_admin на Eltex

#### HQ-RTR и BR-RTR

<div class="cmd">

<div class="cmd-title">

Одинаково на обоих маршрутизаторах**ELTEX**

</div>

    configure
    username net_admin
      password ascii-text P@ssw0rd
      privilege 15
    exit
    ip ssh server
    commit
    confirm
    save

</div>

<div class="note ok">

На Eltex уровень **15** - максимальный. Команда `privilege 15` предусмотрена режимом CONFIG-USER справочника CLI 1.37.

</div>

</div>

<div id="m1-5" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задания 4-5. Маршрутизация HQ и безопасный SSH

</div>

### 4. Маршрутизация трафика HQ-RTR

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

Проверка connected-маршрутов**ELTEX**

</div>

    show ip route
    # Должны быть connected:
    # 192.168.100.0/27
    # 192.168.200.0/27
    # 172.16.1.0/28

</div>

<div class="note info">

Отдельные статические маршруты между HQ-SRV-Net и HQ-CLI-Net не нужны: обе сети directly connected к HQ-RTR.

</div>

### 5. SSH: порт 2026, только sshuser, 2 попытки, баннер

#### HQ-SRV и BR-SRV

<div class="cmd">

<div class="cmd-title">

Одинаково на обоих серверах**BASH**

</div>

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

</div>

<div class="note info">

Если на сервере включён firewalld, откройте порт: `firewall-cmd --permanent --add-port=2026/tcp && firewall-cmd --reload`.

</div>

</div>

<div id="m1-6" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задание 6. GRE-туннель между офисами

</div>

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR: GRE 1**ELTEX**

</div>

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

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR: GRE 1**ELTEX**

</div>

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

</div>

<div class="note ok">

**Проверка:** HQ-RTR: `ping 10.10.10.2`; BR-RTR: `ping 10.10.10.1`.

</div>

</div>

<div id="m1-7" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задание 7. OSPF только через GRE

</div>

Используется link-state протокол OSPF. Соседство разрешается только на интерфейсе `tunnel gre 1`. Пароль `P@ssw0rd` состоит из 8 символов и укладывается в ограничение cleartext-аутентификации Eltex.

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR: OSPF**ELTEX**

</div>

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

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR: OSPF**ELTEX**

</div>

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

</div>

<div class="note ok">

**Проверка:** на каждом маршрутизаторе должен отображаться один OSPF-сосед. HQ-SRV должен пинговать 192.168.30.2, BR-SRV - 192.168.100.2.

</div>

</div>

<div id="m1-8" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задание 8. NAT офисов и DHCP для HQ-CLI

</div>

### 8.1. Динамический NAT офисов в сторону ISP

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR: SNAT**ELTEX**

</div>

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

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR: SNAT**ELTEX**

</div>

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

</div>

### 8.2. DHCP для HQ-CLI

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR: DHCP-пул**ELTEX**

</div>

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

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI: получить адрес**BASH**

</div>

    nmcli con show
    nmcli con mod "Проводное подключение 1" ipv4.method auto
    nmcli con down "Проводное подключение 1"
    nmcli con up "Проводное подключение 1"
    ip -br a
    ip r
    cat /etc/resolv.conf

</div>

<div class="note info">

Пул содержит только **192.168.200.10**, чтобы A-запись DNS для HQ-CLI оставалась стабильной.

</div>

</div>

<div id="m1-9" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 1. Задание 9. DNS и часовой пояс

</div>

### 9.1-9.3. DNS-сервер BIND на HQ-SRV

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

Установить BIND**BASH**

</div>

    dnf install -y bind bind-utils
    mkdir -p /var/named/master
    nano /etc/named.conf

</div>

<div class="cmd">

<div class="cmd-title">

/etc/named.conf**CONF**

</div>

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

</div>

<div class="cmd">

<div class="cmd-title">

Прямая зона**BASH**

</div>

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

</div>

<div class="cmd">

<div class="cmd-title">

PTR-зоны**BASH**

</div>

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

</div>

<div class="cmd">

<div class="cmd-title">

Проверить и запустить BIND**BASH**

</div>

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

</div>

### 9.4. Часовой пояс

#### ISP, HQ-SRV, BR-SRV, HQ-CLI

<div class="cmd">

<div class="cmd-title">

RED OS**BASH**

</div>

    timedatectl set-timezone Europe/Moscow
    timedatectl

</div>

#### HQ-RTR и BR-RTR

<div class="cmd">

<div class="cmd-title">

Eltex**ELTEX**

</div>

    configure
    clock timezone gmt +3
    commit
    confirm
    save
    show date

</div>

<div class="note warn">

Если площадка находится не в часовом поясе Москвы, замените Europe/Moscow и gmt +3 на актуальный часовой пояс.

</div>

</div>

<div id="m2-1" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 1. Samba на BR-SRV

</div>

Word-задание требует службу Samba, FQDN `br-srv.sirius-exam.org`, пять пользователей, группу `hq` и возможность аутентификации с HQ-CLI. Поэтому используется файловый сервер Samba с локальными Samba-учётными записями. Контроллер домена `au-team.irpo` из учебного PDF не добавляется: в Word-задании он не требуется.

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

Установить и подготовить Samba**BASH**

</div>

    hostnamectl hostname br-srv.sirius-exam.org
    dnf install -y samba samba-client
    groupadd hq
    mkdir -p /srv/samba/hq
    chown root:hq /srv/samba/hq
    chmod 2770 /srv/samba/hq

</div>

<div class="cmd">

<div class="cmd-title">

Создать пользователей Linux и Samba**BASH**

</div>

    for n in 1 2 3 4 5; do
      useradd -m -G hq hquser$n
      echo "P@ssw0rd" | passwd --stdin hquser$n
      (echo "P@ssw0rd"; echo "P@ssw0rd") | smbpasswd -a -s hquser$n
    done

    getent group hq
    pdbedit -L

</div>

<div class="cmd">

<div class="cmd-title">

/etc/samba/smb.conf**CONF**

</div>

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

</div>

<div class="cmd">

<div class="cmd-title">

Запуск и проверка Samba**BASH**

</div>

    testparm
    systemctl enable --now smb
    systemctl restart smb
    smbclient -L //127.0.0.1 -U hquser1

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

Проверить аутентификацию с HQ-CLI**BASH**

</div>

    dnf install -y samba-client
    smbclient //192.168.30.2/hq -U hquser1

    # Внутри smbclient:
    ls
    put /etc/hosts hosts-test.txt
    ls
    exit

</div>

<div class="note ok">

Проверка выполняется именно с HQ-CLI: пользователь группы hq вводит пароль P@ssw0rd и получает доступ к ресурсу BR-SRV.

</div>

</div>

<div id="m2-2" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 2. RAID0 на HQ-SRV

</div>

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

Найти дополнительные диски**BASH**

</div>

    dnf install -y mdadm
    lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
    # Ниже предполагаются два дополнительных диска: /dev/sdb и /dev/sdc

</div>

<div class="cmd">

<div class="cmd-title">

Создать md0 и сохранить конфигурацию**BASH**

</div>

    mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
    cat /proc/mdstat
    mdadm --detail /dev/md0
    mdadm --detail --scan > /etc/mdadm.conf
    cat /etc/mdadm.conf

</div>

<div class="cmd">

<div class="cmd-title">

Создать раздел md0p1**BASH**

</div>

    fdisk /dev/md0

    # В fdisk нажать:
    n
    p
    1


    w

    partprobe /dev/md0
    lsblk

</div>

<div class="cmd">

<div class="cmd-title">

ext4 и автомонтирование /raid**BASH**

</div>

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

</div>

<div class="note ok">

Официальная База знаний РЕД ОС рекомендует mdadm для программного RAID. В этом задании после создания массива дополнительно создаётся раздел md0p1, как требует Word-файл.

</div>

</div>

<div id="m2-3" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 3. NFS

</div>

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

Экспортировать /raid/nfs только для HQ-CLI-Net**BASH**

</div>

    dnf install -y nfs-utils
    mkdir -p /raid/nfs
    chmod 777 /raid/nfs
    nano /etc/exports

    # Вставить:
    /raid/nfs 192.168.200.0/27(rw,sync,no_subtree_check)

    systemctl enable --now nfs-server
    exportfs -rav
    exportfs -v

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

Автомонтирование /mnt/nfs**BASH**

</div>

    dnf install -y nfs-utils
    mkdir -p /mnt/nfs
    nano /etc/fstab

    # Добавить:
    192.168.100.2:/raid/nfs /mnt/nfs nfs defaults,_netdev,nofail 0 0

    systemctl daemon-reload
    mount -a
    df -h /mnt/nfs
    touch /mnt/nfs/hq-cli-test.txt
    ls -l /mnt/nfs

</div>

<div class="note info">

В отчёт занесите: экспорт **/raid/nfs**, разрешённая сеть **192.168.200.0/27**, режим **rw**, клиентская точка **/mnt/nfs**.

</div>

</div>

<div id="m2-4" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 4. Chrony на ISP

</div>

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP: NTP-сервер stratum 5**BASH**

</div>

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

</div>

#### HQ-SRV и HQ-CLI

<div class="cmd">

<div class="cmd-title">

Клиенты HQ**BASH**

</div>

    nano /etc/chrony.conf
    # Добавить:
    server 172.16.1.1 iburst

    systemctl enable --now chronyd
    systemctl restart chronyd
    chronyc sources -v

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

Клиент BR-SRV**BASH**

</div>

    nano /etc/chrony.conf
    # Добавить:
    server 172.16.2.1 iburst

    systemctl enable --now chronyd
    systemctl restart chronyd
    chronyc sources -v

</div>

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

Клиент BR-RTR**ELTEX**

</div>

    configure
    ntp enable
    ntp server 172.16.2.1
    exit
    commit
    confirm
    save
    show ntp configuration
    show ntp peers

</div>

</div>

<div id="m2-5" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 5. Docker testapp на BR-SRV

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

Установить Docker и Compose**BASH**

</div>

    dnf install -y docker-ce docker-ce-cli docker-compose
    systemctl enable docker --now
    systemctl status docker --no-pager
    docker --version
    docker-compose --version

</div>

<div class="cmd">

<div class="cmd-title">

Подключить Additional.iso и загрузить образы**BASH**

</div>

    lsblk
    mkdir -p /mnt/additional
    mount /dev/sr0 /mnt/additional
    ls -l /mnt/additional/docker
    cat /mnt/additional/docker/readme.txt

    docker load < /mnt/additional/docker/site_latest.tar
    docker load < /mnt/additional/docker/mariadb_latest.tar
    docker image ls

</div>

<div class="note warn">

После `docker image ls` проверьте фактические теги. В примере ниже используются `site:latest` и `mariadb:10.11`, как в учебном PDF.

</div>

<div class="cmd">

<div class="cmd-title">

/root/web.yaml**YAML**

</div>

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
          - "8080:8000"
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

</div>

<div class="cmd">

<div class="cmd-title">

Запуск и проверка**BASH**

</div>

    cd /root
    docker-compose -f web.yaml up -d
    docker ps
    docker logs tespapp --tail 50
    curl http://127.0.0.1:8080

</div>

<div class="note ok">

Word-файл требует имя контейнера **tespapp** - именно с буквой **s** после **te**. В Compose оно задано через `container_name: tespapp`.

</div>

</div>

<div id="m2-6" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 6. Apache и MariaDB на HQ-SRV

</div>

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

Установить и запустить службы**BASH**

</div>

    dnf install -y httpd php php-mysqlnd mariadb-server mariadb
    systemctl enable --now mariadb
    systemctl enable --now httpd
    systemctl status mariadb httpd --no-pager

</div>

<div class="cmd">

<div class="cmd-title">

Подключить Additional.iso**BASH**

</div>

    lsblk
    mkdir -p /mnt/additional
    mount /dev/sr0 /mnt/additional
    find /mnt/additional/web -maxdepth 2 -type f | sort

</div>

<div class="cmd">

<div class="cmd-title">

Создать webdb и пользователя web**SQL**

</div>

    mysql -u root

    CREATE DATABASE webdb;
    CREATE USER 'web'@'localhost' IDENTIFIED BY 'P@ssw0rd';
    GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

</div>

<div class="cmd">

<div class="cmd-title">

Импортировать приложение**BASH**

</div>

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

</div>

<div class="note info">

В отчёт занесите: Apache на порту 80, БД webdb, пользователь web, импорт dump.sql, каталог приложения /var/www/html.

</div>

</div>

<div id="m2-7" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задание 7. DNAT на Eltex

</div>

### HQ-RTR: 8080 → HQ-SRV:80 и 2026 → HQ-SRV:2026

#### HQ-RTR

<div class="cmd">

<div class="cmd-title">

HQ-RTR**ELTEX**

</div>

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

</div>

### BR-RTR: 8080 → BR-SRV:8080 и 2026 → BR-SRV:2026

#### BR-RTR

<div class="cmd">

<div class="cmd-title">

BR-RTR**ELTEX**

</div>

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

</div>

#### ISP

<div class="cmd">

<div class="cmd-title">

Проверка с ISP**BASH**

</div>

    curl http://172.16.1.2:8080
    curl http://172.16.2.2:8080
    ssh -p 2026 sshuser@172.16.1.2
    ssh -p 2026 sshuser@172.16.2.2

</div>

</div>

<div id="m2-8" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Модуль 2. Задания 8-10. Nginx, Basic Auth, Яндекс Браузер

</div>

### 8. Reverse proxy на ISP

#### ISP

<div class="cmd">

<div class="cmd-title">

Установить Nginx**BASH**

</div>

    dnf install -y nginx httpd-tools
    nano /etc/nginx/conf.d/exam.conf

</div>

<div class="cmd">

<div class="cmd-title">

/etc/nginx/conf.d/exam.conf**NGINX**

</div>

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

</div>

<div class="cmd">

<div class="cmd-title">

Проверить Nginx**BASH**

</div>

    nginx -t
    systemctl enable --now nginx
    systemctl restart nginx
    curl -H 'Host: web.sirius-exam.org' http://127.0.0.1
    curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1

</div>

### 9. Basic Auth только для web.sirius-exam.org

#### ISP

<div class="cmd">

<div class="cmd-title">

Создать /etc/nginx/.htpasswd**BASH**

</div>

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

</div>

### 10. Яндекс Браузер

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI**BASH**

</div>

    sudo dnf install -y yandex-browser-stable

    # Если пакет не найден:
    sudo dnf install -y yandex-browser-release
    sudo dnf install -y yandex-browser-stable

    rpm -qa | grep -i yandex

</div>

<div class="note ok">

Официальная инструкция РЕД ОС указывает пакет `yandex-browser-stable`; при отсутствии репозитория сначала устанавливается `yandex-browser-release`.

</div>

</div>

<div id="variant" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Вариативная часть. Nextcloud и права доступа

</div>

### Вариативный модуль 1. Nextcloud на BR-SRV

<div class="note info">

**Порт Nextcloud изменён на 8081.** Основной контейнер testapp продолжает работать на порту 8080. Команды ниже основаны на приложенной инструкции по развёртыванию Nextcloud, но адаптированы для одновременной работы двух приложений.

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

Установить совместимый Docker CLI и Docker Compose**BASH**

</div>

    sudo yum update
    sudo yum install -y podman-docker
    sudo yum install -y docker-compose

    # Проверка установки
    docker --version
    docker-compose --version

</div>

<div class="note info">

Пакет `podman-docker` предоставляет совместимый интерфейс команды `docker`. Это соответствует приложенной инструкции для РЕД ОС.

</div>

<div class="cmd">

<div class="cmd-title">

Создать рабочий каталог**BASH**

</div>

    sudo mkdir -p /opt/nextcloud
    cd /opt/nextcloud
    sudo nano docker-compose.yml

</div>

<div class="cmd">

<div class="cmd-title">

/opt/nextcloud/docker-compose.yml**YAML**

</div>

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
          - "127.0.0.1:8081:80"
        volumes:
          - ./nextcloud_data:/var/www/html
        environment:
          - MYSQL_HOST=db
          - MYSQL_USER=nextcloud
          - MYSQL_PASSWORD=NextcloudDBpass2025
          - MYSQL_DATABASE=nextcloud
        depends_on:
          - db

</div>

<div class="note warn">

**Отличие от приложенной инструкции:** локальный порт изменён с `8080` на `8081`, чтобы не останавливать testapp. Контейнер testapp остаётся доступным на 8080.

</div>

<div class="cmd">

<div class="cmd-title">

Запустить контейнеры Nextcloud**BASH**

</div>

    cd /opt/nextcloud
    sudo docker-compose up -d
    sudo docker ps

</div>

<div class="cmd">

<div class="cmd-title">

Установить Nginx и создать самоподписанный сертификат**BASH**

</div>

    sudo yum install -y nginx openssl
    sudo mkdir -p /etc/ssl/certs /etc/ssl/private

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout /etc/ssl/private/nextcloud-selfsigned.key \
        -out /etc/ssl/certs/nextcloud-selfsigned.crt \
        -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=127.0.0.1"

</div>

<div class="cmd">

<div class="cmd-title">

Создать конфигурацию Nginx**BASH**

</div>

    sudo mkdir -p /etc/nginx/sites-available
    sudo nano /etc/nginx/sites-available/nextcloud

</div>

<div class="cmd">

<div class="cmd-title">

/etc/nginx/sites-available/nextcloud**NGINX**

</div>

    server {
        listen 443 ssl;
        server_name 127.0.0.1;

        ssl_certificate /etc/ssl/certs/nextcloud-selfsigned.crt;
        ssl_certificate_key /etc/ssl/private/nextcloud-selfsigned.key;

        client_max_body_size 10G;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            add_header Strict-Transport-Security "max-age=15552000; includeSubDomains" always;

            proxy_pass http://127.0.0.1:8081;
            proxy_redirect off;
        }
    }

</div>

<div class="cmd">

<div class="cmd-title">

Создать ссылку sites-enabled и подключить каталог**BASH**

</div>

    sudo mkdir -p /etc/nginx/sites-enabled
    sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/nextcloud

    sudo nano /etc/nginx/nginx.conf
    # Внутри блока http { ... } добавить строку:
    include /etc/nginx/sites-enabled/*;

</div>

<div class="cmd">

<div class="cmd-title">

Проверить и запустить Nginx**BASH**

</div>

    sudo nginx -t
    sudo systemctl enable nginx
    sudo systemctl start nginx
    curl -kI https://127.0.0.1

</div>

<div class="note ok">

**Локальная проверка:** на BR-SRV откройте `https://127.0.0.1` или выполните `curl -kI https://127.0.0.1`. Должна открыться страница начальной настройки Nextcloud.

</div>

### Вариативный модуль 2. Пользователи и каталоги на HQ-CLI

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

Создать группы, пользователей и каталоги**BASH**

</div>

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

    ip -br a
    ls -ld /home/Folder/work_shared /home/Folder/job_readonly

</div>

### Вариативный модуль 3. Эксплуатационные проверки

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

Проверка прав**BASH**

</div>

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

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

Просмотр access.log**BASH**

</div>

    tail -f /var/log/nginx/access.log

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

Запрос к BR-SRV и проверка HSTS**BASH**

</div>

    curl -k https://192.168.30.2 > /dev/null
    curl -kI https://192.168.30.2 | grep -i strict-transport-security

    # Ожидается:
    # Strict-Transport-Security: max-age=15552000; includeSubDomains

</div>

<div class="note info">

Для удалённой проверки с HQ-CLI разрешите Nginx принимать запросы по адресу BR-SRV: при необходимости добавьте в `server_name` значение `192.168.30.2` и пересоздайте сертификат с подходящим CN или SAN. Формальная локальная проверка задания остаётся `https://127.0.0.1`.

</div>

</div>

<div id="checks" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Финальная проверка после каждого модуля

</div>

### После Модуля 1

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP**BASH**

</div>

    ip -br a
    ip r
    sysctl net.ipv4.ip_forward
    nft list ruleset
    ping -c 3 172.16.1.2
    ping -c 3 172.16.2.2
    ping -c 3 192.168.100.2
    ping -c 3 192.168.30.2

</div>

#### HQ-RTR и BR-RTR

<div class="cmd">

<div class="cmd-title">

Eltex**ELTEX**

</div>

    show ip interfaces
    show ip route
    show tunnels status gre 1
    show ip ospf neighbors
    show ip nat source rulesets
    show ip dhcp binding   # только HQ-RTR

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI**BASH**

</div>

    ip -br a
    ip r
    cat /etc/resolv.conf
    ping -c 3 192.168.100.2
    ping -c 3 192.168.30.2
    nslookup hq-srv.sirius-exam.org
    nslookup ya.ru

</div>

### После Модуля 2

#### HQ-SRV

<div class="cmd">

<div class="cmd-title">

HQ-SRV**BASH**

</div>

    cat /proc/mdstat
    mdadm --detail /dev/md0
    df -h /raid
    exportfs -v
    curl http://127.0.0.1

</div>

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

BR-SRV**BASH**

</div>

    testparm
    pdbedit -L
    docker ps
    curl http://127.0.0.1:8080
    chronyc sources -v

</div>

#### ISP

<div class="cmd">

<div class="cmd-title">

ISP**BASH**

</div>

    nginx -t
    curl -I -H 'Host: web.sirius-exam.org' http://127.0.0.1
    curl -u WEB:P@ssw0rd -H 'Host: web.sirius-exam.org' http://127.0.0.1
    curl -H 'Host: docker.sirius-exam.org' http://127.0.0.1

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI**BASH**

</div>

    df -h /mnt/nfs
    smbclient //192.168.30.2/hq -U hquser1
    rpm -qa | grep -i yandex

</div>

### После вариативной части

#### BR-SRV

<div class="cmd">

<div class="cmd-title">

BR-SRV**BASH**

</div>

    docker ps
    nginx -t
    curl -kI https://127.0.0.1
    ls -l /etc/nginx/sites-enabled/

</div>

#### HQ-CLI

<div class="cmd">

<div class="cmd-title">

HQ-CLI**BASH**

</div>

    curl -kI https://192.168.30.2 | grep -i strict-transport-security
    ls -ld /home/Folder/work_shared /home/Folder/job_readonly

</div>

</div>

<div id="sources" class="section sheet">

<div class="doc-header">

<div class="brand-text">

*Образовательные треки ООО «Ред Софт»*  
КОД-09.02.06-1-2026  
Сетевой и системный администратор

</div>

<div class="redos-logo">

**РЕДОС**

</div>

</div>

<div class="rule">

</div>

<div class="section-title">

## Официальные источники и границы проверки

</div>

Команды в runbook сверены с официальными документами и официальной Базой знаний. Генерация файла не выполняет команды на вашем реальном стенде, поэтому финальная гарантия достигается контрольным прогоном на VM.

- [Eltex ESR-Series. Справочник команд CLI. Версия 1.37](https://api.prod.eltex-co.ru/storage/upload_center/files/53/ESR-Series_CLI_1.37.pdf)
- [РЕД ОС: установка и настройка Docker](https://redos.red-soft.ru/base/redos-7_3/7_3-administation/7_3-containers/7_3-docker-install/)
- [РЕД ОС: создание программного RAID0 и RAID1](https://redos.red-soft.ru/base/redos-7_3/7_3-install/7_3-alter-install/7_3-install-on-raid/7_3-program-raid0-and-raid1/)
- [РЕД ОС: установка Яндекс Браузера](https://redos.red-soft.ru/base/redos-7_3/7_3-users-tasks/7_3-browser/7_3-yandex-browser-install/)
- [Nginx: proxy_pass и proxy_set_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Nginx: auth_basic и auth_basic_user_file](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)
- [Docker Compose documentation](https://docs.docker.com/compose/)
- [Official MySQL Docker image](https://hub.docker.com/_/mysql)
- [Official Nextcloud Docker image](https://hub.docker.com/_/nextcloud)

<div class="note warn">

**Особенно важные проверки перед сдачей:** интерфейсы ISP, наличие Additional.iso как /dev/sr0, фактические Docker-теги после docker load, а также доступность репозиториев Docker на BR-SRV.

</div>

</div>

<div class="footer">

КОД-09.02.06-1-2026 · Сетевой и системный администратор · RED OS / Eltex ESR 1.37.4

</div>
