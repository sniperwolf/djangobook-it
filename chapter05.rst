===================
Capitolo 5: Modelli
===================

Nel Capitolo 3, abbiamo parlato delle basi della costruzione di siti web
dinamici con Django: la creazione di view/viste e URLconfs. Come abbiamo
spiegato, la view si occupa della *logica arbitraria*, e il ritorna una risposta.
In uno degli esempi, la nostra logica arbitraria è stata calcolare la data e ora
correnti.

Nelle moderne applicazioni Web, la logica arbitraria spesso comporta
l'*interazione con un database*. Dietro le quinte, un sito web basato su
database si connette a un server di database, recupera alcuni dati fuori di esso,
e li mostra una pagina web. Il sito potrebbe anche fornire modi ai visitatori
del sito di popolare il database.

Molti siti Web complessi forniscono una combinazione dei due. Amazon.com, per
esempio, è un grande esempio di un sito basato su database. Ogni pagina del
prodotto è essenzialmente una query nel database di prodotti di Amazon in
formato HTML, e quando scrivi una recensione cliente, esso viene inserito nel
database di controllo.

Django è adatto per la produzione di siti web basati su database, perché dispone
di diversi strumenti semplici ma potenti per l'esecuzione di query di database
utilizzando Python. Questo capitolo spiega questa funzionalità: il livello
(layer in inglese) database di Django.


(Nota: Anche se non è strettamente necessario conoscere la teoria dei database
relazionali di base e SQL per utilizzare layer di database di Django, è
altamente consigliato una introduzione a questi concetti, va oltre lo scopo di
questo libro, ma continuate a leggere, anche se sei un principiante del database.
Probabilmente sarei in grado di seguire e comprendere i concetti in base al
contesto).

La strada "stupida" di fare Query al Database in una View
=========================================================

Proprio come nel Capitolo 3 abbiamo  un modo "stupido" per produrre output
all'interno di una vista (codificando il testo direttamente all'interno della
vista), c'è un modo "stupido" per recuperare i dati da un database in una view.
E' semplice: basta usare qualsiasi libreria Python esistente per eseguire una
query SQL e fare qualcosa con i risultati.

In questa prospettiva, ad esempio, si usa la libreria ``MySQLdb`` (disponibile
all'indirizzo http://www.djangoproject.com/r/python-mysql/) per connettersi a un
database MySQL, recuperare alcuni record, e dar loro come cibo a un template per
la visualizzazione come una pagina web::

    from django.shortcuts import render
    import MySQLdb

    def book_list(request):
        db = MySQLdb.connect(user='me', db='mydb', passwd='secret', host='localhost')
        cursor = db.cursor()
        cursor.execute('SELECT name FROM books ORDER BY name')
        names = [row[0] for row in cursor.fetchall()]
        db.close()
        return render(request, 'book_list.html', {'names': names})

Questo approccio funziona, ma alcuni problemi dovrebbero saltare all'occhio
subito:


* We're hard-coding the database connection parameters. Ideally, these
  parameters would be stored in the Django configuration.

* Stiamo codificando i parametri di connessione al database. Idealmente, questi
  parametri saranno memorizzati nella configurazione Django.

* Siamo costretti a dover scrivere un bel po' di codice standard: creazione di
  una connessione, creazione di un cursore, esecuzione di una dichiarazione e
  chiusura della connessione. Idealmente, tutto ciò è necessario per fare ciò
  che vogliamo.

* E ci lega a MySQL. Se, lungo il cammino, si passa da MySQL a PostgreSQL,
  dovremo utilizzare un adattatore di database diverso (ad esempio, ``psycopg``
  piuttosto che ``MySQLdb``), modificare i parametri di connessione, e .- a
  seconda della natura della dichiarazione SQL - forse riscrivere l'SQL.
  Idealmente, il server di database che stiamo usando è astratto, cosicché un
  cambiamento di database sia fatto in un unico luogo. (Questa funzione è
  particolarmente rilevante se si sta costruendo un'applicazione Django
  open-source che si desidera utilizzare dal maggior numero possibile di
  persone).

Come ci si potrebbe aspettare, il layer database di Django mira a risolvere
questi problemi. Ecco un'anteprima di come la vista precedente può essere
riscritta utilizzando le API di Django::

    from django.shortcuts import render
    from mysite.books.models import Book

    def book_list(request):
        books = Book.objects.order_by('name')
        return render(request, 'book_list.html', {'books': books})

Spiegheremo questo codice un po' più avanti nel capitolo. Per ora, basta avere
un'idea di come appare.

Il modello di sviluppocMTV (o MVC)
==================================

Prima di approfondire qualsiasi altro codice, prenditi un momento per
considerare il design complessivo di un database-driven in una applicazione web
Django.

Come abbiamo accennato nei capitoli precedenti, Django è stato progettato per
incoraggiare l'accoppiamento lasco ed una rigida separazione tra le parti di
un'applicazione. Se si segue questa filosofia, è facile apportare modifiche a un
particolare pezzo di applicazione senza influenzare gli altri pezzi. Nelle
funzioni di visualizzazione, per esempio, abbiamo discusso l'importanza di
separare la logica di business dalla logica di presentazione utilizzando un
template di sistema. Con il livello/layer database, stiamo applicando la stessa
filosofia alla logica di accesso ai dati.

Questi tre pezzi insieme -- la logica di accesso ai dati, la logica di business
e la logica di presentazione -- comprendono un concetto che è talvolta chiamato
*Model-View-Controller* (MVC) dell'architettura software. In questo modello di
architettura software, "modello" si riferisce al livello di accesso ai dati,
"View" si riferisce alla parte del sistema che seleziona cosa visualizzare e
come visualizzare, mentre "Controller" si riferisce alla parte del sistema che
decide quale vista utilizzare, secondo gli input dell'utente, l'accesso al
modello come necessario.

.. admonition:: Perché l'Acronimo?

    L'obiettivo di modelli che si definiscono esplicitamente come MVC è per lo
    più per semplificare la comunicazione tra gli sviluppatori. Invece di dover
    dire ai vostri colleghi, "Facciamo una astrazione della connessione dati,
    quindi diamo uno strato separato che gestisce la visualizzazione dei dati, e
    cerchiamo di mettere uno strato in mezzo che regola questo", è possibile
    usufruire di un vocabolario condiviso e dicono: "Qui usiamo il pattern MVC."

Django segue questo pattern MVC da vicino abbastanza da poter essere definito un
framework MVC. Ecco grosso modo come la M, V e C appaiono in Django:

* *M*, la parte di accesso ai dati, è gestito dal layer database di Django, che
  è descritto in questo capitolo.

* *V*, la parte che seleziona i dati da visualizzare e come visualizzarli, è
  gestita da vista e template.

* *C*, la parte che mostra una view in base all'input dell'utente, viene
  gestito dal framework stesso, seguendo l'URLconf e chiamando la funzione
  Python appropriata per l'URL specificato.

Because the "C" is handled by the framework itself and most of the excitement
in Django happens in models, templates and views, Django has been referred to
as an *MTV framework*. In the MTV development pattern,

Poiché la "C" è gestita dal framework stesso e la maggior parte del divertimento
in Django sta nei modelli, template e view, Django è stato descritto come un
*framework MTV*. Nel modello di sviluppo di MTV,

* *M* sta per "Modello", il livello di accesso ai dati. Questo livello non
  contiene nulla a parte tutto ciò che riguarda i dati: come accedervi, come per
  convalidarlo, che i comportamenti che ha, e le relazioni tra i dati.

* *T* sta per "Template", il livello di presentazione. Questo livello contiene
  le decisioni di presentazione-correlati: come qualcosa deve essere
  visualizzato in una pagina Web o di altro tipo di documento.

* *V* sta per "View", il livello di logica di business. Questo strato contiene
  la logica che accede ad un modello e lo rinvia al template appropriato. Si può
  pensare ad esso come il ponte tra modelli e template.

Se si ha familiarità con altri framework web MVC per lo sviluppo, come ad
esempio Ruby on Rails, si possono considerare le viste di Django come i
"controllers" mentre i modelli di Django sono le "viste". Si tratta di una
confusione spiacevole causata da interpretazioni divergenti di MVC.
Nell'interpretazione di Django di MVC, la "vista" descrive i dati che vengono
presentati all'utente, non necessariamente *come* sono fatti i dati, ma *da
quali* dati sono mostrati. Al contrario, Ruby on Rails e framework simili dicono
che il lavoro del controller include decidere quali dati viene presentato
all'utente, mentre la vista è fatta strettamente solo da *come* sono fatti i
dati, non *da quali* dati sono mostrati.

Nessuna interpretazione è più "corretta" dell'altra. La cosa importante è capire
i concetti di base.

Configurazione del Database
===========================

Con tutta la filosofia in mente, cominciamo ad esplorare il layer database di
Django. In primo luogo, abbiamo bisogno di prenderci cura di qualche
configurazione iniziale, abbiamo bisogno di dire a Django quale server database
utilizzare e come connettersi ad esso.

Diamo per scontato che hai impostato un server database, attivato, e creato un
database all'interno di esso (ad esempio, utilizzando un'istruzione ``CREATE
DATABASE``). Se stai usando SQLite, non è richiesta alcuna configurazione,
perché SQLite utilizza dei file standalone sul filesystem per memorizzare i
propri dati.

