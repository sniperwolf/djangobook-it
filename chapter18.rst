============================================================
Capitolo 18: Integrazione con database legacy e applicazioni
============================================================

Django è più adatto per il cosiddetto sviluppo green-field -- cioè, a partire
progetti da zero, come se si stesse costruendo un edificio su un campo fresco
di erba verde. Ma nonostante il suo favorire i progetti da-zero, è possibile
integrare il framework in database legacy e applicazioni già esistenti. Questo
capitolo spiega alcune strategie di integrazione.

L'integrazione con un database legacy
=====================================

Il livello di database di Django genera schemi SQL da codice Python -- ma con
un database legacy, hai già degli schemi SQL. In tal caso, hai bisogno di creare
modelli per le tabelle di database esistenti. Per questo scopo, Django è dotato
di uno strumento in grado di generare del codice per un modello leggendo il tuo
layout delle tabelle di database. Questo strumento si chiama ``inspectdb``, e si
può richiamare eseguendo il comando ``manage.py inspectdb``.

Utilizzare ``inspectdb``
------------------------

L'utilità ``inspectdb`` controlla il database a cui puntano le impostazioni
dei file, determina una rappresentazione in un modello Django per ogni tabella,
e stampa tale modello in codice Python standard output.

Ecco una guida passo per passo di un tipico processo di integrazione con database
legacy da configurare. Le uniche ipotesi sono che Django sia installato e che si
dispone di un database legacy.

1. Creare un progetto Django eseguendo ``django-admin.py startproject mysite``
   (dove ``mysite`` è il nome del progetto). Useremo ``mysite`` come nome del
   progetto in questo esempio.

2. Modificare il file di impostazioni del progetto, ``mysite/settings.py``,
   per dire a Django quali sono i parametri di connessione al database e quale è
   il nome del database. In particolare, bisogna specificare le impostazioni per
   ``DATABASE_NAME``, ``DATABASE_ENGINE``, ``DATABASE_USER``, ``DATABASE_PASSWORD``,
   ``DATABASE_HOST``, and ``DATABASE_PORT``. (Da notare che alcune di queste
   impostazioni sono opzionali. Fare riferimento al Capitolo 5 per
   ulteriori informazioni).

3. Creare un'applicazione Django all'interno del progetto eseguendo
   ``python mysite/manage.py startapp myapp`` (dove ``myapp`` è il nome dell'
   applicazione). Useremo ``myapp`` come nome dell'applicazione.

4. Eseguire il comando ``python mysite/manage.py inspectdb``. Questo esamina
   le tabelle nel database ``DATABASE_NAME`` e stampa un modello generato per
   ogni tabella. Dai un'occhiata all'output per avere un'idea di quello che
   ``inspectdb`` può fare.

5. Salvare l'output sul file ``models.py`` all'interno dell'applicazione
   utilizzando lo standard output shell::

       python mysite/manage.py inspectdb > mysite/myapp/models.py

6. Modificare il file ``mysite/myapp/models.py`` per ripulire i modelli
   generati e fare tutte le personalizzazioni necessarie. Daremo alcuni
   suggerimenti per fare questo nella prossima sezione.

Cleaning Up Generated Models
----------------------------

Ripulitura modelli generati
----------------------------

Come ci si potrebbe aspettare, l'introspezione del database non è perfetta, e
avrai bisogno di fare un po' di pulizia del codice del modello risultante. Qui
descriviamo alcune metodologie per trattare con i modelli generati::

1. Ogni tabella del database viene convertita in una classe del modello (cioè,
    c'è una corrispondenza uno-a-uno tra le tabelle del database e le classi del
    modello). Questo significa che avrai bisogno di un refactoring sui modelli
    per fare le tabelle molti-a-molti create tramite join in oggetti ``ManyToManyField``.

2. Ogni modello generato ha un attributo per ogni campo, compreso la chiave
   primaria ``id``. Tuttavia, ricorda che Django aggiunge automaticamente una
   chiave ``id`` primaria, se un modello ne ne ha una. Pertanto, ti consigliamo
   di rimuovere tutte le righe che somigliano a questa::

       id = models.IntegerField(primary_key=True)

    Non solo queste linee sono ridondanti, ma possono anche causare problemi se
    la tua applicazione aggiunga *nuovi* record a queste tabelle.

