+++
banner = "banner/unison-ssh.jpg"
categories = ["system"]
date = "2018-08-05T18:00:00+01:00"
description = ""
images = []
menu = ""
tags = ["linux"]
title = "Sincronizzare cartelle in SSH con UNISON"

+++

Recentemente, mi è capitato di dover risolvere questo problema.
Su un server ftp, vengono copiati files in 3 separate cartelle.
Tali file devono essere sincronizzati con 3 cartelle situate su 3 diverse macchine.
Tale sincronizzazione deve essere bidirezionale e i files creati sul server ftp devono scomparire quanto sulle cartelle remote qualcuno li cancella.

Per risolvere questo problema ho utilizzato unison, un tool che consente di sincronizzare in maniera bidirezionale due cartelle anche via ssh.

Ecco quindi quello che ho fatto:

<!--more-->

Installazione di UNISON
---

Per prima cosa bisogna installare unison.
Nel mio caso sto lavorando su Red-Hat (versioni 6.9 e 7.4).
Unison non è nei repo standard e non posso attivare i repo epel su questi server di produzione, quindi decido di scaricare i pacchetti rpm e installarli a mano.

Li scarico da qui: 

* Per Red-Hat 6.9: http://mirror.onet.pl/pub/mirrors/fedora/linux/epel/6/x86_64/Packages/u/
* Per Red-Hat 7.4: http://mirror.onet.pl/pub/mirrors/fedora/linux/epel/7/x86_64/Packages/u/

Il procedimento è identico, quindi descrivo solo quello che ho fatto per la versione 6.9

```bash

# come utente root
cd /tmp
wget http://mirror.onet.pl/pub/mirrors/fedora/linux/epel/6/x86_64/Packages/u/unison240-2.40.102-5.el6.x86_64.rpm
wget http://mirror.onet.pl/pub/mirrors/fedora/linux/epel/6/x86_64/Packages/u/unison240-text-2.40.102-5.el6.x86_64.rpm

rpm -ilv ./unison240-2.40.102-5.el6.x86_64.rpm ./unison240-text-2.40.102-5.el6.x86_64.rpm

```

Questa operazione va fatta su tutte le macchine coinvolte nella sincronizzazione. L'FTP Server e le macchine remote su cui devo inviare i files.

Copia delle chiavi SSH
---
Unison utilizza le chiavi ssh per accedere alle macchine remote. Per questo motivo occorre generare sull'FTP server una chiave (es: id_rsa.pub) e accodarla nel file authorized_keys nella cartella utente delle macchine remote.

Potete leggere come fare qui [SSH senza password]({{< ref "ssh-senza-password.md" >}})

Configurazione di UNISON
---
A questo punto andiamo nella cartella /root/.unison del server ftp, ovvero andiamo sulla macchina su cui faremo girare la sincronizzazione.

In questa cartella unison tiene registra gli snapshot delle cartelle locali che deve sincronizzare sotto forma di cartelle con un guid nel nome. La stessa cartella è presente anche nella home dell'utente delle macchina remote.

Queste cartelle sono gestite da unison in autonomia e non dobbiamo toccarle.
Quello che dobbiamo fare però è creare un file di configurazione che unison utilizzarà per sincronizzare i dati.
Creiamo quindi i files serverA.prf, serverB.prf e serverC.prf.

Nel file /root/.unison/serverA.prf mettiamo:

```
root = /var/ftp/dir1/
root = ssh://utente@10.10.10.1//files

include default
```

Nel file /root/.unison/serverB.prf mettiamo:

```
root = /var/ftp/dir2/
root = ssh://utente@10.10.10.2//files

include default
```

Nel file /root/.unison/serverC.prf mettiamo:

```
root = /var/ftp/dir3/
root = ssh://utente@10.10.10.3//files

include default
```

Noterete che alla fine dei files, c'è l'include del file /root/.unison/default.prf
Tale file è già presente ed è stato creato durante l'installazione di unison.
Dentro di mettiamo le impostazioni comuni a tutte configurazioni.
Nel mio caso:

```
ignore = Path .unison
confirmbigdel = false
owner = false
batch=true
logfile = /var/log/unison.log
```

Ovviamente questo non è obbligatorio farlo. Se preferite potete mettere tutto nei singoli files.


Esecuzione della sincronizzazione
---
Bene. Ora possiamo innanzitutto provare la sincronizzazione.
Per farlo basta eseguire:

```bash
unison serverA 
```

Se tutto è andato bene vedremo questo:

```bash
[root@serverFTP .unison]# /usr/bin/unison serverA
Contacting server...
Connected [//SERVERA//var/ftp/dir1/  -> //serverA//files]
Looking for changes
  Waiting for changes from server
Reconciling changes
Nothing to do: replicas have not changed since last sync.
[root@serverFTP .unison]#
```

Fate il test con gli altri 2 profili

E alla fine crontabbiamo il tutto

```
*/5 * * * * for i in $(echo A B C); do /usr/bin/unison server${i}; done
```

Bello è ?
