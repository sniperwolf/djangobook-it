=============================================================
Chapter 6: L'interfaccia di amministrazione (admin) di Django
=============================================================

Per una certa classe di siti web, una *interfaccia d'amministrazione* è una parte
essenziale dell'infrastruttura. Si tratta di una interfaccia web, limitata agli
amministratori del sito, che consente l'aggiunta, la modifica e la cancellazione
dei contenuti del sito. Alcuni esempi comuni: l'interfaccia da utilizzare per
postare sul tuo blog, i gestori del sito la usano per moderare i commenti
scritti dagli utenti, mentre i clienti la utilizzano per aggiornare i comunicati
stampa sul sito web che hai costruito per loro.

Però, c'è un problema con le interfacce d'amministrazione: è noioso per
costruirle. Lo sviluppo web è divertente quando si sta sviluppando funzionalità
rivolte al pubblico, mentre la creazione di interfacce di amministrazione è
sempre la stessa storia. È necessario autenticare gli utenti, visualizzare e
gestire i form, convalidare l'input, e così via. E' noioso e ripetitivo.

Allora, qual è l'approccio di Django a questi compiti ripetitivi? Fa tutto per
noi -- in appena un paio di righe di codice, niente di meno. Con Django, la
costruzione di un interfaccia di amministrazione è un problema risolto.

Questo capitolo riguarda l'interfaccia di amministrazione automatica di Django.
La feature funge leggendo i metadati nel modello per fornire un'interfaccia
potente e pronta per la produzione che gli amministratori del sito possono
iniziare ad utilizzare immediatamente. Qui, discutiamo come attivare, utilizzare
e personalizzare queste funzioni.

Consigliamo di leggere questo capitolo, anche se si utilizzerà mai
l'admin di Django, poiché vengono introdotti alcuni concetti che si applicano
spesso in varie parte di Django, indipendentemente dall'utilizzo dell'admin.

I pacchetti django.contrib
==========================

L'Admin automatico di Django è parte di una più ampia suite di funzionalità
Django chiamate ``django.contrib`` -- la parte di codebase che contiene vari
utili add-on per il framework. Si può pensare a django.contrib come l'equivalente
Django della libreria standard di Python -- opzionale, ma di fatto, implementata
in pattern di sviluppo assai comuni. Sono integrati in Django in modo che non
è necessario di reinventare la ruota in tutte le applicazioni.

L'amministratore del sito è la prima parte di ``django.contrib`` di cui parliamo
in questo libro, tecnicamente, si chiama ``django.contrib.admin``. Altre
funzioni disponibili in ``django.contrib`` includono un sistema di
autenticazione utente (``django.contrib.auth``), il supporto per le sessioni
anonime (``django.contrib.session``) e anche un sistema per i commenti degli
utenti (``django.contrib.comments``). Si arriva a conoscere tutte le varie
caratteristiche ``django.contrib`` solo quando si diventa un esperto di Django,
e passeremo un po 'di tempo a discutere di loro nel Capitolo 16. Per ora, è
sufficiente sapere che Django ha molti utili add-on, e ``django.contrib`` è
generalmente il luogo in cui si trovano.

Attivazione dell'interfaccia Admin
==================================

L'amministratore del sito Django è facoltativa, perché solo alcuni tipi di siti
hanno bisogno di questa funzionalità. Questo significa che hai bisogno di fare
qualche passo aggiuntivo per attivarlo nel tup progetto.

In primo luogo, bisogna fare un paio di modifiche al file di impostazioni::

1. Aggiungi ``'django.contrib.admin'`` all'impostazione ``INSTALLED_APPS``.
   (l'ordine in ``INSTALLED_APPS`` non importa, ma a noi piace mantenere le cose
   in ordine alfabetico, in modo che sia più facile da leggere per un essere
   umano).

2. Assicurarsi che ``INSTALLED_APPS`` contiene ``'django.contrib.admin'``,
   ``'django.contrib.contenttypes'``, ``'django.contrib.messages'`` e
   ``'django.contrib.sessions'``. L'admin richiede questi tre pacchetti. (Se
   stai continuando con il progetto ``mysite``, nota che abbiamo commentato
   queste quattro voci nel Capitolo 5. Basta decommentarle adesso).

3. Assicurarsi che ``MIDDLEWARE_CLASSES`` contenga
   ``'django.middleware.common.CommonMiddleware'``,
   ``'django.contrib.messages.middleware.MessageMiddleware'``,
   ``'django.contrib.sessions.middleware.SessionMiddleware'`` e
   ``'django.contrib.auth.middleware.AuthenticationMiddleware'``. (Anche in
   questo caso, se stai seguendo l'esempio, li abbiamo commentati nel capitolo
   5).

In secondo luogo, esegui ``python manage.py syncdb``. Questo passaggio
installerà delle tabelle extra nel database che utilizza l'interfaccia di
amministrazione. La prima volta che si esegue ``syncdb`` con ``'django.contrib.auth'``
in ``INSTALLED_APPS``, viene chiesto di creare un superuser. Se non esegui
questa operazione, è necessario separatamente eseguire il comando
``python manage.py createsuperuser`` per creare un account utente amministratore,
altrimenti non sarai in grado di accedere al sito admin. (Potenziale problema:
il comando ``python manage.py createsuperuser`` è disponibile solo se
``'django.contrib.auth'`` è presente in ``INSTALLED_APPS``).