Come fatto con ``TEMPLATE_DIRS`` nel capitolo precedente, la configurazione del
database sta nel file di impostazioni Django, chiamato ``settings.py`` per
impostazione predefinita. Basta modificare il file e cercare le impostazioni del
database::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
            'NAME': '',                      # Or path to database file if using sqlite3.
            'USER': '',                      # Not used with sqlite3.
            'PASSWORD': '',                  # Not used with sqlite3.
            'HOST': '',                      # Set to empty string for localhost. Not used with sqlite3.
            'PORT': '',                      # Set to empty string for default. Not used with sqlite3.
        }
    }

Ecco una carrellata di ciascuna impostazione.

* ``ENGINE`` dice a Django quale motore di database da utilizzare. Se si
  utilizza un database con Django, ``ENGINE`` deve essere impostato su una delle
  stringhe riportate nella Tabella 5-1.

  .. table:: Table 5-1. Impostazioni del Database

      ============================================ ============ ================================================
      Impostazione                                 Database     Adattatore Richiesto
      ============================================ ============ ================================================
      ``django.db.backends.postgresql_psycopg2``   PostgreSQL   ``psycopg`` versione 2.x,
                                                                http://www.djangoproject.com/r/python-pgsql/.

      ``django.db.backends.mysql``                 MySQL        ``MySQLdb``,
                                                                http://www.djangoproject.com/r/python-mysql/.

      ``django.db.backends.sqlite3``               SQLite       Non e' necessario alcun adattatore.

      ``django.db.backends.oracle``                Oracle       ``cx_Oracle``,
                                                                http://www.djangoproject.com/r/python-oracle/.
      ============================================ ============ ================================================

  Nota che per qualsiasi database back-end si utilizza, è necessario scaricare e
  installare l'adattatore di database appropriato. Ognuno è disponibile
  gratuitamente sul web, basta seguire i link nella "Scheda richiesti" colonna
  nella Tabella 5-1. Se sei su Linux, il sistema di gestione dei pacchetti della
  tua distribuzione potrebbe offrire dei comodi pacchetti. (Cercare i pacchetti
  con nomi del tipo ``python-postgresql`` o ``python-psycopg``)

  Esempio::

      'ENGINE': 'django.db.backends.postgresql_psycopg2',

* ``NAME`` dice a Django il nome del database. Per esempio::

      'NAME': 'mydb',

  Se stai usando SQLite, specifica il percorso completo del filesystem per il
  file di database nel filesystem. Per esempio::

      'NAME': '/home/django/mydata.db',


  Per quanto riguarda il dove mettere il database SQLite, stiamo usando la
  directory ``/home/django`` in questo esempio, ma si potrebbe scegliere una
  directory che funziona meglio.

* ``USER`` dice a Django il nome utente da utilizzare per la connessione al
  database. Ad esempio: se si utilizza SQLite, lasciarlo vuoto.

* ``PASSWORD`` dice a Django la password da utilizzare per la connessione al
  database. Se stai usando SQLite o hai una password vuota, lasciare il campo
  vuoto.

* ``HOST`` dice a Django quale è l'indirizzo a cui collegarsi. Se il database si
  trova sullo stesso computer come l'installazione Django (cioè, localhost),
  lasciare il campo vuoto. Se stai usando SQLite, lascia il campo vuoto.

  MySQL è un caso speciale qui. Se questo valore inizia con una barra (``'/'``)
  e si sta utilizzando MySQL, MySQL si connette tramite un socket Unix per il
  socket specificato, ad esempio::

      'HOST': '/var/run/mysql',

  Se stai usando MySQL e questo valore *non* inizia con uno slash, allora questo
  valore viene considerato l'host.

* ``PORT`` dice a Django la porta da utilizzare per la connessione al database.
  Se stai usando SQLite, lasciare il campo vuoto. In caso contrario, se lo lasci
  vuoto, l'adattatore di database sottostante ne userà predefinita per il server
  di database specificato. Nella maggior parte dei casi, la porta di default va
  bene, quindi è possibile lasciare il campo vuoto.

Una volta inserite le impostazioni e salvato il file ``settings.py``, è una
buona idea verificare la configurazione. Per far questo, eseguire
``python manage.py shell`` come nel precedente capitolo, all'interno della
directory del progetto ``mysite``. (Come abbiamo sottolineato nell'ultimo
capitolo ``manage.py shell`` è un modo per eseguire l'interprete Python con le
impostazioni relative a Django attivate. Ciò è necessario nel nostro caso,
perché Django ha bisogno di sapere quali file di impostazione bisogna utilizzare
per una corretta connessione al database informazioni).

Nella shell, digitare i seguenti comandi per verificare la configurazione del
database::

    >>> from django.db import connection
    >>> cursor = connection.cursor()

Se non succede nulla, allora il database è configurato correttamente. In caso
contrario, controllare il messaggio di errore per avere degli indizi su ciò che
è sbagliato. La tabella 5-2 mostra alcuni errori comuni.

