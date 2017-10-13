+++
banner = "banner/digital-signage-con-pi-kiosk.jpg"
categories = ["linux"]
date = "2017-05-20T20:50:06+02:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "Digital signage con Pi-Kiosk"

+++

Conosco Luigi. Ha una agenzia di viaggi a Rimini, e un'anno fa ha pensato di mettere, nella vetrina del suo negozio, un bel televisore per convincere i passanti che i viaggi della sua agenzia sono grandiosi.

E' fortunato. Il televisore che ha comprato, dispone di una funzione di slideshow. Gli basta copiare le immagini (che il mio amico [Sergio](https://www.sergiogridelli.it/) gli prepara) su una chiavetta USB e infilare la chiavetta nel televisore.

<!--more-->

Purtroppo però, dopo aver provato per un po questa soluzione, sono venuti fuori alcuni problemi.

* **Non era possibile copiare le immagini da remoto.** Per farlo bisognava essere fisicamente sul posto, scaricare le immagini che Sergio gli mandava per posta, copiare le immagini sulla chiavetta, mettere la chiavetta nel televisore. Inoltre durante questa operazione il televisore non proiettava niente.

* **Non tutte le immagini venivano mostrate correttamente.** Alcune formati non erano supportati.

* **Il televisore rimaneva inutilmente accesso anche la notte.** Luca vuole risparmiare sulla bolletta della luce

* **Il caricamento delle immagini era lento.** Le immagini venivano caricate lentamente e l'effetto in effetti non era grachè.

Luca ha quindi pensato di chiedere al sottoscritto se, con tutti i progetti e smanettamenti inutili che faccio, ce ne era uno che poteva risolvergli tutti questi problemi.

Così è nato Pi-Kiosk.

## Pi-Kiosk

Pi-Kiosk è una soluzione basata su raspberry pi, che consente di eseguire lo slideshow di una cartella di immagini su un televisore.

Prevede l'utilizzo di btsync per l'aggiornamento delle immagini. Grazie a btsync è possibile utilizzare un computer desktop o uno smartphone per tenere aggiornata la presentazione.

Pi-Kiosk spegne il televisore la sera e lo riaccende la mattina utilizzando lo standard cec dell'HDMI presente ormai in quasi tutti i televisori di ultima generazione.

## Materiale occorrente

