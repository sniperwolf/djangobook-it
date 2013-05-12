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

Specificare un Backend di Autenticazione
----------------------------------------

Dietro le quinte, Django mantiene una lista di "backend di autenticazione" che
controlla per l'autenticazione. Quando qualcuno chiama
``django.contrib.auth.authenticate()`` (come descritto nel capitolo 14), Django
cerca prova l'autenticazione in tutti i suoi backend di autenticazione. Se il
primo metodo di autenticazione non va a buon fine, Django tenta il seconda, e
 così via, finché tutti backend non siano stati provati.

La lista dei backend di autenticazione da utilizzare è specificata
nell'impostazione ``AUTHENTICATION_BACKENDS``. Questa dovrebbe essere una tupla
del nome del percorso Python che fa riferimento alle classi Python che sanno
come autenticare. Queste classi possono essere ovunque sul tuo percorso Python.

Per impostazione predefinita, ``AUTHENTICATION_BACKENDS`` è impostato come segue::

    ('django.contrib.auth.backends.ModelBackend',)

Questo è lo schema di autenticazione di base che controlla il database utenti di
Django.

L'ordine del ``AUTHENTICATION_BACKENDS`` ha un senso, quindi se lo stesso nome
utente e la password sono validi in diversi backend, Django si ferma
nell'elaborazione al primo riscontro positivo.

Scrivere un backend di autenticazione
-------------------------------------

Un backend di autenticazione è una classe che implementa due metodi:
``get_user (id)`` e ``authenticate(**credentials)``.

Il metodo ``get_user``  prende un ``id`` -- che può essere un nome utente, un ID
di un database o qualsiasi altra cosa -- e restituisce un oggetto ``User``.

Il metodo `authenticate`` prende le credenziali come argomenti chiave. Somiglia a
questo::

    class MyBackend(object):
        def authenticate(self, username=None, password=None):
            # Check the username/password and return a User.

Ma potrebbe autenticare anche un token, come il seguente::

    class MyBackend(object):
        def authenticate(self, token=None):
            # Check the token and return a User.

Altrimenti, ``authenticate`` può controllare le credenziali che gli si passa e
restituire un oggetto ``User`` che corrisponde con quelle credenziali, se quelle
sono valide. Se non lo sono, dovrebbe restituire un ``None``.


Il sistema admin di Django è strettamente accoppiato al proprio oggetto ``User``
presente nel database, come descritto nel capitolo 14. Il modo migliore per
affrontare questo problema è creare un oggetto ``User`` per ogni utente che
esiste per sul tuo backend (ad esempio, nella directory LDAP, il database SQL
esterno, ecc.) In entrambi i casi è possibile scrivere uno script per far fare
questo in anticipo o il tuo metodo ``authenticate`` lo può fare la prima volta
che un utente esegue il login.

Ecco un esempio di backend che autentica le variabili che rappresentano nome
utente e password variabile definita nel tuo ``settings.py`` e crea un oggetto
Django ``User`` la prima volta che un utente esegue l'autenticazione::

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

Per altri backend di autenticazione, leggi la documentazione ufficiale di Django.

Integrazione con Applicazioni Web Legacy
========================================

E 'possibile eseguire un'applicazione Django sullo stesso server Web come
un'applicazione alimentato da un'altra tecnologia. Il modo più diretto per farlo
è usare il file di configurazione di Apache, ``httpd.conf``, per delegare
diversi URL pattern in diverse tecnologie. (Da notare che il capitolo 12 copre
la distribuzione di Django su Apache/mod_python, quindi potrebbe valere la pena
leggere che capitolo prima di provare con questa integrazione).

La chiave è che Django sarà attivato per un particolare pattern di URL solo se
il tuo file ``httpd.conf`` dice così. L'implementazione predefinita è spiegata
nel capitolo 12 e si presuppone che si desidera dare a Django il potere di
alimentare tutte le pagina su un particolare dominio::

    <Location "/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Qui, la linea ``<Location "/">`` significa "gestisci ogni URL, a partire dalla
root, con Django".

E' perfettamente corretto limitare questa direttiva ``<Location>`` ad un ben
determinato albero di directory. Ad esempio, supponiamo di avere un'applicazione
PHP ereditata che ha il potere di gestire la maggior parte delle pagine di un
dominio e si desidera installare il pannello di amministratore di Django su
``/admin/`` senza smettere di far funzionare il codice PHP. Per fare ciò, è
sufficiente impostare la direttiva ``<Location>`` a ``/admin/``::

    <Location "/admin/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Con tutto questo, solo gli URL che iniziano con ``/admin/`` attiveranno
Django. Qualsiasi altra pagina utilizzerà qualsiasi infrastruttura già esistente.

Si noti che il collegamento Django ad un URL specifico (ad esempio ``/admin/``
in questo esempio) non influenza l'URL parsing di Django. Django si basa
sull'URL assoluto (ad esempio, ``/admin/people/person/add/``), non su una
versione "spogliata" (ad esempio, ``/people/person/add/``). Questo significa che
il tuo URLconf di base dovrebbe includere all'inizio ``/admin/``.

Cosa c'è adesso?
================

Se sei un madrelingua inglese, potresti non aver notato una delle
caratteristiche più interessanti del pannello di amministrazione di Django: è
disponibile in più di 50 lingue diverse! Ciò è reso possibile
dall'internazionalizzazione del framework Django (e il duro lavoro dei traduttori
volontari di Django). Nel `prossimo capitolo`_ spiega come utilizzare questo
framework per fornire una versione localizzata dei siti web Django.

.. _prossimo capitolo: chapter19.html