.. table:: Table 5-2. Messaggi d'errore della Configurazione del Database

    =========================================================  ===============================================
    Messaggio d'errore                                         Soluzione
    =========================================================  ===============================================
    You haven't set the ENGINE setting yet.                    Impostare la flag ``ENGINE`` con qualcosa di
                                                               diverso dalla stringa vuota. Valori validi sono
                                                               presenti nella Tabella 5-1.
    Environment variable DJANGO_SETTINGS_MODULE is undefined.  Eseguire il comando ``python manage.py shell``
                                                               piuttosto che ``python``.
    Error loading _____ module: No module named _____.         Non ha installato un corretto adattatore
                                                               specifico per il database (per esempio
                                                               ``psycopg`` o ``MySQLdb``). Gli adattatore *non*
                                                               sono inclusi in Django, perciò è tua
                                                               responsabilità scaricarli ed installarli da solo.
    _____ isn't an available database backend.                 Imposta la flag ``ENGINE`` con un valore valido
                                                               come descritto precedentemente.
                                                               Hai forse fatto un "typo"?
    database _____ does not exist                              Cambiare la flag ``NAME`` in modo che punti ad
                                                               un database esistente o eseguire il comando
                                                               appropriato ``CREATE DATABASE`` per crearlo.
    role _____ does not exist                                  Cambiare la flag ``USER`` in modo che punti ad
                                                               un utente esistente o crearlo nel tuo database.
    could not connect to server                                Accertati che ``HOST`` e
                                                               ``PORT`` siano settati correttamente e che
                                                               il server database sia in esecuzione.
    =========================================================  ===============================================

La tua prima App
================

Ora che hai verificato che la connessione funziona, è il momento di creare un
*app Django* -- un insieme di codice Django, compresi i template e le view, che
stanno insieme in un unico pacchetto Python e rappresentano una completa
applicazione Django.

Vale la pena di spiegare la terminologia qui, perché questo tende a fare
barcollare i principianti. Avevamo già creato un *progetto*, nel Capitolo 2,
quindi qual è la differenza tra un *progetto* e un'*app*? La differenza è che di
configurazione vs codice:

* Un progetto è un esempio di un particolare insieme di applicazioni Django, più
  la configurazione per queste applicazioni.

  Tecnicamente, l'unico requisito di un progetto è che esso fornisce un file di
  impostazioni, che definisce le informazioni riguardo il database usato,
  l'elenco delle applicazioni installate, i ``TEMPLATE_DIRS``, e così via.


* L'app è un set portatile di funzionalità Django, di solito fatto da template
  e view, che sta insieme in un unico pacchetto Python.

  Ad esempio, Django viene riempito con qualche di applicazione, come ad esempio
  un sistema di commento e un'interfaccia di amministrazione automaticamente.
  Una cosa fondamentale da notare su queste applicazioni è che sono portatili e
  riutilizzabili in più progetti.


Ci sono poche regole rigide e veloci su come si forma il tuo codice Django in
questo schema. Se si sta costruendo un semplice sito Web, è possibile utilizzare
solo una singola applicazione. Se si sta costruendo un sito web complesso con
diversi pezzi non correlati, quali un sistema di e-commerce e una message board,
probabilmente si vorrebbe poter dividere quelli in applicazioni separate in modo
che sia possibile riutilizzarle singolarmente in futuro.

In effetti, non è necessario creare applicazioni a tutti i costi, come dimostra
l'esempio di funzioni di visualizzazione che abbiamo creato finora in questo
libro. In questi casi, abbiamo semplicemente creato un file chiamato ``views.py``,
riempito con funzioni di visualizzazione ed abbiamo messo in circuito le
suddette funzioni usando URLconf. Non c'era bisogno di "apps".

Tuttavia, un requisito per quanto riguarda la convenzione delle applicazioni
Django: se si sta utilizzando il layer database di Django (e quindi i modelli),
è necessario creare un'app Django. I modelli devono stare all'interno delle app.
Pertanto, per iniziare a scrivere i nostri modelli, abbiamo bisogno di creare
una nuova app.

All'interno della directory del progetto ``mysite``, digita questo comando per
creare un'app ``books``::

    python manage.py startapp books

Questo comando non produce alcun output, ma crea una directory di ``books``
all'interno della directory ``mysite``. Guardiamo il contenuto di quella
directory::

    books/
        __init__.py
        models.py
        tests.py
        views.py

These files will contain the models and views for this app.

Questi file contengono i modelli e le viste per quest'applicazione.

Dai un'occhiata a ``models.py`` e ``views.py`` nel tuo editor di testo preferito.
Entrambi i file sono vuoti, ad eccezione dei commenti e un import in ``models.py``.
Questa è la base delle app Django.

Definizione di modelli in Python
================================

Come abbiamo discusso in precedenza in questo capitolo, la "M" di "MTV" sta per
"Model." Un modello di Django è una descrizione dei dati nel database,
rappresentate come codice Python. E' il layout dei dati -- l'equivalente del
nostro SQL ``CREATE TABLE`` -- tranne che è scritto in Python invece di SQL, e
comprende più di semplici definizioni di colonna di database. Django utilizza un
modello per eseguire codice SQL dietro le quinte e tornare strutture dati Python
convenienti che rappresentano le righe nelle tabelle del database. Django
utilizza anche modelli per rappresentare i concetti di livello superiore che SQL
non è in grado di gestire necessariamente.

Se hai familiarità con i database, il tuo primo pensiero potrebbe essere: "Non è
forse ridondante dover definire dei modelli in Python piuttosto che in SQL?".
Django funziona nel modo in cui funziona per diversi motivi:

