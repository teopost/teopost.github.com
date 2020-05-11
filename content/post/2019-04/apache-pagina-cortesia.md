+++
banner = "banner/apache-pagina-cortesia.png"
categories = ["linux"]
date = "2019-04-24T18:42:26+01:00"
description = ""
images = []
menu = ""
tags = ["apache"]
title = "Pagina di cortesia su Apache"
+++

Ci sono molti modi per attivare la cosiddetta pagina di cortesia di un sito.

Nella maggior parte dei casi che mi è capitato di vedere, l'abitudine è quella di editare il file di configurazione di apache.
In questo articolo vi mostro un modo più elegante per farlo.

<!--more-->

Aggiungete questa regola nel file **httpd.conf**:

```
RewriteCond %{REMOTE_ADDR} !^172\.27\.
RewriteCond %{REMOTE_ADDR} !^172\.16\.8\.102
RewriteCond %{REMOTE_ADDR} !^172\.16\.8\.112
RewriteCond %{REMOTE_ADDR} !^192\.168\.
RewriteCond %{X-Forwarded-For} !^192\.168\.
RewriteCond %{DOCUMENT_ROOT}/maintenance.txt -f
RewriteRule "/(.*)$" "/500.html" [L,NC]
```

Questo esempio si legge così:

1. **RewriteCond %{REMOTE_ADDR} !^172\.27\.**  - Tutti gli indirizzi che non sono 172.27.*.* vengono esclusi. Le successive righe sono in AND
2. **RewriteCond %{DOCUMENT_ROOT}/maintenance.txt -f** - ... inoltre, se esiste un file nella DOCUMENT_ROOT chiamato maintenance.txt...
3. Effettuo la rewrite alla pagina di maintenance (500.html)

Come dicevo, usando questa sintassi non è necessario editare il file di configurazione per mettere in maintenance il sito.

Per creare il file maintenance.txt si può usare uno scrippettino, tipo questo:

```bash
#!/bin/bash

# Script for website maintenance mode
# Author: Stefano Teodorani 13 feb 2019

maintenance_file="/tmp/maintenance.txt"

if [ $# == 1 ]; then
   if [ $1 != "on" -a $1 != "off" ]; then
      echo "ERR: bad parameter"
      echo "Usage $(basename $0) [on|off]"
      exit
   fi
else
   if [ -f $maintenance_file ]; then
      echo "Maintenance mode is ON"
   else
      echo "Maintenance mode is OFF"
   fi

   echo "Usage $(basename $0) [on|off]"
   exit
fi

case $1 in
   "on")
      touch $maintenance_file
      echo "Maintenance mode is ON"
      ;;
   "off")
      rm -f $maintenance_file
      echo "Maintenance mode is OFF"
      ;;
esac
```

Se siamo un su un cluster di webserver, il suddetto file deve essere messo su tutti gli host.

Per mettere in maintenance tutti i web server si può usare uno script tipo questo:

```bash
#!/bin/bash

# Script for website maintenance mode
# Author: Stefano Teodorani 13 feb 2019

absolute_filename=$(readlink -f $0)
export HOME_DIR=$(dirname $absolute_filename)

$HOME_DIR/maintenance.sh on

if [ $(hostname) = "host-prd-1" ];then
  remote_host="host-prd-11"
elif [ $(hostname) = "host-prd-1" ]; then
  remote_host="host-prd-11"
fi


if [ ! -z $remote_host ]; then
  echo "Execute remote maintenanche script"
  ssh root@${remote_host} "/opt/scripts/maintenance.sh on"
fi

echo "Maintenance mode ON for local site $(hostname)"
echo "Maintenance mode ON for remote site $remote_host"
```

## Pianificazione della pagina di cortesia

In linux è possibile effettuare una schedulazione puntuale ed estemporanea con il comando at.

Ora che non è necessario editare il file di configurazione per editare la pagina, possiamo mettere in manutenzione in sito ad un'ora specifica con il seguente comando:

```bash
# Metto in maintenance il sito alle ore 22
/opt/scripts/maintenance.sh on >> /tmp/maintenance-status.log | at 22:00

# Tolgo il maintenance dal sito alle ore 09
/opt/scripts/maintenance.sh on >> /tmp/maintenance-status.log | at 09:00
```

* Puoi vedere la lista dei jobs at con il comando at -l
* Puoi cancellare un job con il comando atrm <job-number> (vedi comando precedente)

## Riferimenti

* https://www.computerhope.com/unix/uat.htm
