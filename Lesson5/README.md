### Практическое задание к уроку 4


#### 1. На сервере server3 добавить еще один интерфейс — dummy с IP-адресом 33.33.33.33/32.

##### 1.1 Файл /etc/modprobe.d/dummy.conf

    options dummy numdummies=2

##### 1.2 Файл /etc/sysconfig/network-scripts/ifcfg-dummy1


    DEVICE=dummy1
    IPADDR=33.33.33.33
    PREFIX=32
    NM_CONTROLLED=no
    ONBOOT=yes

#### 2. НЕ анонсировать этот интерфейс в OSPF.

Действий не требуется.


#### 3. Поднять openvpn-сервер на server3 и обеспечить возможность подключения клиента server1, используя сертификаты.

##### 3.1 Файл /etc/openvpn/easy-rsa/3/vars
    

    set_var EASYRSA                 "$PWD"
    set_var EASYRSA_PKI             "$EASYRSA/pki"
    set_var EASYRSA_DN              "cn_only"
    set_var EASYRSA_REQ_COUNTRY     "RU"
    set_var EASYRSA_REQ_PROVINCE    "Irkutsk"
    set_var EASYRSA_REQ_CITY        "Irkutsk"
    set_var EASYRSA_REQ_ORG         "EXAMPLE CERTIFICATE AUTHORITY"
    set_var EASYRSA_REQ_EMAIL       "openvpn@example.com"
    set_var EASYRSA_REQ_OU          "Example.com EASY CA"
    set_var EASYRSA_KEY_SIZE        2048
    set_var EASYRSA_ALGO            rsa
    set_var EASYRSA_CA_EXPIRE       7500
    set_var EASYRSA_CERT_EXPIRE     365
    set_var EASYRSA_NS_SUPPORT      "no"
    set_var EASYRSA_NS_COMMENT      "EXAMPLE CERTIFICATE AUTHORITY"
    set_var EASYRSA_EXT_DIR         "$EASYRSA/x509-types"
    set_var EASYRSA_SSL_CONF        "$EASYRSA/openssl-1.0.cnf"
    set_var EASYRSA_DIGEST          "sha256"


##### 3.2 Создание инфраструктуры PKI

    
    ./easyrsa init-pki
    ./easyrsa build-ca nopass


##### 3.2 Создание запроса на сертификат для server3


    ./easyrsa gen-req server3 nopass


##### 3.3 Создание (подписание) сертификата для server3


    ./easyrsa sign-req server server3


##### 3.4 Создание запроса на сертификат для клиента (server1)


    ./easyrsa gen-req server1 nopass


##### 3.5 Создание (подписание) сертификата для клиента (server1)


    ./easyrsa gen-req server1 nopass


##### 3.6 Создание Diffie-Hellman-ключа


    ./easyrsa gen-dh

#### 3.7 Структура папок с ключами и сертификатами


    .
    ├── client
    │   ├── ca.crt
    │   ├── server1.crt
    │   └── server1.key
    ├── easy-rsa
    │   ├── 3 -> 3.0.8
    │   ├── 3.0 -> 3.0.8
    │   └── 3.0.8
    │       ├── easyrsa
    │       ├── openssl-1.0.cnf
    │       ├── openssl-easyrsa.cnf
    │       ├── pki
    │       │   ├── ca.crt
    │       │   ├── certs_by_serial
    │       │   │   ├── D0985CB1A09EAA32200FC5150DB5ED8E.pem
    │       │   │   └── ECB465133AA80912D2E8A8E0BCF221F8.pem
    │       │   ├── dh.pem
    │       │   ├── index.txt
    │       │   ├── index.txt.attr
    │       │   ├── index.txt.attr.old
    │       │   ├── index.txt.old
    │       │   ├── issued
    │       │   │   ├── server1.crt
    │       │   │   └── server3.crt
    │       │   ├── private
    │       │   │   ├── ca.key
    │       │   │   ├── server1.key
    │       │   │   └── server3.key
    │       │   ├── renewed
    │       │   │   ├── certs_by_serial
    │       │   │   ├── private_by_serial
    │       │   │   └── reqs_by_serial
    │       │   ├── reqs
    │       │   │   ├── server1.req
    │       │   │   └── server3.req
    │       │   ├── revoked
    │       │   │   ├── certs_by_serial
    │       │   │   ├── private_by_serial
    │       │   │   └── reqs_by_serial
    │       │   ├── safessl-easyrsa.cnf
    │       │   ├── serial
    │       │   └── serial.old
    │       ├── vars
    │       └── x509-types
    │           ├── ca
    │           ├── client
    │           ├── code-signing
    │           ├── COMMON
    │           ├── email
    │           ├── kdc
    │           ├── server
    │           └── serverClient
    └── server
        ├── ca.crt
        ├── dh.pem
        ├── server3.crt
        └── server3.key


