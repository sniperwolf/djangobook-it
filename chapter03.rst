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

Come possiamo allora riprogettare la nostra applicazione per gestire offset
arbitrari? La chiave è quella di utilizzare gli *URLpattern jolly*. Come
abbiamo accennato in precedenza, un urlPattern è un'espressione regolare, di
conseguenza, siamo in grado di utilizzare il modello di espressione regolare
``\d+`` per poter abbinare una o più cifre::

    urlpatterns = patterns('',
        # ...
        url(r'^time/plus/\d+/$', hours_ahead),
        # ...
    )

(Stiamo usando il ``# ...`` per dire che potrebbero esserci altri URLpattern
che abbiamo rimosso da questo esempio).

Questo nuovo urlPattern corrisponderà a qualsiasi URL, da ``/time/plus/2/``, a
``/time/plus/25/`` o anche ``/time/plus/100000000000/``. Vale la pena pensarci
bene, cerchiamo di limitare il tutto in modo che il massimo consentito sia di
99 ore. Questo significa che vogliamo consentire l'inserimento di uno o due
valori numerici - o nella sintassi delle espressioni regolari, che si traduce
in ``\d{1,2}``::

    url(r'^time/plus/\d{1,2}/$', hours_ahead),

.. note::

    Quando si creano applicazioni Web, è sempre importante considerare l'input
    dati più stravagante possibile, e decidere se l'applicazione deve supportare
    o meno tale ingresso. Abbiamo ridotto le possibilità qui limitando l'offset
    a 99 ore.

Ora che abbiamo indicato un jolly per l'URL, abbiamo bisogno di un modo di
trasmettere tali dati "jolly" per la funzione di visualizzazione, in modo da
poter utilizzare una singola funzione di visualizzazione per ogni ora offset
arbitrario. Lo facciamo mettendo tra parentesi i dati degli urlPattern che
vogliamo salvare. Nel caso del nostro esempio, vogliamo salvare qualsiasi numero
presente nell'URL, quindi cerchiamo di mettere tra parentesi la ``\d{1,2}``,
come questo:

    url(r'^time/plus/(\d{1,2})/$', hours_ahead),

Se hai familiarità con le espressioni regolari, ti troverai a casa e sai che
stiamo usando le parentesi per acquisire i dati dal testo corrispondente.

L'URLconf finale, comprese le nostre precedenti view, si presentano così::

    from django.conf.urls.defaults import *
    from mysite.views import hello, current_datetime, hours_ahead

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^time/plus/(\d{1,2})/$', hours_ahead),
    )

Fatto questo, scriviamo la view ``hours_ahead``.

``hours_ahead`` è molto simile alla view ``current_datetime`` che abbiamo
scritto in precedenza, con una differenza fondamentale: ci vuole un argomento in
più, il numero di ore da aggiungere. Ecco il codice::

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

Analizziamo questo codice una riga alla volta:

* La funzione di visualizzazione, ``hours_ahead`` accetta *due* parametri:
  ``request`` e ``offset``.

  * La ``request`` è un oggetto ``HttpRequest``, proprio come in ``hello`` e
    ``current_datetime``. Lo scriviamo ancora: ogni view ha *sempre* un oggetto
    ``HttpRequest`` come primo parametro.

  * ``offset`` è la stringa catturata dalle parentesi nell'urlPattern. Ad
    esempio, se l'URL richiesto era ``/time/plus/3/``, la stringa ``offset`` è
    ``'3'``. Se l'URL richiesto era ``/time/plus/21/``, l'offset è la stringa
    ``'21'``. Si noti che i valori acquisiti saranno sempre stringhe, non interi,
    anche se la stringa è composta da solo cifre, ad esempio ``'21'``.

    (Tecnicamente, i valori acquisiti saranno sempre gli oggetti Unicode, non
    semplici stringhe di byte Python, ma non preoccuparti di questa distinzione
    al momento).

    Abbiamo deciso di chiamare la variabile ``offset``, ma si può chiamare come vuoi,
    a patto che si tratti di un identificatore Python valido. Il nome della
    variabile non importa, ciò che davvero conta è che sia il secondo argomento
    della funzione, dopo ``request``. (E 'anche possibile utilizzare una parole
    chiave, piuttosto che una variabile posizionale in un URLconf. Lo vedremo
    nel capitolo 8).

* La prima cosa che facciamo all'interno della funzione è chiamare ``int()`` su
  ``offset``. Questo converte la stringa in un intero.

  Si noti che Python solleva un'eccezione ``ValueError`` se si chiama ``int()``
  su un valore che non può essere convertito in un intero, come ad esempio la
  stringa ``'foo'``. In questo esempio, se incontriamo la ``ValueError``,
  solleviamo l'eccezione ``django.http.Http404``, che, come puoi immaginare, si
  traduce in un errore 404, "Page not found".

  I lettori più attenti si chiederanno: come potremmo mai raggiungere il caso
  ``ValueError``, in ogni caso, dato che l'espressione regolare nel nostro
  urlPattern -- ``(\d{1,2})`` -- raccoglie solo cifre, e pertanto l'``offset``
  sarà sempre e solo essere una stringa composta da cifre? La risposta è, non lo
  faremo, perché l'urlPattern fornisce un livello modesto, *ma* utile, di
  convalida dell'input, ma dobbiamo ancora verificare il ``ValueError`` nel caso
  in cui questa funzione di visualizzazione venga chiamata in qualche altro modo.
  E' buona norma implementare funzioni tali che non facciano alcuna ipotesi
  riguardo i loro parametri. Accoppiamento lasco, ricordi?

