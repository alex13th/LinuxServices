## Практическое задание к уроку 8


### (ВЫПОЛНЕНО) 1. Развернуть 3 виртуальные машины и настроить HA-кластер на них.

#### 1.1 Установка компонент

    yum install pcs fence-agents-all -y
    systemctl enable pcsd
    systemctl start pcsd
    systemctl status pcsd

#### 1.2 Настройка кластера


    firewall-cmd --permanent --add-service=high-availability
    firewall-cmd --reload

    pcs cluster auth 192.168.43.201 192.168.43.202 192.168.43.203
    pcs cluster setup --start --name mycluster 192.168.43.201 192.168.43.202 192.168.43.203
    pcs cluster enable --all

#### 1.3 Состояние кластера


    [root@r1 ~]# pcs cluster status
    Cluster Status:
     Stack: corosync
     Current DC: r1 (version 1.1.21-4.el7-f14e36fd43) - partition with quorum
     Last updated: Wed Oct 21 04:12:38 2020
     Last change: Wed Oct 21 04:12:31 2020 by hacluster via crmd on r1
     3 nodes configured
     0 resources configured

    PCSD Status:
     r2: Online
     r3: Online
     r1: Online

    [root@r1 ~]# corosync-quorumtool

    Quorum information
    ------------------
    Date:             Wed Oct 21 04:14:54 2020
    Quorum provider:  corosync_votequorum
    Nodes:            3
    Node ID:          1
    Ring ID:          1/31
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   3
    Highest expected: 3
    Total votes:      3
    Quorum:           2
    Flags:            Quorate

    Membership information
    ----------------------
        Nodeid      Votes Name
            1          1 r1 (local)
            2          1 r2
            3          1 r3


### (ВЫПОЛНЕНО) 2. Развернуть на этом кластере высокодоступный веб-сервер Apache. 

#### 2.1 Установка пакета httpd на каждом сервере.

    yum install httpd -y
    firewall-cmd --permanent --add-service=http
    firewall-cmd --reload

#### 2.2 Отключение STONITH (из-за несовершенства тестовой среды).

    pcs property set stonith-enabled=false
    pcs property set no-quorum-policy=ignore

#### 2.3 Создание ресурса виртуального IP привязанного к интерфейсу ens160.

    pcs resource create VirtualIP IPaddr2 ip=192.168.43.210 cidr_netmask=24 nic=ens160 op monitor interval=30s

\* Не хватило времени сделать по феншую через интерфейс **dummy**, но я помню, что так бы было правильней.

#### 2.4 Создание ресурса веб-сервера.

    pcs resource create WebServer ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://127.0.0.1/server-status" op monitor interval=20s

#### 2.5 Настройка ограничения для запуска обоих ресурсов вместе.

    pcs constraint colocation add WebServer VirtualIP INFINITY

#### 2.6 Проверка состояния ресурсов кластера

#### 2.6.1 Проверка состояния ресурсов кластера с включенной нодой r1

    [root@r1 ~]# pcs status
        Cluster name: apachecluster
        Stack: corosync
        Current DC: r1 (version 1.1.21-4.el7-f14e36fd43) - partition with quorum
        Last updated: Wed Oct 21 04:41:57 2020
        Last change: Wed Oct 21 04:30:31 2020 by root via cibadmin on r1

        3 nodes configured
        2 resources configured

        Online: [ r1 r2 r3 ]

        Full list of resources:

         VirtualIP      (ocf::heartbeat:IPaddr2):       Started r1
         WebServer      (ocf::heartbeat:apache):        Started r1

        Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled

#### 2.6.2 Проверка состояния ресурсов кластера с выключенной нодой r1

    [root@r3 ~]# pcs status
        Cluster name: apachecluster
        Stack: corosync
        Current DC: r2 (version 1.1.21-4.el7-f14e36fd43) - partition with quorum
        Last updated: Wed Oct 21 04:42:57 2020
        Last change: Wed Oct 21 04:30:31 2020 by root via cibadmin on r1

        3 nodes configured
        2 resources configured

        Online: [ r2 r3 ]
        OFFLINE: [ r1 ]

        Full list of resources:

         VirtualIP      (ocf::heartbeat:IPaddr2):       Started r2
         WebServer      (ocf::heartbeat:apache):        Started r2

        Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
