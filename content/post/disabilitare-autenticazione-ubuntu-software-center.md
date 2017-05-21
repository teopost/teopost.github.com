+++
banner = "banner/disabilitare-autenticazione-ubuntu-software-center.png"
categories = []
date = "2016-12-27T18:45:00+01:00"
description = ""
images = ["http://core0.staticworld.net/images/article/2015/08/ubuntu-software-center-whats-new-100609202-orig.png"]
menu = ""
tags = ["ubuntu"]
title = "Disabilitare l'autenticazione in Ubuntu Software Center"

+++

E' una rottura di scatole dover digitare sempre la password per installare gli aggiornamenti su Ubuntu.
Quindi, ecco cosa si può fare:

* Aprire una finestra terminale

* Incollare questo comando

  ```bash
  # (do not put .xml at the end...)
  sudo gedit /usr/share/polkit-1/actions/org.debian.apt.policy
  ```

Si aprirà gedit già posizionato per la modifica dell'impostazione desiderata (che è di proprietà di root)

* Tenere premuto CTRL e premere F, quindi nel box di ricerca incollare il seguente testo:

  ```
  org.debian.apt.install-or-remove-packages
  ```

* Sotto la linea <defaults>, modificare le 3 linee con questo:

  ```xml
  <allow_any>yes</allow_any>
  <allow_inactive>yes</allow_inactive>
  <allow_active>yes</allow_active>
  ```

* Cliccare Save and exit. Non è necessario far ripartire Ubuntu.

Da questo momento gli aggiornamenti e le installazioni di nuovo software non chiederanno più la password.
