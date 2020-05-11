+++
banner = "nmon-as-a-service/nmon-as-a-service.jpg"
categories = ["linux"]
date = "2020-05-01T18:42:26+01:00"
description = ""
images = []
menu = ""
tags = ["linux"]
title = "Schedulare nmon nel crontab"
+++

Per monitorare lo stato dei server in un infrastruttura Linux, esistono oggi molti strumenti (zabbix, netdata, zenoss, ecc).
In alcuni casi, dove non si ha la possibilità di installare questi validissimi tools, può essere utile fare un salto indietro nel tempo e utilizzare il buon "vecchio" nmon.

In questo articolo vi mostro come installarlo come servizio su Red Hat.

<!--more-->

Per prima cosa installiamo nmon.

```bash
yum install -y nmon
```

Bene. Già ora puoi lanciare da linea di comando nmon e sbirciare il tuo sistema con la sua interfaccia, ma la cosa interessante è attivarlo come vero e proprio servizio di linux per raccogliere e memorizzare tutte le metriche della giornata.

```bash
curl  https://www.stefanoteodorani.it/nmon-as-a-service/nmon_initd_rhel  --output /etc/init.d/nmon
chown root:root /etc/init.d/nmon
chmod 755 /etc/init.d/nmon

curl  https://www.stefanoteodorani.it/nmon-as-a-service/nmon_logrotated  --output /etc/logrotate.d/nmon
chown root:root /etc/logrotate.d/nmon
chmod 644 /etc/logrotate.d/nmon

chkconfig --add nmon
service nmon start
```

Ora troverete nella cartella ``/var/log/nmon/`` il file del giorno corrente, e nella cartella ``/var/log/nmon/old`` i files dei giorni precedenti.

## Riferimenti

* https://nmonvisualizer.github.io/nmonvisualizer/startup.html
