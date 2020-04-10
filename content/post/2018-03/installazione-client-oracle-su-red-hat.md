+++
banner = "banner/oracle-logo.png"
categories = ["database"]
date = "2018-03-30T09:18:00+01:00"
description = ""
images = []
menu = ""
tags = ["oracle"]
title = "Installazione client Oracle 12.x su Red-Hat"

+++

Oracle Instant client è una versione minimale del client Oracle che contiene esclusivamente le librerie indispensabili per la connessione al db.

In questo articolo vediamo come installarlo su una macchina Red-Hat

<!--more-->

Nell'Instant client non sono compresi una serie di tool tipicamente presenti in una installazione full (es: exp, imp, sqlldr, tnsping).

Per installare Oracle Instant client su una macchina Red-Hat si possono usare 3 modi diversi.

* Installazione dei pacchetti rpm
* Installazione manuale con lo zip del client
* Installazione del modulo client con il setup di Oracle Server
* Questo documento descrive l'installazione del client utilizzando i pacchetti rpm.

## Scaricare il software

I pacchetti rpm del client si scaricano dal sito Oracle. Nella fattispecie la pagina per il download è:

* http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html

Accettare la licenza Oracle e scaricare i seguenti pacchetti:

* oracle-instantclient12.1-basic-12.1.0.2.0-1.x86_64.rpm
* oracle-instantclient12.1-sqlplus-12.1.0.2.0-1.x86_64.rpm
* oracle-instantclient12.1-jdbc-12.1.0.2.0-1.x86_64.rpm

Depositare i file in una cartella temporanea (es: /tmp/oracle-sw)

## Creazione utente oracle

L'installazione dell'Oracle Instant client "standard" (da best practice Oracle) non prevede necessariamente l'installazione di un utente dedicato, tuttavia nella maggior parte dei casi è buona norma creare un utente specifico.

Eseguire quindi:

```bash
# groupadd -g 502 oracle
# useradd -c "oracle user" -g 502 -m -d /home/oracle -s /bin/bash -u 502 oracle
```

Nota: Se l'uid è già presente utilizzarne un'altro (es: 504) 

### Installazione pacchetti

L'installazione dei pacchetti rpm è molto semplice, con utenza root eseguire:

```bash
# rpm -ilv ./oracle-instantclient12.1-basic-12.1.0.2.0-1.x86_64.rpm
# rpm -ilv ./oracle-instantclient12.1-sqlplus-12.1.0.2.0-1.x86_64.rpm
# rpm -ilv ./oracle-instantclient12.1-jdbc-12.1.0.2.0-1.x86_64.rpm
```

L'installazione del client fatta con i pacchetti rpm, installa il software in  /usr/lib/oracle/12.1/client64/

Se si desidera configurare files tnsnames.ora e sqlnet.ora, creare manualmente la seguente cartella (netword/admin):

```bash
# mkdir -p /usr/lib/oracle/12.1/client64/network/admin
```

Dentro la cartella admin si può creare il file tnsnames.ora in cui mettere i profili di connessione al DB.

Esempio:

```sh
TEST = 
  (DESCRIPTION = 
  	 (ADDRESS_LIST = 
  	 	(ADDRESS = 
  	 		(COMMUNITY = tcp.world) 
  	 		(PROTOCOL = TCP) 
  	 		(Host = tst-dbhost) 
  	 		(Port = 1521) 
  	 	) 
  	 ) 
  (CONNECT_DATA = 
  	 (SID = ORADB) 
  ) 
)
```

Per funzionare, il client oracle ha bisogno del settaggio di alcune variabili d'ambiente.
Nel .bash_profile dell'utente oracle creato impostare le seguenti variabili:

```bash
export ORACLE_HOME=/usr/lib/oracle/12.1/client64
export TNS_ADMIN=$ORACLE_HOME/network/admin/
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export PATH=/usr/lib/oracle/12.1/client64/bin:$PATH
```















