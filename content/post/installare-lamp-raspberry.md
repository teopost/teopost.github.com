+++
banner = ""
categories = ["linux"]
date = "2017-01-07T16:10:00+01:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "LAMP sulla Raspberry"

+++

Voglio installare un blog sulla raspberry.
Per farlo, dopo aver [configurato un ip statico]({% post_url 2017-01-09-ip-statico-raspberry %}), devo installare i pacchetti software necessari per far girare il [sito](http://www.wedoit.io).

Ecco cosa ho fatto:

### Installazione di Apache

```bash
sudo apt-get update
sudo apt-get install apache2
```

Se tutto ha funzionato vai su http://your_server_IP_address e vedi il web server attivo

### Installazione di MySql

```bash
sudo apt-get install mysql-server php5-mysql
```

Inserire una password

### Configurazione di MySql

```bash
# Creare la struttura di directory del database dove sarà memorizzare le informazioni
sudo mysql_install_db
# Rimuoviamo alcuni valori di default pericolosi e limitiamo l'accesso al db
sudo mysql_secure_installation
```

### Installazione di PHP

```bash
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt
```

### Riferimenti

Questa documentazione è l'estratto di questo articolo molto più completo

* https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04