* L'introspezione richiede tempo di calcolo ed è imperfetta. Per dare una API
  per l'accesso ai dati conveniente, Django deve conoscere il layout del database
  in qualche modo, e ci sono due modi per realizzare ciò. Il primo modo è quello
  di descrivere in modo esplicito i dati in Python, mentre il secondo modo sarebbe
  quello di eseguire una introspezione del database in fase di esecuzione per
  determinare i modelli di dati.

  Questo secondo modo sembra più pulito, perché i metadati sulle nostre tabelle
  stanno in un solo luogo, ma introduce alcuni problemi. In primo luogo,
  l'introspezione di un database in fase di esecuzione, ovviamente, necessita di
  tempo di calcolo. Se il framework dovesse eseguire una introspezione del
  database ogni volta che debba elaborare una richiesta, o anche solo quando il
  server Web venga inizializzato, questo comporterebbe un'inaccettabile livello
  di carico. (Se alcuni pensano che il livello di sovraccarico sia
  accettabile, gli sviluppatori di Django mirano a tagliare il più framework in
  modo che esso sia meno relativo all'ambientale possibile). In secondo luogo,
  alcuni database, in particolare le versioni più vecchie di MySQL, non
  memorizzano i metadati sufficienti per un'accurata e completa introspezione.

* Scrivere in Python è divertente, e tenere tutto in Python limita il numero di
  volte che il tuo cervello deve fare un "cambio di contesto". Aiuta quindi la
  produttività, poiché si mantiene lo sviluppo in un unico ambiente di
  programmazione/mentalità. Dover scrivere prima in SQL, poi in Python, e poi
  di nuovo in SQL è fastidioso.

* Avendo modelli di dati memorizzati come codice piuttosto che nel database
  rende più facile mantenere i modelli con un controllo della versione. In
  questo modo, si può facilmente tenere traccia delle modifiche fatte ai layout
  dei dati.

* SQL permette solo un certo livello di metadati relativi a un layout di dati.
  La maggior parte dei sistemi di database, ad esempio, non forniscono un tipo
  di dato specializzato per rappresentare indirizzi e-mail o URL. I modelli di
  Django lo fanno. Il vantaggio dato dai tipi di dati a livello superiore sono
  una maggiore produttività ed un codice più riutilizzabile.

* SQL non è coerente fra le piattaforme di database. Se stai distribuendo
  un'applicazione Web, per esempio, è molto più comprensibile distribuire un
  modulo Python che descrive il layout dei dati in gruppi separati piuttosto che
  istruzioni ``CREATE TABLE`` per MySQL, PostgreSQL e SQLite.

Uno svantaggio di questo approccio, tuttavia, è che è possibile che il codice
Python non sia sincronizzato con ciò che sta effettivamente sul database. Quindi,
se si apportano modifiche ad un modello di Django, è necessario apportare le
stesse modifiche all'interno del database per mantenere il database coerente con
il modello. Discuteremo di alcune strategie per la gestione di questo problema
più avanti in questo capitolo.

Infine, bisogna  notare che Django include un programma di utilità che può
generare modelli analizzando un database esistente. Questo è utile per
rapportarsi rapidamente con i dati preesistenti. Parleremo di questo nel
capitolo 18.

Il tuo primo modello
====================

A titolo di esempio, in questo e nel prossimo capitolo, ci concentreremo su un
layout base libro/autore/editore. Usiamo questo layout esempio perché le
relazioni concettuali tra i libri, gli autori e gli editori sono intuitive, e
questo è un layout di dati molto comune utilizzato in diversi libri di testo
introduttivi su SQL. Stai anche leggendoun libro che è stato scritto da autori e
prodotto da un editore!

Si suppongono i seguenti concetti, campi e relazioni:

* Un autore ha un nome, un cognome e un indirizzo email.

* Un editore ha un nome, un indirizzo, una città, uno stato/provincia, un paese,
  e un sito web.

* Un libro ha un titolo e una data di pubblicazione. Dispone inoltre di uno o
  più autori (relazione molti-a-molti con gli autori) ed un singolo editore
  (relazione uno-a-molti -- conosciuta come foreign key o chiave esterna in
  italiano -- per gli editori).

Il primo passo per utilizzare questo layout con Django è esprimerlo come codice
Python. Nel file ``models.py`` creato dal comando ``startapp``, digita il
seguente codice::

    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField()

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

Esaminiamo rapidamente il codice di coprire le idee di base. La prima cosa da
notare è che ogni modello è rappresentato da una classe Python che è una
sottoclasse di ``django.db.models.Model``. La classe padre, ``Model``, contiene
tutti gli strumenti necessari per rendere questi oggetti in grado di interagire
con un database -- il che lascia i nostri modelli responsabile esclusivamente
di definire i loro campi con una sintassi bella e compatta. Che tu ci creda o no,
questo è tutto il codice che bisogna scrivere per avere accesso ai dati con
Django.

Ogni modello corrisponde generalmente ad una singola tabella di database, ed ogni
attributo di un modello corrisponde generalmente ad una colonna nella tabella
del database. Il nome dell'attributo corrisponde al nome della colonna, e il
tipo di campo (ad esempio, ``CharField``) corrisponde al tipo di colonna di
database (ad es, ``varchar``). Ad esempio, il modello ``Publisher`` è
equivalente alla seguente tabella (assumendo di scrivere nella sintassi
``CREATE TABLE`` di PostgreSQL)::

    CREATE TABLE "books_publisher" (
        "id" serial NOT NULL PRIMARY KEY,
        "name" varchar(30) NOT NULL,
        "address" varchar(50) NOT NULL,
        "city" varchar(60) NOT NULL,
        "state_province" varchar(30) NOT NULL,
        "country" varchar(50) NOT NULL,
        "website" varchar(200) NOT NULL
    );

Invece, Django può generare istruzioni ``CREATE TABLE`` automaticamente, come ti
mostreremo fra un attimo.

C'è una eccezione alla regola "una-classe-per-tabella" nel caso di relazioni
molti-a-molti. Nel nostro esempio, ``Book`` ha un campo ``ManyToManyField``
chiamato ``authors``. Questo specifica che un libro ha uno o molti autori, ma la
tabella del database libro non ha una colonna ``authors``. Django, invece, crea
una tabella aggiuntiva -- una tabella join molti-a-molti -- che gestisce la
relazone dei libri con gli autori.

Per un elenco completo dei tipi di campo e le opzioni di sintassi del modello,
leggi l'Appendice B.

Infine, nota che non abbiamo definito esplicitamente una chiave primaria in
ognuno di questi modelli. A meno di modifiche, Django da automaticamente ad ogni
modello una chiave primaria rappresentata da un campo intero che si
auto-incrementa chiamato ``id``. Ogni modello Django deve avere una singola
colonna che funge da chiave primaria.

Installare un Modello
=====================

We've written the code; now let's create the tables in our database. In order
to do that, the first step is to *activate* these models in our Django project.
We do that by adding the ``books`` app to the list of "installed apps" in the
settings file.

Abbiamo scritto il codice, ora creiamo le tabelle nel database. Per fare questo,
il primo passo è quello di *attivare* questi modelli nel nostro progetto Django.
Dobbiamo quindi aggiungere l'app ``books`` alla lista delle "installed apps" nel
file di impostazioni.

Modificare nuovamente il file ``settings.py``, e cercare l'impostazione
``INSTALLED_APPS``. ``INSTALLED_APPS`` dice a Django quali applicazioni sono
attivate per un dato progetto. Per impostazione predefinita, assomiglia qualcosa
di simile::

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    )

