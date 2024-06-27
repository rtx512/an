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
    7. vim /etc/strongswan/ipsec.secrets
       ```
       100.100.100.10 150.150.150.10 : PSK “P@ssw0rd”
       ```
    8. systemctl enable --now ipsec.service
