+++
banner = "nmon-as-a-service/nmon-as-a-service.jpg"
categories = ["work"]
date = "2020-05-01T18:42:26+01:00"
description = ""
images = []
menu = ""
tags = ["linux","utility"]
title = "Schedulare nmon nel crontab"
+++

Per monitorare lo stato dei server in un infrastruttura Linux, esistono oggi molti strumenti (zabbix, netdata, zenoss, ecc.).
In alcuni casi, dove non si ha la possibilità di installare questi validissimi tools, può essere utile fare un salto indietro nel tempo e utilizzare il buon "vecchio" nmon.

In questo articolo vi mostro come installarlo come servizio su Red Hat.

<!--more-->

Per prima cosa installiamo nmon con il comando ``yum install -y nmon``

Già ora puoi lanciare da linea di comando nmon e sbirciare il tuo sistema con la sua interfaccia, ma la cosa interessante è attivarlo come vero e proprio servizio di linux per raccogliere e memorizzare tutte le metriche della giornata.

Per farlo ti basta eseguire questi comandi

{{< gist teopost 41d2ea849422243075dbd3f4ba791ab7 "installer-nmon-as-a-service.sh" >}}

E se sei pigro come me, vai sulla macchina come root e incolla questo comando:

```bash
curl https://gist.githubusercontent.com/teopost/41d2ea849422243075dbd3f4ba791ab7/raw/installer-nmon-as-a-service.sh | bash
```

Ora troverete nella cartella ``/var/log/nmon/`` il file del giorno corrente, e nella cartella ``/var/log/nmon/old`` i files dei giorni precedenti.

## Riferimenti

* https://nmonvisualizer.github.io/nmonvisualizer/startup.html
