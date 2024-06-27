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
