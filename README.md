### Задание 1
- ISP
    1. hostnamectl set-hostname isp
    2. nano /etc/net/sysctl.conf 
        1. net.ipv4.ip_forward = 1
    3. cd /etc/net/ifaces
    4. cp -r ens18/ ens19
    5. vim ens19/options
        1. BOOTPROTO=static
    6. cp -r ens19/ ens20
    7. cp -r ens19/ ens21
    8. vim ens19/ipv4address
        1. 100.100.100.1/28
    9. vim ens19/ipv4route
        1. 10.10.10.0/24 via 100.100.100.10
    10. vim ens20/ipv4address
        1. 150.150.150.1/28
    11. vim ens20/ipv4route
        1. 20.20.20.0/24 via 150.150.150.10
    12. vim ens21/ipv4address
        1. 35.35.35.1/28
    13. systemctl restart network
    14. reboot
    15. apt-get update && apt-get install nftables chrony -y
    16. vim /etc/nftables/nftables.nft
        1. в начало:
           ```
           flush ruleset;
           ```
        3. в конец:
           ```
           table ip nat {
             chain postrouting {
               type nat hook postrouting priority 0;
               oifname ens18 masquerade;
             }
           }
           ```
    17. systemctl enable --now nftables
    18. nft -f /etc/nftables/nftables.nft

- CLI
    1. hostnamectl set-hostname cli
    2. cd /etc/net/ifaces
    3. cp -r ens18/ ens19
    4. vim ens19/options
        1. BOOTPROTO=static
    5. vim ens19/ipv4address
        1. 35.35.35.10/28
    6. vim ens19/ipv4route
        1. default via 35.35.35.1
    7. systemctl restart network
    8. reboot
    9. apt-get update && apt-get install chrony yandex-browser -y

- RTR-L
    1. hostnamectl set-hostname rtr-l
    2. vim /etc/net/sysctl.conf 
        1. net.ipv4.ip_forward = 1
    3. cd /etc/net/ifaces
    4. vim ens18/options
        1. BOOTPROTO=static
    5. cp -r ens18/ ens19
    6. vim ens18/ipv4address
        1. 100.100.100.10/28
    7. vim ens18/ipv4route
        1. default via 100.100.100.1
    8. vim ens19/ipv4address
        1. 10.10.10.1/24
    9. systemctl restart network
    10. reboot
    11. apt-get update && apt-get install nftables chrony strongswan -y

- RTR-R
    1. hostnamectl set-hostname rtr-r
    2. vim /etc/net/sysctl.conf 
        1. net.ipv4.ip_forward = 1
    3. cd /etc/net/ifaces
    4. vim ens18/options
        1. BOOTPROTO=static
    5. cp -r ens18/ ens19
    6. vim ens18/ipv4address
        1. 150.150.150.10/28
    7. vim ens18/ipv4route
        1. default via 150.150.150.1
    8. vim ens19/ipv4address
        1. 20.20.20.1/24
    9. systemctl restart network
    10. reboot
    11. apt-get update && apt-get install chrony nftables strongswan -y

- WEB-L
    1. hostnamectl set-hostname web-l
    2. cd /etc/net/ifaces/ens18/
    3. vim options
        1. BOOTPROTO=static
    4. vim ipv4address
        1. 10.10.10.110/24
    5. vim ipv4route
        1. default via 10.10.10.1
    6. systemctl restart network
    7. reboot
    8. apt-get update && apt-get install chrony docker-io docker-compose nfs-clients -y

- WEB-R
    1. hostnamectl set-hostname web-r
    2. cd /etc/net/ifaces/ens18/
    3. vim options
        1. BOOTPROTO=static
    4. vim ipv4address
        1. 20.20.20.100/24
    5. vim ipv4route
        1. default via 20.20.20.1
    6. systemctl restart network
    7. reboot
    8. apt-get update && apt-get install chrony bind bind-utils nfs-clients -y

