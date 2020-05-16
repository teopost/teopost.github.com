+++
banner = "rete-non-funzionante-con-centos-su-virtual-box/rete-non-funzionante-con-centos-su-virtual-box.png"
categories = ["work"]
date = "2017-12-28T21:36:13+01:00"
description = ""
images = []
menu = ""
tags = ["linux"]
title = "Rete non funzionante con CentOS su VirtualBox"

+++

Ho scaricato da [osboxes.org](http://www.osboxes.org/) una VM con dentro CentoOS (sono pigro e se posso uso cose già fatte), ho avviato la macchina virtuale e ho provato a navigare su internet. Non funzionava.

<!--more-->

Eseguendo il comando ifconfig non appariva la scheda di rete. Eppure sembrava tutto configurato bene.
Ecco come ho [soluzionato](http://anonimoconiglio.blogspot.it/), come direbbe il mio amico coniglio.

```bash
[teopost@localhost ~]$ ifconfig
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
ether 00:0c:29:68:22:e2  txqueuelen 1000  (Ethernet)
RX packets 0  bytes 0 (0.0 B)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 0  bytes 0 (0.0 B)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
inet 127.0.0.1  netmask 255.0.0.0
inet6 ::1  prefixlen 128  scopeid 0x10<host>
loop  txqueuelen 0  (Local Loopback)
RX packets 642  bytes 55820 (54.5 KiB)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 642  bytes 55820 (54.5 KiB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[teopost@localhost ~]$
```

## Soluzione

* Aprire il terminale
* Diventare root (su - root)
* Eseguire ```dhclient –v```

```bash
[root@localhost network-scripts]# dhclient -v
Internet Systems Consortium DHCP Client 4.2.5 Copyright 2004-2013
Internet Systems Consortium. All rights reserved. For info, please visit https://www.isc.org/software/dhcp/ Listening on LPF/ens32/00:0c:29:68:22:e2
Sending on   LPF/ens32/00:0c:29:68:22:e2
Sending on   Socket/fallback DHCPDISCOVER on ens32 to 255.255.255.255 port 67 interval 4 (xid=0x433a9e33) DHCPREQUEST on ens32 to 255.255.255.255 port 67 (xid=0x433a9e33)
DHCPOFFER from 172.16.179.254 DHCPACK from 172.16.179.254 (xid=0x433a9e33) bound to 172.16.179.136 -- renewal in 822 seconds.
[root@localhost network-scripts]#
```
* Funziona. Se ora eseguo ```ifconfig```

```bash
[root@localhost network-scripts]# ifconfig
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
inet 172.16.179.136  netmask 255.255.255.0  broadcast 172.16.179.255
ether 00:0c:29:68:22:e2  txqueuelen 1000  (Ethernet)
RX packets 11  bytes 1255 (1.2 KiB)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 23  bytes 3536 (3.4 KiB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
inet 127.0.0.1  netmask 255.0.0.0
inet6 ::1  prefixlen 128  scopeid 0x10<host>
loop  txqueuelen 0  (Local Loopback)
RX packets 770  bytes 66956 (65.3 KiB)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 770  bytes 66956 (65.3 KiB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## Eseguire il comando allo startup

Purtroppo se spegnete la macchina virtuale vi accorgerete che questa soluzione (se così la vogliamo chiamare) non è persistente.
Se si vuole rendere definitiva seguire i seguenti passi:

* Andare in ```/etc/init.d```
* Creare un file chiamato ```net-autostart```
* Incollarci dentro sta roba:

```bash
#!/bin/bash
# Fix for "No Internet Connection"
#
### BEGIN INIT INFO
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
### END INIT INFO
dhclient -v
```
4. Salvare il file
5. Cambiare i permessi del file (```chmod 755 net-autostart```)
6. Aggiungere l'esecuzione automatica dello script (```chkconfig --add net-autostart```)
7. Fare un bel reboot


## Riferimenti
Questo articolo non è farina del mio sacco.
Ho avuto il problema, ho gugolato un po e ho trovato la soluzione che ho riportato qui come mio appunto.

* https://geekflare.com/no-internet-connection-from-vmware-with-centos-7/
