+++
banner = "banner/ffmpeg-su-raspberrypi.png"
categories = ["linux"]
date = "2017-05-27T16:18:40+02:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "Installare FFmpeg sulla Raspberry Pi 3"

+++
Purtroppo installare FFmpeg sulla Raspberry non è mai una cosa semplice. Non si capisce per quale motivo non si decidono a mettere i compilati nel repository.
Ad ogni modo riporto qui i miei appunti su come farlo partendo dai sorgenti.

<!--more-->

Tutte le seguenti operazioni vanno fatte come utente root (`sudo -s`).

Per prima cosa occorre installare git col solito comando:

    apt-get install git

Dopo aver installato `git` procediamo con lo scaricamento dei sorgenti.

## Compilazione e installazione libreria video x264

Visto che ci siamo installiamo le librerie video x264, quindi:

    cd /usr/src
    git clone git://git.videolan.org/x264
    cd x264
    ./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
    make
    make install

Se dovesse essere necessario installare il supporto FFmpeg per altre librerie (mp3, aac, ecc..), bisogna farlo ora. Prima di compilare ffmpeg.

## Compilazione e installazione FFmpeg:

    cd /usr/src
    git clone git://source.ffmpeg.org/ffmpeg.git
    cd ffmpeg/
    ./configure --arch=armel --target-os=linux --enable-gpl --enable-libx264 --enable-nonfree
    make

Notate l'opzione `--enable-libx264` che compila ffmpeg con il supporto alle suddette librerie

Questa seconda fase è un po lunghina. Fatela mentre vi guardate un film. Sulla Raspberry 3 potete eseguire la compilazione parallela `make -j4`. Dovrebbe velocizzare un po la cosa.

Per ultimo:

    make install

L'installazione di FFmpeg comprende anche il tool ffserver che permette di fare lo streaming di video e audio.