- SRV-L
    1. hostnamectl set-hostname srv-l
    2. cd /etc/net/ifaces/ens18/
    3. vim options
        1. BOOTPROTO=static
    4. vim ipv4address
        1. 10.10.10.100/24
    5. vim ipv4route
        1. default via 10.10.10.1
    6. systemctl restart network
    7. reboot
    8. apt-get update && apt-get install chrony bind bind-utils nfs-server -y


### Задание 2
- RTR-L
    1. vim /etc/nftables/nftables.nft
        1. в начало:
        ```
        flush ruleset;
        ```
        2. в конец:
        ```
        table ip nat {
          chain postrouting {
            type nat hook postrouting priority 0;
            ip saddr 10.10.10.0/24 oifname ens18 masquerade;
          }
          chain prerouting {
            type nat hook prerouting priority 0;
            tcp dport 2024 dnat to 10.10.10.110:2024;
          }
        }
        ```
  2. nft -f /etc/nftables/nftables.nft
  3. systemctl enable --now nftables
- RTR-R
    1. vim /etc/nftables/nftables.nft
        1. в начало:
        ```
        flush ruleset;
        ```
        2. в конец:
        ```
        table ip nat {
          chain postrouting {
            type nat hook postrouting priority 0;
            ip saddr 20.20.20.0/24 oifname ens18 masquerade;
          }
          chain prerouting {
            type nat hook prerouting priority 0;
            tcp dport 2024 dnat to 20.20.20.100:2024;
          }
        }
        ```
    2. nft -f /etc/nftables/nftables.nft
    3. systemctl enable --now nftables


### Задание 3
- RTR-L
    1. vim /etc/gre.up
       ```
        #!/bin/bash
        ip tunnel add tun0 mode gre local 100.100.100.10 remote 150.150.150.10
        ip addr add 10.5.5.1/30 dev tun0
        ip link set up tun0
        ip route add 20.20.20.0/24  via 10.5.5.2
       ```
    2. chmod +x /etc/gre.up
    3. /etc/gre.up
    4. vim /etc/crontab
        1. в конец добавляем:
        ```
          @reboot root /etc/gre.up
        ```
    5. vim /etc/strongswan/ipsec.conf
      1. ниже “config setup” пишем:
        ```
        conn vpn
        (следующие строки через tab)
          auto=start
          type=tunnel
          authby=secret
          left=100.100.100.10
          right=150.150.150.10
          leftsubnet=0.0.0.0/0
          rightsubnet=0.0.0.0/0
          leftprotoport=gre
          rightprotoport=gre
          ike=aes128-sha256-modp3072
          esp=aes128-sha256
        ```
    6. vim /etc/strongswan/ipsec.secrets
       ```
       100.100.100.10 150.150.150.10 : PSK “P@ssw0rd”
       ```
    7. systemctl enable --now ipsec.service

- RTR-R
    1. vim /etc/gre.up
    ```
       #!/bin/bash
       ip tunnel add tun0 mode gre local 150.150.150.10 remote 100.100.100.10
       ip addr add 10.5.5.2/30 dev tun0
       ip link set up tun0
       ip route add 10.10.10.0/24  via 10.5.5.1
    ```
    2. chmod +x /etc/gre.up
    3. /etc/gre.up
    4. vim /etc/crontab
        1. в конец добавляем:
        ```
        @reboot root /etc/gre.up
        ```
    5. vim /etc/strongswan/ipsec.conf
        1. ниже “config setup” пишем:
        ```
        conn vpn
        (следующие строки через tab)
            auto=start
            type=tunnel
            authby=secret
            left=150.150.150.10
            right=100.100.100.10
            leftsubnet=0.0.0.0/0
            rightsubnet=0.0.0.0/0
            leftprotoport=gre
            rightprotoport=gre
            ike=aes128-sha256-modp3072
            esp=aes128-sha256
        ```
    6. vim /etc/strongswan/ipsec.secrets
        1. 100.100.100.10 150.150.150.10 : PSK “P@ssw0rd”
    7. systemctl enable --now ipsec.service

