+++
banner = "banner/usare-file-ppk-con-ssh-sul-mac.png"
categories = ["system"]
date = "2018-01-02T22:45:00+01:00"
description = ""
images = []
menu = ""
tags = ["osx"]
title = "Usare file ppk con ssh sul Mac"

+++

Amo Linux.
Sembra una contraddizione ma è proprio per questo che amo anche i Mac, e ne possiedo uno.
Recentemente mi è capitato di dovermi collegare ad un server remoto via ssh utilizzando una chiave privata diversa dalla mia.
Mi è stata fornita una chiave privata ma, ahimè, in un formato proprietario di putty (con estensione .ppk).
Ecco quello che ho fatto:

Conversione dile .ppk in .pem
---
Per prima cosa ho convertito il file. Con brew ho installato una utility.

```bash
$ brew install putty
```
questo comando installa ``puttygen``
Ora lo usiamo per convertire il file ppk.

```bash
$ puttygen  cert.ppk -O private-openssh -o cert.pem
```


Installiamo il certificato
---
Copiamo ora il file .pem dove è giusto che stia. Quindi:

```bash
$ cp cerp.pem ~/.ssh/cert.pem
```

Diamo i giusti permessi

```bash
$ chmod 400 ~/.ssh/cert.pem
```

Controlliamo che ssh funzioni senza che mi chieda la password

```bash
$ ssh -i ~/.ssh/cert.pem user@host.com
```

Aggiorniamo il config
---
Ora configuro il mio config di ssh per utilizzare il file cert.pem.
Apro il config e metto:

```
Host live_frontend
    HostName www.server.com
    User user
    IdentityFile ~/.ssh/cert.pem
```
Nota: Eventualmente si può anche aggiungere a ssh-agent con ```ssh-add ~/.ssh/cert.pem```
