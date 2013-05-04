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

.. admonition:: Trucco:

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

Setting Up a Database
=====================

At this point, you could very well begin writing a Web application with Django,
because Django's only hard-and-fast prerequisite is a working Python
installation. However, odds are you'll be developing a *database-driven* Web
site, in which case you'll need to configure a database server.

If you just want to start playing with Django, skip ahead to the
"Starting a Project" section -- but keep in mind that all the examples in this
book assume you have a working database set up.

Django supports four database engines:

* PostgreSQL (http://www.postgresql.org/)
* SQLite 3 (http://www.sqlite.org/)
* MySQL (http://www.mysql.com/)
* Oracle (http://www.oracle.com/)

For the most part, all the engines here work equally well with the core Django
framework. (A notable exception is Django's optional GIS support, which is much
more powerful with PostgreSQL than with other databases.) If you're not tied to
any legacy system and have the freedom to choose a database backend, we
recommend PostgreSQL, which achieves a fine balance between cost, features,
speed and stability.

Setting up the database is a two-step process:

* First, you'll need to install and configure the database server itself.
  This process is beyond the scope of this book, but each of the four
  database backends has rich documentation on its Web site. (If you're on
  a shared hosting provider, odds are that they've set this up for you
  already.)

* Second, you'll need to install the Python library for your particular
  database backend. This is a third-party bit of code that allows Python to
  interface with the database. We outline the specific, per-database
  requirements in the following sections.

If you're just playing around with Django and don't want to install a database
server, consider using SQLite. SQLite is unique in the list of supported
databases in that it doesn't require either of the above steps. It merely reads
and writes its data to a single file on your filesystem, and Python versions 2.5
and higher include built-in support for it.

On Windows, obtaining database driver binaries can be frustrating. If you're
eager to jump in, we recommend using Python 2.7 and its built-in support for
SQLite.

Using Django with PostgreSQL
----------------------------

If you're using PostgreSQL, you'll need to install either the ``psycopg`` or
``psycopg2`` package from http://www.djangoproject.com/r/python-pgsql/. We
recommend ``psycopg2``, as it's newer, more actively developed and can be
easier to install. Either way, take note of whether you're using version 1 or
2; you'll need this information later.

If you're using PostgreSQL on Windows, you can find precompiled binaries of
``psycopg`` at http://www.djangoproject.com/r/python-pgsql/windows/.

If you're on Linux, check whether your distribution's package-management
system offers a package called "python-psycopg2", "psycopg2-python",
"python-postgresql" or something similar.

Using Django with SQLite 3
--------------------------

You're in luck: no database-specific installation is required, because Python
ships with SQLite support. Skip ahead to the next section.

Using Django with MySQL
-----------------------

Django requires MySQL 4.0 or above. The 3.x versions don't support nested
subqueries and some other fairly standard SQL statements.

You'll also need to install the ``MySQLdb`` package from
http://www.djangoproject.com/r/python-mysql/.

If you're on Linux, check whether your distribution's package-management system
offers a package called "python-mysql", "python-mysqldb", "mysql-python" or
something similar.

Using Django with Oracle
------------------------

Django works with Oracle Database Server versions 9i and higher.

If you're using Oracle, you'll need to install the ``cx_Oracle`` library,
available at http://cx-oracle.sourceforge.net/. Use version 4.3.1 or higher, but
avoid version 5.0 due to a bug in that version of the driver.  Version 5.0.1
resolved the bug, however, so you can choose a higher version as well.

Using Django Without a Database
-------------------------------

As mentioned earlier, Django doesn't actually require a database. If you just
want to use it to serve dynamic pages that don't hit a database, that's
perfectly fine.

With that said, bear in mind that some of the extra tools bundled with Django
*do* require a database, so if you choose not to use a database, you'll miss
out on those features. (We highlight these features throughout this book.)

Starting a Project
==================

Once you've installed Python, Django and (optionally) your database
server/library, you can take the first step in developing a Django application
by creating a *project*.

A project is a collection of settings for an instance of Django, including
database configuration, Django-specific options and application-specific
settings.

If this is your first time using Django, you'll have to take care of some
initial setup. Create a new directory to start working in, perhaps something
like ``/home/username/djcode/``.

.. admonition:: Where Should This Directory Live?

    If your background is in PHP, you're probably used to putting code under the
    Web server's document root (in a place such as ``/var/www``). With Django,
    you don't do that. It's not a good idea to put any of this Python code
    within your Web server's document root, because in doing so you risk the
    possibility that people will be able to view your raw source code over the
    Web. That's not good.

    Put your code in some directory **outside** of the document root.

Change into the directory you created, and run the command
``django-admin.py startproject mysite``. This will create a ``mysite``
directory in your current directory.

.. note::

    ``django-admin.py`` should be on your system path if you installed Django
    via its ``setup.py`` utility.

    If you're using the development version, you'll find ``django-admin.py`` in
    ``djmaster/django/bin``. Because you'll be using ``django-admin.py``
    often, consider adding it to your system path. On Unix, you can do so by
    symlinking from ``/usr/local/bin``, using a command such as ``sudo ln -s
    /path/to/django/bin/django-admin.py /usr/local/bin/django-admin.py``. On
    Windows, you'll need to update your ``PATH`` environment variable.

    If you installed Django from a packaged version for your Linux
    distribution, ``django-admin.py`` might be called ``django-admin`` instead.

If you see a "permission denied" message when running
``django-admin.py startproject``, you'll need to change the file's permissions.
To do this, navigate to the directory where ``django-admin.py`` is installed
(e.g., ``cd /usr/local/bin``) and run the command ``chmod +x django-admin.py``.

The ``startproject`` command creates a directory containing five files::

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

.. note:: Doesn't match what you see?

    The default project layout recently changed. If you're seeing a
    "flat" layout (with no inner ``mysite/`` directory), you're probably using
    a version of Django that doesn't match this tutorial version. This book covers
    Django 1.4 and above, so if you're using an older version you probably want to
    consult Django's official documentation.

    The documentation for Django 1.X version is available at https://docs.djangoproject.com/en/1.X/.

These files are as follows:

* ``mysite/``: The outer ``mysite/`` directory is just a container for your project.
  Its name doesn't matter to Django; you can rename it to anything you like.

* ``manage.py``: A command-line utility that lets you interact with this
  Django project in various ways. Type ``python manage.py help`` to get a
  feel for what it can do. You should never have to edit this file; it's
  created in this directory purely for convenience.

* ``mysite/mysite/``: The inner ``mysite/`` directory is the actual Python package
  for your project. Its name is the Python package name you'll need to use to
  import anything inside it (e.g. ``import mysite.settings``).

* ``__init__.py``: A file required for Python to treat the ``mysite``
  directory as a package (i.e., a group of Python modules). It's an empty
  file, and generally you won't add anything to it.

* ``settings.py``: Settings/configuration for this Django project. Take a
  look at it to get an idea of the types of settings available, along with
  their default values.

* ``urls.py``: The URLs for this Django project. Think of this as the
  "table of contents" of your Django-powered site.

* ``wsgi.py``: An entry-point for WSGI-compatible webservers to serve your project.
  See How to deploy with WSGI (https://docs.djangoproject.com/en/1.4/howto/deployment/wsgi/) for more details.

Despite their small size, these files already constitute a working Django
application.

Running the Development Server
------------------------------

For some more post-installation positive feedback, let's run the Django
development server to see our barebones application in action.

The Django development server (also called the "runserver" after the command
that launches it) is a built-in, lightweight Web server you can use while
developing your site. It's included with Django so you can develop your site
rapidly, without having to deal with configuring your production server (e.g.,
Apache) until you're ready for production. The development server watches your
code and automatically reloads it, making it easy for you to change your code
without needing to restart anything.

To start the server, change into your project container directory (``cd mysite``),
if you haven't already, and run this command::

    python manage.py runserver

You'll see something like this::

    Validating models...
    0 errors found.

    Django version 1.4.2, using settings 'mysite.settings'
    Development server is running at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.

This launches the server locally, on port 8000, accessible only to connections
from your own computer. Now that it's running, visit http://127.0.0.1:8000/
with your Web browser. You might see a different Django version depending on
which version of Django you have installed. You'll see a "Welcome to Django" page shaded in a
pleasant pastel blue. It worked!

One final, important note about the development server is worth mentioning
before proceeding. Although this server is convenient for development, resist
the temptation to use it in anything resembling a production environment. The
development server can handle only a single request at a time reliably, and it
has not gone through a security audit of any sort. When the time comes to
launch your site, see Chapter 12 for information on how to deploy Django.

.. admonition:: Changing the Development Server's Host or Port

    By default, the ``runserver`` command starts the development server on port
    8000, listening only for local connections. If you want to change the
    server's port, pass it as a command-line argument::

        python manage.py runserver 8080

    By specifying an IP address, you can tell the server to allow non-local
    connections. This is especially helpful if you'd like to share a
    development site with other members of your team. The IP address
    ``0.0.0.0`` tells the server to listen on any network interface::

        python manage.py runserver 0.0.0.0:8000

    When you've done this, other computers on your local network will be able
    to view your Django site by visiting your IP address in their Web browsers,
    e.g., http://192.168.1.103:8000/ . (Note that you'll have to consult your
    network settings to determine your IP address on the local network. Unix
    users, try running "ifconfig" in a command prompt to get this information.
    Windows users, try "ipconfig".)

What's Next?
============

Now that you have everything installed and the development server running,
you're ready to :doc: learn the basics `Chapter 3`_, of serving Web pages with Django.

.. _Chapter 3: chapter03.html
