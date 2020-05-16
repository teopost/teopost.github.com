+++
banner = "oracle-tde-transparent-data-encryption/oracle-tde-transparent-data-encryption.jpg"
categories = ["work"]
date = "2018-03-29T09:18:00+01:00"
description = ""
images = []
menu = ""
tags = ["oracle","database"]
title = "Oracle Transparent Data Encryption (TDE)"

+++

Ultimamente sono tutti molto preoccupati per una cosa chiamata [GDPR](https://it.wikipedia.org/wiki/Regolamento_generale_sulla_protezione_dei_dati), una legge europea che riguarda la protezione dei dati degli utenti (era ora dico io) che descrive una serie di adempimenti, non esclusivamente tecnici, da adottare per la salvaguardia dei dati.

<!--more-->

Il GDPR prevede responsabilità molto forti per chi manipola e mantiene tali dati, quindi se avete sui vostri server i databases dei vostri clienti, cominciate a tremare perchè le multe sono salatissime.

Siccome le responsabilità non riguardano esclusivamente chi detiene i dati, ma anche a chi fornisce il software per il loro trattamento, Oracle risolve questo problema con una serie di [moduli e funzionalità aggiuntive](https://www.oracle.com/it/applications/gdpr/index.html).

Ce ne sono parecchie, ma quella che interessa noi oggi è Oracle Transparent Data Encryption. Oracle TDE per gli amici.

## Cosa fa Oracle TDE

Quando un dato viene registrato fisicamente sul disco, le informazioni sono scritte in formato binario, ma possono essere identificate senza troppi problemi da un malintenzionato (se non ci credete, provate a usare il comando linux ```string```su un datafile).

Oracle TDE risolve il problema criptando tali dati e rendendoli illeggibili. Questo può essere fatto:

* Sulle singole colonne delle tabelle
* Su una intera [Tablespace](https://it.wikipedia.org/wiki/Tablespace)

Tralasciamo la modalità per colonna, che ha molte limitazioni in più rispetta a quella per tablespace (ad esempio non puoi creare indici sulle colonne criptate) e parliamo di quella per Tablespace.

Prima di cominciare preciso che:

1. Le cose scritte in questo articolo non si applicano ad un RAC, ma solo ad una installazione single instance basata su datafile (no ASM).
2. Le cose scritte in questo articolo valgono per la Oracle 12.2. I concetti sono identici nella versione 11,ma in comandi sono un po diversi.

## Creare il wallet

Per criptare i dati, è necessario creare un wallet, e per farlo occorre prima creare la cartella che deve ospitare tale wallet.

{{% notice info %}}
Oracle consiglia di creare il wallet all'esterno dell'albero di directory $ORACLE_BASE per evitare di archiviare accidentalmente il wallet con i dati crittografati su un supporto di backup. La cartella suggerita da Oracle è ```/etc/ORACLE/WALLETS/<$ORACLE_UNQNAME>```.
{{% /notice %}}

Per comodità da adesso in avanti sostituiamo <$ORACLE_UNQNAME> con DB01, quindi come utente root:

```
# cd /etc
# mkdir -pv oracle/wallets/DB01
# chown -R oracle:oinstall oracle
# chmod -R 700 oracle
```

## Configurare sqlnet.ora

A questo punto occorre dire ad Oracle si trova la cartella. Questa informazione si configura nel file sqlnet.ora, quindi:

```bash
# vi $ORACLE_HOME/network/admin/sqlnet.ora
```

e aggiungiamo queste righe:

```
ENCRYPTION_WALLET_LOCATION =
  (SOURCE = (METHOD = FILE)
    (METHOD_DATA =
      (DIRECTORY = /etc/oracle/wallets/DB01/)))
```

{{% notice info %}}
Se non diversamente configurato nel file sqlnet.ora, Oracle cerca il wallet in $ORACLE_BASE/admin/DB01/wallet/.
Si può verificare questa configurazione facendo una query sulla vista di sistema V$ENCRYPTION_WALLET prima di modificare il file sqlnet.ora.
E' importante sapere che se si modifica tale percorso (come suggerito nell'articolo), occorre effettuare uno shutdown e uno statup dell'istanza per far leggere ad Oracle la nuova configurazione.
{{% /notice %}}

## Creare il wallet

Entrare come utente SYS o comunque con un utente che abbia il ruolo SYSKM, ed eseguire:

```bash
# sqlplus / as sysdba
```

Verificare la compatibilità del DB. Deve essere almeno 12.x

```sql
SQL> show parameter compatible
NAME                TYPE     VALUE
----------------------------------------
compatible          string   12.1.0.2.0
noncdb_compatible   boolean  FALSE
```

A questo punto possiamo creare il wallet (file p12 nella cartella wallet) specificando una master key password:

```sql
SQL> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/u01/app/oracle/admin/DB01/wallet' identified by walletpwd;
keystore altered.
SQL> host ls /etc/oracle/wallets/DB01/
ewallet.p12
```

{{% notice warning %}}
walletpwd è la password del wallet. Non perdetela assolutamente!!!!
{{% /notice %}}

## Aprire il wallet

Apriamo il wallet. Il wallet rimane aperto fino a quando non si fa lo shutdown (a meno che non venga creato di tipo auto_login).
Chiudere il wallet permette di disabilitare l'accesso alla masterkey e di inibile l'accesso ai dati criptati.

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY walletpwd;
```

## Creare e attivare la masterkey

Ora che il wallet è aperto, occorre creare la chiave (master encryption key).
Il seguente comando crea la chiave e contestualmente, per sicurezza, ne crea anche una copia di backup.
Se si sputtana questo file sono [volatili per diabetici](https://www.youtube.com/watch?v=KeCwQjhX-R8)

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY walletpwd WITH BACKUP USING 'key_backup';
```

## Creare la Tablespace Encrypted

Per creare la tablespace encripted usare questa sintassi

```sql
CREATE TABLESPACE USERS_CRYPT
    DATAFILE
        '/u01/app/oracle/oradata/DB01/users01_crypt.dbf'
        SIZE 5242880 AUTOEXTEND ON NEXT 1310720 MAXSIZE 34358689792
        BLOCKSIZE 8192
    DEFAULT
      NO INMEMORY   
      STORAGE (ENCRYPT)
    ONLINE
    SEGMENT SPACE MANAGEMENT AUTO
    EXTENT MANAGEMENT LOCAL AUTOALLOCATE
        ENCRYPTION USING 'AES256';
```

Et voilà. Da adesso in poi, tutti gli oggetti creati su questa tablespace verranno scritti sul filesystem in modo criptato.
Poichè non è possibile criptare una tablespace esistente, per mettere i dati nella nuova tablespace ci sono 2 modi:

1. Un Export (expdp) ed un Import (impdb) con un remap della tablespace
2. Muovere gli oggetti con il comando ```alter table TABLE_NAME move tablespace TABLESPACE_ENCRYPTED```


## Creare un wallet AUTO_LOGIN

Per evitare di dover riaprire sempre manualmente il wallet, è possibile crearlo di tipo AUTOLOGIN. Un Wallet autologin si riapre automaticamente ogni volta che il database viene fatto ripartire.

Il comando è il seguente:

```sql
SQL> ADMINISTER KEY MANAGEMENT CREATE AUTO_LOGIN KEYSTORE FROM KEYSTORE '/u01/app/oracle/admin/INTDB01/wallet' IDENTIFIED BY walletpwd;
```

## Disabilitare l'AUTO_LOGIN da un wallet esistente

Un wallet autologin è comodo, ma è molto meno sicuro. Un malintenzionato puo' accedere ai dati semplicemente facendo ripartire l'istanza.
Se volete disabilitare l'AUTO_LOGIN potete sempre farlo in questo modo:

Andare nella cartella del wallet. Noterete un file chiamato cwallet.sso.
Questo file viene creato solo quando il wallet viene generato di tipo auto_login.
Per prima cosa rinominate il file:

```bash
mv cwallet.sso cwallet.sso.bkp
```

Poi entrate come ```sqlplus / as sysdba``` e chiudete il wallet aperto.

```sql
SQL> ALTER SYSTEM SET WALLET CLOSE;
```

Fatto. Per riaprire il wallet potete sempre eseguire il seguente comando:

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY walletpwd;
```

## Conclusioni

Indubbiamente Oracle TDE è una funzionalità molto interessante, fate molta attenzione, ma soprattutto:


1. Non perdete mai la masterkey, se no sono grossi guai.
2. Fate mille backup della cartella che contiene il wallet, se no sono grossi guai.
3. Gli ambienti di produzione non mai il posto ideale per imparare queste cose.
4. Criptare e decriptare i dati ha un costo in termini di CPU, anche se minimo.
5. Oracle è progettato per funzionare sempre. La sua architettura consente una alta disponibilità come nessun altro database. Pensare che la perdita di un singolo piccolo e insignificante file .p12 possa impedirvi di recuperare i dati fa venire i brividi.
