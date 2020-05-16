+++
banner = "liquibase-vs-flyway/liquibase-vs-flyway.png"
categories = ["work"]
date = "2018-11-22T18:00:00+01:00"
description = ""
images = []
menu = ""
tags = ["database","utility"]
title = "Liquibase vs Flyway"

+++

Liquibase e Flyway sono "database migration tool", ovvero strumenti per l'applicazione di modifiche al database.
Questo articolo descrive le principali funzionalità e le  differenze che li caratterizzano.

<!--more-->

Siquibase che flyway sono scritti in Java.
Utilizzano jdbc per l'accesso al database e supportano sia Oracle che DB2.

Entrambi, dato un elenco di file che descrivono le modifiche da applicare al database (DBFIX), si collegano ad un database target ed eseguono tali file escludendo quelli già eseguiti in una precedente migrazione.
Tutte le modifiche vengono registrate in una tabella di metadati.

## Checksum dei files e tabella di modifiche

Come fanno liquibase e flyway a sapere se una DBFIX è da applicare o se è già stata applicata ?

Entrambi i tool registrano, su una propria tabella, la storia delle DBFIX applicate.
In tale tabella vengono memorizzate varie informazioni come, ad esemepio, la data di esecuzione, il nome del file e la versione della fix.
Oltre a ciò, si salvanno anche un checksum che identifica univocamente il file applicato.
Tale checksum viene sempre controllato quando la fix viene eseguita.
Tale dettaglio è importante perché se un file viene modificato dopo essere stato applicato ad un db, la migrazione fallisce.

Esiste il modo di ripristinare manualmente tale checksum ma in entrambi i casi è una operazione da fare manualmente e abbastanza delicata da effettuare.

## Versioni delle DBFIX

### Flyway

Flyway identifica la versione e la descrizione di una fix estraendola dal nome del file.
Sulla base di una particolare naming convention (che può essere  personalizzata) è in grado di determinare anche la sequenza di applicazione di tali fix.

Esempio:

```
V1.0.0__Creazione_tabella_USERS.sql
V2.0.0__Aggiunta_campo_SEX_a_tabella_USERS.sql
```

Dopo l'applicazione, nella tabella di changelog (chiamata schema_version), viene riportato:

| installed_rank | version | description                        |
| -------------- | ------- | ---------------------------------- |
| 1              | 1.0.0   | Creazione tabella USERS            |
| 2              | 2.0.0   | Aggiunta campo SEX a tabella USERS |

Flyway, nel sopracitato esempio, applica prima la V1.0.0 ed in seguito la V2.0.0.
La sequenza di applicazione delle fix la si può evincere dal campo installed_rank.

Per Flyway quindi ogni file è una dbfix con una versione.

### Liquibase

Per liquibase il concetto di versione della fix non è necessariamente associato al singolo file.
Un file può anche contenere una serie di modifiche (changeset) che devono essere "marcate" all'interno del file.
Se si usa il formato .sql, i changeset devono essere esplicitati sotto forma di commenti.

Ogni changeset è marcato dalla coppia di valori "autore" e "id".
Tale coppia di valori rende univoca la fix nel caso in cui più sviluppatori lavorano in contemporanea su uno stesso set di funzioni.

Esempio:

```
--liquibase formatted sql

--changeset teodorani:1
create table test_sql(
    id   number(32),
    name varchar2(255)
);

--rollback drop table test_sql;

--changeset teodorani:2
insert into test_sql (id, name) values (1, 'nama 1');
insert into test_sql (id, name) values (2, 'name 2');

--changeset teodorani:3
insert into test_sql (id, name) values (3, 'name 3');
```

Nel suddetto esempio, la DBFIX contiene 2 commenti che vengono interpretati da liquibase come direttive, ovvero:

