+++
banner = "banner/syncthing-su-raspberrypi.png"
categories = ["linux"]
date = "2017-06-20T22:11:56+02:00"
description = ""
images = []
menu = ""
tags = ["raspberry"]
title = "Syncthing su Raspberrypi"

+++

Btsync era veramente un bel programma, ma dopo aver fatto venire l'acquolina in bocca l'hanno messo a pagamento.
La cosa peggiore tuttavia, restava il fatto che non fosse opensource.

<!--more-->

Per fortuna esiste una validissima alternativa che risolte questi 2 problemi. Syncthing.
Syncthing che è scritto in go ed è per questo molto veloce e adatto ad essere installato su dispositivi che non hanno molta memorie e potenza di calcolo come, ad esempio, la mia amata Raspberry Pi.
La cosa che contraddistingue Syncthing, rispetto altre soluzioni opensource simili, è la sua architettura che si appoggia su relay server in cloud per la sincronizzazione dei file.
In questo modo la sincronizzazione può avvenire senza dover aprire porte nel vostro router, esattamente come accade con programmi tipo Dropbox.

Prima di partire, tenete presente che alla fine di questa guida:
1. Syncthing dovrà funzionare sotto l'utente pi
2. Come conseguente del punto 1, i files di configurazione saranno creati sotto la home dell'utente pi (/home/pi/.config).
2. Le cartelle che potrebe condividere dovranno stare sotto la home dell'utente pi.

Bene. Vediamo ora come possiamo installarlo sulla Raspbian.

## Installazione

Innanzitutto scarichiamo la chiave gpg per poter aggiungere in seguito il repository debian.

```bash
wget -O - https://syncthing.net/release-key.txt | sudo apt-key add -
```

Ora aggiungiamo il repository

```bash
echo "deb http://apt.syncthing.net/ syncthing release" | sudo tee -a /etc/apt/sources.list.d/syncthing-release.list
```

Aggiorniamo il repository

```bash
sudo apt-get update
sudo apt-get install syncthing -y
```

Ora facciamo partire syncthing una prima volta. Questa operazione crea i files di configurazione nella home dell'utente con cui abbiamo lanciato il programma. Digitate quindi

```bash
syncthing
```

Dovreste vedere un output tipo questo

```bash
[monitor] 17:45:33 INFO: Starting syncthing
[start] 17:45:33 INFO: Generating RSA key and certificate for syncthing...
[HR2B5] 17:47:33 INFO: syncthing v0.11.12 (go1.4.2 linux-arm default) unknown-user@syncthing-builder 2015-07-05 09:24:21 UTC
[HR2B5] 17:47:33 INFO: My ID: HR2B56D-52WUAGB-36PBQRH-VBU3AAN-YS6SXIM-LJXVBZP-BR3CEMP-STI4EQW
[HR2B5] 17:47:33 INFO: No config file; starting with empty defaults
[HR2B5] 17:47:33 INFO: Edit /home/pi/.config/syncthing/config.xml to taste or use the GUI
[HR2B5] 17:47:33 INFO: Database block cache capacity 8192 KiB
[HR2B5] 17:47:33 OK: Ready to synchronize default (read-write)
[HR2B5] 17:47:33 INFO: Starting web GUI on http://127.0.0.1:8384/
[HR2B5] 17:47:33 INFO: Loading HTTPS certificate: open /home/pi/.config/syncthing/https-cert.pem: no such file or directory
[HR2B5] 17:47:33 INFO: Creating new HTTPS certificate
[HR2B5] 17:47:33 INFO: Generating RSA key and certificate for raspberrypi...
[HR2B5] 17:47:33 INFO: Completed initial scan (rw) of folder default
```

Ora fermiamo il programma premendo Ctrl-C

La configurazione standard di syncthing, prevede che l'accesso alla sua pagina web di configurazione, possa avvenire solo dalla macchina locale. Stiamo installando syncthing su una raspberry quindi è ovvio che dobbiamo accedervi da remoto. Apriamo quindi il file di configurazione per modificare tale impostazione.

```bash
vi /home/pi/.config/syncthing/config.xml
```

