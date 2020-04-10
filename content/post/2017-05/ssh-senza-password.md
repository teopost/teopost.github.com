+++
banner = "banner/ssh-senza-password.jpg"
categories = ["linux"]
date = "2017-05-17T21:39:51+02:00"
description = ""
images = []
menu = ""
tags = ["bash"]
title = "SSH senza password"

+++

Uso spesso ssh per collegarmi sui server linux dei clienti.
Per evitare di dover digitare sempre la password del server, è possibile configurare il vostro client per effettuare un collegamento senza dover digitare la password.

Creare sul vostro computer la chiave pubblica e privata
---
La prima cosa da fare è creare, sul client che deve collegarsi al server, le chiavi

Digitare quindi il comando ``ssh-keygen``

```bash
    teopost@local-host$ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/jsmith/.ssh/id_rsa):[Enter key]
    Enter passphrase (empty for no passphrase): [Press enter key]
    Enter same passphrase again: [Press enter key]
    Your identification has been saved in /home/teopost/.ssh/id_rsa.
    Your public key has been saved in /home/teopost/.ssh/id_rsa.pub.
    The key fingerprint is:
    33:b3:fe:af:95:95:18:11:31:d5:de:96:2f:f2:35:f9 teopost@local-host
```
Questa operazione crea nella cartella .ssh nella home del vostro computer 2 file. Una chiave privata (id_rsa) e una chiave pubblica (id_rsa.pub).

Copiare la chiave pubblica sul server
---
Ora dobbiamo far conoscere al server la nostra identità. Per farlo dobbiamo copiare la nostra chiave pubblica sul server. Si può fare anche a mano, ma c'è un comando che lo fa per noi chiamato ``ssh-copy-id``.

```
    teopost@local-host$ ssh-copy-id -i ~/.ssh/id_rsa.pub remote-host
    teopost@remote-host's password:
    Now try logging into the machine, with "ssh 'remote-host'", and check in:

    .ssh/authorized_keys

    to make sure we haven't added extra keys that you weren't expecting.
```

In pratica questo comando non fa altro che accodare la vostra chiave pubblica (scritta nel file id_rsa.pub), nel file del server ```authorized_key```situato sotto la cartella .ssh.

Effettuare il login senza password
---
Il gioco è fatto. Proviamo a vedere se il nostro lavoro è andato a buon fine. Effettuiamo quindi da linea di comando il collegamento ssh al server.

```bash
    teopost@local-host$ ssh remote-host
    Last login: Sun Nov 16 17:22:33 2008 from 192.168.1.2
    [Note: SSH did not ask for password.]

    teopost@remote-host$ [Note: You are on remote-host here]
```

Et voilà.