#### 3.8 Файл /etc/openvpn/server.conf


    # OpenVPN Port, Protocol and the Tun
    port 1194
    proto udp
    dev tun

    # OpenVPN Server Certificate - CA, server key and certificate
    ca /etc/openvpn/server/ca.crt
    cert /etc/openvpn/server/server3.crt
    key /etc/openvpn/server/server3.key

    #DH key
    dh /etc/openvpn/server/dh.pem

    # Network Configuration - Internal network
    server 192.168.0.0 255.255.255.0
    push "route 33.33.33.33 255.255.255.255"

    #Enable multiple client to connect with same Certificate key
    duplicate-cn

    # TLS Security
    cipher AES-256-CBC
    tls-version-min 1.2
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256
    auth SHA512
    auth-nocache

    # Other Configuration
    keepalive 20 60
    persist-key
    persist-tun
    comp-lzo yes
    daemon
    user nobody
    group nobody

    # OpenVPN Log
    log-append /var/log/openvpn.log
    verb 3



#### 3.9 Файл /etc/openvpn/client/server1.ovpn


    client
    dev tun
    proto udp

    remote 192.168.23.1 1194 # IP адрес сервера

    ca ca.crt
    cert server1.crt
    key server1.key

    cipher AES-256-CBC
    auth SHA512
    auth-nocache
    tls-version-min 1.2
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256

    resolv-retry infinite
    compress lzo
    nobind
    persist-key
    persist-tun
    mute-replay-warnings
    verb 3


#### 3.11 Правила firewalld на Server3


    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: ens33 ens34
      sources: 
      services: ssh dhcpv6-client dns openvpn
      ports: 3260/tcp
      protocols: ospf
      masquerade: no
      forward-ports: 
      source-ports: 
      icmp-blocks: 
      rich rules: 

#### 3.12 Результат запуска openvpn --config server1.ovpn


**Состояние интерфейса tun0 на клиенте**


    26: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/none 
        inet 192.168.0.6 peer 192.168.0.5/32 scope global tun0
        valid_lft forever preferred_lft forever
        inet6 fe80::fe44:a835:2184:864d/64 scope link flags 800 
        valid_lft forever preferred_lft fore


**Маршруты на клиенте**


    default via 192.168.1.254 dev ens33 proto static metric 100 
    2.2.2.2 via 192.168.12.2 dev team0 proto 188 metric 20 
    3.3.3.3 via 192.168.12.2 dev team0 proto 188 metric 20 
    4.4.4.4 via 192.168.12.2 dev team0 proto dhcp metric 350 
    5.5.5.0/24 via 192.168.12.2 dev team0 proto dhcp metric 350 
    33.33.33.33 via 192.168.0.5 dev tun0 
    169.254.0.0/16 dev dummy0 scope link metric 1002 
    192.168.0.1 via 192.168.0.5 dev tun0 
    192.168.0.5 dev tun0 proto kernel scope link src 192.168.0.6 
    192.168.1.0/24 dev ens33 proto kernel scope link src 192.168.1.111 metric 100 
    192.168.12.0/24 dev team0 proto kernel scope link src 192.168.12.1 metric 350 
    192.168.23.0/24 via 192.168.12.2 dev team0 proto 188 metric 20 


#### 4. Убедиться, что server1 может пропинговать 33.33.33.33, когда VPN подключен, и не может этого сделать, когда VPN не подключен.


##### 4.1 Результат ping при подключенном VPN

**PING** 


    PING 33.33.33.33 (33.33.33.33) 56(84) bytes of data.
    64 bytes from 33.33.33.33: icmp_seq=1 ttl=64 time=0.829 ms
    64 bytes from 33.33.33.33: icmp_seq=2 ttl=64 time=0.957 ms
    64 bytes from 33.33.33.33: icmp_seq=3 ttl=64 time=0.842 ms
    64 bytes from 33.33.33.33: icmp_seq=4 ttl=64 time=0.742 ms
    64 bytes from 33.33.33.33: icmp_seq=5 ttl=64 time=0.776 ms
    64 bytes from 33.33.33.33: icmp_seq=6 ttl=64 time=0.776 ms
    ^C
    --- 33.33.33.33 ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5002ms
    rtt min/avg/max/mdev = 0.742/0.820/0.957/0.073 ms


**TRACEPATH**


    tracepath 33.33.33.33 -n
    1?: [LOCALHOST]                                         pmtu 1500
    1:  33.33.33.33                                           0.882ms reached
    1:  33.33.33.33                                           0.553ms reached

##### 4.2 Результат ping при отключеном VPN


**PING** 


    PING 33.33.33.33 (33.33.33.33) 56(84) bytes of data.
    ^C
    --- 33.33.33.33 ping statistics ---
    9 packets transmitted, 0 received, 100% packet loss, time 8000ms


**TRACEPATH**


    tracepath 33.33.33.33 -n
    1?: [LOCALHOST]                                         pmtu 1500
    1:  192.168.1.254                                         0.648ms 
    1:  192.168.1.254                                         0.406ms 
    2:  192.168.1.254                                         0.447ms pmtu 1492
    2:  no reply
    3:  192.168.255.5                                         2.140ms 
    4:  188.43.24.66                                          3.146ms !N