3. Ogni tipo di campo (ad esempio, ``CharField``, ``DateField``) è determinato
   guardando il tipo di colonna del database (ad esempio, ``VARCHAR``, ``DATA``).
   Se ``inspectdb`` non può mappare il tipo di una colonna di un tipo di campo modello,
   utilizzerà ``TextField`` ed inserirà il commento ``'This field type is a guess.'``
   accanto al campo generato per il modello. Presta attenzione a questo commento,
   e, di conseguenza, cambia il tipo di campo, se necessario.

   Se un campo del database non ha buon equivalente in Django, è possibile
   tranquillamente lasciarlo fuori. I modelli di Django non sono tenuti ad
   includere ogni campo della tabella.

4. Se il nome della colonna di database è una parola riservata di Python (come
   ``pass``, ``class``, o ``for``) ``inspectdb`` aggiungerà `` '_field' ``
   all'attributo del nome e imposterà l'attributo ``db_column`` il nome del
   campo reale (ad esempio, ``pass``, ``class``, o ``for``).

   Ad esempio, se una tabella ha una colonna ``INT`` chiamata ``for``, il
   modello generato avrà un campo così::

       for_field = models.IntegerField(db_column='for')

   ``inspectdb`` inserirà il commento Python
   ``'Field renamed because it was a Python reserved word.'`` accanto al campo.


5. Se il database contiene tabelle che fanno riferimento ad altre tabelle (come
   fanno la maggior parte dei Database), potrebbe essere necessario modificare
   l'ordine dei modelli generati in modo che i suddetti facciano riferimento
   agli altri modelli correttamente. Per esempio, se il modello ``Book`` ha una
   ``ForeignKey`` al modello ``Author``, il modello ``Author`` deve essere
   definito prima del modello ``Book``. Se hai bisogno di creare una relazione
   con un modello che non è ancora stato definito, si può utilizzare una stringa
   contenente il nome del modello, piuttosto che l'oggetto modello stesso.

6. ``inspectdb`` rileva le chiavi primarie per PostgreSQL, MySQL e SQLite.
    Cioè, inserisce ``primary_key=True`` nel caso appropriato. Per altri
    database, è necessario inserire ``primary_key=True`` per almeno un campo in
    ciascun modello, perché i modelli Django devono avere almeno uno di questi campi.

7. L'ndividuazione di chiave funziona solo con PostgreSQL e con alcuni tipi
    di tabelle MySQL. In altri casi, i campi di chiave verranno generati come
    ``IntegerField``, assumendo che la colonna di chiave esterna è un ``INT``.

Integrazione con un Sistema di Autenticazione
=============================================

È possibile integrare Django con un sistema di autenticazione già esistente --
un'altra fonte di nomi utente e password o metodi di autenticazione.

Ad esempio, la società può già avere una configurazione LDAP che memorizza un
nome utente e password per ogni dipendente. Sarebbe una seccatura sia per l'
amministratore di rete che per gli utenti stessi, avere account separati fra LDAP
e applicazioni basate su Django.

Per gestire situazioni come questa, il sistema di autenticazione Django consente
di collegare altre fonti di autenticazione. È possibile eseguire l'override del
sistema di default di Django, oppure è possibile utilizzare il sistema
di default in accoppiata con altri sistemi.

Specifying Authentication Backends
----------------------------------

Behind the scenes, Django maintains a list of "authentication backends" that it
checks for authentication. When somebody calls
``django.contrib.auth.authenticate()`` (as described in Chapter 14), Django
tries authenticating across all of its authentication backends. If the first
authentication method fails, Django tries the second one, and so on, until all
backends have been attempted.

The list of authentication backends to use is specified in the
``AUTHENTICATION_BACKENDS`` setting. This should be a tuple of Python path
names that point to Python classes that know how to authenticate. These classes
can be anywhere on your Python path.

By default, ``AUTHENTICATION_BACKENDS`` is set to the following::

    ('django.contrib.auth.backends.ModelBackend',)

That's the basic authentication scheme that checks the Django users database.