Nel file dobbiamo cambiare la riga con scritto 127.0.0.1 (solo accessibile dalla macchina locale) in 0.0.0.0 (accessibile da tutte le macchine)

Modificare tls in true se volete attivare l'ssl per l'interfaccia di configurazione.


```bash
<gui enabled="true" tls="false">
 <address>0.0.0.0:8384</address>
 <apikey>VbsKT2fCELYldTI74Tk4BKCbJP8Frlij</apikey>
</gui>
```

## Autostart di Syncthing su Raspberry Pi

Creare il file di init.d

```bash
sudo vi /etc/init.d/syncthing
```

Incollateci dentro questo:

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          Syncthing
# Required-Start:    $local_fs $remote_fs $network
# Required-Stop:     $local_fs $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Syncthing
# Description:       Syncthing is for backups
### END INIT INFO


# Documentation available at
# http://refspecs.linuxfoundation.org/LSB_3.1.0/LSB-Core-generic/LSB-Core-generic/iniscrptfunc.html
# Debian provides some extra functions though
. /lib/lsb/init-functions


DAEMON_NAME="syncthing"
DAEMON_USER=pi
DAEMON_PATH="/usr/bin/syncthing"
DAEMON_OPTS=""
DAEMON_PWD="${PWD}"
DAEMON_DESC=$(get_lsb_header_val $0 "Short-Description")
DAEMON_PID="/var/run/${DAEMON_NAME}.pid"
DAEMON_NICE=0
DAEMON_LOG='/var/log/syncthing'

[ -r "/etc/default/${DAEMON_NAME}" ] && . "/etc/default/${DAEMON_NAME}"

do_start() {
  local result

	pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
	if [ $? -eq 0 ]; then
		log_warning_msg "${DAEMON_NAME} is already started"
		result=0
	else
		log_daemon_msg "Starting ${DAEMON_DESC}" "${DAEMON_NAME}"
		touch "${DAEMON_LOG}"
		chown $DAEMON_USER "${DAEMON_LOG}"
		chmod u+rw "${DAEMON_LOG}"
		if [ -z "${DAEMON_USER}" ]; then
			start-stop-daemon --start --quiet --oknodo --background \
				--nicelevel $DAEMON_NICE \
				--chdir "${DAEMON_PWD}" \
				--pidfile "${DAEMON_PID}" --make-pidfile \
				--exec "${DAEMON_PATH}" -- $DAEMON_OPTS
			result=$?
		else
			start-stop-daemon --start --quiet --oknodo --background \
				--nicelevel $DAEMON_NICE \
				--chdir "${DAEMON_PWD}" \
				--pidfile "${DAEMON_PID}" --make-pidfile \
				--chuid "${DAEMON_USER}" \
				--exec "${DAEMON_PATH}" -- $DAEMON_OPTS
			result=$?
		fi
		log_end_msg $result
	fi
	return $result
}

do_stop() {
	local result

	pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
	if [ $? -ne 0 ]; then
		log_warning_msg "${DAEMON_NAME} is not started"
		result=0
	else
		log_daemon_msg "Stopping ${DAEMON_DESC}" "${DAEMON_NAME}"
		killproc -p "${DAEMON_PID}" "${DAEMON_PATH}"
		result=$?
		log_end_msg $result
		rm "${DAEMON_PID}"
	fi
	return $result
}

do_restart() {
	local result
	do_stop
	result=$?
	if [ $result = 0 ]; then
		do_start
		result=$?
	fi
	return $result
}

do_status() {
	local result
	status_of_proc -p "${DAEMON_PID}" "${DAEMON_PATH}" "${DAEMON_NAME}"
	result=$?
	return $result
}

do_usage() {
	echo $"Usage: $0 {start | stop | restart | status}"
	exit 1
}

case "$1" in
start)   do_start;   exit $? ;;
stop)    do_stop;    exit $? ;;
restart) do_restart; exit $? ;;
status)  do_status;  exit $? ;;
*)       do_usage;   exit  1 ;;
esac
```

Rendere il file eseguibile

```bash
sudo chmod +x /etc/init.d/syncthing
```

Impostare lo startup del servizio alla partenza della raspberrypi

```bash
sudo update-rc.d syncthing defaults
```
