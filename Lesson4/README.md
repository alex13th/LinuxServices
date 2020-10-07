## Практическое задание к уроку 4

### 1. **(ВЫПОЛНЕНО)** Настроить nic teaming между двумя интерфейсами — server1 и server2. Подсеть 192.168.12.0/24 будет находиться теперь на team0-интерфейсе.

#### 1.1 Настройка Server1
##### 1.1.1 Файл /etc/sysconfig/network-scripts/ifcfg-ens34
        DEVICETYPE=TeamPort
        HWADDR=00:0c:29:fc:b1:2c
        DEVICE=ens34
        ONBOOT=yes
        TEAM_MASTER=team0
        TEAM_PORT_CONFIG='{"prio": 100}'

##### 1.1.2 Файл /etc/sysconfig/network-scripts/ifcfg-ens35
        DEVICETYPE=TeamPort
        HWADDR=00:0c:29:fc:b1:36
        DEVICE=ens35
        ONBOOT=yes
        TEAM_MASTER=team0
        TEAM_PORT_CONFIG='{"prio": 100}'

##### 1.1.3 Файл /etc/sysconfig/network-scripts/ifcfg-team0
        DEVICE=team0
        DEVICETYPE=Team
        ONBOOT=yes
        BOOTPROTO=dhcp
        TEAM_CONFIG='{"runner": {"name": "activebackup"}, "link_watch": {"name": "ethtool"}}'

#### 1.2 Настройка Server1
##### 1.2.1 Файл /etc/sysconfig/network-scripts/ifcfg-ens34
        DEVICETYPE=TeamPort
        HWADDR=00:0c:29:1c:81:bc
        DEVICE=ens34
        ONBOOT=yes
        TEAM_MASTER=team0
        TEAM_PORT_CONFIG='{"prio": 100}'

##### 1.2.2 Файл /etc/sysconfig/network-scripts/ifcfg-ens35
    DEVICETYPE=TeamPort
    HWADDR=00:0c:29:1c:81:c6
    DEVICE=ens35
    ONBOOT=yes
    TEAM_MASTER=team0
    TEAM_PORT_CONFIG='{"prio": 100}'

##### 1.2.3 Файл /etc/sysconfig/network-scripts/ifcfg-team0
    DEVICE=team0
    DEVICETYPE=Team
    ONBOOT=yes
    BOOTPROTO=none
    TEAM_CONFIG='{"runner": {"name": "activebackup"}, "link_watch": {"name": "ethtool"}}'

### 2. **(ВЫПОЛНЕНО)** На интерфейсе team0 сервера server2 назначить статический IP из подсети 192.168.12.0/24. 
#### Файл /etc/sysconfig/network-scripts/ifcfg-team0
    DEVICE=team0
    DEVICETYPE=Team
    ONBOOT=yes
    BOOTPROTO=none
    IPADDR=192.168.12.2
    PREFIX=24
    TEAM_CONFIG='{"runner": {"name": "activebackup"}, "link_watch": {"name": "ethtool"}}'

### 3. **(ВЫПОЛНЕНО)** На сервере server2 настроить DHCP-сервер для выдачи динамического IP-адреса интерфейсу team0 сервера server1, а также IP-адрес DNS-сервера 3.3.3.3.

#### Файл /etc/dhcp/dhcpd.conf

        subnet 192.168.12.0 netmask 255.255.255.0 {
        range 192.168.12.10 192.168.12.20;
        option domain-name-servers 3.3.3.3;
        option domain-name "example.com";
        default-lease-time 600;
        max-lease-time 7200;
        }

## 4. **(ВЫПОЛНЕНО)** При помощи DHCP выдать серверу Server1 2 статических маршрута 4.4.4.4/32 и 5.5.5.0/24 с next hop интерфейса team0 на сервере server2

#### Файл /etc/dhcp/dhcpd.conf

        subnet 192.168.12.0 netmask 255.255.255.0 {
            range 192.168.12.10 192.168.12.20;
            option domain-name-servers 3.3.3.3;
            option domain-name "example.com";
            option classless-static-routes 32.4.4.4.4 192.168.12.2, 24.5.5.5 192.168.12.2;
            default-lease-time 600;
            max-lease-time 7200;
        }

## 5. **(ВЫПОЛНЕНО)** Настроить DNS-сервер для зоны example.com на сервере server3. Создать прямую и обратную зоны, а также несколько записей с разными RR. Убедиться, что только запросы на IP-адрес 3.3.3.3 будут обслуживаться этим DNS-сервером.

