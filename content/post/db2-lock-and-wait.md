+++
banner = "banner/lock-wait-e-timeout-in-db2.jpg"
categories = ["database"]
date = "2019-01-20T18:42:26+01:00"
description = ""
images = []
menu = ""
tags = ["db2"]
title = "Lock wait e timeout in DB2"

+++

In DB2 un lock wait è una condizione che si verifica quanto una sessione attente di prendere un lock su un oggetto che sul quale un'altra sessione l'ha già preso.

In questa situazione la sessione che attende è una sessione in Lock Wait.

Con questa query è possibile vedere quali sessioni sono in attesa

<!--more-->

```sql
select
      substr(HLD_APPLICATION_NAME,1,10) as "Hold App",  -- Who is holding the lock
      substr(HLD_USERID,1,10) as "Holder",
      substr(REQ_APPLICATION_NAME,1,10) as "Wait App",  -- Who is waiting on the lock
      substr(REQ_USERID,1,10) as "Waiter",
      LOCK_MODE as "Hold Mode",
      LOCK_OBJECT_TYPE as "Obj Type",
      substr(TABNAME,1,10) as "TabName",
      substr(TABSCHEMA,1,10) as "Schema",
      LOCK_WAIT_ELAPSED_TIME as "waiting (s)", -- How long is the wait
      REQ_STMT_TEXT, -- new
      HLD_CURRENT_STMT_TEXT, -- new
      LOCK_NAME -- new
from
      SYSIBMADM.MON_LOCKWAITS
```

Fra le colonne estratte ce ne è una interessante. La colonna waiting, ovvero da quanto tempo la sessione è in attesa.

Nella suddetta query è in attesa da 15 secondi.

La cosa interessante ti questo tempo è che in DB2 è possibile impostare un tempo massimo oltre il quale la sessione in attesa viene sbloccata.

Questo tempo si chiama Lock timeout.

Per sapere l'impostazione del proprio database eseguire questo comando

```bash
db2 get db cfg for <DB_NAME> | grep Lock
```
esempio:

```bash
[dbtkadm@db2-10-5-0-7-sandbox ~]$ db2 get db cfg for TESTDB | grep Lock
 Lock timeout (sec)                        (LOCKTIMEOUT) = 45
 Lock timeout events                   (MON_LOCKTIMEOUT) = NONE
 Lock wait events                         (MON_LOCKWAIT) = NONE
 Lock wait event threshold               (MON_LW_THRESH) = 5000000
 Lock event notification level         (MON_LCK_MSG_LVL) = 1
```

## Riferimenti
* https://www.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.admin.perf.doc/doc/c0021311.html
