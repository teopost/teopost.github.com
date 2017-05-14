+++
banner = "https://s.w.org/images/backgrounds/wordpress-bg-medblue.png"
categories = ["linux"]
date = "2017-01-08T16:15:00+01:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "Wordpress su Raspberry"

+++

Voglio installare un blog sulla raspberry.
Per farlo, dopo aver [installato un LAMP server]({{< ref "installare-lamp-raspberry.md" >}}), procedo come segue:


Ecco cosa ho fatto:

### Preparare lo schema di DATABASE

```sql
mysql -uroot -p<password>
CREATE DATABASE wordpress;
CREATE USER wordpressuser@localhost IDENTIFIED BY 'wppassword'
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;
```

dove <password> è la password di mysql che ho impostato durante l'installazione del LAMP server e wppassword è un esempio
di password per l'utente wordpress che ho appenta creato su mysql

Se tutto ha funzionato vai su http://your_server_IP_address e vedi il web server attivo

### Installazione di Wordpress

Ora scarico l'ultima versione di wordpress e la installo nella Rasp.
Queste operazioni le devo eseguire come root.

```bash
sudo -s
cd ~
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
cd wordpress
cp wp-config-sample.php wp-config.php

apt-get update
apt-get install php5-gd libssh2-php
cp wp-config-sample.php wp-config.php
vi wp-config.php
```

a questo punto inserisco i dati per l'accesso a wordpress. Poi proseguo:

```bash
apt-get install rsync
sudo rsync -avP ~/wordpress/ /var/www/html/
sudo chown -R root:www-data *
mkdir /var/www/html/wp-content/uploads
```

### Riferimenti

Questa documentazione è l'estratto di questo articolo molto più completo

* https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-14-04
