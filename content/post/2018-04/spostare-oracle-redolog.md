+++
banner = "spostare-oracle-redolog/spostare-oracle-redolog.jpg"
categories = ["work"]
date = "2018-04-05T11:18:00+01:00"
description = ""
images = []
menu = ""
tags = ["oracle","database"]
title = "Spostare i redolog su Oracle 12.x"

+++

## Introduzione

Oggi mi è capitato di dover spostare alcuni redolog che erano stati creati in una cartella sbagliata.
Ecco come ho fatto

<!--more-->

## Procedimento

Per prima cosa vediamo dove sono posizionati tali files:

```bash
# sqlplus "/ as sysdba"

SQL>  select member from v$logfile;

MEMBER
---------------------------------------
/u02/oradata/INTDB01/controlfile/o1_mf_f3h3nxgs_.ctl
/u03/oradata/INTDB01/controlfile/o1_mf_f3h3nxlh_.ctl
/u04/oradate/INTDB01/controlfile/o1_mf_f3h3nxol_.ctl
```

Il problema è la cartella oradate (si deve chiamare oradata).
I redo logs non possono essere rinominati (o spostati) mentre il database è online.
Il database deve essere in stato mount.
Per prima cosa faccio lo shutdown del DB.

Quindi:

```bash
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
```

Ora posso spostare il file:

```bash
$ mv /u04/oradate/INTDB01/controlfile/o1_mf_f3h3nxol_.ctl /u04/oradata/INTDB01/controlfile/
```

Ora rimetto il DB in mount e aggiorno il dizionario dati di Oracle con la nuova posizione dei files.

```bash
SQL> startup mount
ORACLE instance started.

Total System Global Area 1849530880 bytes
Fixed Size                 31339824 bytes
Variable Size            5528485968 bytes
Database Buffers         5314572800 bytes
Redo Buffers               55132288 bytes
Database mounted.
SQL> alter database rename file '/u04/oradate/INTDB01/controlfile/o1_mf_f3h3nxol_.ctl' to '/u04/oradata/INTDB01/controlfile/o1_mf_f3h3nxol_.ctl';

Database altered.

SQL> alter database open;

Database altered.
```

Controlliamo che tutto sia andato ben

```bash
# sqlplus "/ as sysdba"
SQL>  select member from v$logfile;

MEMBER
---------------------------------------
/u02/oradata/INTDB01/controlfile/o1_mf_f3h3nxgs_.ctl
/u03/oradata/INTDB01/controlfile/o1_mf_f3h3nxlh_.ctl
/u04/oradata/INTDB01/controlfile/o1_mf_f3h3nxol_.ctl
```

Controllo i parametri db_create_online_log_dest


```bash
SQL> show parameter db_create_online_log_dest

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_create_online_log_dest_1	     string	 /u02/oradata
db_create_online_log_dest_2	     string	 /u03/oradata
db_create_online_log_dest_3	     string	 /u04/oradate
db_create_online_log_dest_4	     string
db_create_online_log_dest_5	     string
```

Lo sistemo e controllo

```bash
SQL> alter system set db_create_online_log_dest_3='/u04/oradata' scope=spfile;

System altered.

SQL> show parameter db_create_online_log_dest

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_create_online_log_dest_1	     string	 /u02/oradata
db_create_online_log_dest_2	     string	 /u03/oradata
db_create_online_log_dest_3	     string	 /u04/oradata
db_create_online_log_dest_4	     string
db_create_online_log_dest_5	     string

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> startup
ORACLE instance started.

Total System Global Area 1849530880 bytes
Fixed Size                 31339824 bytes
Variable Size            5528485968 bytes
Database Buffers         5314572800 bytes
Redo Buffers               55132288 bytes
Database mounted.
```


FATTO
