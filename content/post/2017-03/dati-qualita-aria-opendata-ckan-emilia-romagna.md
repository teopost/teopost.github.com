+++
banner = "dati-qualita-aria-opendata-ckan-emilia-romagna/dati-qualita-aria-opendata-ckan-emilia-romagna.jpg"
categories = ["fun"]
date = "2017-03-24T18:42:26+01:00"
description = ""
images = []
menu = ""
tags = ["ckan","opendata"]
title = "Recuperare i dati sull'inquinamento della mia città da CKAN"

+++

Ho intenzione di scrivere un piccolo bot di telegram che mi dia indicazioni sulla qualità dell'aria che ogni giorno respiro nella mia città.

<!--more-->

Sono partito, nella mia indagine, da qui:

    https://www.arpae.it/liberiamo/statistiche_riepilogative.asp?idlivello=822

Questo sito è pieno zeppo di dati utili e fra questi ci sono anche quelli che servono a me; ma come recuperarli ?

Per fortuna la regione Emilia Romagna espone tutte queste informazioni sotto forma di Open Data utilizzando un software open source chiamato CKAN.

CKAN consente di raggruppare dati da fonti eterogene, li trasforma e li rende disponibili in vari modi, fra cui, formati JSON che tanto piacciono ai programmatori.

A Savignano sul Rubicone, luogo in cui vivo, purtroppo nessuno dell'amministrazione comunale conosce questo strumento, e questo è un vero peccato.

Scopriamo il perchè.

## Leggere i dati da CKAN

Come dicevo, la regione espone i dati con CKAN. Lo fa con questo sito:

    https://dati.arpae.it

i dati sui PM10 sono qua:

    https://dati.arpae.it/dataset/qualita-dell-aria-rete-di-monitoraggio

l'insieme di dati che serve a me (chiamato dataset) e che devo recuperare e' questo:

    https://dati.arpae.it/dataset/qualita-dell-aria-rete-di-monitoraggio/resource/a1c46cfe-46e5-44b4-9231-7d9260a38e68

per leggere i dati in formato json (grazie al mitico CKAN) basta chiamare questa url per ottenere i primi 5 risultati:

    https://dati.arpae.it/api/action/datastore_search?resource_id=a1c46cfe-46e5-44b4-9231-7d9260a38e68&limit=5

Questa lista contiene tutti i rilevamenti di tutte le stazioni nella provincia di forlì-cesena.
In questa lista di dati, i campi che ci interessano sono:

* ```campo station_id```: identifica la stazione di rilevamento
* ```reftime```: rappresenta la data di rilevamento
* ```variable_id```: rappresenta la tipologia di dato recuperato

c'è una tabella completa delle stazioni qui:

    https://dati.arpae.it/dataset/qualita-dell-aria-rete-di-monitoraggio/resource/6b1c73c6-2fa9-40b2-854a-d3f822d1451a

e c'è anche una legenda dei parametri disponibili qui:

    https://dati.arpae.it/dataset/qualita-dell-aria-rete-di-monitoraggio/resource/6fb8af6d-a3cc-4cf7-b098-ed95c5744cee

i dati che interessano me sono:

* 6000031: La stazione di Savignano
* 5: Il dato che identifica i PM10

Bene, ora grazie a questi dati posso inviare una query a CKAN tipo questa:

```sql
SELECT * FROM "a1c46cfe-46e5-44b4-9231-7d9260a38e68"
WHERE station_id = '6000031' and reftime='2017-02-22T00:00:00'and variable_id=5
```

per farlo basta sostituire con caratteri d'escape spazi, apici e affini, nonchè passarlo in query string, incollarlo nel browser e il gioco è fatto.

    https://dati.arpae.it/api/action/datastore_search_sql?sql=SELECT%20*%20from%20%22a1c46cfe-46e5-44b4-9231-7d9260a38e68%22%20WHERE%20station_id%20=%20%276000031%27%20and%20reftime=%272017-02-22T00:00:00%27and%20variable_id=5

questo è il risultato:

```json
{ "help" : "https://dati.arpae.it/api/3/action/help_show?name=datastore_search_sql",
  "result" : { "fields" : [ { "id" : "_id",
            "type" : "int4"
          },
          { "id" : "_full_text",
            "type" : "tsvector"
          },
          { "id" : "station_id",
            "type" : "int4"
          },
          { "id" : "variable_id",
            "type" : "int4"
          },
          { "id" : "reftime",
            "type" : "timestamp"
          },
          { "id" : "value",
            "type" : "float8"
          }
        ],
      "records" : [ { "_full_text" : "'00':4,5 '03/23/2017':3 '36':6 '5':1 '6000031':2",
            "_id" : 571594,
            "reftime" : "2017-03-23T00:00:00",
            "station_id" : 6000031,
            "value" : 54.0,
            "variable_id" : 5
          } ],
      "sql" : "SELECT * from \"a1c46cfe-46e5-44b4-9231-7d9260a38e68\" WHERE station_id = '6000031' and reftime='2017-02-22T00:00:00'and variable_id=5"
    },
  "success" : true
}
```

a questo punto con un po di python è semplice leggere il dato che mi interessa

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

__author__ = 'teopost'

import urllib
import json


url = "https://dati.arpae.it/api/action/datastore_search_sql?sql=SELECT * from %22a1c46cfe-46e5-44b4-9231-7d9260a38e68%22 WHERE station_id = %276000031%27 and reftime=%272017-02-22T00:00:00%27 and variable_id=5 order by value desc"

fileobj = urllib.urlopen(url)
results = fileobj.read().decode("utf-8")

data = json.loads(results)

print data["result"]["records"][0]["value"]

```
e cioè, il giorno 23 Marzo 2017, a Savignano sul Rubicone, i livello di PM10 rilevato era 54 µg/m3 (il limite consentito per legge è 50 µg/m3).
Forse era meglio non saperlo !

## Conclusioni

CKAN è una figata pazzesca. Peccato che uno strumento così utile e oltretutto opensource non sia usato da tutte le pubbliche amministrazioni (come quella in cui abito).

## Riferimenti

* https://www.arpae.it/liberiamo/statistiche_riepilogative.asp?idlivello=822
* https://www.arpae.it/v2_aria.asp?idlivello=134&tema=valutazioni#
* http://service.arpa.emr.it/qualita-aria/bollettino.aspx?prov=FC
* https://www.arpae.it/liberiamo/dati_giornalieri_14gg.asp?idlivello=821

{{% button href="https://gist.github.com/teopost/299bb00cea2988bfae3994cc1af2900a" icon="fa fa-github" %}}Get source code from Git-Hub{{% /button %}}
<br/>
