+++
banner = "banner/oracle-logo.png"
categories = ["database"]
date = "2018-04-05T11:18:00+01:00"
description = ""
images = []
menu = ""
tags = ["oracle"]
title = "Spostare gli Oracle controlfiles su Oracle 12.x"

+++

## Introduzione

Oggi mi è capitato di dover spostare alcuni controlfile Oracle che erano stati creati in una cartella sbagliata. Spostare i controlfile non è difficile, ma essendo files molto critici per la sopravvivenza del DB, questa attività va fatta senza troppo casino nelle orecchie.

Vi avviso però. Per spostare i controlfiles bisogna spegnere l'istanza. Vediamo come fare.

<!--more-->

## Procedimento

Per prima cosa vediamo dove sono posizionati tali files:

```bash
# sqlplus "/ as sysdba"

SQL> show parameter control_files

NAME                 TYPE     VALUE
-------------------- -------- ------------------------------
control_files        string   /u02/oradata/TEST/control01.ctl,
                              /u02/oradata/TEST/control02.ctl, 
                              /u02/oradate/TEST/control03.ctl

```

In questo caso i control files sono posizionati in:

* /u02/oradata/TEST/control01.ctl
* /u02/oradata/TEST/control02.ctl
* /u02/**oradate**/TEST/control03.ctl

Il problema è la cartella oradate (si deve chiamare oradata).

Quindi:

```
SQL> shutdown immediate;

Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> exit
```

A questo punto, col DB spento, creo la cartella /u02/oradata e ci sposto dento il controlfile.

```bash
$ mkdir /u02/oradata/TEST
$ mv /u02/oradate/TEST/control03.ctl /u02/oradata/TEST/control03.ctl
```

Dopo aver spostato il file, bisogna aggiornare il pfile di Oracle per configurare la nuova posizione.
Aprire il pfile con un editor di testo (sotto $ORACLE_HOME/dbs) e cambiare il valore.

Quasi sempre si utilizza l'spfile. In questo caso, per cambiare il valore, bisogna aprire il DB con l'opzione nomount e dare il comando per aggiornare l'spfile.

``` bash
$ sqlplus "/ as sysdba"

Connected to an idle instance.

SQL> startup nomount;

ORACLE instance started.

SQL> alter system set control_files='/u02/oradata/TEST/control01.ctl',
'/u02/oradata/TEST/control02.ctl', '/u02/oradata/TEST/control03.ctl'
scope=SPFILE;

System altered.
```

A questo punto spengo l'istanza e la faccio ripartire per vedere se riparte senza problemi.

```bash
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup nomount
```

Prima di montare il DB cotrollo che i controlfile siano stati modificati correttamente

```bash
SQL> show parameter control_files

NAME                 TYPE     VALUE
-------------------- -------- ------------------------------
control_files        string   /u02/oradata/TEST/control01.ctl,
                              /u02/oradata/TEST/control02.ctl, 
                              /u02/oradata/TEST/control03.ctl
```

Visto che tutto è OK faccio partire l'istanza

```bash
SQL> alter database mount;

Database altered.

SQL> alter database open;

Database altered.
```

Finito












