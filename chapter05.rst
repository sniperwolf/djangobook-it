==================
Chapter 5: Modelli
==================

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

If you're familiar with databases, your immediate thought might be, "Isn't it
redundant to define data models in Python instead of in SQL?" Django works the
way it does for several reasons:

* Introspection requires overhead and is imperfect. In order to provide
  convenient data-access APIs, Django needs to know the
  database layout *somehow*, and there are two ways of accomplishing this.
  The first way would be to explicitly describe the data in Python, and the
  second way would be to introspect the database at runtime to determine
  the data models.

  This second way seems cleaner, because the metadata about your tables
  lives in only one place, but it introduces a few problems. First,
  introspecting a database at runtime obviously requires overhead. If the
  framework had to introspect the database each time it processed a
  request, or even only when the Web server was initialized, this would
  incur an unacceptable level of overhead. (While some believe that level
  of overhead is acceptable, Django's developers aim to trim as much
  framework overhead as possible.) Second, some databases, notably older
  versions of MySQL, do not store sufficient metadata for accurate and
  complete introspection.

* Writing Python is fun, and keeping everything in Python limits the number
  of times your brain has to do a "context switch." It helps productivity
  if you keep yourself in a single programming environment/mentality for as
  long as possible. Having to write SQL, then Python, and then SQL again is
  disruptive.

* Having data models stored as code rather than in your database makes it
  easier to keep your models under version control. This way, you can
  easily keep track of changes to your data layouts.

* SQL allows for only a certain level of metadata about a data layout. Most
  database systems, for example, do not provide a specialized data type for
  representing email addresses or URLs. Django models do. The advantage of
  higher-level data types is higher productivity and more reusable code.

* SQL is inconsistent across database platforms. If you're distributing a
  Web application, for example, it's much more pragmatic to distribute a
  Python module that describes your data layout than separate sets of
  ``CREATE TABLE`` statements for MySQL, PostgreSQL, and SQLite.

A drawback of this approach, however, is that it's possible for the Python code
to get out of sync with what's actually in the database. If you make changes to
a Django model, you'll need to make the same changes inside your database to
keep your database consistent with the model. We'll discuss some strategies for
handling this problem later in this chapter.

Finally, we should note that Django includes a utility that can generate models
by introspecting an existing database. This is useful for quickly getting up
and running with legacy data. We'll cover this in Chapter 18.

Your First Model
================

As an ongoing example in this chapter and the next chapter, we'll focus on a
basic book/author/publisher data layout. We use this as our example because the
conceptual relationships between books, authors, and publishers are well known,
and this is a common data layout used in introductory SQL textbooks. You're
also reading a book that was written by authors and produced by a publisher!

We'll suppose the following concepts, fields, and relationships:

* An author has a first name, a last name and an email address.

* A publisher has a name, a street address, a city, a state/province, a
  country, and a Web site.

* A book has a title and a publication date. It also has one or more
  authors (a many-to-many relationship with authors) and a single publisher
  (a one-to-many relationship -- aka foreign key -- to publishers).

The first step in using this database layout with Django is to express it as
Python code. In the ``models.py`` file that was created by the ``startapp``
command, enter the following::

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

Let's quickly examine this code to cover the basics. The first thing to notice
is that each model is represented by a Python class that is a subclass of
``django.db.models.Model``. The parent class, ``Model``, contains all the
machinery necessary to make these objects capable of interacting with a
database -- and that leaves our models responsible solely for defining their
fields, in a nice and compact syntax. Believe it or not, this is all the code
we need to write to have basic data access with Django.

Each model generally corresponds to a single database table, and each attribute
on a model generally corresponds to a column in that database table. The
attribute name corresponds to the column's name, and the type of field (e.g.,
``CharField``) corresponds to the database column type (e.g., ``varchar``). For
example, the ``Publisher`` model is equivalent to the following table (assuming
PostgreSQL ``CREATE TABLE`` syntax)::

    CREATE TABLE "books_publisher" (
        "id" serial NOT NULL PRIMARY KEY,
        "name" varchar(30) NOT NULL,
        "address" varchar(50) NOT NULL,
        "city" varchar(60) NOT NULL,
        "state_province" varchar(30) NOT NULL,
        "country" varchar(50) NOT NULL,
        "website" varchar(200) NOT NULL
    );

Indeed, Django can generate that ``CREATE TABLE`` statement automatically, as
we'll show you in a moment.

The exception to the one-class-per-database-table rule is the case of
many-to-many relationships. In our example models, ``Book`` has a
``ManyToManyField`` called ``authors``. This designates that a book has one or
many authors, but the ``Book`` database table doesn't get an ``authors``
column. Rather, Django creates an additional table -- a many-to-many "join
table" -- that handles the mapping of books to authors.

For a full list of field types and model syntax options, see Appendix B.