* Nella riga successiva della funzione, calcoliamo la data/ora corrente e
  aggiungiamo il numero adeguato di ore. Abbiamo già visto
  ``datetime.datetime.now()`` dalla vista ``current_datetime``, il nuovo
  concetto qui è che è possibile eseguire la somma aritmetica sulla data/ora con
  la creazione di un oggetto ``datetime.timedelta`` e l'aggiunta di un oggetto
  ``datetime.datetime``. Il nostro risultato viene memorizzato nella variabile
  ``dt``.

  Questa linea mostra anche perché abbiamo chiamato ``int()`` su ``offset`` --
  la funzione ``datetime.timedelta`` richiede il parametro ``hours`` per essere
  un numero intero.

* Più avanti, costruiamo l'output HTML di questa view, proprio come abbiamo
  fatto in ``current_datetime``. Una piccola differenza in questa linea dalla
  linea precedente è che utilizza la stringa di formattazione Python con *due*,
  valori non solo uno. Quindi, ci sono due simboli ``%s`` nella stringa e una
  tupla di valori da inserire: ``(offset, dt)``.


* Infine, ritorniamo un ``HttpResponse`` del HTML. Ormai, lo abbiamo capito.

Avendo scritto la funzione di visualizzazione ed il corrispondente URLconf,
avviare il server di sviluppo (se non è già in esecuzione), e visita
``http://127.0.0.1:8000/time/plus/3/`` per verificarne il funzionamento. Quindi
prova ``http://127.0.0.1:8000/time/plus/5/``. Poi ``http://127.0.0.1:8000/time/plus/24/``.
Infine, visita ``http://127.0.0.1:8000/time/plus/100/`` per verificare che il
pattern configurato da URLconf accetta solo o una o due cifre; Django dovrebbe
visualizzare un errore "Page not found" in questo caso, proprio come abbiamo
visto nella sezione "Una breve nota sui 404 Errori" ("Page not found")
precedenti. Anche gli URL ``http://127.0.0.1:8000/time/plus/`` (*senza* l'ora)
dovrebbero sollevare un 404.

.. admonition:: Ordine di programmazione

    In questo esempio, abbiamo scritto per prima l'urlPattern e la view in
    secondo luogo, ma negli esempi precedenti, abbiamo scritto la vista, poi
    l'urlPattern. Quale tecnica è migliore?

    Beh, ogni sviluppatore è diverso.

    Se sei un tipo di persona che immagina in grande, può avere più senso per te
    scrivere tutti gli URLpatterns per la tua applicazione, allo stesso tempo,
    all'inizio del progetto, e quindi codificare le view. Questo ha il vantaggio
    di dare una chiara "to-do list", e definisce essenzialmente i requisiti dei
    parametri per le funzioni di visualizzazione che avrai bisogno di scrivere.

    Se sei più di uno sviluppatore più build&fix preferirai scrivere prima le
    view, e poi determinare gli URL dopo. Va bene allo stesso modo.

    Alla fine, si tratta di quale tecnica si adatta meglio il tuo cervello.
    Entrambi gli approcci sono validi.

Le belle pagine d'errore di Django
==================================

Prenditi un momento per ammirare l'applicazione che abbiamo fatto finora... ora
cerchiamo di rompere! Deliberatamente, introduciamo un errore di Python nel
nostro file ``views.py`` commentando ``offset = int(offset)``  nella vista
``hours_ahead``::

    def hours_ahead(request, offset):
        # try:
        #     offset = int(offset)
        # except ValueError:
        #     raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Carica il server di sviluppo e visita la pagina ``/time/plus/3/``. Vedrai una
pagina di errore con una notevole quantità di informazioni, compreso un
messaggio ``TypeError`` mostrato in cima: ``"unsupported type for timedelta hours
component: unicode"`` ("tipo non supportato per timedelta componente ore: unicode").

Che cosa è successo? Beh, la funzione ``datetime.timedelta`` si aspetta che il
parametro ``hours`` debba essere un numero intero, e abbiamo commentato il
codice che fa proprio questo. Questo ha fatto sollevare a ``datetime.timedelta``
un errore ``TypeError``. È in genere un tipico bug che ogni programmatore
incontra a un certo punto.

L'obiettivo di questo esempio è mostrare le pagine di errore di Django. Prenditi
del tempo per esplorare la pagina di errore e conoscere i vari pezzi di
informazioni che ti restituisce.

