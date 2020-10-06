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

## 4. При помощи DHCP выдать серверу Server1 2 статических маршрута 4.4.4.4/32 и 5.5.5.0/24 с next hop интерфейса team0 на сервере server2

#### Файл /etc/dhcp/dhcpd.conf

        subnet 192.168.12.0 netmask 255.255.255.0 {
            range 192.168.12.10 192.168.12.20;
            option domain-name-servers 3.3.3.3;
            option classless-static-routes 32.4.4.4.4 192.168.12.2, 24.5.5.5 192.168.12.2;
            default-lease-time 600;
            max-lease-time 7200;
        }

5. Настроить DNS-сервер для зоны example.com на сервере server3. Создать прямую и обратную зоны, а также несколько записей с разными RR. Убедиться, что только запросы на IP-адрес 3.3.3.3 будут обслуживаться этим DNS-сервером.
6. Настроить фаерволл на серверах server2 и server3, чтобы разрешить только соответствующие запросы (DHCP/DNS).
7. * Настроить slave для DNS-сервера server3. Убедиться, что репликация записей происходит.