The order of ``AUTHENTICATION_BACKENDS`` matters, so if the same username and
password are valid in multiple backends, Django will stop processing at the
first positive match.

Writing an Authentication Backend
---------------------------------

An authentication backend is a class that implements two methods:
``get_user(id)`` and ``authenticate(**credentials)``.

The ``get_user`` method takes an ``id`` -- which could be a username, database
ID, or whatever -- and returns a ``User`` object.

The  ``authenticate`` method takes credentials as keyword arguments. Most of
the time it looks like this::

    class MyBackend(object):
        def authenticate(self, username=None, password=None):
            # Check the username/password and return a User.

But it could also authenticate a token, like so::

    class MyBackend(object):
        def authenticate(self, token=None):
            # Check the token and return a User.

Either way, ``authenticate`` should check the credentials it gets, and it
should return a ``User`` object that matches those credentials, if the
credentials are valid. If they're not valid, it should return ``None``.

The Django admin system is tightly coupled to Django's own database-backed
``User`` object described in Chapter 14. The best way to deal with this is to
create a Django ``User`` object for each user that exists for your backend
(e.g., in your LDAP directory, your external SQL database, etc.). Either you can
write a script to do this in advance or your ``authenticate`` method can do it
the first time a user logs in.

Here's an example backend that authenticates against a username and password
variable defined in your ``settings.py`` file and creates a Django ``User``
object the first time a user authenticates::

    from django.conf import settings
    from django.contrib.auth.models import User, check_password

    class SettingsBackend(object):
        """
        Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.

        Use the login name, and a hash of the password. For example:

        ADMIN_LOGIN = 'admin'
        ADMIN_PASSWORD = 'sha1$4e987$afbcf42e21bd417fb71db8c66b321e9fc33051de'
        """
        def authenticate(self, username=None, password=None):
            login_valid = (settings.ADMIN_LOGIN == username)
            pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
            if login_valid and pwd_valid:
                try:
                    user = User.objects.get(username=username)
                except User.DoesNotExist:
                    # Create a new user. Note that we can set password
                    # to anything, because it won't be checked; the password
                    # from settings.py will.
                    user = User(username=username, password='get from settings.py')
                    user.is_staff = True
                    user.is_superuser = True
                    user.save()
                return user
            return None

        def get_user(self, user_id):
            try:
                return User.objects.get(pk=user_id)
            except User.DoesNotExist:
                return None

For more on authentication backends, see the official Django documentation.

Integrating with Legacy Web Applications
========================================

It's possible to run a Django application on the same Web server as an
application powered by another technology. The most straightforward way of
doing this is to use Apache's configuration file, ``httpd.conf``, to delegate
different URL patterns to different technologies. (Note that Chapter 12 covers
Django deployment on Apache/mod_python, so it might be worth reading that
chapter first before attempting this integration.)

The key is that Django will be activated for a particular URL pattern only if
your ``httpd.conf`` file says so. The default deployment explained in Chapter
12 assumes you want Django to power every page on a particular domain::

    <Location "/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Here, the ``<Location "/">`` line means "handle every URL, starting at the
root," with Django.

It's perfectly fine to limit this ``<Location>`` directive to a certain
directory tree. For example, say you have a legacy PHP application that powers
most pages on a domain and you want to install a Django admin site at
``/admin/`` without disrupting the PHP code. To do this, just set the
``<Location>`` directive to ``/admin/``::

    <Location "/admin/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

With this in place, only the URLs that start with ``/admin/`` will activate
Django. Any other page will use whatever infrastructure already existed.

Note that attaching Django to a qualified URL (such as ``/admin/`` in this
section's example) does not affect the Django URL parsing. Django works with the
absolute URL (e.g., ``/admin/people/person/add/``), not a "stripped" version of
the URL (e.g., ``/people/person/add/``). This means that your root URLconf
should include the leading ``/admin/``.

What's Next?
============

If you're a native English speaker, you might not have noticed one of the
coolest features of Django's admin site: it's available in more than 50
different languages! This is made possible by Django's internationalization
framework (and the hard work of Django's volunteer translators). The
`next chapter`_ explains how to use this framework to provide localized Django
sites.

.. _next chapter: chapter19.html
