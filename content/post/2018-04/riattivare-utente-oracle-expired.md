+++
banner = "riattivare-utente-oracle-expired/riattivare-utente-oracle-expired.jpg"
categories = ["work"]
date = "2018-04-17T12:00:00+01:00"
description = ""
images = []
menu = ""
tags = ["oracle","database"]
title = "Riattivare un utente Oracle Expired"

+++

## Introduzione

Un utente Oracle può essere creato in modo che la password scada dopo un certo periodo di tempo definito nel profilo assegnato. Per riattivare l'utente esistente è necessario reimpostare la password.

In alcuni casi questo può essere un problema.
In questo articolo viene descritta una procedura un pò inconsueta per ripristinare un utente senza conoscere la sua password.

<!--more-->

## Procedimento

Il trucco sta nel cambiare la password dell'utente specificando il valore criptato recuperato dalle tabelle di catalogo.

Prima di tutto imposto l'utente come EXPIRED.

```bash
# sqlplus "/ as sysdba"

SQL> alter user SCOTT password expire;

User altered.

SQL> select username, account_status from dba_users where username='SCOTT';

USERNAME                       ACCOUNT_STATUS
------------------------------ --------------------------------
SCOTT                          EXPIRED

1 row selected.

```

A questo punto costruisco lo statement da eseguire per modificare la password:

```
SQL> SELECT 'ALTER USER '|| name ||' IDENTIFIED BY VALUES '''|| spare4 ||';'|| password ||''';' FROM sys.user$ WHERE name='SCOTT';

'ALTERUSER'||NAME||'IDENTIFIEDBYVALUES'''||SPARE4||';'||PASSWORD||''';'
----------------------------------------------------------------------------------------------------------------------------------------------------------------
ALTER USER SCOTT IDENTIFIED BY VALUES 'S:ADEB92DE98CCE3A01033BFE530092B43AD1AE394220C93BA4ED3813C05C6;B0334ACB686B0325';

1 row selected.

```

Poi eseguo lo statement creato

```
SQL> ALTER USER SCOTT IDENTIFIED BY VALUES 'S:ADEB92DE98CCE3A01033BFE530092B43AD1AE394220C93BA4ED3813C05C6;B0334ACB686B0325';

User altered.
```


Verifico lo stato dell'utente

```
SQL> select username, account_status from dba_users where username='SCOTT';

USERNAME                       ACCOUNT_STATUS
------------------------------ --------------------------------
SCOTT                          OPEN

1 row selected.

SQL> connect simon/tiger
Connected.
```


Finito
