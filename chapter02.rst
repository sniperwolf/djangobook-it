=========================
Capitolo 2: Come iniziare
=========================

Installare Django è un processo multi-fase, a causa delle parti multiple mobili
in moderni ambienti di sviluppo web. In questo capitolo, ti guideremo attraverso
l'installazione del framework e le sue poche dipendenze.

Poiché Django è "solo" codice Python, funziona ovunque gira Python -- tra cui
alcuni telefoni cellulari! Ma questo capitolo riguarda solo gli scenari comuni
in cui installare Django. Qui si suppone che si voglia installare Django su un
computer desktop/laptop o un server.

Più tardi, nel capitolo 12, vedremo come distribuire Django su un sito di
produzione.

Installare Python
=================

Django stesso è scritto in puro Python, quindi il primo passo per
l'installazione del framework è quello di assicurarsi di avere installato Python.

Versioni di Python
------------------

Il nucleo di Django (versione 1.4 +) funziona con qualsiasi versione di Python
dalla 2.5 alla 2.7 compresa. Il sistema opzionale GIS (Geographic Information
Systems) di Django richiede Python dalla versione 2.5 alla 2.7.

Se non siete sicuri di quale versione di Python installare ed hai la possibilità
di prendere una qualunque decisione, scegli quello più recente della serie 2.x:
la versione 2.7. Anche Django funziona altrettanto bene con qualsiasi versione
2.5-2.7, le versioni di Python hanno miglioramenti delle prestazioni ed alcune
caratteristiche aggiunte che si potrebbero voler utilizzare per le proprie
applicazioni. Inoltre, alcuni componenti aggiuntivi di Django che si potrebbe
voler utilizzare, potrebbero richiedere una versione più recente della 2.5,
quindi usare una versione più recente di Python mantiene aperte diverse opzioni.

.. admonition:: Django e Python 3.x

    Al momento della scrittura, Python 3.3 è stato rilasciato, ma Django
    la supporta solo sperimentalmente. Questo perché fa parte della serie Python
    3.x, che introduce un numero significativo di modifiche non retrocompatibili
    al linguaggio stesso, e, di conseguenza, molte grandi librerie Python e
    framework, tra cui Django (dalla versione 1.4), non hanno ancora raggiunto.

    Django 1.5 è previsto per supportare Python 2.6, 2.7 e 3.2. tuttavia,
    il supporto per la 3.2 è considerata un "anteprima", il che significa che gli
    sviluppatori non sono ancora abbastanza sicuri da promettere stabilità nella
    produzione. Pertanto, si suggerisce attendere Django 1.6 prima di passare a
    Python 3.x.

Installazione
-------------

Se siete su Linux o Mac OS X, probabilmente avete già installato Python.
Digita ``python`` al prompt dei comandi (o in Applicazioni/Utility/Terminale, in
OS X). Se vedi qualcosa di simile, allora Python è installato::

    Python 2.7.3rc2 (default, Apr 22 2012, 22:30:17)
    [GCC 4.6.3] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

In caso contrario, sarà necessario scaricare e installare Python. E' veloce e
semplice, e le istruzioni dettagliate sono disponibili all'indirizzo
http://www.python.org/download/

Installare Django
=================

In qualsiasi momento, due versioni distinte di Django sono a tua
disposizione: l'ultima release ufficiale e la versione di sviluppo bleeding-edge.
La versione da installare dipende dalle tue priorità. Vuoi una versione
stabile e testata di Django, o vuoi una versione con tutte le ultime
caratteristiche, magari per poter contribuire a Django stesso, a spese della
stabilità?

Vi consigliamo di iniziare con una release ufficiale, ma è importante sapere
che la versione di sviluppo esiste, perché lo troverete citato
nella documentazione e dai membri della comunità.

Installare la Release Ufficiale
-------------------------------

Le release ufficiali hanno un numero di versione, come ad esempio 1.4.2, 1.4.1 o
1.4, e l'ultima è sempre disponibile all'indirizzo
http://www.djangoproject.com/download/.

Se siete su una distribuzione Linux che include un pacchetto di Django, è un
buona idea utilizzare la versione del distributore. In questo modo, avrai la
sicurezza di aggiornare Django con il resto dei tuoi pacchetti di sistema.