Infine, aggiungi l'admin del sito all'URLconf (ricorda, in ``urls.py``). Per
impostazione predefinita, l'``urls.py`` generato da ``django-admin.py startproject``
contiene un po' di codice commentato per abilitare l'admin di Django, e quindi
tutto quello che bisogna fare è semplicemente rimuovere il commento. Per la
cronaca, qui c'è lo snippet necessario per verificare tutto è al suo posto::

    # Include these import statements...
    from django.contrib import admin
    admin.autodiscover()

    # And include this URLpattern...
    urlpatterns = patterns('',
        # ...
        (r'^admin/', include(admin.site.urls)),
        # ...
    )

Con queste configurazioni, ora è possibile vedere l'admin Django in azione.
Basta eseguire il server di sviluppo (``python manage.py runserver``, come nei
capitoli precedenti) e visitare ``http://127.0.0.1:8000/admin/`` nel browser.

Utilizzare l'interfaccia Admin
==============================

L'interfaccia Admin è stato progettato per essere utilizzato da utenti non
tecnici, e per questa ragione dovrebbe essere abbastanza intuitivo. Tuttavia,
ti daremo una guida rapida alle caratteristiche base.

La prima cosa che vedremo è una schermata di login, come mostrato nella
Figura 6-1.

.. figure:: graphics/chapter06/login.png
   :alt: Screenshot della schermata di login di Django.

   Figura 6-1. La schermata di login di Django

Accedi con il nome utente e la password che hai impostato quando hai aggiunto
il tuo superutente/superuser. Se non riesci ad accedere, assicurati di aver
effettivamente creato un superuser -- prova ad eseguire il comando
``python manage.py createsuperuser``.

Una volta effettuato l'accesso, la prima cosa che vedrai sarà la home page
dell'interfaccia admin. Questa pagina elenca tutti i tipi di dati disponibili
che possono essere modificati all'interno dell'admin. A questo punto, poiché non
abbiamo ancora attivato alcun modello, la lista è abbastanza scarna: include
solo 'Gruppi' e 'Utenti', che sono i due modelli personalizzabili di default.

.. figure:: graphics/chapter06/admin_index.png
   :alt: Screenshot della homepage dell'admin di Django

   Figura 6-2. La homepage dell'admin di Django

Ogni tipo di dato nel sito d'amministrazione ha una *pagina di elenco* ed un
*modulo/form di modifica*. Le liste dei cambiamenti mostrano tutti gli oggetti
disponibili nel database, mentre i moduli di modifica permettono di aggiungere,
modificare o eliminare particolari record nel database.

.. admonition:: Altre lingue

    Se la lingua principale non è l'inglese e il browser Web è configurato per
    preferire una lingua diversa dall'inglese, è possibile effettuare un rapido
    cambio per verificare se l'admin è stato tradotto in questa lingua. Basta
    aggiungere ``'django.middleware.locale.LocaleMiddleware'`` nelle impostazioni
    ``MIDDLEWARE_CLASSES``, facendo attenzione che appaia dopo
    ``'django.contrib.sessions.middleware.SessionMiddleware'``.

    Una volta fatto ciò, ricaricare l'homepage dell'admin. Se è disponibile una
    traduzione, le varie parti dell'interfaccia - dal "Modifica password" al
    link "Esci" nella parte superiore della pagina, o i link "Gruppi" e "Utenti"
    in mezzo -- appariranno in questa lingua, invece che in inglese. Django ha
    traduzioni per decine di lingue.

    E' possibile ottenere più informazioni sulle caratteristiche di
    internazionalizzazione di Django nel Capitolo 19.

Per caricare la pagina di gestione degli utenti, cliccare sul link "Modifica"
nella sezione "Utenti".

.. figure:: graphics/chapter06/user_changelist.png
   :alt: Screenshot della pagina di gestione dell'utente.

   Figure 6-3. La pagina di gestione dell'utente

In questa pagina vengono visualizzati tutti gli utenti nel database, e la si può
interpretare come una versione web abbellita di una query SQL ``SELECT * FROM auth_user;``.
Seguendo il nostro esempio, vedrai solo un utente, assumendo che ne sia stato
aggiunto solo uno, ma una volta aggiunti altri utenti, probabilmente troverai
molto utili le opzioni per il filtraggio, l'ordinamento e di ricerca. Le opzioni
di filtraggio sono a destra, l'ordinamento è disponibile cliccando su una
colonna, e la casella di ricerca nella parte superiore consente di cercare per
nome utente.

Cliccando sul nome dell'utente creato, vedremo il form di modifica per l'utente.

.. figure:: graphics/chapter06/user_editform.png
   :alt: Screenshot del modulo di modifica per l'utente.

   Figure 6-4. Il modulo di modifica per l'utente

Questa pagina permette di modificare gli attributi dell'utente, come nome e
cognome, ed i vari permessi. (Da notare che per cambiare la password di un
utente, è necessario cliccare su "Modulo di modifica della password" sotto il
campo password invece di modificare direttamente il codice hash). Un'altra cosa
da notare qui è che campi ottengono diversi widgets a seconda del loro tipo --
per esempio, i campi di data/ora hanno controlli basati su calendario, i campi
booleani hanno dei checkbox, i campi di carattere sono semplici campi di input
di testo.