* n.1 [Raspberry Pi](https://www.amazon.it/Raspberry-PI-Model-Scheda-madre/dp/B01CD5VC92/ref=sr_1_4?s=pc&ie=UTF8&qid=1495318603&sr=1-4&keywords=raspberry+pi+3)
* n.1 [Case](https://www.amazon.it/Raspberry-Pi-3-Case-BLACK/dp/B00W7S1BFG/ref=sr_1_4?s=pc&ie=UTF8&qid=1495318648&sr=1-4&keywords=case+raspberry+pi)
* n.1 [Scheda di memoria](http://goo.gl/3OPHrh)
* n.1 [Alimentatore 3A per Raspberry](https://www.amazon.it/Aukru-Caricabatterie-3000mA-Alimentatore-Raspberry/dp/B01566WOAG/ref=sr_1_4?s=pc&ie=UTF8&qid=1495318694&sr=1-4&keywords=alimentatore+raspberry+pi+3)
* n.1 Cavo HDMI
* n.1 Televisore con HDMI cec

## Preparare la Raspberry

Installate l'ultima versione del sistema operativo raspbian. Ci sono centinaia di guide su internet su come farlo. Se hai linux, ecco la centunesima:

Aprire una finestra terminal e da root inserire l'SD nel card reader del computer. Eseguire questo comando:

    # df -h

Eseguire l'umount della scheda: (sdb1 è solo un esempio, il nome potrebbe essere diverso)

    # umount /dev/sdb1

Scrivere l'immagine nella SD

    # dd if=./immagine.img of=/dev/sdX bs=4k

Eseguire questo comando per essere sicuri che tutta la cache sia scritta nell'SD

    # sync


## Installare pi-kiosk

Installare il programma di visualizzazione immagini

    $ sudo apt-get install feh unclutter

Collegarsi in ssh sulla rasp e posizionarsi nell home

    $ cd

Scaricare il software

    $ git clone https://github.com/teopost/pi-kiosk

Aggiornare il software

    $ sudo apt-get dist-upgrade
    $ sudo rpi-update
    $ sudo apt-get remove mathematica* sonic-pi wolfram*
    $ rm python_games

## Configurare il software

Rendere eseguibili gli script:

```
chmod 777 ./pi-kiosk/bin/*.sh
```

Per disabilitare lo screensaver editare il file autostart situato sotto /etc/xdg/lxsession/LXDE-pi. Quindi:

    sudo vi /etc/xdg/lxsession/LXDE-pi/autostart

quindi

```
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
# @xscreensaver -no-splash           # <-- COMMENTARE
@xset s off
@xset -dpms
@xset s noblank
@/home/pi/pi-kiosk/bin/slideshow.sh      # <-- AGGIUNGERE
```

Nel file, commentare la riga che contiene xscreensaver e aggiungere la riga in fondo per l'esecuzione automatica di pi-kiosk.

## Installazione di btsync

Per sincronizzare le immagini installare [btsync](http://getsync.com). Ovviamente la versione per ARM.

## Spegnimento automatico

Per spegnere e riaccendere automaticamente il televisore occorre installare la libreria cec per raspberry. Operazione da fare come root.

```
# apt-get instal cec-utils
# apt-get instal cec-client		+# apt-get instal cec-utils

# apt-get -y install udev libudev-dev autoconf automake libtool gcc liblockdev1
# Invece del git clone prendere questa versione : https://github.com/Pulse-Eight/libcec/tree/2a80b46be78e9d849de223ab73b6f3e7b4d9fc46
# cd libcec/
# ./bootstrap
# ./configure --with-rpi-include-path=/opt/vc/include --with-rpi-lib-path=/opt/vc/lib --enable-rpi
# cec-client
# make
# make install
# ldconfig
```

## Pianificare lo spegnimento e la riaccensione del TV

Nel crontab dell'utente pi, incollare le seguenti righe:

```bash
# .---------------- [m]inute: minuto (0 - 59)
# |  .------------- [h]our: ora (0 - 23)
# |  |  .---------- [d]ay [o]f [m]onth: giorno del mese (1 - 31)
# |  |  |  .------- [mon]th: mese (1 - 12) OPPURE jan,feb,mar,apr...
# |  |  |  |  .---- [d]ay [o]f [w]eek: giorno della settimana (0 - 6) (domenica=0 o 7)  OPPURE sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |

54 23 * * * /mnt/pi-kiosk/bin/turntv.sh off
30  7 * * * /mnt/pi-kiosk/bin/turntv.sh on && sleep 3 && /mnt/pi-kiosk/bin/turntv.sh input
```

Riferimenti
---
* http://raspberry-at-home.com/control-rpi-with-tv-remote/
* http://raspberrypi.stackexchange.com/questions/8698/how-can-my-raspberry-pi-turn-on-off-my-samsung-tv

## Aggiornamento del 20 maggio 2017
Btsync è morto. Poco male. [Sostituitelo con syncthing]({{< ref "syncthing-su-raspberrypi.md" >}})


{{% button href="https://github.com/teopost/pi-kiosk" icon="fa fa-github" %}}Get source code from Git-Hub{{% /button %}}
<br/>

<!--

Alcuni appunti:

* https://info-beamer.com/blog/raspberry-pi-hardware-video-scaler
* http://www.whizzy.org/wp-content/uploads/2012/11/cecsimple.sh_.txt

# lista comandi
echo h | cec-client -s -d 1

# Attiva la porta cec come attiva
echo "as" | cec-client -s

-->