Qui ci sono alcune cose da notare:

* Nella parte superiore della pagina, si trovano le informazioni chiave: il tipo
  di eccezione, tutti i parametri dell'eccezione (il messaggio
  ``"unsupported type"`` in questo caso), il file in cui è stata sollevata
  l'eccezione ed il numero della riga implicata.

* Sotto queste informazioni, la pagina mostra l'intero traceback di Python
  relativo a questa eccezione. Questo è simile alla traccia standard che si
  riceve quando si è davanti all'interprete della riga di comando di Python. Per
  ogni livello ("frame") nella pila, Django mostra il nome del file, il nome
  della funzione/metodo, il numero di riga e il codice sorgente presente in
  quella linea.

  Cliccare sulla riga di codice sorgente (in grigio scuro), e sarà possibile
  vedere diverse linee di codice prima e dopo la linea che ha causato l'errore,
  per darci il contesto.

  Cliccando su "Local vars" sotto qualsiasi fotogramma dello stack per vedere
  una tabella di tutte le variabili locali ed i loro valori attuali nel punto
  esatto del codice in cui è stata sollevata l'eccezione. Queste informazioni
  di debug possono essere di grande aiuto.

* Nota il testo "Switch to copy-and-paste view"  sotto   l'intestazione
  "Traceback". Cliccando su quelle parole, il traceback passerà ad una versione
  alternativa che può essere facilmente copiata e incollata. Utilizza questa
  funzione quando vuoi condividere il traceback dell'eccezione con altri per
  ottenere supporto tecnico - come ad esempio le gentili persone nella chat room
  di Django o sulla mailing list degli utenti Django.

  Più in basso, il pulsante "Share this traceback on a public Web site"
  ("Condividi questa traceback su un pubblico sito Web", in inglese) farà questo
  lavoro per noi in un solo click. Cliccando, il traceback viene inserito sul
  servizio web http://www.dpaste.com/ con un indirizzo univoco che è possibile
  condividere con altre persone.


* Successivamente, la sezione "Richiedi informazioni" comprende una serie di
  informazioni sulla richiesta web in ingresso che ha generato l'errore: le
  informazioni GET e POST, i valori dei cookie, e altre meta-informazioni, come
  ad esempio le intestazioni CGI. L'appendice G ha un riferimento completo di
  tutte le informazioni presenti su un oggetto di richiesta.

  All'interno della sezione "Request information" ("Richiedi informazioni"), la
  sezione "Settings" elenca tutte le impostazioni di questa particolare
  installazione di Django. (Abbiamo già scritto di ``ROOT_URLCONF``, e vi
  mostreremo le varie impostazioni Django in tutto il libro. Tutte le
  impostazioni disponibili sono trattate in dettaglio in Appendice D).

La pagina di errore è in grado di mostrare ancor più informazioni in taluni casi
particolari, come ad esempio nel caso di errori di sintassi nella costruzione di
un template. Arriveremo a quelli più tardi, quando discuteremo del sistema di
template incluso in Django. Ora, è possibile rimuovere il commento
all'istruzione ``offset = int(offset)`` per far tornare a funzionare tutto
correttamente.

Sei il tipo di programmatore che ama scrivere il codice usando ``print``
accuratamente posizionati? È possibile utilizzare la pagina di errore Django per
farlo -- ma senza ``print``. In qualsiasi punto della tua view, è possibile
inserire temporaneamente un ``assert False`` per attivare la pagina di errore.
Quindi, è possibile visualizzare le variabili locali e l'intero stato del
programma. Ecco un esempio, utilizzando la view ``hours_ahead``::

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        assert False
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Infine, è ovvio che molte di queste informazioni sono sensibili -- poiché espone
le "interiora" delle impostazioni e del codice Python e Django -- e sarebbe
sciocco per visualizzare queste informazioni su pubblicamente. Un utente
malevolo potrebbe usarlo per tentare di decodificare l'applicazione Web e fare
cose brutte. Per questo motivo, la pagina di errore Django viene mostrata solo
quando il progetto è in modalità di debug. Spiegheremo come disattivare la
modalità di debug nel Capitolo 12. Per ora, è sufficiente sapere che ogni
progetto Django è in modalità di debug automaticamente quando lo si avvia.

(Ti suona familiare? Gli errori "Page not found", descritti in precedenza in
questo capitolo, funzionano allo stesso modo).

Cosa c'è adesso?
================

Finora, abbiamo scritto le nostre view in HTML a livello di codice direttamente
nel codice Python. Lo abbiamo fatto per mantenere le cose semplici, ed abbiamo
dimostrato concetti fondamentali, ma nel mondo reale, questa è quasi sempre una
cattiva idea.

Vedremo che Django ha un motore di template semplice ma potente che permette di
separare la progettazione della pagina dal codice sottostante. Ci immergeremo
all'interno di questo motore di template nel prossimo capitolo `Capitolo 4`_.

.. _Capitolo 4: chapter04.html
