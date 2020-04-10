+++
banner = "banner/oracle-char-vs-bytes.jpg"
categories = ["database"]
date = "2017-05-24T22:23:55+02:00"
description = ""
images = []
menu = ""
tags = ["oracle"]
title = "Oracle Char vs Bytes"

+++
Nei database Oracle, i campi che possono contenere dati stringa possono assumere datatype diversi

- NVARCHAR2
- VARCHAR2(n)
- VARCHAR2(n CHAR)
- VARCHAR2(n BYTE)

Ma che differenza c'è ?
<!--more-->

Se un campo viene definito ```VARCHAR2(20 byte)```, questo vuol dire che Oracle può memorizzare al massimo 20 byte in quel campo.
Ma 20 byte non corrispondono a 20 caratteri nel caso in cui il database debba memorizzare dati in formato UTF-8.

Se invece si definisce un campo ```VARCHAR2(20 char)```, oracle sa che deve essere in grado di memorizzare 20 caratteri. In questo caso non è importante quanti byte sono necessari per memorizzare un singolo carattere (un singolo carattere UTF-8 può occupare 4 byte).

Quando si crea una colonna sul db usando il datatype ```VARCHAR2(x)```, Oracle decide di creare il datatype ```VARCHAR2(x char)``` oppure ```VARCHAR2(x byte)``` in base alla configurazione di Oracle definita in fase di installazione.
La modifica di questa configurazione predefinita purtroppo non è semplice e, in alcuni casi, occorre reinstallare l'intero  db.

Quindi. Quale usare ?

Nelle fix di database usare sempre

``VARCHAR2(x CHAR)``

NVARCHAR2
----------
Il datatype NVARCHAR2 memorizza dati in formato unicode e quindi, per certi versi, è come scrivere VARCHAR2(x CHAR).
Tuttavia, Oracle ne sconsiglia l'utilizzo.

http://docs.oracle.com/database/121/NLSPG/ch6unicode.htm#NLSPG318

dove è scritto:

> Oracle recommends using SQL CHAR, VARCHAR2, and CLOB data types in AL32UTF8 database to store Unicode character data. SQL NCHAR, NVARCHAR2, and NCLOB data types are not supported by some database features. Most notably, Oracle Text and XML DB do not support these data types.

RIFERIMENTI
---
* http://docs.oracle.com/database/121/NLSPG/ch6unicode.htm#NLSPG318
