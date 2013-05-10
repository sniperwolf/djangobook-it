===========================
Chapter 3: Views e URLconfs
===========================

Nel capitolo precedente, abbiamo spiegato come creare un progetto Django e come
eseguire il server di sviluppo integrato. In questo capitolo, imparerai le basi
della creazione di pagine web dinamiche con Django.

La tua prima pagina Django-Powered: Hello world
===============================================

Come nostro primo obiettivo, cerchiamo di creare una pagina web che mostra il
più famoso messaggio di esempio: "Hello world".

Per pubblicare  un semplice "Hello world" in una pagina web senza un framework,
basta semplicemente digitare "Hello world" in un file di testo, chiamarlo
``hello.html``, e caricarlo in una directory su un server Web da qualche parte.
Da notare che in questo processo, sono stati specificati due informazioni chiave
su quella pagina web: il suo contenuto (la stringa `"Hello world"``) e il relativo
URL (``http://www.example.com/hello.html``, o forse ``http://www.example.com/files/hello.html``
 se lo hai inserito in una sottodirectory).

Con Django, si specificano le stesse due cose, ma in un modo diverso. I
contenuti della pagina sono prodotti da una *funzione di visualizzazione* e l'URL
è specificato in un *URLconf*. In primo luogo, scriviamo la funzione di
visualizzazione "Hello world".

La tua prima View/Vista
-----------------------

All'interno della directory miosito dove abbiamo fatto ``django-admin.py startproject``
nel capitolo precedente, creare un file vuoto chiamato ``views.py``. Questo
modulo Python conterrà le nostre elucubrazioni per questo capitolo. Da notare
che non c'è niente di speciale nel nome ``views.py`` -- Django non si cura del
nome del file, come vedremo tra un po' -- ma è una buona idea lasciarlo ``views.py``
come convenzione, per il bene di altri sviluppatori che leggono il tuo codice.

La nostra view/vista "Hello world" è semplice. Ecco l'intera funzione, oltre
alle istruzioni di importazione, da digitare nel file ``views.py``::

    from django.http import HttpResponse

    def hello(request):
        return HttpResponse("Hello world")

Chiariamo ciò che abbiamo scritto per ogni riga di codice:

* In primo luogo, importiamo la classe ``HttpResponse``, che sta nel modulo
  ``django.http``. Abbiamo bisogno di importare questa classe perché è
  utilizzato in seguito nel nostro codice.


* Più avanti, definiamo una funzione chiamata `hello`` -- la funzione di
  visualizzazione.

  Ogni funzione di visualizzazione richiede almeno un parametro, chiamato
  ``request`` ("richiesta", in inglese) per convenzione. Questo è un oggetto che
  contiene informazioni relative alla richiesta web corrente che ha innescato
  questa vista/view, ed è un'istanza della classe ``django.http.HttpRequest``.
  In questo esempio, non facciamo nulla con ``request``, ma deve essere comunque
  il primo parametro della view/vista.

  Da notare che il nome della funzione di visualizzazione non è importante, non
  deve essere nominata in un certo modo affinché Django lo riconosca.
  L'abbiamo chiamata ``hello`` qui perché questo nome indica chiaramente il
  senso della view/vista, ma potrebbe benissimo essere nominato ``hello_wonderful_beautiful_world``,
  o qualcosa di altrettanto rivoltante. La sezione successiva,
  "Il tuo primo URLconf", farà luce su come Django trova questa funzione.

* La funzione è un semplice: si limita a restituire un oggetto ``HttpResponse``
  che è stato istanziato con il testo ``"Hello world"``.

La lezione principale è questa: una vista è semplicemente una funzione Python
che prende un ``HttpRequest`` come primo parametro e restituisce un'istanza di
``HttpRequest``. Affinché una funzione Python sia una view/vista per Django,
deve fare queste due cose. (Ci sono delle eccezioni, ma arriveremo a quelli
più tardi)

Il tuo primo URLconf
------------------

Se, a questo punto, è stato eseguito di nuovo ``python manage.py runserver``,
dovresti ancora vedere il messaggio "Welcome to Django", senza alcuna traccia
della nostra view/vista "Hello world" da nessuna parte. Questo perché il nostro
progetto ``mysite`` non sa ancora della view `hello``, abbiamo bisogno di dire
esplicitamente a Django che stiamo attivando questa view in un determinato URL.
(Continuando la nostra precedente analogia con la pubblicazione di file HTML
statici, a questo punto abbiamo creato il file HTML, ma non l'abbiamo ancora
caricato in una directory sul server). Per aggiungere una funzione di
visualizzazione ad un particolare URL in Django, bisogna utilizzare un URLconf.

Un *URLconf* è come una tabella di contenuti per il tuo sito web Django-powered.
In sostanza, si tratta di un mapping (ovvero creare delle relazioni) tra URL e
funzioni di visualizzazione che dovrebbero essere chiamate per tali URL. E' come
dire a Django, "Per questo URL, devi chiamare questo codice, e per questo URL,
devi chiamare quel codice". Per esempio, "Quando qualcuno visita l'URL ``/foo/``,
chiama la funzione vista ``foo_view()``, che sta nel modulo ``views.py``".

Quando si esegue ``django-admin.py startproject`` nel capitolo precedente, lo
script crea un URLconf per te automaticamente: il file ``urls.py``. Per
impostazione predefinita, è qualcosa di simile a questo::

    from django.conf.urls import patterns, include, url

    # Uncomment the next two lines to enable the admin:
    # from django.contrib import admin
    # admin.autodiscover()

    urlpatterns = patterns('',
        # Examples:
        # url(r'^$', 'mysite.views.home', name='home'),
        # url(r'^mysite/', include('mysite.foo.urls')),

        # Uncomment the admin/doc line below to enable admin documentation:
        # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

        # Uncomment the next line to enable the admin:
        # url(r'^admin/', include(admin.site.urls)),
    )

Questo URLconf di default include alcune caratteristiche di Django comunemente
utilizzate commentate, in modo che l'attivazione di queste caratteristiche sia
facile, basta togliere il commento alle righe appropriate. Se ignoriamo il
codice commentato, ecco l'essenza di un URLconf::

    from django.conf.urls.defaults import patterns, include, url

    urlpatterns = patterns('',
    )

Esaminiamo questo codice una riga alla volta:

* Nella prima linea, stiamo importando tre moduli da
  ``django.conf.urls.defaults``, che è la base per gli URLconf di Django:
  ``patterns`` ``include`` e ``urls``;

* La seconda riga chiama le funzioni ``patterns`` e salva il risultato in una
  variabile chiamata ``urlpatterns``. Ad essa viene passato un solo argomento --
  la stringa vuota. (La stringa può essere utilizzata per fornire un prefisso
  comune a tutte le funzioni, come vedremo nel :doc:`chapter08`).

La cosa più importante da notare qui è vedere come variabile ``urlpatterns``,
che Django si aspetta di trovare nel tuo modulo URLconf. Questa variabile
definisce il mapping tra URL e codici che gestiscono tali URL. Per impostazione
predefinita, come possiamo vedere, l'URLconf è vuoto -- l'applicazione Django è
un tabula rasa. (Come nota a margine, è il modo che Django ha per mostrare la
pagina "Welcome to Django" nel capitolo precedente. Se il tuo URLconf è vuoto,
Django assume che hai appena iniziato un nuovo progetto e, quindi, mostra quel
messaggio).

Per aggiungere un URL all'URLconf, basta aggiungere una relazione tra un pattern
relativo all'URL e la funzione di visualizzazione. Ecco come collegare la nostra
view ``hello``::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
    )


(Si noti che abbiamo rimosso il codice commentato per brevità. È possibile
lasciare quelle linee senza problemi).

Abbiamo fatto due modifiche qui:

* In primo luogo, abbiamo importato la view ``hello`` dal suo modulo --
  ``mysite/views.py``, che si traduce con ``mysite.views`` nella sintassi di
  import. (Premesso ``mysite/views.py`` è sul tuo percorso Python, vedi la barra
  laterale per i dettagli);

* Successivamente, abbiamo aggiunto la linea ``url(r'^hello/$', hello),`` a
  ``urlpatterns``. Questa linea è detta come un *URLpattern*. La funzione ``url()``
  dice a Django come gestire l'url che si sta configurando. Il primo argomento è
  una stringa pattern-matching (una espressione regolare; ne discuteremo meglio
  fra un po') e il secondo argomento è la funzione di visualizzazione da
  utilizzare per quel pattern. ``url()`` può assumere altri argomenti opzionali, che
  vedremo più in dettaglio in :doc:`chapter08`.

.. note::

  Il dettaglio più importante che abbiamo introdotto qui è il carattere ``r``
  che sta davanti alla stringa di espressione regolare. Questo dice a Python che
  la stringa è una "stringa raw" -- il suo contenuto non deve essere
  interpretato. Nelle normali stringhe Python, i backslash sono usati per
  scrivere caratteri speciali - come la stringa ``'\n'`` per andare a capo.
  Quando si aggiunge la ``r`` per renderlo una stringa raw, Python non lo
  considera un carattere di escape -- così, ``r'\n'`` è una stringa di due
  caratteri che contiene una barra rovesciata e un carattere minuscolo "n".
  C'è una problematica naturale fra l'utilizzo di backslash in Python ed i
  backslash che si trovano nelle espressioni regolari, quindi è fortemente
  consigliato l'uso di stringhe raw in tutti i casi in cui si sta definendo
  un'espressione regolare in Python. Tutte gli URLpattern in questo libro
  sono stringhe raw.

In poche parole, abbiamo appena detto a Django che qualsiasi richiesta all'URL
``/hello/`` dovrebbe essere gestita dalla funzione di visualizzazione ``hello``.

.. admonition:: Il tuo percorso di Python

    Il *tuo percorso di Python* è l'elenco delle directory del sistema in cui
    Python appare quando si utilizza l'istruzione ``import`` Python.

    Ad esempio, supponiamo che il vostro percorso di Python è impostato su ``['',
    '/usr/lib/python2.7/site-packages', '/home/username/djcode']``. Se si esegue
    l'istruzione Python ``from foo import bar``, Python cercherà un modulo
    chiamato ``foo.py`` nella directory corrente. (La prima voce nel percorso di
    Python, una stringa vuota, significa sostanzialmente "la directory corrente").
    Se questo file non esiste, Python cercherà il file ``/usr/lib/python2.7/site-packages/foo.py``.
    Se questo file non esiste, proverà ``/home/username/djcode/foo.py``.
    Infine, se *neanche questo* file non esiste, solleva l'eccezione ``ImportError``.

    Se sei interessato a vedere il suddetto percorso, avviare l'interprete
    interattivo Python e digitare::

        >>> import sys
        >>> print sys.path

    Generalmente non devi preoccuparti di impostare il percorso di Python --
    Python e Django si prendono automaticamente cura di queste cose per noi
    dietro le quinte. (l'impostazione del percorso di Python è una delle cose
    che fa lo script ``manage.py``).


Vale la pena discutere la sintassi di questo urlPattern, in quanto potrebbe
non essere immediatamente evidente. Anche se vogliamo lavorare con l'URL
``/hello/``, il pattern è un po' diverso da questo. Ecco perché:

* Django rimuove la barra dalla parte anteriore di ogni URL in ingresso prima di
  controllare gli URLpatterns. Questo significa che il nostro urlPattern non
  include la barra iniziale in ``/hello/``. (In un primo momento, può sembrare
  poco intuitivo, ma questo requisito semplifica le cose -- come l'inserimento
  di alcuni URLconf all'interno di altri URLconf, come vedremo nel capitolo 8);

* Il pattern include un accento circonflesso (``^``) e il simbolo del dollaro
  (``$``). Questi sono i tipici caratteri delle espressioni regolari e che hanno
  un significato particolare: l'accento circonflesso significa "richiedo che il
  pattern corrisponda all'inizio della stringa," e il simbolo del dollaro
  significa "richiedo che il pattern corrisponda alla fine della stringa";

  Questo concetto si spiega meglio con un esempio. Se avessimo usato invece il
  pattern ``'^hello/'`` (senza il simbolo del dollaro alla fine), quindi
  qualsiasi URL che inizia con ``/hello/`` potrebbe corrispondere, ad esempio
  ``/hello/foo`` e ``/hello/bar``, e non solo ``/hello/``. Allo stesso modo, se
  avessimo lasciato fuori l'accento circonflesso iniziale (vale a dire, ``'hello/$'``),
  Django avrebbe scelto qualsiasi URL che termina con ``hello/`` come ``/foo/bar/hello/``.
  Se avessimo usato semplicemente ``hello/``, senza un segno di accento
  circonflesso o dollaro, qualsiasi URL contenente ``hello/`` sarebbe stato
  valido, come ``/foo/hello/bar``. Quindi, usiamo sia il segno di accento
  circonflesso che il dollaro per garantire che solo gli URL ``/hello/`` siano
  validi -- niente di più, niente di meno.

  La maggior parte dei tuoi URLpattern iniziano con accenti e terminano con
  segni di dollaro, ma è bello avere comunque la flessibilità necessaria per
  svolgere compiti più sofisticati.

  Ci si potrebbe chiedere cosa succederebbe se qualcuno richiedess l'URL ``/hello``
  (cioè *senza* lo slash finale). Poiché il nostro urlPattern richiede una slash
  finale, tale URL *non* dovrebbe funzionare. Tuttavia, per impostazione
  predefinita, qualsiasi richiesta per un URL che *non corrisponde* a un
  urlPattern e *non* termina con uno slash verrà comunque reindirizzato allo
  stesso URL con una barra finale. (questa regola è data dall'impostazione
  ``APPEND_SLASH`` di Django, trattata nell'Appendice D).

  Se sei il tipo di persona che ama tutti gli URL che terminano con gli slash
  (che è la preferenza degli sviluppatori di Django), tutto quello che bisogna
  fare è aggiungere uno slash per ciascun urlPattern e lasciare ``APPEND_SLASH``
  impostata su ``True``. Se si preferisce invece che gli URL non debbano avere
  lo slash, o se si vuole decidere in maniera mirata come comportarsi, impostare
  ``APPEND_SLASH`` su ``False`` ed inserire gli slash finali negli URLpattern
  come meglio si crede.

L'altra cosa da notare in questo URLconf è che abbiamo passato la view ``hello``
come un oggetto, senza chiamare la funzione. Questa è una caratteristica
fondamentale di Python (e altri linguaggi dinamici): le funzioni sono oggetti di
prima classe, il che significa che è possibile passarli in giro, proprio come
tutte le altre variabili. Belle cose eh?

Per testare le nostre modifiche a URLconf, avviare il server di sviluppo, come
fatto nel capitolo 2, eseguendo il comando ``python manage.py runserver``. (se
lo si aveva lasciato in esecuzione, va bene, poiché il server di sviluppo rileva
automaticamente le modifiche al codice Python e si ricarica, se necessario, in
modo che non sia necessario riavviare il server per vedere i cambiamenti). Il
server è in esecuzione all'indirizzo ``http://127.0.0.1:8000/``, quindi aprire
un browser ed aprire l'indirizzo ``http://127.0.0.1:8000/hello/``. Si dovrebbe
vedere il testo "Hello world" -- La tua view in Django.

Evviva! Hai creato la tua prima pagina web Django-powered.

.. admonition:: Espressioni regolari

    Le *espressioni regolari* (o regex) sono un modo compatto di specificare dei
    pattern nel testo. Mentre gli URLconf consentono di scrivere regex
    arbitrarie per fare qualunque magheggio con gli URL, probabilmente userai
    solo pochi simboli regex nella pratica. Ecco una selezione di simboli comuni:

    ============  ==========================================================
    Simboli        Corrispondenza
    ============  ==========================================================
    ``.`` (punto) Qualsiasi carattere singolo

    ``\d``        Qualsiasi cifra singola

    ``[A-Z]``     Qualsiasi carattere tra ``A`` e ``Z`` (maiuscolo)

    ``[a-z]``     Qualsiasi carattere tra ``a`` e ``z`` (minuscolo)

    ``[A-Za-z]``  Qualsiasi carattere tra ``a`` e ``z`` (maiuscole e minuscole)

    ``+``         Uno o più elementi dell'espressione anteposta ad esso
                  (ad esempio, ``\d+`` corrisponde ad una o più cifre)

    ``[^/]+``     Uno o più caratteri fino (e non incluso) allo slash

    ``?``         Zero o una delle espressioni anteposta (ad esempio, ``\d?``
                  corrisponde a zero o una cifra)

    ``*``         Zero o più dell'espressione anteposta (ad esempio, ``\d*``
                  corrisponde a zero o una o più d'una cifra)

    ``{1,3}``     Tra uno e tre (compreso) dell'espressione anteposta (ad
                  esempio, ``\d{1,3}`` corrisponde a uno, due o tre cifre)
    ============  ==========================================================

    Per ulteriori informazioni sulle espressioni regolari, leggere la pagina
    all'indirizzo http://www.djangoproject.com/r/python/re-module/.

Breve nota sui errori 404
-------------------------

A questo punto, il nostro URLconf definisce un solo urlPattern: quello che
gestisce le richieste per l'URL ``/hello/``. Cosa succede quando si richiede un
URL diverso?

Per scoprirlo, prova ad eseguire il server di sviluppo e visitare una pagina
come ``http://127.0.0.1:8000/goodbye/`` o ``http://127.0.0.1:8000/hello/subdirectory/``,
o anche ``http://127.0.0.1:8000/`` (la "root" del sito web). Si dovrebbe ricevere
un errore con il messaggio "Page not found" (vedi Figura 3-1). Django mostra
questo messaggio perché hai richiesto un URL che non è definito in URLconf.

.. figure:: graphics/chapter03/404.png
   :alt: Screenshot della pagina 404 di Django.

   Figura 3-1. Pagina 404 di Django

L'utilità di questa pagina va oltre il messaggio di errore 404 di base. Ci dice
anche con precisione quali URLconf ha usato Django e ogni modello in
quell'URLconf. Da queste informazioni, si dovrebbe essere in grado di dire
perché l'URL richiesto ha provocato un errore 404.

Naturalmente, si tratta di informazioni sensibili destinato solo per te,
sviluppatore web. Se fossimo su un sito di produzione distribuito su Internet,
non vorremmo esporre tali informazioni al pubblico. Per questo motivo, questa
pagina "Pagina non trovata" viene mostrata solo se il progetto Django è in
*modalità di debug*. Spiegheremo come disattivare la modalità di debug in seguito.
Per ora, è sufficiente sapere che ogni progetto Django è in modalità di debug
quando viene creato per la prima volta, e se il progetto non è in modalità di
debug, Django genera delle pagine di risposta di 404 differenti.

Breve nota sulla root del sito
------------------------------

Come spiegato nel paragrafo precedente, viene mostrato un messaggio di errore
404 se si naviga verso la root del sito -- ``http://127.0.0.1:8000/``. Django non
aggiunge magicamente nulla alla root del sito, l'URL non è speciale o diverso in
alcun modo da qualunque altro. Sta a noi assegnarlo ad un urlPattern, proprio
come ogni altra voce della URLconf.

Però, l'urlPattern da abbinare alla radice del sito è poco intuitivo, vale
quindi la pena specificare meglio. Quando si è pronti ad implementare una
view/vista per la root principale del sito, utilizzare l'urlPattern ``'^$'``,
che corrisponde a una stringa vuota. Per esempio::

    from mysite.views import hello, my_homepage_view

    urlpatterns = patterns('',
        url(r'^$', my_homepage_view),
        # ...
    )


Come Django Elabora una richiesta
=================================

Prima di continuare con le view, facciamo una pausa per imparare un po' di più
come funziona Django. In particolare, quando viene mostrato il messaggio
"Hello world" visitando ``http://127.0.0.1:8000/hello/`` nel browser, cosa fa
Django dietro le quinte?

Tutto inizia con il *file delle impostazioni*. Quando si esegue ``python
manage.py runserver``, lo script cerca un file chiamato settings.py nella
directory miosito interna. Questo file contiene tutti i tipi di configurazione
per questo particolare progetto Django, tutto in maiuscolo, ``TEMPLATE_DIRS``,
``DATABASES`` ecc. L'impostazione più importante è chiamata ``ROOT_URLCONF``.
``ROOT_URLCONF`` dice a Django quale modulo Python dovrebbe essere usato come
URLconf per questo sito web.

Ricordate quando ``django-admin.py startproject`` ha creato il file ``settings.py``
e ``urls.py``? Il ``settings.py`` autogenerato contiene un'impostazione ``ROOT_URLCONF``
che punta al file ``urls.py`` generato automaticamente. Apri il file ``settings.py``
e guarda di persona, dovrebbe essere qualcosa di simile a questo::

    ROOT_URLCONF = 'mysite.urls'

Questo corrisponde al file ``mysite/urls.py``.

Quando arriva una richiesta per un URL specifico - per esempio, una richiesta
per ``/hello/`` --  Django carica la URLconf dall'impostazione ``ROOT_URLCONF``.
Infine controlla ciascuna delle URLpatterns in tale URLconf, in ordine dall'alto
verso il basso, confrontando l'URL richiesto con i modelli di uno alla volta,
fino a che non ne trova uno che corrisponde. Quando ne trova uno che corrisponde,
chiama la funzione di visualizzazione associata a tale modello, passandogli un
oggetto ``HttpRequest`` come primo parametro. (parleremo delle specifiche ``HttpRequest``
più tardi).

Come abbiamo visto nella nostra prima view, ad esempio, le funzioni per la
visualizzazione devono restituire un ``HttpResponse``. Una volta che lo fa,
Django fa il resto, convertendo l'oggetto Python in una giusta risposta web, con
intestazioni HTTP appropriate e corpo (ad esempio, il contenuto della pagina
web).

In sintesi:

1. Arriva una richiesta per ``/hello/``.
2. Django determina la root degli URLconf, cercando nell'impostazione ``ROOT_URLCONF``.
3. Django legge tutti gli URLpatterns nell'URLconf e sceglie il primo che corrisponde a / ciao /.
4. Se trova una corrispondenza, chiama la funzione di visualizzazione associata.
5. La funzione di visualizzazione restituisce un ``HttpResponse``.
6. Django converte l'``HttpResponse`` nella risposta HTTP corretta, che risulta
in una pagina web.

Ora conosci le basi per creare pagine Django-powered. E' abbastanza semplice,
in realtà -- basta solo scrivere le funzioni di visualizzazione ed il file
URLconf con gli url.

La tua seconda View: Contenuto Dinamico
=======================================

La nostra view  "Hello world" è stata istruttiva nel mostrare le basi di come
funziona Django, ma non è stato un esempio di una pagina Web dinamica, perché il
contenuto della pagina è sempre lo stesso. Ogni volta che si richiede ``/hello/``,
vedremo la stessa cosa, come se fosse un normale file HTML statico.

Per la nostra seconda view, creiamo qualcosa di più dinamico -- una pagina Web
che visualizza la data e l'ora attuali. Questo è passo semplice perché non
comporta l'uso di un database o di qualsiasi input dell'utente -- mostriamo solo
dati già in nostro possesso. E' poco più emozionante di "Hello world," ma
dimostrerà alcuni nuovi concetti.

Questa view ha bisogno di fare due cose: calcolare la data e l'ora corrente e
restituire un ``HttpResponse`` contenente tale valore. Se avete esperienza con
Python, si sa che include un modulo ``datetime`` per calcolare le date. Ecco come
usarlo::

    >>> import datetime
    >>> now = datetime.datetime.now()
    >>> now
    datetime.datetime(2008, 12, 13, 14, 9, 39, 2731)
    >>> print now
    2008-12-13 14:09:39.002731

Questo è abbastanza semplice e non ha nulla a che fare con Django. E' solo
codice Python. (Vogliamo sottolineare che si dovrebbe essere consapevoli di ciò
che è codice "solo Python" rispetto a ciò che è specifico di Django. Mentre
impari ad utilizzare Django, vorremmo che tu sia in grado di applicare le tue
conoscenze ad altri progetti Python che non usano necessariamente Django).

Per fare una view che mostra la data e l'ora correnti, quindi, abbiamo solo
bisogno di collegare l'istruzione ``datetime.datetime.now()`` in una view e
restituire il suo risultato in un ``HttpResponse``. Ecco come dovrebbe sembrare::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Come con la nostra funzione di visualizzazione ``hello``, questo dovrebbe stare
all'interno di ``views.py``. Si noti che abbiamo nascosto la funzione ``hello``
da questo esempio per brevità, ma per amor di completezza, ecco come somiglia
l'intera ``views.py``::

    from django.http import HttpResponse
    import datetime

    def hello(request):
        return HttpResponse("Hello world")

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

(D'ora in poi, noi non mostreremo più il codice di esempi precedenti, se non
necessario. Si dovrebbe essere in grado di capire dal contesto se le parti di
un esempio sono nuove o vecchie).

Stiliamo un sommario dei cambiamenti che abbiamo apportato a ``views.py`` per
mostrare la vista/view ``current_datetime``.

* Abbiamo aggiunto un datetime import alla parte superiore del modul ``views.py``,
  in modo da poter calcolare le date.

* La nuova funzione ``current_datetime`` calcola la data e l'ora corrente, come
  un oggetto ``datetime.datetime``, e funziona come la variabile locale ``now``.

* La seconda riga di codice all'interno della vista costruisce una risposta HTML
  con la funzionalità di formattazione delle stringhe integrata in Python. Il
  ``%s`` all'interno della stringa è un "segnaposto", mentre il segno di
  percentuale dopo la stringa significa "Sostituire il ``%s`` nella stringa
  precedente con il valore della variabile". La variabile ora è tecnicamente un
  oggetto datetime.datetime, non una stringa , ma il ``%s`` converte
  implicitamente nella rappresentazione l'oggetto in stringa, che è qualcosa
  come ``"2008-12-13 14:09:39.002731"``. Questo si traduce in una stringa HTML
  come ``"<html><body>It is now 2008-12-13 14:09:39.002731.</body></html>"``.

  (Sì, il nostro codice HTML non è valido, ma stiamo cercando di mantenere
  l'esempio semplice e breve).

* Infine, la vista restituisce un oggetto ``HttpResponse`` che contiene la risposta
  generata -- proprio come abbiamo fatto in ``hello``.

Dopo aver aggiunto tutto questo a ``views.py``, ci tocca aggiungere un
urlPattern di ``urls.py`` per dire a Django quale URL dovrebbe gestire questa
view. Qualcosa di simile a ``/time/`` potrebbe avere un senso::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello, current_datetime

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
    )

Abbiamo fatto due cambiamenti qui. Prima, abbiamo importato la funzione
``current_datetime`` in alto. In secondo luogo, ancor più importante, abbiamo
aggiunto il collegamento fra urlPattern e URL ``/time/`` a quella nuova view.
Come vedere tutto questo di questo?

Con la view scritta e l'URLconf aggiornato, lanciare il comando ``runserver`` e
visitare ``http://127.0.0.1:8000/time/`` nel browser. Si dovrebbero vedere la
data e l'ora correnti.

.. admonition:: Il fuso orario di Django

    A seconda del computer, data e ora possono avere un paio d'ore di differenza.
    Questo perché Django ha come impostazioni predefinite il fuso orario
    ``America/Chicago``. (Per un qualche difetto con cui bisogna fare i conti,
    questo è il fuso orario in cui vivono gli sviluppatori originali). Se si
    vive altrove, ti consigliamo di cambiarla da ``settings.py``. Leggi la
    pagina segnalata per avere una idea di tutti i fuso orario del mondo.

URLconfs e Accoppiamento Lasco
==============================

Ora è un buon momento per evidenziare una filosofia chiave dietro gli URLconf e
dietro Django in generale: il *principio di accoppiamento lasco*. In poche parole,
l'accoppiamento lasco è un approccio allo sviluppo software che valorizza
l'importanza di costruire pezzi intercambiabili. Se due pezzi di codice non sono
accoppiati fra loro, poi le modifiche apportate a uno dei pezzi avranno poco o
nessun effetto sull'altro.

Gli URLconf di Django sono un buon esempio di questo principio nella pratica. In
una applicazione web Django, le definizioni di URL e le view che le richiamano
sono debolmente accoppiate, cioè la decisione di ciò che l'URL deve essere per
una data funzione, e l'attuazione della funzione stessa, risiede in due luoghi
separati. Ciò consente di manipolare un unico pezzo senza influenzare l'altro.

Per esempio, consideriamo la nostra view ``current_datetime``. Se volessimo
modificare l'URL per l'applicazione -- per esempio, per spostarla da ``/time/``
a ``/current-time/`` -- potremmo fare una rapida modifica al'URLconf, senza
doverci preoccupare della vista stessa. Allo stesso modo, se volessimo cambiare
la funzione di visualizzazione -- alterarne la logica in qualche modo --
potremmo farlo senza intaccare l'URL a cui è legata la funzione.

Inoltre, se avessimo voluto mostrare la funzionalità ``current_datetime`` a
*diversi* URL, potremmo semplicemente modificare l'URLconf, senza dover toccare
il codice della view. In questo esempio, il nostro ``current_datetime`` è
disponibile a due URL. E' un esempio forzato, ma questa tecnica può tornare
utile::

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^another-time-page/$', current_datetime),
    )

Le URLconf e le view sono sciolti accoppiamento in azione. Continueremo a
sottolineare degli esempi di questa importante filosofia in questo libro.

La tua terza view: URL dinamici
===============================

A nostro avviso la vista ``current_datetime``, il contenuto della pagina --
la data/ora di oggi -- erano dinamica, ma l'URL (``/time/``) era statico. Nella
maggior parte delle applicazioni web dinamiche, però, un URL contiene parametri
che influenzano l'output della pagina. Ad esempio, una libreria online potrebbe
dare ad ogni libro il proprio URL, come ``/books/243/`` e ``/books/81196/``.

Creiamo un terzo punto di vista che mostra la data e l'ora correnti compensato
da un certo numero di ore. L'obiettivo è quello di predisporre un sito web in
modo tale che la pagina ``/time/plus/1/`` mostri la data/ora di un'ora nel
futuro, la pagina ``/time/plus/2/`` mostra la data/ora di due ore nel futuro, la
pagina ``/time/plus/3/`` mostra la data/ora di tre ore nel futuro, e così
via.

Un principiante potrebbe pensare di codificare una funzione di visualizzazione
separata per ogni ora di offset, che potrebbe tradursi in una URLconf come
questo::

    urlpatterns = patterns('',
        url(r'^time/$', current_datetime),
        url(r'^time/plus/1/$', one_hour_ahead),
        url(r'^time/plus/2/$', two_hours_ahead),
        url(r'^time/plus/3/$', three_hours_ahead),
        url(r'^time/plus/4/$', four_hours_ahead),
    )

Chiaramente, questa linea di pensiero è scorretta. Non solo questo risultato è
fatta da funzioni ridondanti, ma anche l'applicazione è fondamentalmente
limitata a sostenere solo le gamme di ore predefinite - uno, due, tre o quattro
ore. Volendo creare una pagina che mostra il tempo di *cinque* ore nel futuro,
dovremmo creare una vista separata ed una nuova linea di URLconf per questo,
favorire la duplicazione. Abbiamo bisogno di fare un po' di astrazione qui.

.. admonition:: Una parola riguardo ai "Pretty URL"

    Se siete esperti in un'altra piattaforma di sviluppo web, come PHP o Java,
    si potrebbe pensare: "Ehi, usiamo un parametro di stringa di query!" --
    qualcosa come ``/time/plus?hours=3``, in cui le ``hours`` sarebbero
    designate dal parametro ore nella stringa di query dell'URL (la parte dopo
    il ``?``).

    È *possibile* farlo con Django (e ti diremo come nel Capitolo 7), ma una
    delle filosofie di base di Django è che gli URL dovrebbe essere belli.
    L'URL ``/time/plus/3/`` è molto più pulito, più semplice, più leggibile,
    più facile da dire ad alta voce e qualcuno... semplicemente più bella
    rispetto al suo omologo di una stringa di query. I pretty URL sono una
    caratteristica di un'applicazione Web di qualità.

    Il sistema URLconf di Django incoraggia la creazione di URL di questo tipo,
    rendendoli più semplici da usare rispetto agli altri che *non* lo sono.

How, then do we design our application to handle arbitrary hour offsets? The
key is to use *wildcard URLpatterns*. As we mentioned previously, a URLpattern
is a regular expression; hence, we can use the regular expression pattern
``\d+`` to match one or more digits::

    urlpatterns = patterns('',
        # ...
        url(r'^time/plus/\d+/$', hours_ahead),
        # ...
    )

(We're using the ``# ...`` to imply there might be other URLpatterns that we
trimmed from this example.)

This new URLpattern will match any URL such as ``/time/plus/2/``,
``/time/plus/25/``, or even ``/time/plus/100000000000/``. Come to think of it,
let's limit it so that the maximum allowed offset is 99 hours. That means we
want to allow either one- or two-digit numbers -- and in regular expression
syntax, that translates into ``\d{1,2}``::

    url(r'^time/plus/\d{1,2}/$', hours_ahead),

.. note::

    When building Web applications, it's always important to consider the most
    outlandish data input possible, and decide whether or not the application
    should support that input. We've curtailed the outlandishness here by
    limiting the offset to 99 hours.

Now that we've designated a wildcard for the URL, we need a way of passing that
wildcard data to the view function, so that we can use a single view function
for any arbitrary hour offset. We do this by placing parentheses around the
data in the URLpattern that we want to save. In the case of our example, we
want to save whatever number was entered in the URL, so let's put parentheses
around the ``\d{1,2}``, like this::

    url(r'^time/plus/(\d{1,2})/$', hours_ahead),

If you're familiar with regular expressions, you'll be right at home here;
we're using parentheses to *capture* data from the matched text.

The final URLconf, including our previous two views, looks like this::

    from django.conf.urls.defaults import *
    from mysite.views import hello, current_datetime, hours_ahead

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^time/plus/(\d{1,2})/$', hours_ahead),
    )

With that taken care of, let's write the ``hours_ahead`` view.

``hours_ahead`` is very similar to the ``current_datetime`` view we wrote
earlier, with a key difference: it takes an extra argument, the number of hours
of offset. Here's the view code::

    from django.http import Http404, HttpResponse
    import datetime

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Let's step through this code one line at a time:

* The view function, ``hours_ahead``, takes *two* parameters: ``request``
  and ``offset``.

  * ``request`` is an ``HttpRequest`` object, just as in ``hello`` and
    ``current_datetime``. We'll say it again: each view *always* takes an
    ``HttpRequest`` object as its first parameter.

  * ``offset`` is the string captured by the parentheses in the
    URLpattern. For example, if the requested URL were ``/time/plus/3/``,
    then ``offset`` would be the string ``'3'``. If the requested URL were
    ``/time/plus/21/``, then ``offset`` would be the string ``'21'``. Note
    that captured values will always be *strings*, not integers, even if
    the string is composed of only digits, such as ``'21'``.

    (Technically, captured values will always be *Unicode objects*, not
    plain Python bytestrings, but don't worry about this distinction at
    the moment.)

    We decided to call the variable ``offset``, but you can call it
    whatever you'd like, as long as it's a valid Python identifier. The
    variable name doesn't matter; all that matters is that it's the second
    argument to the function, after ``request``. (It's also possible to
    use keyword, rather than positional, arguments in an URLconf. We cover
    that in Chapter 8.)

* The first thing we do within the function is call ``int()`` on ``offset``.
  This converts the string value to an integer.

  Note that Python will raise a ``ValueError`` exception if you call
  ``int()`` on a value that cannot be converted to an integer, such as the
  string ``'foo'``. In this example, if we encounter the ``ValueError``, we
  raise the exception ``django.http.Http404``, which, as you can imagine,
  results in a 404 "Page not found" error.

  Astute readers will wonder: how could we ever reach the ``ValueError``
  case, anyway, given that the regular expression in our URLpattern --
  ``(\d{1,2})`` -- captures only digits, and therefore ``offset`` will only
  ever be a string composed of digits? The answer is, we won't, because
  the URLpattern provides a modest but useful level of input validation,
  *but* we still check for the ``ValueError`` in case this view function
  ever gets called in some other way. It's good practice to implement view
  functions such that they don't make any assumptions about their
  parameters. Loose coupling, remember?

* In the next line of the function, we calculate the current date/time and
  add the appropriate number of hours. We've already seen
  ``datetime.datetime.now()`` from the ``current_datetime`` view; the new
  concept here is that you can perform date/time arithmetic by creating a
  ``datetime.timedelta`` object and adding to a ``datetime.datetime``
  object. Our result is stored in the variable ``dt``.

  This line also shows why we called ``int()`` on ``offset`` -- the
  ``datetime.timedelta`` function requires the ``hours`` parameter to be an
  integer.

* Next, we construct the HTML output of this view function, just as we did
  in ``current_datetime``. A small difference in this line from the previous
  line is that it uses Python's format-string capability with *two* values,
  not just one. Hence, there are two ``%s`` symbols in the string and a
  tuple of values to insert: ``(offset, dt)``.

* Finally, we return an ``HttpResponse`` of the HTML. By now, this is old
  hat.

With that view function and URLconf written, start the Django development server
(if it's not already running), and visit ``http://127.0.0.1:8000/time/plus/3/``
to verify it works. Then try ``http://127.0.0.1:8000/time/plus/5/``. Then
``http://127.0.0.1:8000/time/plus/24/``. Finally, visit
``http://127.0.0.1:8000/time/plus/100/`` to verify that the pattern in your
URLconf only accepts one- or two-digit numbers; Django should display a "Page
not found" error in this case, just as we saw in the section "A Quick Note
About 404 Errors" earlier. The URL ``http://127.0.0.1:8000/time/plus/`` (with
*no* hour designation) should also throw a 404.

.. admonition:: Coding Order

    In this example, we wrote the URLpattern first and the view second, but in
    the previous examples, we wrote the view first, then the URLpattern. Which
    technique is better?

    Well, every developer is different.

    If you're a big-picture type of person, it may make the most sense to you
    to write all of the URLpatterns for your application at the same time, at
    the start of your project, and then code up the views. This has the
    advantage of giving you a clear to-do list, and it essentially defines the
    parameter requirements for the view functions you'll need to write.

    If you're more of a bottom-up developer, you might prefer to write the
    views first, and then anchor them to URLs afterward. That's OK, too.

    In the end, it comes down to which technique fits your brain the best. Both
    approaches are valid.

Django's Pretty Error Pages
===========================

Take a moment to admire the fine Web application we've made so far . . . now
let's break it! Let's deliberately introduce a Python error into our
``views.py`` file by commenting out the ``offset = int(offset)`` lines in the
``hours_ahead`` view::

    def hours_ahead(request, offset):
        # try:
        #     offset = int(offset)
        # except ValueError:
        #     raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Load up the development server and navigate to ``/time/plus/3/``. You'll see an
error page with a significant amount of information, including a ``TypeError``
message displayed at the very top: ``"unsupported type for timedelta hours
component: unicode"``.

What happened? Well, the ``datetime.timedelta`` function expects the ``hours``
parameter to be an integer, and we commented out the bit of code that converted
``offset`` to an integer. That caused ``datetime.timedelta`` to raise the
``TypeError``. It's the typical kind of small bug that every programmer runs
into at some point.

The point of this example was to demonstrate Django's error pages. Take some
time to explore the error page and get to know the various bits of information
it gives you.

Here are some things to notice:

* At the top of the page, you get the key information about the exception:
  the type of exception, any parameters to the exception (the ``"unsupported
  type"`` message in this case), the file in which the exception was raised,
  and the offending line number.

* Under the key exception information, the page displays the full Python
  traceback for this exception. This is similar to the standard traceback
  you get in Python's command-line interpreter, except it's more
  interactive. For each level ("frame") in the stack, Django displays the
  name of the file, the function/method name, the line number, and the
  source code of that line.

  Click the line of source code (in dark gray), and you'll see several
  lines from before and after the erroneous line, to give you context.

  Click "Local vars" under any frame in the stack to view a table of all
  local variables and their values, in that frame, at the exact point in the
  code at which the exception was raised. This debugging information can be
  a great help.

* Note the "Switch to copy-and-paste view" text under the "Traceback"
  header. Click those words, and the traceback will switch to a alternate
  version that can be easily copied and pasted. Use this when you want to
  share your exception traceback with others to get technical support --
  such as the kind folks in the Django IRC chat room or on the Django users
  mailing list.

  Underneath, the "Share this traceback on a public Web site" button will
  do this work for you in just one click. Click it to post the traceback to
  http://www.dpaste.com/, where you'll get a distinct URL that you can
  share with other people.

* Next, the "Request information" section includes a wealth of information
  about the incoming Web request that spawned the error: GET and POST
  information, cookie values, and meta information, such as CGI headers.
  Appendix G has a complete reference of all the information a request
  object contains.

  Below the "Request information" section, the "Settings" section lists all
  of the settings for this particular Django installation. (We've already
  mentioned ``ROOT_URLCONF``, and we'll show you various Django settings
  throughout the book. All the available settings are covered in detail in
  Appendix D.)

The Django error page is capable of displaying more information in certain
special cases, such as the case of template syntax errors. We'll get to those
later, when we discuss the Django template system. For now, uncomment the
``offset = int(offset)`` lines to get the view function working properly again.

Are you the type of programmer who likes to debug with the help of carefully
placed ``print`` statements? You can use the Django error page to do so -- just
without the ``print`` statements. At any point in your view, temporarily insert
an ``assert False`` to trigger the error page. Then, you can view the local
variables and state of the program. Here's an example, using the
``hours_ahead`` view::

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        assert False
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Finally, it's obvious that much of this information is sensitive -- it exposes
the innards of your Python code and Django configuration -- and it would be
foolish to show this information on the public Internet. A malicious person
could use it to attempt to reverse-engineer your Web application and do nasty
things. For that reason, the Django error page is only displayed when your
Django project is in debug mode. We'll explain how to deactivate debug mode
in Chapter 12. For now, just know that every Django project is in debug mode
automatically when you start it. (Sound familiar? The "Page not found" errors,
described earlier in this chapter, work the same way.)

What's next?
============

So far, we've been writing our view functions with HTML hard-coded directly
in the Python code. We've done that to keep things simple while we demonstrated
core concepts, but in the real world, this is nearly always a bad idea.

Django ships with a simple yet powerful template engine that allows you to
separate the design of the page from the underlying code. We'll dive into
Django's template engine in the next chapter `Chapter 4`_.

.. _Chapter 4: chapter04.html
