## Практическое задание к уроку 7

### 1. (ВЫПОЛНЕНО) На сервере Server1 установить ipvsadm.
### 2. (ВЫПОЛНЕНО) На сервере Server1 установить docker и запустить 2 контейнера с Nginx.
### 3. Создать интерфейс dummy2 и назначить ему IP-адрес 111.111.111.111/32.

#### Файл /etc/sysconfig/network-scripts/ifcfg-dummy1

    DEVICE=dummy1
    IPADDR=111.111.111.111
    PREFIX=32
    NM_CONTROLLED=no
    ONBOOT=yes


### 4. Проанонсировать этот IP в сеть из трёх серверов.

    router ospf
    network 1.1.1.1/32 area 0
    network 111.111.111.111/32 area 0
    network 192.168.12.0/24 area 0
    !


### 5. Настроить ipvs для передачи http запросов с сервера server3 в оба контейнера в Nginx используя механизм балансировки round-robin.


    ipvsadm -A -t 111.111.111.111:80 -s rr
    ipvsadm -a -t 111.111.111.111:80 -r 172.17.0.2 -m
    ipvsadm -a -t 111.111.111.111:80 -r 172.17.0.3 -m

### 6.Удостовериться, что запросы балансируются между контейнерами путем проверки acсess.log внутри каждого из них.


#### 6.1. Результат curl 111.111.111.111 на server3

    [root@server3 ~]# curl 111.111.111.111
    This is B
    [root@server3 ~]# curl 111.111.111.111
    This is A
    [root@server3 ~]# curl 111.111.111.111
    This is B
    [root@server3 ~]# curl 111.111.111.111
    This is A

#### 6.2 Содержание access.log контейнера nginx-A

    172.17.0.1 - - [20/Oct/2020:10:38:11 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"
    192.168.23.1 - - [20/Oct/2020:10:43:22 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"
    192.168.23.1 - - [20/Oct/2020:10:43:23 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"

#### 6.3 Содержание access.log контейнера nginx-B

    172.17.0.1 - - [20/Oct/2020:10:38:15 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"
    192.168.23.1 - - [20/Oct/2020:10:43:20 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"
    192.168.23.1 - - [20/Oct/2020:10:43:23 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/7.29.0" "-"