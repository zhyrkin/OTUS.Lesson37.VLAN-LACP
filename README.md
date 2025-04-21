# OTUS.Lesson37.VLAN-LACP
Тестовый стенд:
7 VM (2CPU,2GB RAM, 8GB HDD, 5 OS Centos7 и 2 Ubuntu22.04)

Домашнее задание: 

1. в Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами в internal сети testLAN:

testClient1 - 10.10.10.254
testClient2 - 10.10.10.254
testServer1- 10.10.10.1
testServer2- 10.10.10.1
Равести вланами:
testClient1 <-> testServer1
testClient2 <-> testServer2

2. Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов

Домашнее задание:
Запускаем playbook vlan.yml. 
После выполнения смотрим результаты: 
1) Проверяем связанность testClient1 <-> testServer1  и testClient2 <-> testServer2
На серверах запустим TCPdump и будем пинговать их с клиентов
```
[root@testClient1 ~]# ping 10.10.10.1 -c2
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.656 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.481 ms

--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.481/0.568/0.656/0.090 ms

```
мы видим что пакеты приходят на Server1
```
[root@testServer1 ~]# tcpdump -ni any icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
11:51:14.740260 ethertype IPv4, IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 23267, seq 1, length 64
11:51:14.740260 IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 23267, seq 1, length 64
11:51:14.740324 IP 10.10.10.1 > 10.10.10.254: ICMP echo reply, id 23267, seq 1, length 64
11:51:15.740662 ethertype IPv4, IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 23267, seq 2, length 64
11:51:15.740662 IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 23267, seq 2, length 64
11:51:15.740717 IP 10.10.10.1 > 10.10.10.254: ICMP echo reply, id 23267, seq 2, length 64
```
так же пингуем с client2 до Server2
```
root@testClient2:~# ping 10.10.10.1 -c 2
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.544 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.550 ms

--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1029ms
rtt min/avg/max/mdev = 0.544/0.547/0.550/0.003 ms
```
```
root@testServer2:~# tcpdump -ni any icmp
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
12:10:50.979834 ens19 In  IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 2, seq 1, length 64
12:10:50.979834 vlan2 In  IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 2, seq 1, length 64
12:10:50.979906 vlan2 Out IP 10.10.10.1 > 10.10.10.254: ICMP echo reply, id 2, seq 1, length 64
12:10:52.008786 ens19 In  IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 2, seq 2, length 64
12:10:52.008786 vlan2 In  IP 10.10.10.254 > 10.10.10.1: ICMP echo request, id 2, seq 2, length 64
12:10:52.008871 vlan2 Out IP 10.10.10.1 > 10.10.10.254: ICMP echo reply, id 2, seq 2, length 64
```

2. Пингуем от CentralRouter InetRouter и на InetRouter выключим интерфейс eth1. Убедимся что пинги не пропадут
```
[root@inetrouter ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether bc:24:11:e5:63:7e brd ff:ff:ff:ff:ff:ff
    inet 10.200.3.221/24 brd 10.200.3.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::be24:11ff:fee5:637e/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN group default qlen 1000
    link/ether bc:24:11:8c:05:5f brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether bc:24:11:75:61:3f brd ff:ff:ff:ff:ff:ff
5: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether bc:24:11:75:61:3f brd ff:ff:ff:ff:ff:ff
    inet 192.168.255.1/30 brd 192.168.255.3 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::be24:11ff:fe8c:55f/64 scope link 
       valid_lft forever preferred_lft forever
```
```
[root@centralrouter ~]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=1.04 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.563 ms
64 bytes from 192.168.255.1: icmp_seq=3 ttl=64 time=0.543 ms
64 bytes from 192.168.255.1: icmp_seq=4 ttl=64 time=0.561 ms
64 bytes from 192.168.255.1: icmp_seq=5 ttl=64 time=0.351 ms
64 bytes from 192.168.255.1: icmp_seq=6 ttl=64 time=0.583 ms
64 bytes from 192.168.255.1: icmp_seq=7 ttl=64 time=0.480 ms
64 bytes from 192.168.255.1: icmp_seq=8 ttl=64 time=0.490 ms
64 bytes from 192.168.255.1: icmp_seq=9 ttl=64 time=0.365 ms
64 bytes from 192.168.255.1: icmp_seq=10 ttl=64 time=0.828 ms
64 bytes from 192.168.255.1: icmp_seq=11 ttl=64 time=0.509 ms
64 bytes from 192.168.255.1: icmp_seq=12 ttl=64 time=0.511 ms
^C
--- 192.168.255.1 ping statistics ---
12 packets transmitted, 12 received, 0% packet loss, time 11003ms
rtt min/avg/max/mdev = 0.351/0.568/1.041/0.185 ms

```