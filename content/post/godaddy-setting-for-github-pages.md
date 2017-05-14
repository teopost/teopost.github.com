+++
banner = "https://partnernoc.cpanel.net/logo/436-5141-logo.jpg"
categories = ["system"]
date = "2016-12-28T22:45:00+01:00"
description = ""
images = []
menu = ""
tags = ["github pages"]
title = "Configurare GoDaddy per le Github Pages"

+++

Possiedo il dominio [www.stefanoteodorani.it][sito] su GoDaddy.com. Per far funzionare tale dominio con questo blog ho seguito i seguenti passi:

Creazione sito teopost.github.io
---
Ho creato sul mio profilo github un repository chiamato teopost.github.com.
Ho committato il sorgente del mio sito sul master di tale repository.
Github è così gentile da accorgersi che il sorgente committato sul master è un sorgente di
jekyll, quindi provvede ad eseguirne la build e a pubblicarne il risultato.
Lo fa automaticamente sull'indirizzo [teopost.github.io][teopost].

Aggiunta file CNAME
---
Nella root del repository ho aggiunto un file chiamato CNAME con una sola riga al suo interno. L'indirizzo del mio dominio personalizzato ovvero www.stefanoteodorani.it
Questo file serve a github per riuscire a collegare il sito al dominio.

![](/post_images/godaddy-setting-for-github-pages.png)

Attesa aggiornamento DNS
---
Quando gli aggiornamenti del DNS diventeranno effettivi, si dovrebbe essere in grado di navigare su [www.stefanoteodorani.it][sito] e vedere la pagina teopost.github.io.

È possibile verificare l'aggiornamento del DNS con il seguente comando:

```bash
$ dig www.stefanoteodorani.it +nostats +nocomments +nocmd
```

Questo è il risultato

```bash
; <<>> DiG 9.8.3-P1 <<>> www.stefanoteodorani.it +nostats +nocomments +nocmd
;; global options: +cmd
;www.stefanoteodorani.it.	IN	A
www.stefanoteodorani.it. 923	IN	CNAME	stefanoteodorani.it.
stefanoteodorani.it.	206	IN	A	192.30.252.153
```

[sito]:      http://www.stefanoteodorani.it
[teopost]:   http://teopost.github.io