Commentare temporaneamente tutte e sei le righe mettendo un cancelletto (``#``)
davanti a loro. (Sono inclusi di default, ma li attiveremo e discuteremo di loro
nei capitoli successivi). Mentre ci siamo, commentare l'impostazione predefinita
``MIDDLEWARE_CLASSES``; i valori di default in ``MIDDLEWARE_CLASSES`` dipendono
da alcune delle applicazioni che abbiamo appena commentato. Quindi, aggiungere
``'books'`` alla lista ``INSTALLED_APPS``, quindi l'impostazione finisce per
assomigliare a qualcosa di simile::

    MIDDLEWARE_CLASSES = (
        # 'django.middleware.common.CommonMiddleware',
        # 'django.contrib.sessions.middleware.SessionMiddleware',
        # 'django.middleware.csrf.CsrfViewMiddleware',
        # 'django.contrib.auth.middleware.AuthenticationMiddleware',
        # 'django.contrib.messages.middleware.MessageMiddleware',
    )

    INSTALLED_APPS = (
        # 'django.contrib.auth',
        # 'django.contrib.contenttypes',
        # 'django.contrib.sessions',
        # 'django.contrib.sites',
        'books',
    )

(As we mentioned last chapter when setting ``TEMPLATE_DIRS``, you'll need to be
sure to include the trailing comma in ``INSTALLED_APPS``, because it's a
single-element tuple. By the way, this book's authors prefer to put a comma
after *every* element of a tuple, regardless of whether the tuple has only a
single element. This avoids the issue of forgetting commas, and there's no
penalty for using that extra comma.)

(Come abbiamo accennato nell'ultimo capitolo quando si imposta ``TEMPLATE_DIRS``,
è necessario essere sicuri di includere la virgola finale in ``INSTALLED_APPS``,
perché è una tupla a singolo elemento. In proposito, gli autori di questo libro
preferiscono mettere una virgola dopo *ogni* elemento di un tupla,
indipendentemente dal fatto che la tupla abbia un solo elemento. Ciò evita il
problema di virgole dimenticate, e non c'è nessun problema nell'utilizzare
quella virgola in più).

``'mysite.books'`` si riferisce all'applicazione ``books`` su cui stiamo
lavorando. Ogni applicazione in ``INSTALLED_APPS`` è rappresentata dal suo
percorso completo Python -- cioè, il percorso dei pacchetti, separati da punti,
che portano al vero pacchetto dell'applicazione.

Now that the Django app has been activated in the settings file, we can create
the database tables in our database. First, let's validate the models by
running this command::

Ora che l'app Django è stata attivata nel file delle impostazioni, possiamo
creare le tabelle nel nostro database. In primo luogo, cerchiamo di validare i
modelli eseguendo questo comando::

    python manage.py validate

Il comando ``validate`` controlla se la sintassi e la logica dei tuoi modelli
sono corretti. Se tutto va bene, vedrai il messaggio ``0 errors found``. Se non
va bene, assicurati di aver digitato correttamente il codice del modello.
L'uscita di un errore dovrebbe dare informazioni utili su che cosa c'è di
sbagliato nel codice.

Ogni volta che si pensa di avere problemi con i modelli, eseguire
``python manage.py validate``. Esso tende a catturare tutti i problemi comuni
dei modelli.

Se i modelli sono validi, esegui il seguente comando per generare istruzioni
``CREATE TABLE`` dei modelli nella app ``books`` (con la sintassi evidenziata d
colori, se stai utilizzando Unix)::

    python manage.py sqlall books

In questo comando, ``books`` è il nome dell'app. E' quello che è stato
specificato quando è stato eseguito il comando ``manage.py startapp``. Quando si
esegue il comando, si dovrebbe vedere qualcosa di simile a questo::

    BEGIN;
    CREATE TABLE "books_publisher" (
        "id" serial NOT NULL PRIMARY KEY,
        "name" varchar(30) NOT NULL,
        "address" varchar(50) NOT NULL,
        "city" varchar(60) NOT NULL,
        "state_province" varchar(30) NOT NULL,
        "country" varchar(50) NOT NULL,
        "website" varchar(200) NOT NULL
    )
    ;
    CREATE TABLE "books_author" (
        "id" serial NOT NULL PRIMARY KEY,
        "first_name" varchar(30) NOT NULL,
        "last_name" varchar(40) NOT NULL,
        "email" varchar(75) NOT NULL
    )
    ;
    CREATE TABLE "books_book" (
        "id" serial NOT NULL PRIMARY KEY,
        "title" varchar(100) NOT NULL,
        "publisher_id" integer NOT NULL REFERENCES "books_publisher" ("id") DEFERRABLE INITIALLY DEFERRED,
        "publication_date" date NOT NULL
    )
    ;
    CREATE TABLE "books_book_authors" (
        "id" serial NOT NULL PRIMARY KEY,
        "book_id" integer NOT NULL REFERENCES "books_book" ("id") DEFERRABLE INITIALLY DEFERRED,
        "author_id" integer NOT NULL REFERENCES "books_author" ("id") DEFERRABLE INITIALLY DEFERRED,
        UNIQUE ("book_id", "author_id")
    )
    ;
    CREATE INDEX "books_book_publisher_id" ON "books_book" ("publisher_id");
    COMMIT;

Da notare che:

* I nomi delle tabelle vengono generati automaticamente combinando il nome dell'app
  (``books``) e il nome del modello in minuscolo (``publisher``, ``book``, e
  ``author``). È possibile sovrascrivere questo comportamento, come indicato
  nell'Appendice B.

* Come abbiamo accennato in precedenza, Django aggiunge una chiave primaria per
  ogni tabella automaticamente -- i campi ``id``. È possibile ignorare anche
  questo.

* Per convenzione, Django aggiunge ``"_id"`` al nome dei campi chiave esterna.
  Come si può immaginare, è possibile anche ignorare questo comportamento.

* La relazione di chiave esterna è resa esplicita da una dichiarazione
  ``REFERENCES``.

* Queste istruzioni ``CREATE TABLE`` sono su misura per il database che si sta
  utilizzando, quindi vengono utilizzati i tipi di campi specifici del database,
  ad esempio ``auto_increment``   (MySQL), ``serial`` (PostgreSQL), o
  ``integer primary key`` (SQLite) sono gestite in maniera automatica. Lo stesso
  vale per le virgolette da usare (ad esempio, usare le virgolette doppie o
  singole). Questo output di esempio è scritto seguendo la sintassi PostgreSQL.

Il comando ``sqlall`` non crea le tabelle o comunque non tocca il database --
stampa solo l'output sullo schermo in modo da poter vedere quale istruzioni
SQL Django avrebbe eseguito se gli fosse richiesto. Se si volesse, si potrebbe
copiare e incollare questo SQL nel database client, o utilizzarlo con l'operatore
pipe di Unix per passarlo direttamente (ad esempio, ``python manage.py sqlall
books | psql mydb``). Tuttavia, Django fornisce un modo più semplice di inviare
SQL per il database: il comando ``syncdb`::

    python manage.py syncdb

Eseguendo questo comando, vedremo qualcosa di simile a questo::

    Creating table books_publisher
    Creating table books_author
    Creating table books_book
    Installing index for books.Book model


Il comando ``syncdb`` è un semplice "sincronizzatore" dei modelli per il
database. Controlla tutti i modelli di ogni app nell'ambiente ``INSTALLED_APPS``,
controlla il database per vedere se esistono le tabelle e le crea se non esistono
ancora. Da notare che ``syncdb`` *non* sincronizza i cambiamenti o l'eliminazione
dei modelli, se si apporta una modifica ad un modello o lo si elimina e si
desidera aggiornare il database, ``syncdb`` non ti può aiutare. (Maggiori
informazioni al riguardo nella sezione "Modifica dello SCHEMA di database" verso
la fine di questo capitolo).

Eseguendo nuovamente ``python manage.py syncdb``, non succede niente, perché
non hai aggiunto i modelli per l'applicazione ``books`` o aggiunto Apps per
``INSTALLED_APPS``. Ergo, è sempre sicuro eseguire ``python manage.py syncdb``
-- non sovrascrive le cose.

Se sei interessato, prenditi un momento per tuffarti nel client a riga di
comando del server di database e vedere le tabelle create da Django sul database.
È possibile eseguire manualmente il client a riga di comando (ad esempio, ``psql``
per PostgreSQL) oppure è possibile eseguire il comando
``python manage.py dbshell``, che sarà in grado di capire quale client a riga di
comando eseguire, in base all'impostazione ``DATABASE_SERVER``. Quest'ultimo è
quasi sempre più conveniente.

Accesso al Database
===================

Una volta creato un modello, Django fornisce automaticamente una API Python ad
alto livello per lavorare con tali modelli. Provalo eseguendo
``python manage.py shell`` e digitando il seguente::

    >>> from books.models import Publisher
    >>> p1 = Publisher(name='Apress', address='2855 Telegraph Avenue',
    ...     city='Berkeley', state_province='CA', country='U.S.A.',
    ...     website='http://www.apress.com/')
    >>> p1.save()
    >>> p2 = Publisher(name="O'Reilly", address='10 Fawcett St.',
    ...     city='Cambridge', state_province='MA', country='U.S.A.',
    ...     website='http://www.oreilly.com/')
    >>> p2.save()
    >>> publisher_list = Publisher.objects.all()
    >>> publisher_list
    [<Publisher: Publisher object>, <Publisher: Publisher object>]

Queste poche righe di codice fanno un bel po' di lavoro. Ecco i punti salienti:

* In primo luogo, importiamo la nostra classe modello ``Publisher``. Questo ci
  permette di interagire con la tabella del database che contiene gli editori;

* Creiamo un oggetto ``Publisher`` istanziandola con dei valori per ogni campo
  -- nome, indirizzo, ecc...;

* Per salvare l'oggetto nel database, viene chiamato il suo metodo ``save()``.
  Dietro le quinte, Django qui esegue un'istruzione SQL ``INSERT``;

* Per recuperare gli editori dal database, viene usato l'attributo
  ``Publisher.objects``, che si può pensare come un insieme di tutti gli editori.
  Per recuperare un elenco di *tutti* gli oggetti ``Publisher`` nel database,
  si usa l'istruzione ``Publisher.objects.all()``. In questi casi, dietro le
  quinte, Django esegue un'istruzione SQL ``SELECT``.

Una cosa è degna di nota, nel caso in cui non fosse stato chiaro da questo
esempio. Quando si creano oggetti utilizzando le API dei modelli, Django non
salva gli oggetti nel database fino a quando si chiama il metodo ``save()``::

    p1 = Publisher(...)
    # A questo punto, p1 non è ancora salvato sul database!
    p1.save()
    # Ora lo è.

Se si desidera creare un oggetto e salvarlo nel database in un unico passaggio,
utilizzare il metodo ``objects.create()``. Questo esempio è equivalente al
precedente esempio::

    >>> p1 = Publisher.objects.create(name='Apress',
    ...     address='2855 Telegraph Avenue',
    ...     city='Berkeley', state_province='CA', country='U.S.A.',
    ...     website='http://www.apress.com/')
    >>> p2 = Publisher.objects.create(name="O'Reilly",
    ...     address='10 Fawcett St.', city='Cambridge',
    ...     state_province='MA', country='U.S.A.',
    ...     website='http://www.oreilly.com/')
    >>> publisher_list = Publisher.objects.all()
    >>> publisher_list

Naturalmente, si può fare molto con le API relative ai database di Django -- ma
in primo luogo, cerchiamo di prenderci cura di un piccolo fastidio.

Aggiungere una rappresentazione ai Modelli
==========================================

Quando abbiamo stampato l'elenco degli editori, abbiamo ottenuto un inutile
messaggio che rende difficile pubblicare gli oggetti ``Publisher``
singolarmente::

    [<Publisher: Publisher object>, <Publisher: Publisher object>]


Siamo in grado di risolvere questo problema facilmente con l'aggiunta di un
metodo chiamato ``__unicode__()`` nella nostra classe Publisher. Un metodo
``__unicode__()`` indica a Python come visualizzare con la rappresentazione
"unicode" un oggetto. Ecco come aggiungere un metodo ``__unicode__()`` ai tre
modelli::

.. parsed-literal::

    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

        **def __unicode__(self):**
            **return self.name**

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField()

        **def __unicode__(self):**
            **return u'%s %s' % (self.first_name, self.last_name)**

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

        **def __unicode__(self):**
            **return self.title**

Come puoi vedere, un metodo ``__unicode__()`` restituisce una rappresentazione
di un oggetto. Nel nostro esempio, il metodo ``__unicode__()`` di ``Publisher``
e ``Book`` semplicemente restituisce il nome e il titolo dell'oggetto,
rispettivamente, ma lo ``__unicode__()`` per ``Author`` è leggermente più
complesso -- è un'unione dei campi ``first_name`` e ``last_name``, separati da
uno spazio.

L'unico requisito per ``__unicode__()`` è che si restituisca un oggetto Unicode.
Se ``__unicode__()`` non restituisce un oggetto Unicode - se restituisce, ad
esempio, un intero -- Python solleva l'eccezione ``TypeError`` con un messaggio
del tipo ``"coercing to Unicode: need string or buffer, int found"``.

.. admonition:: Oggetti Unicode

    Cosa sono gli Oggetti Unicode?

    Si può pensare a un oggetto Unicode come una stringa Python in grado di
    gestire più di un milione di diversi tipi di caratteri, da versioni
    accentate di caratteri latini a caratteri non latini alle virgolette
    tipografiche e simboli oscuri.

    Le Stringhe Python normali sono *codificati*, il che significa che utilizzano
    una codifica come ASCII, ISO-8859-1 o UTF-8. Se devi utilizzare caratteri
    particolari (nulla oltre i 128 caratteri ASCII standard come 0-9 e AZ)
    in una normale stringa di Python, è necessario tenere traccia di quale
    codifica la stringa sta usando, o i caratteri particolari potrebbero
    sembrare incasinati quando vengono visualizzati o stampati. I problemi si
    verificano quando si dispone di dati che sono memorizzati in una codifica e
    si tenta di combinarli con i dati in una codifica diversa, o si tenta di
    visualizzarlo in un ambito che presuppone una certa codifica. Abbiamo visto
    tutti pagine Web e e-mail che sono disseminati di "??? ??????" ed altri
    caratteri in posti strani, che generalmente suggeriscono c'è un problema di
    codifica.

    Gli Oggetti Unicode, tuttavia, non hanno una codifica; essi usano un
    coerente, insieme universale di caratteri chiamato, beh, "Unicode". Quando
    avete a che fare con oggetti Unicode in Python, è possibile combinarli in
    modo sicuro, senza doversi preoccupare di problemi di codifica.

    Django utilizza oggetti Unicode per tutto il framework. Gli oggetti del
    modello vengono recuperati come oggetti Unicode, le viste interagiscono con
    i dati Unicode, e template sono resi come Unicode. In generale, non dovrete
    preoccuparvi di rendere sicuro il vostro codifiche sono giuste, le cose
    dovrebbero solo lavorare.

    Questo è uno strato di astrazione *molto* elevato, e se sei stordito da
    questa panoramica sugli oggetti Unicode, e lo devi a te stesso conoscere
    meglio l'argomento. Un buon punto di partenza è
    http://www.joelonsoftware.com/articles/Unicode.html.

Affinché le modifiche di ``__unicode__()`` abbiano effetto, bisogna uscire fuori
dalla shell Python ed entrare di nuovo con ``python manage.py shell``. (Questo
è il modo più semplice per vedere delle modifiche) Ora la lista di oggetti di
``Publisher`` è molto più facile da capire::

    >>> from books.models import Publisher
    >>> publisher_list = Publisher.objects.all()
    >>> publisher_list
    [<Publisher: Apress>, <Publisher: O'Reilly>]

Assicurati che qualsiasi modello si definisca abbia un metodo ``__unicode__()``
-- non solo per comodità quando si utilizza l'interprete interattivo, ma anche
perché Django utilizza l'output di ``__unicode__()`` in più punti quando ha
bisogno della visualizzazione degli oggetti.

Infine, ricordiamo che aggiungere ``__unicode__()`` ad ogni modello è una
*best-practices* da seguire. Un modello Django descrive non solo la disposizione
della tabella di database per un oggetto, ma anche una funzionalità che un
oggetto deve avere. ``__unicode__()`` è un esempio di tale funzionalità --
un modello sa come visualizzare se stesso.

Inserimento e Aggiornamento dei dati
====================================

You've already seen this done: to insert a row into your database, first create
an instance of your model using keyword arguments, like so::

    >>> p = Publisher(name='Apress',
    ...         address='2855 Telegraph Ave.',
    ...         city='Berkeley',
    ...         state_province='CA',
    ...         country='U.S.A.',
    ...         website='http://www.apress.com/')

As we noted above, this act of instantiating a model class does *not* touch
the database. The record isn't saved into the database until you call
``save()``, like this::

    >>> p.save()

.. SL Tested ok

In SQL, this can roughly be translated into the following::

    INSERT INTO books_publisher
        (name, address, city, state_province, country, website)
    VALUES
        ('Apress', '2855 Telegraph Ave.', 'Berkeley', 'CA',
         'U.S.A.', 'http://www.apress.com/');

Because the ``Publisher`` model uses an autoincrementing primary key ``id``,
the initial call to ``save()`` does one more thing: it calculates the primary
key value for the record and sets it to the ``id`` attribute on the instance::

    >>> p.id
    52    # this will differ based on your own data

.. SL Should be '52L' to match actual output.

Subsequent calls to ``save()`` will save the record in place, without creating
a new record (i.e., performing an SQL ``UPDATE`` statement instead of an
``INSERT``)::

    >>> p.name = 'Apress Publishing'
    >>> p.save()

.. SL Tested ok

The preceding ``save()`` statement will result in roughly the following SQL::

    UPDATE books_publisher SET
        name = 'Apress Publishing',
        address = '2855 Telegraph Ave.',
        city = 'Berkeley',
        state_province = 'CA',
        country = 'U.S.A.',
        website = 'http://www.apress.com'
    WHERE id = 52;

Yes, note that *all* of the fields will be updated, not just the ones that have
been changed. Depending on your application, this may cause a race condition.
See "Updating Multiple Objects in One Statement" below to find out how to
execute this (slightly different) query::

    UPDATE books_publisher SET
        name = 'Apress Publishing'
    WHERE id=52;

Selecting Objects
=================

Knowing how to create and update database records is essential, but chances are
that the Web applications you'll build will be doing more querying of existing
objects than creating new ones. We've already seen a way to retrieve *every*
record for a given model::

    >>> Publisher.objects.all()
    [<Publisher: Apress>, <Publisher: O'Reilly>]

.. SL Tested ok

This roughly translates to this SQL::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher;

.. note::

    Notice that Django doesn't use ``SELECT *`` when looking up data and instead
    lists all fields explicitly. This is by design: in certain circumstances
    ``SELECT *`` can be slower, and (more important) listing fields more closely
    follows one tenet of the Zen of Python: "Explicit is better than implicit."

    For more on the Zen of Python, try typing ``import this`` at a Python
    prompt.

Let's take a close look at each part of this ``Publisher.objects.all()`` line:

* First, we have the model we defined, ``Publisher``. No surprise here: when
  you want to look up data, you use the model for that data.

* Next, we have the ``objects`` attribute. This is called a *manager*.
  Managers are discussed in detail in Chapter 10. For now, all you need to
  know is that managers take care of all "table-level" operations on data
  including, most important, data lookup.

  All models automatically get a ``objects`` manager; you'll use it
  any time you want to look up model instances.

* Finally, we have ``all()``. This is a method on the ``objects`` manager
  that returns all the rows in the database. Though this object *looks*
  like a list, it's actually a *QuerySet* -- an object that represents a
  specific set of rows from the database. Appendix C deals with QuerySets
  in detail. For the rest of this chapter, we'll just treat them like the
  lists they emulate.

Any database lookup is going to follow this general pattern -- we'll call methods on
the manager attached to the model we want to query against.

Filtering Data
--------------

Naturally, it's rare to want to select *everything* from a database at once; in
most cases, you'll want to deal with a subset of your data. In the Django API,
you can filter your data using the ``filter()`` method::

    >>> Publisher.objects.filter(name='Apress')
    [<Publisher: Apress>]

.. SL Tested ok

``filter()`` takes keyword arguments that get translated into the appropriate
SQL ``WHERE`` clauses. The preceding example would get translated into
something like this::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE name = 'Apress';

You can pass multiple arguments into ``filter()`` to narrow down things further::

    >>> Publisher.objects.filter(country="U.S.A.", state_province="CA")
    [<Publisher: Apress>]

.. SL Tested ok

Those multiple arguments get translated into SQL ``AND`` clauses. Thus, the
example in the code snippet translates into the following::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE country = 'U.S.A.'
    AND state_province = 'CA';

Notice that by default the lookups use the SQL ``=`` operator to do exact match
lookups. Other lookup types are available::

    >>> Publisher.objects.filter(name__contains="press")
    [<Publisher: Apress>]

.. SL Tested ok

That's a *double* underscore there between ``name`` and ``contains``. Like
Python itself, Django uses the double underscore to signal that something
"magic" is happening -- here, the ``__contains`` part gets translated by Django
into a SQL ``LIKE`` statement::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE name LIKE '%press%';

Many other types of lookups are available, including ``icontains``
(case-insensitive ``LIKE``), ``startswith`` and ``endswith``, and ``range`` (SQL
``BETWEEN`` queries). Appendix C describes all of these lookup types in detail.

Retrieving Single Objects
-------------------------

The ``filter()`` examples above all returned a ``QuerySet``, which you can
treat like a list. Sometimes it's more convenient to fetch only a single object,
as opposed to a list. That's what the ``get()`` method is for::

    >>> Publisher.objects.get(name="Apress")
    <Publisher: Apress>

.. SL Tested ok

Instead of a list (rather, ``QuerySet``), only a single object is returned.
Because of that, a query resulting in multiple objects will cause an
exception::

    >>> Publisher.objects.get(country="U.S.A.")
    Traceback (most recent call last):
        ...
    MultipleObjectsReturned: get() returned more than one Publisher --
        it returned 2! Lookup parameters were {'country': 'U.S.A.'}

.. SL Tested ok

A query that returns no objects also causes an exception::

    >>> Publisher.objects.get(name="Penguin")
    Traceback (most recent call last):
        ...
    DoesNotExist: Publisher matching query does not exist.

.. SL Tested ok

The ``DoesNotExist`` exception is an attribute of the model's class --
``Publisher.DoesNotExist``. In your applications, you'll want to trap these
exceptions, like this::

    try:
        p = Publisher.objects.get(name='Apress')
    except Publisher.DoesNotExist:
        print "Apress isn't in the database yet."
    else:
        print "Apress is in the database."

.. SL Tested ok

Ordering Data
-------------

As you play around with the previous examples, you might discover that the objects
are being returned in a seemingly random order. You aren't imagining things; so
far we haven't told the database how to order its results, so we're simply
getting back data in some arbitrary order chosen by the database.

In your Django applications, you'll probably want to order your results
according to a certain value -- say, alphabetically. To do this, use the
``order_by()`` method::

    >>> Publisher.objects.order_by("name")
    [<Publisher: Apress>, <Publisher: O'Reilly>]

.. SL Tested ok

This doesn't look much different from the earlier ``all()`` example, but the
SQL now includes a specific ordering::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    ORDER BY name;

You can order by any field you like::

    >>> Publisher.objects.order_by("address")
    [<Publisher: O'Reilly>, <Publisher: Apress>]

    >>> Publisher.objects.order_by("state_province")
    [<Publisher: Apress>, <Publisher: O'Reilly>]

.. SL Tested ok

To order by multiple fields (where the second field is used to disambiguate
ordering in cases where the first is the same), use multiple arguments::

    >>> Publisher.objects.order_by("state_province", "address")
     [<Publisher: Apress>, <Publisher: O'Reilly>]

.. SL Tested ok

You can also specify reverse ordering by prefixing the field name with a ``-``
(that's a minus character)::

    >>> Publisher.objects.order_by("-name")
    [<Publisher: O'Reilly>, <Publisher: Apress>]

.. SL Tested ok

While this flexibility is useful, using ``order_by()`` all the time can be quite
repetitive. Most of the time you'll have a particular field you usually want
to order by. In these cases, Django lets you specify a default ordering in the
model:

.. parsed-literal::

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

        def __unicode__(self):
            return self.name

        **class Meta:**
            **ordering = ['name']**

Here, we've introduced a new concept: the ``class Meta``, which is a class
that's embedded within the ``Publisher`` class definition (i.e., it's indented
to be within ``class Publisher``). You can use this ``Meta`` class on any model
to specify various model-specific options. A full reference of ``Meta`` options
is available in Appendix B, but for now, we're concerned with the ``ordering``
option. If you specify this, it tells Django that unless an ordering is given
explicitly with ``order_by()``, all ``Publisher`` objects should be ordered by
the ``name`` field whenever they're retrieved with the Django database API.

Chaining Lookups
----------------

You've seen how you can filter data, and you've seen how you can order it. Often, of course,
you'll need to do both. In these cases, you simply "chain" the lookups together::

    >>> Publisher.objects.filter(country="U.S.A.").order_by("-name")
    [<Publisher: O'Reilly>, <Publisher: Apress>]

.. SL Tested ok

As you might expect, this translates to a SQL query with both a ``WHERE`` and an
``ORDER BY``::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE country = 'U.S.A'
    ORDER BY name DESC;

Slicing Data
------------

Another common need is to look up only a fixed number of rows. Imagine you have thousands
of publishers in your database, but you want to display only the first one. You can do this
using Python's standard list slicing syntax::

    >>> Publisher.objects.order_by('name')[0]
    <Publisher: Apress>

.. SL Tested ok

This translates roughly to::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    ORDER BY name
    LIMIT 1;

Similarly, you can retrieve a specific subset of data using Python's
range-slicing syntax::

    >>> Publisher.objects.order_by('name')[0:2]

.. SL Tested ok (but should show expected output?)

This returns two objects, translating roughly to::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    ORDER BY name
    OFFSET 0 LIMIT 2;

Note that negative slicing is *not* supported::

    >>> Publisher.objects.order_by('name')[-1]
    Traceback (most recent call last):
      ...
    AssertionError: Negative indexing is not supported.

This is easy to get around, though. Just change the ``order_by()`` statement,
like this::

    >>> Publisher.objects.order_by('-name')[0]

Updating Multiple Objects in One Statement
------------------------------------------

We pointed out in the "Inserting and Updating Data" section that the model
``save()`` method updates *all* columns in a row. Depending on your
application, you may want to update only a subset of columns.

For example, let's say we want to update the Apress ``Publisher`` to change
the name from ``'Apress'`` to ``'Apress Publishing'``. Using ``save()``, it
would look something like this::

    >>> p = Publisher.objects.get(name='Apress')
    >>> p.name = 'Apress Publishing'
    >>> p.save()

.. SL Tested ok

This roughly translates to the following SQL::

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE name = 'Apress';

    UPDATE books_publisher SET
        name = 'Apress Publishing',
        address = '2855 Telegraph Ave.',
        city = 'Berkeley',
        state_province = 'CA',
        country = 'U.S.A.',
        website = 'http://www.apress.com'
    WHERE id = 52;

(Note that this example assumes Apress has a publisher ID of ``52``.)

You can see in this example that Django's ``save()`` method sets *all* of the
column values, not just the ``name`` column. If you're in an environment where
other columns of the database might change due to some other process, it's
smarter to change *only* the column you need to change. To do this, use the
``update()`` method on ``QuerySet`` objects. Here's an example::

    >>> Publisher.objects.filter(id=52).update(name='Apress Publishing')

.. SL Tested ok

The SQL translation here is much more efficient and has no chance of race
conditions::

    UPDATE books_publisher
    SET name = 'Apress Publishing'
    WHERE id = 52;

The ``update()`` method works on any ``QuerySet``, which means you can edit
multiple records in bulk. Here's how you might change the ``country`` from
``'U.S.A.'`` to ``USA`` in each ``Publisher`` record::

    >>> Publisher.objects.all().update(country='USA')
    2

.. SL Tested ok

The ``update()`` method has a return value -- an integer representing how many
records changed. In the above example, we got ``2``.

Deleting Objects
================

To delete an object from your database, simply call the object's ``delete()``
method::

    >>> p = Publisher.objects.get(name="O'Reilly")
    >>> p.delete()
    >>> Publisher.objects.all()
    [<Publisher: Apress Publishing>]

.. SL Tested ok

You can also delete objects in bulk by calling ``delete()`` on the result of
any ``QuerySet``. This is similar to the ``update()`` method we showed in the
last section::

    >>> Publisher.objects.filter(country='USA').delete()
    >>> Publisher.objects.all().delete()
    >>> Publisher.objects.all()
    []

.. SL Tested ok

Be careful deleting your data! As a precaution against deleting all of the data
in a particular table, Django requires you to explicitly use ``all()`` if you
want to delete *everything* in your table. For example, this won't work::

    >>> Publisher.objects.delete()
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
    AttributeError: 'Manager' object has no attribute 'delete'

.. SL Tested ok

But it'll work if you add the ``all()`` method::

    >>> Publisher.objects.all().delete()

.. SL Tested ok

If you're just deleting a subset of your data, you don't need to include
``all()``. To repeat a previous example::

    >>> Publisher.objects.filter(country='USA').delete()

.. SL Tested ok

What's Next?
============

Having read this chapter, you have enough knowledge of Django models to be able
to write basic database applications. Chapter 10 will provide some information
on more advanced usage of Django's database layer.

Once you've defined your models, the next step is to populate your database
with data. You might have legacy data, in which case Chapter 18 will give you
advice about integrating with legacy databases. You might rely on site users
to supply your data, in which case Chapter 7 will teach you how to process
user-submitted form data.

But in some cases, you or your team might need to enter data manually, in which
case it would be helpful to have a Web-based interface for entering and
managing data. The next chapter `Chapter 6`_ covers Django's admin interface, which exists
precisely for that reason.

.. _Chapter 6: chapter06.html
