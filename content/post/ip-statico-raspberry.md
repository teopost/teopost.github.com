+++
banner = "https://upload.wikimedia.org/wikipedia/commons/6/69/Raspberries05.jpg"
categories = ["linux"]
date = "2017-01-09T16:02:00+01:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "IP Statico su Raspberry"

+++

Se si vuole utilizzare una raspberry come un server, la prima cosa da fare è evitare che l'indirizzo IP cambi ad ogni reboot. Insomma va disabilitato il dhcp.
Per farlo agire come segue:

Editare il file `/etc/network/interfaces`

```bash
sudo vi /etc/network/interfaces
```

Commentare `iface eth0 inet manual` e aggiungere in fondo al file i dati dell'ip:

```bash
# iface eth0 inet manual
auto eth0
iface eth0 inet static
address 192.168.10.32
gateway 192.168.10.254
netmask 255.255.255.0
```

Per motivi ignoti il vecchio ip continua a rimanere assegnato se non si disinstalla il dhcp.
Lo si può verificare digitando:

```bash
ip addr show
```

Si può risolvere il problema disinstallando il relativo pacchetto

```bash
apt-get purge dhcpcd5
```

Fare un ``reboot`` e ricontrollare con:

```bash
ip addr show
```
