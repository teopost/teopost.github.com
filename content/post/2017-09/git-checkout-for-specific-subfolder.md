+++
banner = "git-checkout-for-specific-subfolder/git-checkout-for-specific-subfolder.png"
categories = ["work"]
date = "2017-09-17T12:52:00+01:00"
description = ""
images = []
menu = ""
tags = ["git"]
title = "Git checkout di una specifica sottocartella"
+++

Esiste un comando di git chiamato ```sparse checkout``` che permette di popolare la propria cartella di lavoro con una specifica cartella del repository. Utile perché non si è costretti a scaricare il contenuto di tutto il repository git.

<!--more-->

Supponiamo di avere su github un repository al cui interno è contenuta la sottocartella database. Devo scaricare tutti i files SQL contenuti nella cartella. Come posso fare :

Ecco come:


## Procedimento

```bash
mkdir databas
cd database/
# Inizializzo il repo vuoto
git init
# Aggiungo il remote origin
git remote add origin -f https://github.com/wedoit-io/decaduti.git
# ricorsivamente effettuo il checkout della cartella database
echo "database/*" >> .git/info/sparse-checkout
# rispetto alla cartella database effettuo il checkout per un massimo di 2 livelli
git pull --depth=2 origin master
```