### Задание 4
- WEB-L
    1. vim /etc/openssh/banner.txt
        1. Authorized access only
    2. vim /etc/openssh/sshd_config
        1. расскоментируем строчку Port 22
           пишем вместо 22 порт 2024
        2. расскоментируем строчку MaxAuthTries 6
           пишем вместо 6 попыток 2
        3. расскоментируем строчку Banner none
           вместо none пишем путь к banner.txt /etc/openssh/banner.txt
        4. добавляем в конец
           AllowUsers sshuser
    3. adduser sshuser
    4. passwd sshuser
        1. P@ssw0rd два раза
    5. systemctl restart sshd

- WEB-R
    1. vim /etc/openssh/banner.txt
        1. Authorized access only
    2. vim /etc/openssh/sshd_config
        1. расскоментируем строчку Port 22
           пишем вместо 22 порт 2024
        2. расскоментируем строчку MaxAuthTries 6
           пишем вместо 6 попыток 2
        3. расскоментируем строчку Banner none
           вместо none пишем путь к banner.txt /etc/openssh/banner.txt
        4. добавляем в конец
           AllowUsers sshuser
    3. adduser sshuser
    4. passwd sshuser
        1. P@ssw0rd два раза
    5. systemctl restart sshd

### Задание 5
- SRV-L
    1. systemctl enable --now bind
    2. vim /etc/bind/options.conf
        1. что должно быть в options:
        ```
        listen-on { any; };
        forwarders { 94.232.137.104; };
        dnssec-validation no;
        recursion yes;
        allow-query { any; };
        allow-recursion { any; };
        ```
        2. вот так
        ![image](https://github.com/rtx512/an/assets/101506362/56621db6-029d-4568-ab80-15b71960bcbb)

    3. vim /etc/bind/local.conf
        1. добавляем после слов Add other zones here:
        ```
        zone "au.team" {
                        type master;
                        file "au.team";
                        allow-transfer {20.20.20.100;};
        };
        zone "10.10.10.in-addr.arpa" {
                        type master;
                        file "left.reverse";
                        allow-transfer {20.20.20.100;};
        };
        zone "20.20.20.in-addr.arpa" {
                        type master;
                        file "right.reverse";
                        allow-transfer {20.20.20.100;};
        };
        zone "35.35.35.in-addr.arpa" {
                        type master;
                        file "cli.reverse";
                        allow-transfer {20.20.20.100;};
        };
        ```
        2. вот так
        ![image](https://github.com/rtx512/an/assets/101506362/56acf213-10d6-4668-8183-32b6396d0970)

    4. cd /etc/bind/zone/
    5. cp localhost au.team
    6. vim au.team
        1. заменяем localhost. на au.team.  и root.localhost. на root.au.team.
        2. редачим зоны, должны получиться такие зоны, пишем через табуляцию (Tab), а не пробелы:
        ```
        @        IN     NS    au.team.
        @        IN     A     10.10.10.100
        isp      IN     A     100.100.100.1
        rtr-l    IN     A     10.10.10.1
        rtr-r    IN     A     20.20.20.1
        web-l    IN     A     10.10.10.110
        web-r    IN     A     20.20.20.100
        srv-l    IN     A     10.10.10.100
        cli      IN     A     35.35.35.10
        dns      IN     CNAME srv-l
        ntp      IN     CNAME isp
        mediawiki       IN    CNAME    web-l
        ```
        3. должно получиться так
        ![image](https://github.com/rtx512/an/assets/101506362/50cc1868-6280-41c0-a2f4-d06cb3712594)

    7. cp localhost right.reverse
    8. vim right.reverse
        1. заменяем localhost. на 20.20.20.in-addr.arpa. и root.localhost. на root.20.20.20.in-addr.arpa.
        2. редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
        ```
        @        IN     NS    au.team.
        @        IN     A     20.20.20.100
        1        PTR    rtr-r.au.team.
        100      PTR    web-r.au.team.
        ```
        3. должно получиться так
        ![image](https://github.com/rtx512/an/assets/101506362/c673aada-a06f-42be-96aa-6cd4bbcaf6b8)

    9. cp right.reverse left.reverse
    10. vim left.reverse
        1. заменяем 20.20.20.in-addr.arpa. на 10.10.10.in-addr.arpa. и root.20.20.20.in-addr.arpa. на                  root.10.10.10.in-addr.arpa.
        2. редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
        ```
        @        IN     NS    au.team.
        @        IN     A     10.10.10.100
        1        PTR    rtr-l.au.team.
        100      PTR    srv-l.au.team.
        110      PTR    web-l.au.team.
        ```
        3. должно получиться так
        ![image](https://github.com/rtx512/an/assets/101506362/70ddfcfc-f5da-4f8e-a75c-9bb0607e7813)

    11. cp right.reverse cli.reverse
    12. vim cli.reverse
        1. заменяем 10.10.10.in-addr.arpa. на 35.35.35.in-addr.arpa. и root.10.10.10.in-addr.arpa. на                  root.35.35.35.in-addr.arpa.
        2. редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
        ```
        @        IN     NS    au.team.
        @        IN     A     35.35.35.1
        1        PTR    isp.au.team.
        10       PTR    cli.au.team.
        ```
        3. должно получиться вот так:
        ![image](https://github.com/rtx512/an/assets/101506362/fca96f84-50ae-445e-8c54-f1503855f10f)

    13. chmod 777 au.team
    14. chmod 777 right.reverse
    15. chmod 777 left.reverse
    16. chmod 777 cli.reverse
    17. systemctl restart bind
    18. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 127.0.0.1
        ![image](https://github.com/rtx512/an/assets/101506362/65312d9b-1f9f-42c6-9284-e1f36081a998)
- WEB-R
    1. systemctl enable --now bind
    2. vim /etc/bind/options.conf
        1. что должно быть в options:
        ```
        listen-on { any; };
        forwarders { 10.10.10.100; };
        dnssec-validation no;
        recursion yes;
        allow-query { any; };
        allow-recursion { any; };
        ```
        2. вот так:
        ![image](https://github.com/rtx512/an/assets/101506362/85f47899-3497-4c90-b68a-95e08c96939f)
    3. vim /etc/bind/local.conf
        1. добавляем после слов Add other zones here:
        ```
        zone "au.team" {
                        type slave;
                        file "slave/au.team";
                        masters {10.10.10.100;};
        };
        zone "10.10.10.in-addr.arpa" {
                        type slave;
                        file "slave/left.reverse";
                        masters {10.10.10.100;};
        };
        zone "20.20.20.in-addr.arpa" {
                        type slave;
                        file "slave/right.reverse";
                        masters {10.10.10.100;};
        };
        zone "35.35.35.in-addr.arpa" {
                        type slave;
                        file "slave/cli.reverse";
                        masters {10.10.10.100;};
        };
        ```
        2. вот так:
       
        ![image](https://github.com/rtx512/an/assets/101506362/d3bab9f7-9e1f-4172-bfa9-8250e9d9a89e)
        
     4. chown named:named /var/lib/bind/zone/slave/
     5. chown named:named /etc/bind/zone/slave/
     6. systemctl restart bind
     7. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 127.0.0.1
        ![image](https://github.com/rtx512/an/assets/101506362/0ea2ecc5-471d-4689-9d8f-fce1e73341b2)

- CLI
    1. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100
           ![image](https://github.com/rtx512/an/assets/101506362/dacb1237-876a-4547-84f9-bae50a0f9af9)

- ISP
    1. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100
           ![image](https://github.com/rtx512/an/assets/101506362/7d673db5-344f-4ed8-986b-8c8d4badd638)

- RTR-L
    1. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100
           ![image](https://github.com/rtx512/an/assets/101506362/8088bc5d-c2cf-4dd1-8390-4ff80f650406)

- RTR-R
    1. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 20.20.20.100 (если WEB-R не работает, то 10.10.10.100)
           
           ![image](https://github.com/rtx512/an/assets/101506362/8ef0000e-e0a9-4a04-a477-94ccd777d744)

- WEB-L
    1. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100
           ![image](https://github.com/rtx512/an/assets/101506362/186f78c6-e39c-4c64-a863-13f6f204b7f2)


### Задание 6
- ISP
    1. vim /etc/chrony.conf
        1. в конец пишем:
        ```
            server 127.0.0.1
            allow 100.100.100.0/28
            allow 150.150.150.0/28
            allow 35.35.35.0/28
            allow 10.10.10.0/24
            allow 20.20.20.0/24
            local stratum 5
        ```
    2. systemctl restart chronyd

- CLI
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
            server 35.35.35.1 iburst
        ```
    2. systemctl restart chronyd

- RTR-L
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
           server 100.100.100.1 iburst
        ```
    2. systemctl restart chronyd
 
- RTR-R
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
            server 150.150.150.1 iburst
        ```
    2. systemctl restart chronyd

- WEB-R
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
            server 150.150.150.1 iburst
        ```
    2. systemctl restart chronyd
 
- WEB-L
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
            server 100.100.100.1 iburst
        ```
    2. systemctl restart chronyd

- SRV-L
    1. vim /etc/chrony.conf
        1. комментируем (пишем #) перед “pool pool.ntp.org iburst”
        2. в конец пишем:
        ```
            server 100.100.100.1 iburst
        ```
    2. systemctl restart chronyd

### Задание 7
- SRV-L
    1. lsblk проверяем NAME 4 дисков размером 1G: в моем случае 4 диска размером 1 гб это sdb sdc sdd sde          ДОЛЖНО БЫТЬ 4 ДИСКА: sdb sdc sdd sde:
    ![image](https://github.com/rtx512/an/assets/101506362/1a1e22a4-db1a-4cc3-8d17-a6fa5e1b0caa)
    2. cfdisk /dev/sdb
    3. =
    ![image](https://github.com/rtx512/an/assets/101506362/4d2878ae-9d4b-4caa-b3d4-7f10c7591bb7)

    4. Enter
    5. =
    ![image](https://github.com/rtx512/an/assets/101506362/176bdc64-5812-4e28-91c8-3dd35a1bf9e4)

    6. =
    ![image](https://github.com/rtx512/an/assets/101506362/4ca3ac97-c7fa-47fe-a5d6-bbe7003cb9f8)

    7. =
    ![image](https://github.com/rtx512/an/assets/101506362/791c1bee-e0a8-4ab2-8534-87704863c382)

    8. С пункта 2 повторить действия со всеми остальными дисками (sdb, sdc, sdd, sde)
    9. mdadm --create /dev/md0 --level=5 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    10. mdadm --detail --scan --verbose | tee -a /etc/mdadm.conf
    11. mkfs.ext4 /dev/md0
    12. mkdir /raid5
    13. vim /etc/fstab
        1. добавить в конец, пишем через табуляцию, а не пробел:
           /dev/md0 /raid5 ext4 defaults 0 0
        2. так
           ![image](https://github.com/rtx512/an/assets/101506362/f6614c56-743a-43d8-b9a9-19aa244620ce)
    14. reboot
    15. mkdir /raid5/nfs
    16. chmod 777 /raid5/nfs
    17. vim /etc/exports
        1. в конец добавляем:
           ```
           /raid5/nfs 10.10.10.110(rw,sync) 20.20.20.100(rw,sync)
           ```          

- WEB-L
    1. mkdir /mnt/nfs
    2. vim /etc/fstab
        1. добавляем в конец, пишем через табуляцию, а не пробел:
           ```
           10.10.10.100:/raid5/nfs /mnt/nfs nfs rw,sync 0 0
           ```
    3. mount -av

- WEB-R
    1. mkdir /mnt/nfs
    2. vim /etc/fstab
        1. добавляем в конец, пишем через табуляцию, а не пробел:
           ```
           10.10.10.100:/raid5/nfs /mnt/nfs nfs rw,sync 0 0
           ```
    3. mount -av


### Задание 8

- WEB-L
    1. systemctl disable --now ahttpd
       systemctl disable --now alteratord
    2. vim ~/wiki.yml
        1. пишем это:
           ```
           version: '3'
           services:
               MediaWiki:
                   container_name: wiki
                   image: mediawiki
                   restart: always
                   ports:
                       - 8080:80
                   links:
                       - database
                   volumes:
                       - images:/var/www/html/images
                       # - ./LocalSettings.php:/var/www/html/LocalSettings.php
        
              database:
                  container_name: db
                  image: mysql
                  restart: always
                  environment:
                      MYSQL_DATABASE: mediawiki
                      MYSQL_USER: wiki
                      MYSQL_PASSWORD: DEP@ssw0rd
                      MYSQL_RANDOM_ROOT_PASSWORD: 'toor'
                   volumes:
                       - dbvolume:/var/lib/mysql
        
               volumes:
                   images:
                   dbvolume:
                       external: true
           ```
       3. systemctl enable --now docker
       4. docker volume create dbvolume
       5. cd ~
       6. docker-compose -f wiki.yml up -d 
       7. заходим в mozila, пишем в строке url:
          localhost:8080
          ![image](https://github.com/rtx512/an/assets/101506362/f6dd5dd1-e9bf-448f-bbd5-9571f874fc6d)
       8. жмем set up the wiki
          ![image](https://github.com/rtx512/an/assets/101506362/8d6dede0-2356-4ebc-9e67-691b5e59d9f3)
       9. далее
           ![image](https://github.com/rtx512/an/assets/101506362/a4588519-0e87-449e-98d8-287ec3bae455)
       10. далее
           ![image](https://github.com/rtx512/an/assets/101506362/a1f0d7e5-c13c-4140-806e-f8954fe79c6d)
       11. Пароль: DEP@ssw0rd
           ![image](https://github.com/rtx512/an/assets/101506362/2545db74-165e-4c5e-9535-6ae388843f8b)
       12. Далее
           ![image](https://github.com/rtx512/an/assets/101506362/75afca69-2bdc-40c6-9c02-f65c63ae3d1c)
       13. =
           ![image](https://github.com/rtx512/an/assets/101506362/5f9b1820-a469-412d-8fcb-1a6736f92f28)
       14. Пароль: DEP@ssw0rd почту не указываем
           ![image](https://github.com/rtx512/an/assets/101506362/615047be-2977-4b8e-8b01-7ca182450ede)
       15. Далее
           ![image](https://github.com/rtx512/an/assets/101506362/5112b132-b31e-4ac8-90bd-0d860c97be3f)
       16. Жмем до конца далее и скачается файл, надо найти куда этот файл скачался, скорее всего вот сюда             /home/user/Загрузки/
           ![image](https://github.com/rtx512/an/assets/101506362/aa7b1041-a44d-480c-b8f5-fd02d061c2c9)
       17. Копируем скачанный файл:
           ```
           cp /home/user/Загрузки/LocalSettings.php ~/LocalSettings.php
           ```
       18. vim ~/wiki.yml
               1. расскоментируем 
                   - ./LocalSettings.php:/var/www/html/LocalSettings.php
       19. vim ~/LocalSettings.php
           1. $wgServer = “http://mediawiki.au.team:8080”
       20. docker-compose -f wiki.yml stop
       21. docker-compose -f wiki.yml up -d

- WEB-R
    1. systemctl disable --now ahttpd
       systemctl disable --now alteratord

### Задание 9
- CLI
    1. apt-get install yandex-browser -y
    2. запустить НЕ от рута с помощью команды:
       yandex-browser-stable
       запустить от рута с помощью команды:
       yandex-browser-stable --no-sandbox

### Подсказачка: 
Если DNS сервер не работает, systemctl status bind выдает ошибки, надо systemctl restart bind на DNS сервере