Se non si ha accesso a una versione preconfezionata, è possibile scaricare e
installare il framework manualmente. Per fare ciò, prima scaricare il tarball,
che avrà un nome simile a ``Django-1.4.2.tar.gz`` (Non importa in quale
directory locale si scarica questo file, il processo di installazione metterà
tutti i file di Django al posto giusto). Poi, decomprimerlo ed eseguire
``setup.py install``, come si fa con la maggior parte delle librerie Python.

Ecco come si presenta questo processo nei sistemi Unix:

#. ``tar xzvf Django-1.4.2.tar.gz``
#. ``cd Django-*``
#. ``sudo python setup.py install``

In Windows, si consiglia di utilizzare 7-Zip (http://www.djangoproject.com/r/7zip/)
per decomprimere i file ``.tar.gz``. Una volta decompresso il file, avviare una
shell DOS (il "Prompt dei comandi") con i privilegi di amministratore ed
eseguire il seguente comando all'interno della directory il cui nome inizia con
``Django-``::

    python setup.py install

Nel caso tu sia curioso: i file di Django saranno installati nella tua directory
``site-packages`` di Python -- una directory in cui Python cerca per trovare le
librerie di terze parti. Di solito è in un posto come
``/usr/lib/python2.7/site-packages``.

Installare la Versione "Development"
------------------------------------

Django utilizza Git (http://git-scm.com) per il controllo del codice sorgente.
L'ultima e più grande versione di sviluppo Django è disponibile dalla repository
Git ufficiale di Django (https://github.com/django/django). Si dovrebbe prendere
in considerazione l'installazione di questa versione se si vuole lavorare sulle
ultime novità, o se si vuole contribuire al codice di Django stesso.

Git è un sistema libero, open source per la distribuzione, la revisione ed il
controllo, ed il team di Django lo utilizza per gestire le modifiche al codice
di base. È possibile scaricare e installare Git dall'indirizzo
http://git-scm.com/download, ma è più semplice da installare con il gestore dei
pacchetti del sistema operativo (su usi Linux). È possibile utilizzare Git per
scaricare il codice di Django più recente e, in un qualunque momento, è
possibile aggiornare la versione in locale del codice di Django per ottenere gli
ultimi cambiamenti e miglioramenti fatti dagli sviluppatori.

Quando si utilizza la versione di sviluppo, è bene tenere a mente che non ci
sono garanzie che qualcosa vada storta in un particolare momento. Detto questo,
però, alcuni membri della team usando su siti di produzione la versione di
sviluppo, in modo da avere un incentivo per mantenerlo stabile.

Per scaricare l'ultima Django, leggi la seguente procedura:

#. Assicurarsi di avere installato Git. È possibile il software gratuito
    all'indirizzo http://git-scm.com/, mentre una documentazione eccellente si
    trova all'indirizzo http://git-scm.com/documentation.

#. Clonare il repository con il comando
    ``git clone https://github.com/django/django djmaster``

#. Individuare la directory ``site-packages`` nella tua installazione di Python.
    Solitamente è in un posto simile a ``/usr/lib/python2.7/site-packages``.
    Se non hai alcuna idea, digita questo comando dal prompt dei comandi::

       python -c 'import sys, pprint; pprint.pprint(sys.path)'

    L'output risultante dovrebbe includere il tuo ``site-packages``.

#. All'interno di ``site-packages``, creare un file chiamato
    ``djmaster.pth``e modificarlo in modo che contenga il percorso completo
    alla tua directory ``djmaster``. Ad esempio, il file potrebbe semplicemente
    contenere questa riga::

       /path/to/djmaster

#. Localizza ``djmaster/django/bin`` sul tuo percorso di sistema. Questa
   directory include alcune utility di gestione come ad esempio
   ``django-admin.py``.

.. admonition:: Consiglio:

    Se i file ``.pth``  sono una novità per te, puoi imparare qualcosa di più
    sul loro conto alla pagina
    http://www.djangoproject.com/r/python/site-module/.

Dopo aver scaricato il tutto da Git e seguendo i passaggi precedenti, non è
necessario eseguire ``python setup.py install`` -- hai appena fatto il lavoro a
mano!

Poiché il codice Django cambia spesso con correzioni di bug e aggiunte,
ti consigliamo di aggiornarlo di tanto in tanto. Per aggiornare il codice,
basta eseguire il comando ``git pull origin master`` dall'interno della
directory ``djmaster`` Quando si esegue questo comando, Git si metterà in
contatto con https://github.com/django/django per determinare se qualche parte
di codice è cambiata e quindi aggiornare la versione locale con le eventuali
modifiche che sono state compiuti dopo l'ultimo aggiornamento. E 'abbastanza
semplice.

Infine, se si utilizza la versione di sviluppo Django, si dovrebbe sapere come
con quale versione di Django si sta lavorando. Conoscere il proprio numero di
versione è importante se mai avessi bisogno di entrare in contatto con la
comunità per chiedere aiuto, o volessi presentare miglioramenti al framework.
In questi casi, si dovrebbe comunicare la revisione, detto anche "commit", della
versione in uso. Per scoprire l'attuale commit della tua versione, basta
digitare "git log -1" all'interno della directory ``django``, e cercare
l'identificatore dopo "commit". Questo numero cambia ogni volta che Django è
viene modificato, o per di una correzione di bug, o per l'aggiunta di nuove
feature, o per il miglioramento documentazione o qualsiasi altra cosa.

Testare l'installazione di Django
=================================

Per capire se tutto è filato liscio, prenditi un momento per verificare se la
installazione ha fatto il suo lavoro. In una shell dei comandi, cambiare
directory (ad esempio, *non* la directory che contiene ``django``) e avviare
l'interprete interattivo di Python digitando ``python``. Se l'installazione ha
avuto successo, dovresti essere in grado di importare il modulo ``django``:

    >>> import django
    >>> django.VERSION
    (1, 4, 2, 'final', 0)

.. admonition:: Esempi con l'interprete interattivo

    L'interprete interattivo di Python è un programma a riga di comando che
    permette di scrivere un programma Python interattivamente. Per avviarlo,
    eseguire il comando ``python`` dalla riga di comando.

    In questo libro, useremo scrivere esempi funzionali usando questo interprete.
    È possibile riconoscere questi esempi dal triplo segno di maggiore (``>>>``),
    che simbolizza il prompt dell'interprete. Se vuoi copiare gli esempi di
    questo libro, non copiare quei segni di maggiore.

    Dichiarazioni multilinea nell'interprete interattivo sono aggiunti con tre
    puntini (``...``). Per esempio::

        >>> print """This is a
        ... string that spans
        ... three lines."""
        This is a
        string that spans
        three lines.
        >>> def my_function(value):
        ...     print value
        >>> my_function('hello')
        hello

    Quei tre puntini all'inizio delle linee aggiuntive vengono inserite dalla
    Shell di Python -. quindi sono parte del nostro input. Noi li includiamo qui
    per esser fedeli all'effettivo output dell'interprete. Se vuoi copiare i
    nostri esempi a seguire, non copiare quei puntini.

Creare ed impostare un Database
===============================

A questo punto, si potrebbe benissimo iniziare a scrivere un'applicazione web
con Django, perché il solo prerequisito di Django è una installazione
funzionante di Python. Tuttavia, è probabile che tu debba sviluppare un sito web
*basato su database*, nel qual caso è necessario configurare un database server.

Se si desidera solo iniziare a giocare con Django, salta alla sezione successiva
"Avviare un Progetto" -- ma tieni presente che tutti gli esempi in questo
libro suppongono che tu abbia creato un database sul quale lavorare.

Django supporta quattro tipi di database engine:

* PostgreSQL (http://www.postgresql.org/)
* SQLite 3 (http://www.sqlite.org/)
* MySQL (http://www.mysql.com/)
* Oracle (http://www.oracle.com/)

Per la gran parte, tutti i motori qui elencati funzionano altrettanto bene con
il nucleo Django framework (un'eccezione di rilievo va sollevata per il supporto
opzionale GIS di Django, che è molto più potente con PostgreSQL che con altri
database). Se non sei legato ad un particolare sistema legacy ed hai libertà di
scegliere un qualunque database backend, ti raccomandiamo PostgreSQL, che si
basa su un giusto equilibrio tra costi, caratteristiche, velocità e stabilità.

L'impostazione del database è un processo che si svolge in due fasi:

* In primo luogo, è necessario installare e configurare il database server
  stesso. Questo processo va oltre lo scopo di questo libro, ma ciascuno dei
  quattro database qui elencati ha ricca documentazione sul suo sito web.
  (Se sei su un hosting condiviso, è molto probabile che il tuo fornitore abbia
  già impostato tutto questo per te).

* In secondo luogo, è necessario installare la libreria Python per il tuo
  particolare database backend. Questo è fatto tramite codice di terze parti che
  permette a Python di relazionarsi con il database. Descriveremo tutte le
  specifiche da soddisfare per ogni database nelle seguenti sezioni.

Se stai solo giocando con Django e non si desidera installare un database
server, considera l'utilizzo di SQLite. SQLite è unico nella lista dei database
supportati che non richiede nessuno dei due passaggi precedenti. Esso si limita
a leggere e scrivere i dati su un unico file nel filesystem e le versioni di
Python dalla 2.5 lo supportano nativamente.

Su Windows, ottenere i file binari per i database può essere frustrante. Se vuoi
saltare queste fasi, si consiglia di utilizzare Python 2.7 con SQLite.
(N.d.T. Gli utenti Windows hanno la possibilità di usare ActiveState Python, una
versione modificata di Python che integra in sé un sistema di gestione dei
pacchetti/moduli simile a pip che salta la fase di compilazione dei sorgenti
poiché prende tutto da una vasta repository di utili file binari. Questa
repository contiene tutti i moduli che sono descritti di seguito, compreso il
mitico ``MySQLdb``).

Usare Django con PostgreSQL
----------------------------

Se stai usando PostgreSQL, è necessario installare sia il pacchetto ``psycopg``
o ``psycopg2`` dall'indirizzo http://www.djangoproject.com/r/python-pgsql/. Noi
raccomandiamo ``psycopg2``, poiché più recente, più attivamente sviluppato e più
semplice da installare. In entrambi i casi, ricorda se stai utilizzando la
versione 1 o la 2; avrai bisogno di questa informazioni in futuro.

Se stai usando PostgreSQL su Windows, si possono trovare binari precompilati di
``psycopg`` all'indirizzo http://www.djangoproject.com/r/python-pgsql/windows/.

Se sei su Linux, verifica che il gestore di pacchetti presente nella tua
distribuzione offra un pacchetto chiamato "python-psycopg2", "psycopg2-python",
"python-PostgreSQL" o qualcosa di simile.

Usare Django con SQLite 3
--------------------------

Sei fortunato: non è richiesta alcuna installazione specifica del database,
perché Python supporta nativamente SQLite. Salta alla sezione successiva.

Usare Django con MySQL
-----------------------

Django richiede MySQL 4.0 o superiore. Le versioni 3.x non supportano le
subquery annidate e alcune altre istruzioni SQL abbastanza standard.

Hai anche bisogno di installare il pacchetto ``MySQLdb`` dall'indirizzo
http://www.djangoproject.com/r/python-mysql/.

Se sei su Linux, verifica che il sistema di gestione dei pacchetti presente
nella tua distribuzione offra un pacchetto chiamato "python-mysql",
"python-mysqldb", "mysql-python" o qualcosa di simile.

Usare Django con Oracle
------------------------

Django lavora con le versioni 9i e superiori dei server database Oracle.

Se si utilizza Oracle, è necessario installare la libreria ``cx_Oracle``,
disponibile all'indirizzo http://cx-oracle.sourceforge.net/. Utilizza la
versione 4.3.1 o superiore, ma evita la versione 5.0 per via di un bug presente
in questa versione del driver. La versione 5.0.1 ha risolto il bug, in modo da
poter scegliere una versione superiore qualunque.

Usare Django senza Database
-------------------------------

Come accennato in precedenza, Django in realtà non richiede un database.
Se vuoi usarlo solo per fornire pagine dinamiche che non necessitano database,
funziona bene allo stesso modo.

Detto questo, tieni a mente che alcuni degli strumenti extra compresi in Django
*richiedono* un database, quindi se si sceglie questa strada, ti perdi queste
caratteristiche (che evidenziamo in questo libro).

Avviare un Progetto
===================

Una volta installato Python, Django e (opzionalmente) la libreria del database
server, si può compiere il primo passo nello sviluppo di una applicazione con la
creazione di un *progetto*.

Un progetto è quell'insieme di impostazioni di un'istanza di Django, comprese le
opzioni di configurazione, database specifici per Django e le impostazioni delle
terze delle applicazioni.

Se questa è la prima volta che si usa Django, bisogna compiere alcune
configurazione iniziale. Creare una nuova directory per iniziare a lavorare,
qualcosa come ``/home/username/djcode/``.

.. admonition:: Dove mettere tutta la roba per andare online?

    Se sei un programmatore PHP, probabilmente sei abituato a mettere il codice
    nella root del server Web (un posto come /var/www). Con Django, non farlo.
    Non è una buona idea mettere tutto questo codice Python all'interno di
    document root del server Web, perché così facendo si rischia la possibilità
    che qualcuno sia in grado di visualizzare il codice sorgente grezzo/raw sul
    web. E questo non va bene.

    Inserire il codice in una directory **al di fuori** della root dei documenti.

Spostarsi nella directory appena creata, ed eseguire il comando
``django-admin.py startproject mysite``. Questo creerà una directory ``mysite``
nella directory corrente.

.. note::

    ``django-admin.py`` dovrebbe essere presente sul tuo percorso di sistema se
    hai installato Django attraverso ``setup.py`` o pip.

    Se stai utilizzando la versione di sviluppo, troverai ``django-admin.py`` in
    ``djmaster/django/bin``. Poiché userai spesso ``django-admin.py``, può
    essere comodo aggiungerlo al percorso di sistema. Su Unix, è possibile farlo
    con un collegamento simbolico da ``/usr/local/bin`` usando un comando come
    ``sudo ln -s /path/to/django/bin/django-admin.py /usr/local/bin/django-admin.py``.
    In Windows, è necessario aggiornare la variabile d'ambiente ``PATH``.

    Se Django è stato installato da una versione impacchettata di una
    distribuzione Linux, ``django-admin.py`` potrebbe essere chiamato
    ``django-admin``.

Se viene visualizzato il messaggio di un "permission denied" ("permesso negato"
in inglese) durante l'esecuzione di ``django-admin.py startproject``, è
necessario modificare le autorizzazioni del file. Per fare ciò, passare alla
directory in cui è installato ``django-admin.py`` (ad esempio,
``cd /usr/local/bin``) ed eseguire il comando ``chmod +x django-admin.py``.

Il comando ``startproject`` crea una directory che contiene cinque file::

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

.. note:: Non corrisponde a ciò che vedi?

    Il layout di progetto predefinito è recentemente cambiato. Se stai vedendo
    un layout "piatto" (senza ``mysite`` come directory interna), probabilmente
    stai usando una versione di Django che non corrisponde alla versione usata
    nel tutorial. Questo libro tratta Django 1.4 e superiori, quindi se stai
    usando una versione precedente, è meglio consultare la documentazione
    ufficiale di Django.

    La documentazione per la versione 1.x Django è disponibile
    presso https://docs.djangoproject.com/en/1.X/.

Questi file sono i seguenti:

* ``mysite/``: La directory ``mysite/`` è solo un contenitore per il
  tuo progetto. Il suo nome non importa a Django, è possibile rinominarlo
  in qualcosa che ti piace senza provocare danni.

* ``manage.py``: È un programma da riga di comando che permette di interagire
  con questo progetto Django in vari modi. Digita ``python manage.py help``
  per avere un'idea di quello che può fare; non dovrebbe mai essere necessario
  modificare questo file, è creato in questa directory solo per convenienza.

* ``mysite/mysite/``: La directory interna ``mysite/`` è il pacchetto Python
  effettivo per il progetto. Il suo nome è il nome del "pacchetto" ed è
  necessario utilizzare questo nome per importare qualsiasi cosa al suo
  interno (ad esempio, ``import mysite.settings``).

* ``__init__.py``: Un file richiesto da Python per trattare la directory
  ``mysite`` come un pacchetto (ad esempio, un gruppo di moduli Python). E' un
  file vuoto, e generalmente non si aggiunge nulla.

* ``settings.py``: Sono le impostazioni/configurazioni per questo progetto.
  Dai un'occhiata a questo per avere un'idea dei tipi di impostazioni disponibili,
  insieme ai loro valori di default.

* ``urls.py``: Il gestore di URL per questo progetto Django. Pensa a questo come
  il "sommario" del tuo sito Django-powered.

* ``wsgi.py``: un entry-point per i server web compatibili con WSGI. Sul suo
  utilizzo, leggere "How to deploy with WSGI" (https://docs.djangoproject.com/en/1.4/howto/deployment/wsgi/)
  per maggiori dettagli.

Nonostante le piccole dimensioni, questi file già costituiscono una applicazione
Django su cui lavorare.

Avviare il Server per lo Sviluppo
---------------------------------

Per ottenere un feedback sull'effettiva funzionalità del progetti, eseguiamo il
server di sviluppo per vedere la nostra applicazione in azione.

Il server di sviluppo Django (chiamato "runserver" come il comando che lo
lancia) è un web server integrato leggero, che è possibile utilizzare durante lo
sviluppo del sito. E' incluso con Django in modo da poter sviluppare il tuo sito
rapidamente, senza avere a che fare con la configurazione del server di
produzione (ad esempio, Apache) fino a quando non si è pronti per la produzione.
Il server di sviluppo interpreta il codice caricandolo automaticamente, rendendo
più facile modificare il codice senza bisogno di riavviare nulla.

Per avviare il server, passa alla directory che contiene il progetto
(``cd mysite``), se già non lo sei, ed esegui questo comando::

    python manage.py runserver

Vedrai qualcosa di simile a questo::

    Validating models...
    0 errors found.

    Django version 1.4.2, using settings 'mysite.settings'
    Development server is running at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.

Questo lancia il server in locale, sulla porta 8000, accessibile tramite
semplice connessione dal proprio computer. Ora che è in esecuzione, visita
l'indirizzo http://127.0.0.1:8000/ con un browser web qualunque. Si potrebbe
vedere una versione di Django diversa a seconda della versione di Django
installata. Vedrai una pagina "Welcome to Django" ombreggiato in un piacevole
blu pastello. Ha funzionato!

Una ultima, importante nota sul server di sviluppo che vale la pena ricordare
prima di procedere. Anche se questo server è conveniente per lo sviluppo,
bisogna resistere alla tentazione di usarlo in qualcosa di simile ad un ambiente
di produzione. Il server di sviluppo è in grado di gestire una sola richiesta
alla volta in maniera affidabile, e non è testabile a livello sicurezza di
qualsiasi tipo. Quando arriva il momento di lanciare il tuo sito, leggi il
Capitolo 12 per conoscere le informazioni per distribuire Django.

.. admonition:: Modificare Host del server di sviluppo o la porta

    Per impostazione predefinita, il comando ``runserver`` avvia il server di
    sviluppo sulla porta 8000, tipica delle connessioni locali. Se si desidera
    cambiare la porta del server, basta passarla come argomento della riga di
    comando::

        python manage.py runserver 8080

    Specificando un indirizzo IP, è possibile diffondere Django attraverso
    connessioni non locali. Ciò è particolarmente utile se vuoi condividere un
    sito di sviluppo con gli altri membri della tua squadra. L'indirizzo IP
    ``0.0.0.0`` indica al server di inviare a tutto qualsiasi interfaccia di
    rete::

        python manage.py runserver 0.0.0.0:8000

    Fatto questo, gli altri computer della rete locale saranno in grado di
    visualizzare il tuo sito Django, visitando il tuo indirizzo IP con un
    browser, ad esempio, http://192.168.1.103:8000/. (Per informazioni più
    specifiche, consultare le proprie impostazioni di rete per determinare
    l'indirizzo IP sulla rete locale. Gli utenti Unix, possono provare
    "ifconfig" nel prompt dei comandi per ottenere queste informazioni comando.
    Gli utenti di Windows, possono provare con "ipconfig")

Cosa c'è adesso?
================

Ora che abbiamo installato tutto ed abbiamo il server di sviluppo in esecuzione,
si è pronti ad :doc: imparare le basi nel `Capitolo 3`_, di servire le pagine web con
Django.

.. _Chapter 3: chapter03.html