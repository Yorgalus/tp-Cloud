

Faire la cinf reseau:

Vm avec 1er adaptateur en NAT et second en reseaux privé hote "TP_Leo"
Sur la VM modifier fichier de conf de la troisieme carte réseau :
```
DEVICE=enp0s8
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.3.1.10
NETMASK=255.255.255.0

```
Et modifier en fonction des machines:
kvm1.one: 10.3.1.11
kvm2.one: 10.3.1.12
frontend.one: 10.3.1.10



kvm1.one vers frontend.one
```
[rootlefrei-ngtagaul keyket ]# ping frontend.one
PING frontend.one (10.3.1.10) 56(84) bytes of data.
64 bytes from frontend one (10.3.1.10): imp_seq=1 ttl=64 time=1.38 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=2 ttl=64 time=0.977 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=3 ttl=64 time-0.841 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=4 ttl=64 time=1.03 ms
--- frontend.one ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/aug/max/mdev = 0.841/1.056/1.383/0.200 ms
```


kvm2.one vers frontend.one
```
[root@efrei-xmg1agau1 keyket]# ping frontend.one
PING frontend.one (10.3.1.10) 56(84) bytes of data.
64 bytes from frontend.one (10.3.1.10): icmp_seq=1 tt1=64 time=1.18 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=2 tt1=64 time=0.895 ms
64 bytes from frontend.one (10.3.1.10): icmp_seq=3 ttl=64 time=0.905 ms
64
bytes from frontend.one (10.3.1.10): icmp_seq=4
ttl=64 time=0.894 ms
-- frontend.one ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/aug/max/mdev = 0.894/B.967/1.176/0.120 ms
```