* `--liquibase formatted sql`: Valore obbligatorio che indica il formato della DBFIX (puo' essere sql, xml o json).
* `--changeset teodorani:1`: Valore che identifica una specifica modifica composto da nome dello sviluppatore e da un numero (facoltativo).
* `--rollback drop table test_sql;`: Statement da eseguire in caso di fallimento del changeset superiore

Dopo aver applicato la DBFIX, liquibase memorizza nella tabella DATABASECHANGELOG i seguenti dati:

| ID  | AUTHOR    | FILENAME          | DATEEXECUTED               | ORDEREXECUTED.... |
| --- | --------- | ----------------- | -------------------------- | ----------------- |
| 1   | teodorani | changelog-1.0.sql | 22/11/2018 10.17.09,069030 | 1                 |
| 2   | teodorani | changelog-1.0.sql | 22/11/2018 10.17.09,150484 | 2                 |
| 3   | teodorani | changelog-1.0.sql | 22/11/2018 10.17.09,182781 | 3                 |
| 1   | sarli     | changelog-2.0.sql | 22/11/2018 10.17.09,182781 | 3                 |

## Sequenza di applicazione delle DBFIX

### Flyway

Flayway determina la sequenza di applicazione di una fix sulla base dei files contenuti in una o più cartelle.
E' quindi sufficiente mettere i files in una cartella secondo una struttura che puo' essere personalizzata.
Oltre a ciò è possibile applicare dei files di pre e post migrazione (files con un nome particolare).

### Liquibase

Liquibase determina la sequenza di fix di applicare sulla base di un file master xml in cui devono essere inserite le entry dei files da applicare.

Esempio di file master:

```
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                      http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <include file="fix-sql/changelog-1.0.sql"/>
  <include file="fix-sql/changelog-2.0.sql"/>

</databaseChangeLog>
```

File changelog-1.0.sql

```
--liquibase formatted sql

--changeset teodorani:1
create table test_sql(
    id   number(32),
    name varchar2(255)
);

--rollback drop table test_sql;

--changeset teodorani:2
insert into test_sql (id, name) values (1, 'nama 1');
insert into test_sql (id, name) values (2, 'name 2');

--changeset teodorani:3
insert into test_sql (id, name) values (3, 'name 3');
```

File changelog-2.0.sql

```
--liquibase formatted sql

--changeset sarli:1
alter table test_sql add surname varchar2(255);
```

Quindi non importa il nome del file e non è importante il path.
Quello che è importante è solo la corretta creazione del file xml master.
Il vantaggio di questo approccio è notevole perché è possibile far coesistere DBFIX di formati diversi (sql,json, xml) e disporle in una alberatura su disco a piacere.

## Multidatabase

### Flyway

Flyway si limita ad applicare file di tipo SQL. Non è possibile applicare le stesse DBFIX a database target differenti (DB2 e ORACLE) se non creando gli specifici statement con la sintassi oracle e DB2

### Liquibase

Se si utilizza la sintassi basata non su SQL (es: XML), liquibase genera, in base al DB target, gli statement opportuni per le modifiche al db.

## Rollback

Flyway non consente di definire files di rollback. Questa funzione non è disponibile.
Liquibase consente di specificare quali istruzioni di rollback eseguire

## Documentazione

Con Liquibase è possibile generare una documentazione descrittiva con le modifiche applicate per ogni changeset.
E' disponibile un esempio on line a questo indirizzo: http://www.liquibase.org/dbdoc/index.html
Tale funzionalità non è presente in Flyway

## SQL Placeholder

I files SQL, in Flyway possono contenere i placeholder. Vere e proprie variabili che possono essere sostituite in fase di esecuzione:

Esempio:

```sql
INSERT INTO ${tableName} (name) VALUES ('Mr. T');
```

Tali placeholder possono essere valorizzati in files di properties.

Liquibase non dispone di questa funzionalità

## Baseline

In flyway è possibile definire le baseline, ovvero un tab ad una versione specifica.
Questo consente di eseguire migrazioni fino ad una specifica baseline.

In liquibase questa funzionalità non è presente ma è facilmente gestibile con differenti file xml master.

## Precondition

Con liquibase è possibile definire delle condizioni indispensabili per l'applicazione delle DBFIX.

Esempio di precondition nel formato XML:

```xml
<preConditions onFail="WARN">
     <sqlCheck expectedResult="0">select count(*) from my_table</sqlCheck>
 </preConditions>
```

Faccio un esempio più interessate basato su file SQL
Diciamo che dobbiamo inserire un valore nella STORECONF solo se non presente:

```sql
--preconditions onFail:WARN onError:HALT
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM STORECONF where STOREENT_ID='1000' and NAME='CreateOrderServiceURL'
insert into STORECONF (STOREENT_ID, NAME, VALUE,...)
values (1000, 'CreateOrderServiceURL',...)
```

## Context

Talvolta esistono situazioni in cui statement SQL devono essere eseguiti solo se si è su un determinato ambiente (DEV piuttosto che PROD).

Con liquibase è possibile definire i context per questo scopo.

In questo esempio, l'insert viene effettuata solo se si è nell'ambiente UAT.

```sql
--changeset teodorani:3 context=UAT
insert into STORECONF (STOREENT_ID, NAME, VALUE,...)
values (1000, 'CreateOrderServiceURL',...)
```

I context vengono passati via command-line a liquibase in fase di migrazione.

* Elenco pre-condition: http://www.liquibase.org/documentation/preconditions.html

Tabella di confronto funzionalità
---

| Feature              | Flyway | Liquibase |
| -------------------- |:------:|:---------:|
| SQL Placeholder      | X      |           |
| Baseline             | X      |           |
| Precondition         |        | X         |
| Context              |        | X         |
| Supporto jdbc        | X      | X         |
| Supporto fix in XML  |        | X         |
| Supporto fix in JSON |        | X         |
| Supporto fix in YAML |        | X         |
| Supporto fix in SQL  | X      | X         |
| Supporto PL/SQL      | X      | X         |
| Gen. Documentazione  |        | X         |
| DBFIX checksum based | X      | X         |

Conclusioni
-----------

Personalmente ho esperienza diretta sull'utilizzo di flyway su cui ho riscontrato anche alcuni problemi sull'applicazione di DBFIX custom che possono cambiare a seconda delle integrazioni da effettuare.

Flyway determina la sequenza di fix da applicare in base a come vengono trovate in una cartella, questo rende estremamente complicato gestire casi in cui, a seconda delle integrazioni da effettuare, tali cartelle possono essere diverse.
Infatti flyway, a meno che non si attivi una modalità chiamata out-of-order, non permette di applicare una fix con versione minore (es:1.0), se è gia' stata applicata la 1.1.

Liquibase non sembra avere questo problema perché per liquibase quello che comanda é sempre il file xml master.