### 5.1 Ключевые параметры файла /etc/named.conf

        options {
                listen-on port 53 { 127.0.0.1; 3.3.3.3; };
                allow-query     { localhost; 192.168.0.0/16; };
        };

        include "/etc/named.conf.local";

### 5.2 Файл /etc/named.conf.local

        zone "example.com" {
                type master;
                file "/etc/named/zones/db.example.com";
        };
        zone "12.168.192.in-addr.arpa" {
                type master;
                file "/etc/named/zones/db.12.168.192.in-addr.arpa";
        };
        zone "23.168.192.in-addr.arpa" {
                type master;
                file "/etc/named/zones/db.23.168.192.in-addr.arpa";
        };
        zone "1.1.1.in-addr.arpa" {
                type master;
                file "/etc/named/zones/db.1.1.1.in-addr.arpa";     
        };
        zone "2.2.2.in-addr.arpa" {
                type master;
                file "/etc/named/zones/db.2.2.2.in-addr.arpa";
        };
        zone "3.3.3.in-addr.arpa" {
                type master;
                file "/etc/named/zones/db.3.3.3.in-addr.arpa";
        };

### 5.2 Файл /etc/named/zones/db.example.com

        $TTL    604800
        @       IN SOA ns1.example.com. postmaster.example.com. (
                2451
                604800
                86400
                2419200
                604800)

                IN      NS      ns1.example.com.

        ns1.example.com.        IN      A       3.3.3.3

        server1.example.com.    IN      A       192.168.12.15
        server1.example.com.    IN      A       1.1.1.1
        server2.example.com.    IN      A       192.168.12.2
        server2.example.com.    IN      A       192.168.23.2
        server2.example.com.    IN      A       2.2.2.2
        server3.example.com.    IN      A       192.168.23.1
        server3.example.com.    IN      A       3.3.3.3
### 5.3 Файл /etc/named/zones/db.12.168.192.in-addr.arpa

        $TTL    604800
        @       IN      SOA     example.com.    postmaster@example.com. (
                2714652
                604800
                86400
                2419200
                604800)

                IN      NS      ns1.example.com.
        15      IN      PTR     server1.example.com.
        2       IN      PTR     server2.example.com.

### 5.4 Файл /etc/named/zones/db.23.168.192.in-addr.arpa

        $TTL    604800
        @       IN      SOA     example.com.    postmaster@example.com. (
                2714652
                604800
                86400
                2419200
                604800)

                IN      NS      ns1.example.com.
        1       IN      PTR     server3.example.com.
        2       IN      PTR     server2.example.com.

### 5.5 Файл /etc/named/zones/db.1.1.1.in-addr.arpa

        $TTL    604800
        @       IN      SOA     example.com.    postmaster@example.com. (
                2714652
                604800
                86400
                2419200
                604800)

                IN      NS      ns1.example.com.
        1       IN      PTR     server1.example.com.

\* Остальные не стал выкладывать.

## 6. **(ВЫПОЛНЕНО)** Настроить фаерволл на серверах server2 и server3, чтобы разрешить только соответствующие запросы (DHCP/DNS).

### 6.1 Правила firewalld на Server2

        [root@server2 ~]# firewall-cmd --list-all
        public (active)
        target: default
        icmp-block-inversion: no
        interfaces: ens36 ens34 ens35 team0 dummy0 ens33
        sources:
        services: ssh dhcpv6-client
        ports:
        protocols: ospf
        masquerade: no
        forward-ports:
        source-ports:
        icmp-blocks:

### 6.2 Правила firewalld на Server3

        [root@server3 ~]# firewall-cmd --list-all
        public (active)
        target: default
        icmp-block-inversion: no
        interfaces: ens34 dummy0 ens33
        sources:
        services: ssh dhcpv6-client dns
        ports: 3260/tcp
        protocols: ospf
        masquerade: no
        forward-ports:
        source-ports:
        icmp-blocks:
        rich rules:

\* Также добавлены правила для работоспособности OSPF и ISCSI


\* На Server2 добавлены правила direct для пропуска пакетов между с интерфейса team0 на интерфейс ens36 и обрабтно.

## 7. * Настроить slave для DNS-сервера server3. Убедиться, что репликация записей происходит.