Finally, note we haven't explicitly defined a primary key in any of these
models. Unless you instruct it otherwise, Django automatically gives every
model an auto-incrementing integer primary key field called ``id``. Each Django
model is required to have a single-column primary key.

Installing the Model
====================

We've written the code; now let's create the tables in our database. In order
to do that, the first step is to *activate* these models in our Django project.
We do that by adding the ``books`` app to the list of "installed apps" in the
settings file.

Edit the ``settings.py`` file again, and look for the ``INSTALLED_APPS``
setting. ``INSTALLED_APPS`` tells Django which apps are activated for a given
project. By default, it looks something like this::

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    )

Temporarily comment out all six of those strings by putting a hash character
(``#``) in front of them. (They're included by default as a common-case
convenience, but we'll activate and discuss them in subsequent chapters.)
While you're at it, comment out the default ``MIDDLEWARE_CLASSES`` setting, too;
the default values in ``MIDDLEWARE_CLASSES`` depend on some of the apps we
just commented out. Then, add  ``'books'`` to the ``INSTALLED_APPS``
list, so the setting ends up looking like this::

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

``'mysite.books'`` refers to the ``books`` app we're working on. Each app in
``INSTALLED_APPS`` is represented by its full Python path -- that is, the path
of packages, separated by dots, leading to the app package.

Now that the Django app has been activated in the settings file, we can create
the database tables in our database. First, let's validate the models by
running this command::

    python manage.py validate

.. SL Tested ok

The ``validate`` command checks whether your models' syntax and logic are
correct. If all is well, you'll see the message ``0 errors found``. If you
don't, make sure you typed in the model code correctly. The error output should
give you helpful information about what was wrong with the code.

Any time you think you have problems with your models, run
``python manage.py validate``. It tends to catch all the common model problems.

If your models are valid, run the following command for Django to generate
``CREATE TABLE`` statements for your models in the ``books`` app (with colorful
syntax highlighting available, if you're using Unix)::

    python manage.py sqlall books

In this command, ``books`` is the name of the app. It's what you specified when
you ran the command ``manage.py startapp``. When you run the command, you
should see something like this::

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

.. SL Tested ok (sqlall output for postgres matches that shown here)

Note the following:

* Table names are automatically generated by combining the name of the app
  (``books``) and the lowercase name of the model (``publisher``,
  ``book``, and ``author``). You can override this behavior, as detailed
  in Appendix B.

* As we mentioned earlier, Django adds a primary key for each table
  automatically -- the ``id`` fields. You can override this, too.

* By convention, Django appends ``"_id"`` to the foreign key field name. As
  you might have guessed, you can override this behavior, too.

* The foreign key relationship is made explicit by a ``REFERENCES``
  statement.

* These ``CREATE TABLE`` statements are tailored to the database you're
  using, so database-specific field types such as ``auto_increment``
  (MySQL), ``serial`` (PostgreSQL), or ``integer primary key`` (SQLite) are
  handled for you automatically. The same goes for quoting of column names
  (e.g., using double quotes or single quotes). This example output is in
  PostgreSQL syntax.

The ``sqlall`` command doesn't actually create the tables or otherwise touch
your database -- it just prints output to the screen so you can see what SQL
Django would execute if you asked it. If you wanted to, you could copy and
paste this SQL into your database client, or use Unix pipes to pass it
directly (e.g., ``python manage.py sqlall books | psql mydb``). However, Django
provides an easier way of committing the SQL to the database: the ``syncdb``
command::

    python manage.py syncdb

Run that command, and you'll see something like this::

    Creating table books_publisher
    Creating table books_author
    Creating table books_book
    Installing index for books.Book model

.. SL Tested ok

The ``syncdb`` command is a simple "sync" of your models to your database. It
looks at all of the models in each app in your ``INSTALLED_APPS`` setting,
checks the database to see whether the appropriate tables exist yet, and
creates the tables if they don't yet exist. Note that ``syncdb`` does *not*
sync changes in models or deletions of models; if you make a change to a model
or delete a model, and you want to update the database, ``syncdb`` will not
handle that. (More on this in the "Making Changes to a Database Schema" section
toward the end of this chapter.)

If you run ``python manage.py syncdb`` again, nothing happens, because you
haven't added any models to the ``books`` app or added any apps to
``INSTALLED_APPS``. Ergo, it's always safe to run ``python manage.py syncdb``
-- it won't clobber things.

If you're interested, take a moment to dive into your database server's
command-line client and see the database tables Django created. You can
manually run the command-line client (e.g., ``psql`` for PostgreSQL) or
you can run the command ``python manage.py dbshell``, which will figure out
which command-line client to run, depending on your ``DATABASE_SERVER``
setting. The latter is almost always more convenient.

Basic Data Access
=================

Once you've created a model, Django automatically provides a high-level Python
API for working with those models. Try it out by running
``python manage.py shell`` and typing the following::

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

.. SL Tested ok

These few lines of code accomplish quite a bit. Here are the highlights:

* First, we import our ``Publisher`` model class. This lets us interact
  with the database table that contains publishers.

* We create a ``Publisher`` object by instantiating it with values for
  each field -- ``name``, ``address``, etc.

* To save the object to the database, call its ``save()`` method. Behind
  the scenes, Django executes an SQL ``INSERT`` statement here.

* To retrieve publishers from the database, use the attribute
  ``Publisher.objects``, which you can think of as a set of all publishers.
  Fetch a list of *all* ``Publisher`` objects in the database with the
  statement ``Publisher.objects.all()``. Behind the scenes, Django executes
  an SQL ``SELECT`` statement here.

One thing is worth mentioning, in case it wasn't clear from this example. When
you're creating objects using the Django model API, Django doesn't save the
objects to the database until you call the ``save()`` method::

    p1 = Publisher(...)
    # At this point, p1 is not saved to the database yet!
    p1.save()
    # Now it is.

If you want to create an object and save it to the database in a single step,
use the ``objects.create()`` method. This example is equivalent to the example
above::

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

.. SL Tested ok

Naturally, you can do quite a lot with the Django database API -- but first,
let's take care of a small annoyance.

Adding Model String Representations
===================================

When we printed out the list of publishers, all we got was this
unhelpful display that makes it difficult to tell the ``Publisher`` objects
apart::

    [<Publisher: Publisher object>, <Publisher: Publisher object>]

We can fix this easily by adding a method called ``__unicode__()`` to our
``Publisher`` class. A ``__unicode__()`` method tells Python how to display the
"unicode" representation of an object. You can see this in action by adding a
``__unicode__()`` method to the three models:

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

As you can see, a ``__unicode__()`` method can do whatever it needs to do in order
to return a representation of an object. Here, the ``__unicode__()`` methods for
``Publisher`` and ``Book`` simply return the object's name and title,
respectively, but the ``__unicode__()`` for ``Author`` is slightly more complex --
it pieces together the ``first_name`` and ``last_name`` fields, separated by a
space.

The only requirement for ``__unicode__()`` is that it return a Unicode object.
If ``__unicode__()`` doesn't return a Unicode object -- if it returns, say, an
integer -- then Python will raise a ``TypeError`` with a message like
``"coercing to Unicode: need string or buffer, int found"``.

.. admonition:: Unicode objects

    What are Unicode objects?

    You can think of a Unicode object as a Python string that can handle more
    than a million different types of characters, from accented versions of
    Latin characters to non-Latin characters to curly quotes and obscure
    symbols.

    Normal Python strings are *encoded*, which means they use an encoding such
    as ASCII, ISO-8859-1 or UTF-8. If you're storing fancy characters (anything
    beyond the standard 128 ASCII characters such as 0-9 and A-Z) in a normal
    Python string, you have to keep track of which encoding your string is
    using, or the fancy characters might appear messed up when they're
    displayed or printed. Problems occur when you have data that's stored in
    one encoding and you try to combine it with data in a different encoding,
    or you try to display it in an application that assumes a certain encoding.
    We've all seen Web pages and e-mails that are littered with "??? ??????"
    or other characters in odd places; that generally suggests there's an
    encoding problem.

    Unicode objects, however, have no encoding; they use a consistent,
    universal set of characters called, well, "Unicode." When you deal with
    Unicode objects in Python, you can mix and match them safely without having
    to worry about encoding issues.

    Django uses Unicode objects throughout the framework. Model objects are
    retrieved as Unicode objects, views interact with Unicode data, and
    templates are rendered as Unicode. Generally, you won't have to worry about
    making sure your encodings are right; things should just work.

    Note that this has been a *very* high-level, dumbed down overview of
    Unicode objects, and you owe it to yourself to learn more about the topic.
    A good place to start is http://www.joelonsoftware.com/articles/Unicode.html .

For the ``__unicode__()`` changes to take effect, exit out of the Python shell
and enter it again with ``python manage.py shell``. (This is the simplest way
to make code changes take effect.) Now the list of ``Publisher`` objects is
much easier to understand::

    >>> from books.models import Publisher
    >>> publisher_list = Publisher.objects.all()
    >>> publisher_list
    [<Publisher: Apress>, <Publisher: O'Reilly>]

.. SL Tested ok

Make sure any model you define has a ``__unicode__()`` method -- not only for
your own convenience when using the interactive interpreter, but also because
Django uses the output of ``__unicode__()`` in several places when it needs to
display objects.

Finally, note that ``__unicode__()`` is a good example of adding *behavior* to
models. A Django model describes more than the database table layout for an
object; it also describes any functionality that object knows how to do.
``__unicode__()`` is one example of such functionality -- a model knows how to
display itself.

Inserting and Updating Data
===========================

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
