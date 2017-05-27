+++
banner = "banner/oracle-mail-spooler.png"
categories = ["database"]
date = "2017-05-27T23:35:16+02:00"
description = ""
images = []
menu = ""
tags = ["oracle"]
title = "Oracle mail spooler"

+++

Può capitare di dover mandare mail da Oracle ? Si. Ho fatto questo script per l'invio di mail da una istanza Oracle per risolvere un problema immediato, e siccome non c'è niente di più definitivo delle cose provvisorie, eccolo qui come se fosse un prodotto finito.

<!--more-->

L'idea è semplice. Creare una tabella si spool in cui accodare la lista delle mail da inviare. L'invio vero e proprio può essere immediato (eseguendo una specifica stored procedure) o asincrono schedulando l'esecuzione della stored procedure come job Oracle.

Per far funzionare oracle mail spooler occorrono alcuni oggetti di database.

## Tabelle

| Tabella | Descrizione |
|---------|-------------|
| MAIL_BOXES | Contiene la configurazione degli account con cui mandare le mail |
| MAIL_SPOOLER | Contiene la coda di messaggi da inviare |

Procedure
---

| Procedura  | Descrizione |
|------------|-------------|
| MAIL_QUEUE | Procedura da usare per mettere in coda le mail da inviare |
| SEND_MAIL  | Procedura che effettua l'invio della mail |
| SEND_QUEUE | Procedura da schedulare che verifica sulla tabella MAIL_SPOOLER l'esistenza di massaggi da spedire (Statu = Q) e se esistono chiama la SEND_MAIL per l'invio. |

## Schema

```sql
/* 1. Eseguire come utente system */

DROP USER  MAIL_QUEUE CASCADE;

CREATE USER MAIL_QUEUE
  IDENTIFIED BY MAIL_QUEUE
  DEFAULT TABLESPACE USERS
  TEMPORARY TABLESPACE TEMP
  ACCOUNT UNLOCK;

ALTER USER MAIL_QUEUE QUOTA UNLIMITED ON USERS;

GRANT CONNECT, RESOURCE TO MAIL_QUEUE;
GRANT UPDATE ANY TABLE TO MAIL_QUEUE;
GRANT SELECT ANY TABLE TO MAIL_QUEUE;
GRANT CREATE ANY TABLE TO MAIL_QUEUE;
GRANT QUERY REWRITE TO MAIL_QUEUE;
GRANT ALTER ANY TRIGGER TO MAIL_QUEUE;
GRANT EXECUTE ANY PROCEDURE TO MAIL_QUEUE;
GRANT CREATE ANY SYNONYM TO MAIL_QUEUE;
GRANT INSERT ANY TABLE TO MAIL_QUEUE;
GRANT EXECUTE ANY LIBRARY TO MAIL_QUEUE;
GRANT UNLIMITED TABLESPACE TO MAIL_QUEUE;
GRANT DELETE ANY TABLE TO MAIL_QUEUE;
GRANT CREATE VIEW TO MAIL_QUEUE;
```

## Grants

Tutti gli oggetti per l'invio delle mail sono creati in uno schema specifico (nel caso descritto chiamao MAIL_QUEUE).
Per utilizzare la procedure che mette in coda i messaggi e per consultare la lista dei messaggi in coda, occorre dare i seguenti grant:

```sql
-- Eseguire come system
grant execute on mail_queue.MAIL_QUEUE to MYUSER;
grant all on mail_queue.MAIL_SPOOLER to MYUSER;
grant execute on mail_queue.SEND_QUEUE to MYUSER;
```

## Synonyms

Collegarsi all'utente system di oracle.
Al primo utilizzo è consigliabile creare 2 sinomini per accedere piu' facilmente agli oggetti:

```sql
-- Eseguire come system
create synonym MYUSER.MAIL_QUEUE for mail_queue.MAIL_QUEUE;

-- Se si desidera consultare la tabella di spooler
create synonym MYUSER.MAIL_SPOOLER for mail_queue.MAIL_SPOOLER;

-- Se si desidera forzare l'invio delle mail
create synonym MYUSER.SEND_QUEUE for mail_queue.SEND_QUEUE;
```

## Creazione ACL Oracle

Per poter effettuare chiamate http, smtp, ssh, ecc direttamente da una istanza oracle è necessario attivare specifiche ACL.
Per fare questo collegarsi come utente system e creato la necessaria ACL, quindi:

```sql
-- Eseguire come system
-- Creo una ACL per il mio utente MAIL_QUEUE
BEGIN
  DBMS_NETWORK_ACL_ADMIN.create_acl (
    acl          => 'mail_queue.xml',
    description  => 'Acl per invio mail da utente MAIL_QUEUE',
    principal    => 'MAIL_QUEUE',
    is_grant     => TRUE,
    privilege    => 'connect');

  COMMIT;
END;
/

-- Add privilege of ACL to user MAIL_QUEUE
BEGIN
  DBMS_NETWORK_ACL_ADMIN.ADD_PRIVILEGE
    (acl      => 'mail_queue.xml',
    principal => 'MAIL_QUEUE',
    is_grant  =>  true,
    privilege => 'resolve');
  commit;
END;
/

BEGIN
  DBMS_NETWORK_ACL_ADMIN.assign_acl (
    acl => 'mail_queue.xml',
    host => 'smtp.universita.it',
    lower_port => 25,
    upper_port => NULL);
  COMMIT;
END;
/
```

```sql
/* Controllo che l'ACL sia stata creata */
select * from dba_network_acls
```

## Invio della mail

```sql
-- Mette in coda la mail per la spedizione
-- 1 è il numero mailbox configurato in MAIL_BOXES)
exec MAIL_QUEUE.MAIL_QUEUE ('Oggetto', 'Testo in formato TXT', 'Testo in formato HTML', 'destinatazio@gmail.com', 1)

-- Invia la mail
exec MAIL_QUEUE.SEND_QUEUE
```

## Limitazioni

- Non possono essere inviati messaggi a piu' destinatari.
- Attualmente il corpo dei messaggi non può superare i 4000 caratteri (facilmente risolvibile)
- Non possono essere inviati allegati

## Riferimenti

* http://remidian.com/2013/01/email-network-access-in-oracle-11g-network-access-control-list-acl/
* http://oracle-base.com/articles/misc/email-from-oracle-plsql.php
* https://arkatec.wordpress.com/2011/08/15/sending-email-using-oracle-database-and-google-mail-service/

{{% button href="https://github.com/teopost/oracle-mail-spooler/" icon="fa fa-github" %}}Scarica da Git-Hub{{% /button %}}
<br/>