È possibile eliminare un record cliccando sul pulsante "Elimina" nella parte
inferiore sinistra del modulo di modifica. Questo ti reindirizza ad una pagina
di conferma, la quale, in alcuni casi, visualizza tutti gli oggetti che
dipendono da esso e che verranno eliminati. (Per esempio, se si elimina un
editore, qualsiasi libro di quella casa editrice sarà anch'esso cancellato).

È possibile aggiungere un record cliccando sul link "Aggiungi" presente nella
colonna. La pagina che ne verrà fuori sarà una versione vuota della pagina di
modifica, pronta da compilare.

Noterai anche che l'interfaccia di amministrazione gestisce la convalida
dell'input per noi. Prova a lasciare un campo richiesto vuoto o a mettere in un
campo una data non valida, e riceverai degli errori quando si tenta di salvare,
come mostrato nella Figura 6-5.

.. figure:: graphics/chapter06/user_editform_errors.png
   :alt: Screenshot di un modulo di modifica che mostra errori.

   Figure 6-5. Un modulo di modifica che mostra errori

Quando si modifica un oggetto esistente, viene creato un collegamento
nell'angolo superiore destro della finestra "Cronologia". Ogni modifica fatta
attraverso l'interfaccia admin viene registrata ed è possibile esaminare il
registro cliccando sul collegamento "Cronologia" (vedi Figura 6-6).

.. figure:: graphics/chapter06/user_history.png
   :alt: Screenshot di un oggetto nella pagina della cronologia.

   Figure 6-6. Un oggetto nella pagina della cronologia.

Aggiungere i proprio Modelli all'interfaccia Admin
==================================================

C'è una parte cruciale a cui non abbiamo ancora fatto cenno. Andiamo ad
aggiungere i nostri modelli nell'admin, in modo che si possa aggiungere,
modificare ed eliminare gli oggetti contenuti nelle tabelle di database
personalizzati utilizzando questa bella interfaccia. Continueremo l'esempio
dei ``books`` del capitolo 5, dove abbiamo definito tre modelli: ``Publisher``,
``Author`` e ``Book``.

All'interno della directory ``books`` (``mysite/books``), crea un file chiamato
``admin.py``, e digita le seguenti righe di codice::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    admin.site.register(Publisher)
    admin.site.register(Author)
    admin.site.register(Book)

Questo codice dice all'admin di aggiungere un'interfaccia per ciascuno di questi
modelli.

Una volta fatto ciò, apri la pagina di amministrazione nel browser
(``http://127.0.0.1:8000/admin/``), e dovresti vedere una sezione "Books" con i
link degli Autori/Authors, Libri/Books ed Editori/Publishers. (Potrebbe essere
necessario arrestare e avviare il ``runserver`` affinché le modifiche abbiano
effetto).

Abbiamo ora un'interfaccia di amministrazione pienamente funzionale per ciascuno
di questi tre modelli. E' stato facile!

Prenditi del tempo per aggiungere e modificare i record, per popolare il
database con alcuni dati. Se hai seguito gli esempi del Capitolo 5 nela
creazione di oggetti di ``Publisher`` (e non li hai eliminati), avrai già visto
quei record nella pagina di elenco.

Una caratteristica degna di nota è la gestione del sito amministrazione di
chiavi esterne e di relazioni molti-a-molti, entrambi presenti nel modello ``Book``.
Come promemoria, ecco come si presenta il modello di ``Book``::

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

        def __unicode__(self):
            return self.title

Sulla pagina admin di "Aggiungi Libro" (``http://127.0.0.1:8000/admin/books/book/add/``),
l'editore/publisher (una ``ForeignKey``, chiave esterna) è rappresentato da una
casella di selezione, mentre il campo degli autori/authors (un relazione molti-a-molti,
``ManyToManyField``) è rappresentato da un checkbox con selezione multipla.
Entrambi i campi stanno accanto ad un'icona con all'interno un segno più verde
che permette di aggiungere nuovi record correlati al tipo. Per esempio, se
clicchi sul segno più verde accanto al campo "editore/publisher", viene aperta
una finestra di pop-up che da la possibilità di aggiungere un editore/publisher.
Dopo aver creato con successo l'editore all'interno del pop-up, il form di
"Aggiungi Libro" verrà aggiornato con la nuova casa editrice/publisher appena
creato.

Come funziona l'Interfaccia di amministrazione
==============================================

Dietro le quinte, come funziona l'interfaccia di amministrazione? E 'piuttosto
semplice.

Quando Django carica l'URLconf da ``urls.py`` all'avvio del server, esegue
l'istruzione ``admin.autodiscover()`` che abbiamo aggiunto come parte
dell'attivazione dell'admin. Questa funzione itera l'impostazione ``INSTALLED_APPS``
e cerca un file chiamato ``admin.py`` per ed in ogni app installata. Se viene
trovato ``admin.py`` in una determinata app, esegue il codice presente in quel
file.

Nell'``admin.py`` della nostra app ``books``, ogni chiamata ad
``admin.site.register()`` registra semplicemente il modello dato nell'admin.
All'amministratore del sito viene mostrato solo un'interfaccia per la
modifica dei modelli che sono stati registrati in modo esplicito.

L'app ``django.contrib.auth`` include il proprio ``admin.py``, motivo per cui
sono mostrati automaticamente 'Utenti' e 'Gruppi' nell'admin. Inoltre vengono
aggiunte altre app presenti in ``django.contrib``, come ``django.contrib.redirects``,
così come molte app di terze parti che puoi scaricare su internet.

Aldilà di questo fatto, l'admin di Django è solo una app Django, con i suoi
modelli, template, view e URLpatterns. La puoi aggiungere alla tua applicazione
includendola nel tuo URLconf, proprio come si includono le proprie view.
È possibile controllare i suoi modelli, view e URLpatterns rovistando in
``django/contrib/admin`` -- ma fare modifiche direttamente lì, visto la marea
di possibilità che offre il pannello di amministrazione. (Se vuoi vedere l'app
admin di Django, tieni a mente che fa alcune cose piuttosto complicate nel
leggere i metadati relativi a modelli, quindi è bene prendeti un bel po' di
tempo per leggere e comprendere il codice).

Rendere campi opzionali
=======================

Dopo aver giocato con l'admin per un po', probabilmente noterai una limitazione
-- il form di modifica richiede che ogni campo sia compilato, mentre nei casi
reali molti di essi sono opzionali. Assumiamo, per esempio, di volere rendere
opzionale il campo ``email`` del nostro modello ``Author`` -- ovvero, consentire
la possibilità di avere una stringa vuota.

Per specificare che il campo ``email`` è facoltativo, modifica il modello
``Author`` (che, come ricorderai dal capitolo 5, sta in ``mysite/books/models.py``).
Basta aggiungere ``blank=True`` al campo ``email``, in questo modo:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(**blank=True**)

Questo dice a Django che è consentito un valore vuoto per gli indirizzi e-mail
degli autori. Per impostazione predefinita, tutti i campi hanno ``blank=False``,
il che significa che i valori vuoti non sono ammessi.

C'è qualcosa di interessante che accade qui. Fino ad ora, con l'eccezione del
metodo ``__unicode__()``, i nostri modelli ci sono serviti come definizione
delle nostre tabelle -- espressioni 'Pythoneggianci' di istruzioni ``CREATE TABLE``,
essenzialmente. Aggiungendo ``blank=True``, abbiamo iniziato ad espandere il
nostro modello al di là di una semplice definizione, spingendola a farla
diventare a qualcosa di simile alla tabella stessa. Ora, la nostra classe modello
sta cominciando a diventare una collezione più ricca sulla conoscenza di ciò che
gli oggetti ``Author`` sono e cosa possono fare. Non solo il campo email è
rappresentato da una colonna ``VARCHAR`` nel database, ma è anche un campo
facoltativo in contesti quali l'interfaccia d'amministratore Django.

Una volta aggiunto che ``blank=True``, ricaricare il modulo "Aggiungi autore"
di modifica (``http://127.0.0.1:8000/admin/books/author/add/``), e noterai che
l'etichetta del campo -- "Email" -- non è più in grassetto. Questo significa che
non è un campo obbligatorio. È ora possibile aggiungere gli autori, senza
bisogno di specificare indirizzi e-mail, non ottendendo così l'errore rosso
"Questo campo è richiesto" ("This field is required" ), se il campo è vuoto.

Rendere Campi Numerici e Campi relativi a Date Facoltativi
----------------------------------------------------------

Un problema comune legata a ``blank=True``, ha a che fare con la data ed i
campi numerici, ma questo richiede una spiegazione di fondo.

SQL ha il suo modo di specificare valori vuoti -- esiste un valore speciale
chiamato ``NULL``. ``NULL`` potrebbe significare "sconosciuto", o "non valido"
o qualche altro significato specifico dell'applicazione.

In SQL, un valore ``NULL`` è diverso da una stringa vuota, proprio come lo
l'oggetto speciale ``None`` in Python è diverso da una stringa vuota (``""``).
Questo significa che per un particolare campo di caratteri (ad esempio, una
colonna ``VARCHAR``) è possibile contenere sia i valori ``NULL`` che valori
di stringa vuota.

Ciò può causare ambiguità indesiderate e confusione: "Perché questo entry ha un
valore ``NULL``, mentre quest'altro ha una stringa vuota? C'è una differenza, o
i dati appena inseriti sono incoerenti?". E ancora "Come faccio ad ottenere
tutti i record che hanno un valore vuoto - devo cercare entrambi i record ``NULL``
e le stringhe vuote, oppure posso selezionare solo quelli con stringhe vuote?".

Per evitare tale ambiguità, Django è genera automaticamente istruzioni ``CREATE TABLE``
(di cui abbiamo parlato nel Capitolo 5) aggiungendo un esplicito ``NOT NULL``
ad ogni definizione di colonna. Ad esempio, ecco la dichiarazione generata per
il nostro modello ``Author``, dal capitolo 5::

    CREATE TABLE "books_author" (
        "id" serial NOT NULL PRIMARY KEY,
        "first_name" varchar(30) NOT NULL,
        "last_name" varchar(40) NOT NULL,
        "email" varchar(75) NOT NULL
    )
    ;

Nella maggior parte dei casi, questo comportamento predefinito è ottimale per la
tua applicazione e ti farà risparmiare mal di testa a causati da dati incoerenti.
E funziona bene con il resto del Django, come ad esempio l'admin di Django, che
inserisce una stringa vuota (non un valore ``NULL``) quando un campo di
caratteri si lascia vuoto.

Ma c'è un'eccezione con tipi di colonna di database che non accettano stringhe
vuote come valori validi - come date, orari e numeri. Se si prova ad inserire
una stringa vuota in una data o ad una colonna di interi, è probabile ottenere
un errore del database, a seconda del database che si sta utilizzando.
(PostgreSQL, che è rigoroso, qui genera un'eccezione. MySQL potrebbe accettarlo
o meno, a seconda della versione che si sta utilizzando, l'ora del giorno e
della fase della luna). In questo caso, ``NULL`` è l'unico modo per specificare
un valore vuoto. Nei modelli Django, è possibile specificare che ``NULL`` sia
consentito con l'aggiunta di ``null=True`` a un campo.

Ecco, questo è un modo lungo di dire: se vuoi permettere di inserire valori
vuoti ad un campo data (ovvero, ``DateField``, ``TimeField``, ``DateTimeField``)
o ad un campo numerico (ovver, ``IntegerField``, ``DecimalField``, ``FloatField``),
hai bisogno di usare sia ``null=True`` *e* ``blank=True``.

Per fare un esempio banale, cambiamo il nostro modello per permettere che il
campo ``publication_date`` possa essere vuoto. Ecco il codice rivisitato:

.. parsed-literal::

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField(**blank=True, null=True**)

L'aggiunta di ``null=True`` è più complicata dell'aggiunta del ``blank=True``,
perché i ``null=True`` cambiano la semantica del database -- ovvero, cambia
l'istruzione ``CREATE TABLE`` per rimuovere il ``NOT NULL`` dal campo ``publication_date``.
Per completare questo cambiamento, abbiamo bisogno di aggiornare il database.

Per una serie di ragioni, Django non tenta di automatizzare le modifiche agli
schemi del database, quindi è tua responsabilità eseguire l'appropriata
istruzione ``ALTER TABLE`` ogni volta che si apporta una modifica a un tale
modello. Ricordiamo che è possibile utilizzare ``manage.py dbshell`` per avere
la shell del server del database. Ecco come rimuovere il ``NOT NULL`` in questo
caso particolare::

    ALTER TABLE books_book ALTER COLUMN publication_date DROP NOT NULL;

(Nota che questa sintassi SQL è specifica per PostgreSQL).

Parleremo delle modifiche allo schema in modo più approfondito nel capitolo 10.

Ritornando all'admin, ora il form di modifica "Aggiungi Libro" dovrebbe
permettere di inserire date di pubblicazione vuote.

Personalizzare le etichette dei Campi
=====================================

Sulla modifica il form del sito admin, l'etichetta di ciascun campo è generata
dal nome del campo del modello. L'algoritmo è semplice: Django sostituisce solo
underscore con spazi e capitalizza il primo carattere, quindi, ad esempio, il
campo ``publication_date`` del modello ``Book`` ha l'etichetta "Data di pubblicazione"
("Publication date").

Tuttavia, i nomi dei campi non sempre si prestano a buone etichette per campi
d'amministrazione, per cui, in alcuni casi, è bene personalizzare un'etichetta.
Puoi farlo specificando ``verbose_name`` nel modello appropriato.

Per esempio, ecco come possiamo cambiare l'etichetta del campo ``Author.email``,
trasformandolo in "e-mail", con un trattino:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(blank=True, **verbose_name='e-mail'**)

Assicurati di riavviare il server per vedere la nuova etichetta del campo nella
pagina di modifica del modello autore/author.

E' bene evitare di capitalizzare la prima lettera di un ``verbose_name`` a meno
che non debba essere sempre mostrato in maiuscolo (ad esempio, ``"USA state"``).
Django capitalizza questa stringa automaticamente quando è necessario, mentre ne
utilizzerà il valore esatto in altri posti che non richiedono la capitalizzazione.

Infine, nota che è possibile passare il ``verbose_name`` come un argomento
posizionale, con una sintassi leggermente più compatta. Questo esempio è
equivalente a quello precedente:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(**'e-mail',** blank=True)

Questo non funziona con i campi ``ManyToManyField`` o ``ForeignKey``, in quanto
richiedono che il primo argomento sia una classe modello. In questi casi,
specificare ``verbose_name`` esplicitamente è la strada da percorrere.

Classi ModelAdmin Personalizzate
================================

I cambiamenti che abbiamo fatto finora -- ``blank=True``, ``null=True`` e
``verbose_name`` -- sono in effetti cambiamenti fatti a livello di modello, non
al livello admin. Cioè, questi cambiamenti sono fondamentalmente parte del
modello e quindi vengono usate dall'admin, ma su di loro non c'è nulla di
admin-specifico.

Al di là di questi, l'admin offre una grande ricchezza di opzioni che ti
consentono di configurare l'admin per lavorare con un modello particolare. Tali
opzioni stanno in *classi ModelAdmin*, ovvero classi che contengono la
configurazione di uno specifico modello in una specifica istanza dell'interfaccia
di admin.

Personalizzare le liste dei cambiamenti
---------------------------------------

Tuffiamoci nella personalizzazione dell'admin specificando i campi visualizzati
nella pagina di elenco per il nostro modello di ``Author``. Per
impostazione predefinita, l'elenco di modifica visualizza per ogni oggetto il
risultato di ``__unicode__()`` . Nel capitolo 5, abbiamo definito il metodo
``__unicode__()`` negli oggetti ``Author`` per visualizzare il nome ed il
cognome insieme:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(blank=True, verbose_name='e-mail')

        **def __unicode__(self):**
            **return u'%s %s' % (self.first_name, self.last_name)**

Come risultato, la pagina di elenco per gli oggetti ``Author`` visualizza
ogni nome e cognome insieme, come si può vedere nella Figura 6-7.

.. figure:: graphics/chapter06/author_changelist1.png
   :alt: Screenshot della pagina di elenco per l'Autore.

   Figure 6-7. La pagina di elenco per l'Autore.

Possiamo migliorare questo comportamento predefinito aggiungendo un paio d'altri
campi per la cambiare l'elenco. Sarebbe utile, per esempio, vedere l'indirizzo
e-mail di ogni autore in questa lista, e sarebbe bello essere in grado di
ordinare tutto per nome e cognome.

Per fare ciò, si definisce una classe ``ModelAdmin`` per il modello ``Author``.
Questa classe è la chiave per la personalizzazione dell'admin, ed una delle cose
più elementari che permette di fare è specificare l'elenco dei campi da
mostrare sulla pagina di elenco. Modifica ``admin.py`` per apportare queste
modifiche:

.. parsed-literal::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    **class AuthorAdmin(admin.ModelAdmin):**
        **list_display = ('first_name', 'last_name', 'email')**

    admin.site.register(Publisher)
    **admin.site.register(Author, AuthorAdmin)**
    admin.site.register(Book)

Ecco cosa abbiamo fatto:

* Abbiamo creato la classe ``AuthorAdmin``. Questa classe, ha come sottoclassi
  ``django.contrib.admin.ModelAdmin``, che include la configurazione
  personalizzata di uno specifico modello admin. Abbiamo solo specificato una
  personalizzazione -- ``list_display``, che è impostata con una tupla di nomi
  dei campi da visualizzare nella pagina di elenco. Questi nomi devono essere
  presenti nel modello, ovviamente.

* Abbiamo modificato la chiamata ``admin.site.register()`` per aggiungere ``AuthorAdmin``
  dopo ``Author``. Puoi interpretarlo com: "Registra il modello ``Author`` con
  le opzioni ``AuthorAdmin``."

  L'``admin.site.register()`` prende una sottoclasse ``ModelAdmin`` come secondo
  argomento facoltativo. Se non si specifica un secondo argomento (come nel caso
  di ``Publisher`` and ``Book``), Django usa le opzioni di default per quel
  modello.

Con questo tweak, abbiamo cambiato la pagina dell'elenco autore, ed ora la
vedremo tre colonne -- il nome, il cognome e l'indirizzo e-mail. Inoltre,
ciascuna di queste colonne può essere ordinata cliccando sull'intestazione sulla
sua intestazione (Vedi Figura 6-8).

.. figure:: graphics/chapter06/author_changelist2.png
   :alt: Screenshot dell'elenco degli autori dopo list_display.

   Figure 6-8. Elenco degli autori dopo list_display

Adesso, aggiungiamo una semplice barra di ricerca. Aggiungi ``search_fields`` ad
``AuthorAdmin``, in questo modo:

.. parsed-literal::

    class AuthorAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name', 'email')
        **search_fields = ('first_name', 'last_name')**

Ricaricando la pagina nel browser, vedrai una barra di ricerca in alto.
(Vedi Figura 6-9). Abbiamo appena detto all'admin di aggiungere alla pagina di
elenco una barra di ricerca che cerca fra i campi ``first_name`` e ``last_name``.
Come l'utente si potrebbe aspettare, questa ricerca è case-insensitive e si
ricerca in entrambi i campi, quindi se cerchi la stringa ``"bar"`` viene
restituito sia un autore con il nome 'Barney' che un autore con il cognome
'Hobarson'.

.. figure:: graphics/chapter06/author_changelist3.png
   :alt: Screenshot della pagina di elenco degli autori dopo search_fields.

   Figure 6-9. La pagina di elenco degli autori dopo search_fields

Adesso, aggiungiamo alcune date per la pagina d'elenco dei libri del nostro
modello di ``Book``:

.. parsed-literal::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    class AuthorAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name', 'email')
        search_fields = ('first_name', 'last_name')

    **class BookAdmin(admin.ModelAdmin):**
        **list_display = ('title', 'publisher', 'publication_date')**
        **list_filter = ('publication_date',)**

    admin.site.register(Publisher)
    admin.site.register(Author, AuthorAdmin)
    **admin.site.register(Book, BookAdmin)**

Qui, poiché abbiamo a che fare con un diverso insieme di opzioni, abbiamo creato
una classe ``ModelAdmin`` separata -- ``BookAdmin``. In primo luogo, abbiamo
definito un ``list_display`` solo per far sembrare più bella la pagina d'elenco.
Poi, abbiamo utilizzato ``list_filter`` impostandola con una tupla di campi da
utilizzare per creare filtri nel lato destro della pagina. Per i campi data,
Django integra delle scorciatoie per filtrare l'elenco con le opzioni "Oggi",
"Ultimi 7 giorni", "Questo mese" e "Quest'anno" - scorciatoie che gli sviluppatori
di Django hanno pensato siano utili nei casi più comuni per filtrare per data.
La Figura 6-10 mostra a cosa assomiglia tutto questo.

.. figure:: graphics/chapter06/book_changelist1.png
   :alt: Screenshot della pagina d'elenco dei libri dopo list_filter.

   Figure 6-10. La pagina d'elenco dei libri dopo list_filter

``list_filter`` funziona anche su altri tipi di campi, non solo ``DateField``. (Prova con i campi
``ForeignKey`` e ``BooleanField``, per esempio). I filtri appaiono finché ci
sono almeno 2 valori tra cui scegliere.

Un altro modo per aggiungere altri filtri relativi alla data è quella di
utilizzare l'opzione ``date_hierarchy``, come questo:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        **date_hierarchy = 'publication_date'**

In questo modo, la pagina dell'elenco aggiunge una barra di navigazione in cima
alla lista, come mostrato in Figura 6-11. Essa contiene un elenco degli anni
disponibili, avendo la possibilità di scegliere anche fra mesi e giorni singoli.

.. figure:: graphics/chapter06/book_changelist2.png
   :alt: Screenshot della pagina d'elenco dei libri dopo date_hierarchy.

   Figure 6-11. La pagina d'elenco dei libri dopo date_hierarchy

Nota che date_hierarchy prende una *stringa*, non una tupla, poiché solo il
campo data può essere usato per creare una gerarchia.

Infine, cambiamo l'ordinamento predefinito in modo che i libri nella pagina
dell'elenco siano sempre ordinati in maniera discendente dalla loro data di
pubblicazione. Per impostazione predefinita, l'ordinamento scelto di base viene
preso da ``ordering`` all'interno della ``class Meta`` (che abbiamo trattato nel
capitolo 5) -- ma, quando non specificato, allora l'ordinamento rimane non
definito.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        **ordering = ('-publication_date',)**

Questa opzione ``ordering`` funziona esattamente come l'``ordering`` della
``class Meta`` dei modelli, ma utilizza solo il primo nome di campo nella lista.
Basta passare una lista o una tupla dei nomi dei campi, e aggiungere un segno
meno a un campo per imporre un ordinamento discendente.

Ricarica la pagina d'elenco libro/book per vederlo in azione. Nota che adesso
l'intestazione "Data di pubblicazione" include una piccola freccia che indica in
che modo i record vengono ordinati. (Vedi Figura 6-12)

.. DWP Different screenshot needed.

.. figure:: graphics/chapter06/book_changelist3.png
   :alt: Screenshot della pagina d'elenco dei libri dopo l'ordinamento.

   Figure 6-12. La pagina d'elenco dei libri dopo l'ordinamento

Abbiamo coperto le principali opzioni dell'elenco cambiamento. Utilizzando queste
opzioni, è possibile creare una robusta interfaccia di amministrazione, in grado
di elaborare dati in maniera efficiente con poche righe di codice.

Customizing edit forms
----------------------

Just as the change list can be customized, edit forms can be customized in many
ways.

First, let's customize the way fields are ordered. By default, the order of
fields in an edit form corresponds to the order they're defined in the model.
We can change that using the ``fields`` option in our ``ModelAdmin`` subclass:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **fields = ('title', 'authors', 'publisher', 'publication_date')**

.. SL Tested ok

After this change, the edit form for books will use the given ordering for
fields. It's slightly more natural to have the authors after the book title.
Of course, the field order should depend on your data-entry workflow. Every
form is different.

Another useful thing the ``fields`` option lets you do is to *exclude* certain
fields from being edited entirely. Just leave out the field(s) you want to
exclude. You might use this if your admin users are only trusted to edit a
certain segment of your data, or if part of your fields are changed by some
outside, automated process. For example, in our book database, we could
hide the ``publication_date`` field from being editable:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **fields = ('title', 'authors', 'publisher')**

.. SL Tested ok

As a result, the edit form for books doesn't offer a way to specify the
publication date. This could be useful, say, if you're an editor who prefers
that his authors not push back publication dates. (This is purely a
hypothetical example, of course.)

When a user uses this incomplete form to add a new book, Django will simply set
the ``publication_date`` to ``None`` -- so make sure that field has
``null=True``.

Another commonly used edit-form customization has to do with many-to-many
fields. As we've seen on the edit form for books, the admin site represents each
``ManyToManyField`` as a multiple-select boxes, which is the most logical
HTML input widget to use -- but multiple-select boxes can be difficult to use.
If you want to select multiple items, you have to hold down the control key,
or command on a Mac, to do so. The admin site helpfully inserts a bit of text
that explains this, but, still, it gets unwieldy when your field contains
hundreds of options.

The admin site's solution is ``filter_horizontal``. Let's add that to
``BookAdmin`` and see what it does.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **filter_horizontal = ('authors',)**

.. SL Tested ok

(If you're following along, note that we've also removed the ``fields`` option
to restore all the fields in the edit form.)

Reload the edit form for books, and you'll see that the "Authors" section now
uses a fancy JavaScript filter interface that lets you search through the
options dynamically and move specific authors from "Available authors" to
the "Chosen authors" box, and vice versa.

.. DWP screenshot!

.. figure:: graphics/chapter06/book_editform1.png
   :alt: Screenshot of the book edit form after adding filter_horizontal.

   Figure 6-13. The book edit form after adding filter_horizontal

We'd highly recommend using ``filter_horizontal`` for any ``ManyToManyField``
that has more than 10 items. It's far easier to use than a simple
multiple-select widget. Also, note you can use ``filter_horizontal``
for multiple fields -- just specify each name in the tuple.

``ModelAdmin`` classes also support a ``filter_vertical`` option. This works
exactly as ``filter_horizontal``, but the resulting JavaScript interface stacks
the two boxes vertically instead of horizontally. It's a matter of personal
taste.

``filter_horizontal`` and ``filter_vertical`` only work on ``ManyToManyField``
fields, not ``ForeignKey`` fields. By default, the admin site uses simple
``<select>`` boxes for ``ForeignKey`` fields, but, as for ``ManyToManyField``,
sometimes you don't want to incur the overhead of having to select all the
related objects to display in the drop-down. For example, if our book database
grows to include thousands of publishers, the "Add book" form could take a
while to load, because it would have to load every publisher for display in the
``<select>`` box.

The way to fix this is to use an option called ``raw_id_fields``. Set this to
a tuple of ``ForeignKey`` field names, and those fields will be displayed in
the admin with a simple text input box (``<input type="text">``) instead of a
``<select>``. See Figure 6-14.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        filter_horizontal = ('authors',)
        **raw_id_fields = ('publisher',)**

.. SL Tested ok

.. DWP Screenshot!

.. figure:: graphics/chapter06/book_editform2.png
   :alt: Screenshot of edit form after raw_id_fields.

   Figure 6-14. The book edit form after adding raw_id_fields

What do you enter in this input box? The database ID of the publisher. Given
that humans don't normally memorize database IDs, there's also a
magnifying-glass icon that you can click to pull up a pop-up window, from which
you can select the publisher to add.

Users, Groups, and Permissions
==============================

Because you're logged in as a superuser, you have access to create, edit, and
delete any object. Naturally, different environments require different
permission systems -- not everybody can or should be a superuser. Django's
admin site uses a permissions system that you can use to give specific users
access only to the portions of the interface that they need.

These user accounts are meant to be generic enough to be used outside of the
admin interface, but we'll just treat them as admin user accounts for now. In
Chapter 14, we'll cover how to integrate user accounts with the rest of your
site (i.e., not just the admin site).

You can edit users and permissions through the admin interface just like any
other object. We saw this earlier in this chapter, when we played around with
the User and Group sections of the admin. User objects have the standard
username, password, e-mail and real name fields you might expect, along with a
set of fields that define what the user is allowed to do in the admin
interface. First, there's a set of three boolean flags:

* The "active" flag controls whether the user is active at all.
  If this flag is off and the user tries to log in, he won't be allowed in,
  even with a valid password.

* The "staff" flag controls whether the user is allowed to log in to the
  admin interface (i.e., whether that user is considered a "staff member" in
  your organization). Since this same user system can be used to control
  access to public (i.e., non-admin) sites (see Chapter 14), this flag
  differentiates between public users and administrators.

* The "superuser" flag gives the user full access to add, create and
  delete any item in the admin interface. If a user has this flag set, then
  all regular permissions (or lack thereof) are ignored for that user.

"Normal" admin users -- that is, active, non-superuser staff members -- are
granted admin access through assigned permissions. Each object editable through
the admin interface (e.g., books, authors, publishers) has three permissions: a
*create* permission, an *edit* permission and a *delete* permission. Assigning
permissions to a user grants the user access to do what is described by those
permissions.

When you create a user, that user has no permissions, and it's up to you to
give the user specific permissions. For example, you can give a user permission
to add and change publishers, but not permission to delete them. Note that
these permissions are defined per-model, not per-object -- so they let you say
"John can make changes to any book," but they don't let you say "John can make
changes to any book published by Apress." The latter functionality, per-object
permissions, is a bit more complicated and is outside the scope of this book
but is covered in the Django documentation.

.. note::

    Access to edit users and permissions is also controlled by this permission
    system. If you give someone permission to edit users, she will be able to
    edit her own permissions, which might not be what you want! Giving a user
    permission to edit users is essentially turning a user into a superuser.

You can also assign users to groups. A *group* is simply a set of permissions to
apply to all members of that group. Groups are useful for granting identical
permissions to a subset of users.

When and Why to Use the Admin Interface -- And When Not to
==========================================================

After having worked through this chapter, you should have a good idea of how to
use Django's admin site. But we want to make a point of covering *when* and
*why* you might want to use it -- and when *not* to use it.

Django's admin site especially shines when nontechnical users need to be able
to enter data; that's the purpose behind the feature, after all. At the
newspaper where Django was first developed, development of a typical online
feature -- say, a special report on water quality in the municipal supply --
would go something like this:

* The reporter responsible for the project meets with one of the developers
  and describes the available data.

* The developer designs Django models to fit this data and then opens up
  the admin site to the reporter.

* The reporter inspects the admin site to point out any missing or
  extraneous fields -- better now than later. The developer changes the
  models iteratively.

* When the models are agreed upon, the reporter begins entering data using
  the admin site. At the same time, the programmer can focus on developing
  the publicly accessible views/templates (the fun part!).

In other words, the raison d'être of Django's admin interface is facilitating
the simultaneous work of content producers and programmers.

However, beyond these obvious data entry tasks, the admin site is useful in a
few other cases:

* *Inspecting data models*: Once you've defined a few models, it can be
  quite useful to call them up in the admin interface and enter some dummy
  data. In some cases, this might reveal data-modeling mistakes or other
  problems with your models.

* *Managing acquired data*: For applications that rely on data coming from
  external sources (e.g., users or Web crawlers), the admin site gives you
  an easy way to inspect or edit this data. You can think of it as a less
  powerful, but more convenient, version of your database's command-line
  utility.

* *Quick and dirty data-management apps*: You can use the admin site to
  build yourself a very lightweight data management app -- say, to keep
  track of expenses. If you're just building something for your own needs,
  not for public consumption, the admin site can take you a long way. In
  this sense, you can think of it as a beefed up, relational version of a
  spreadsheet.

One final point we want to make clear is: the admin site is not an
end-all-be-all. Over the years, we've seen it hacked and chopped up to serve a
variety of functions it wasn't intended to serve. It's not intended to be a
*public* interface to data, nor is it intended to allow for sophisticated
sorting and searching of your data. As we said early in this chapter, it's for
trusted site administrators. Keeping this sweet spot in mind is the key to
effective admin-site usage.

What's Next?
============

So far we've created a few models and configured a top-notch interface for
editing data. In the next chapter `Chapter 7`_, we'll move on to the real "meat and potatoes"
of Web development: form creation and processing.

.. _Chapter 7: chapter07.html
