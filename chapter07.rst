================
Capitolo 7: Form
================

I form HTML sono l'ossatura di siti web interattivi, dalla semplicità del
singolo box di ricerca di Google agli ubiqui form per la scrittura dei commenti
in un blog alle più complesse interfacce personalizzate per l'inserimento dei
dati. Questo capitolo spiega come puoi usare Django per consentire l'inserimento
di dati tramite form, convalidarli e farci qualcosa. Durante la strada,
parleremo degli oggetti ``HttpRequest`` e ``Form``.

Prendere dati dagli oggetti Request
===================================

Abbiamo introdotto gli oggetti ``HttpRequest`` nel Capitolo 3, quando abbiamo
parlato delle view, ma non abbiamo detto molto. Ricordando che ogni funzione di
view richiede un oggetto ``HttpRequest`` come primo parametro, nella nostra view
``hello()``::

    from django.http import HttpResponse

    def hello(request):
        return HttpResponse("Hello world")

Gli oggetti ``HttpRequest``, come nell'esempio la variabile ``request``, hanno
un numero di attributi e metodi interessanti con i quali è bene familiarizzare,
per capire cosa è possibile fare. Puoi usare questi attributi per avere
informazioni riguardo la richiesta corrente (nel nostro esempio, l'utente che
sta caricando la pagina Django-powered), al momento in cui la funzione di view
viene eseguita.

Informazioni riguardo l'URL
-------------------------

Gli oggetti ``HttpRequest`` contengono diversi pezzi di informazioni riguardo
l'URL richiesto attualmente:

===========================   ====================================  ========================
Attributo/Metodo              Descrizione                           Esempio
===========================   ====================================  ========================
``request.path``              Il percorso completo, escluso il      ``"/hello/"``
                              dominio ma incluso lo slash finale.

``request.get_host()``        L'host (ovvero, il "dominio" nel      ``"127.0.0.1:8000"``
                              comune dire).                         o ``"www.example.com"``

``request.get_full_path()``   Il ``path``, più una stringa di       ``"/hello/?print=true"``
                              query (se disponibile).

``request.is_secure()``       ``True`` se la richiesta è stata      ``True`` or ``False``
                              fatta via HTTPS. Altrimenti,
                              ``False``.
===========================   ====================================  ========================

Usa sempre questi attributi/metodi piuttosto di scrivere URL a mano nelle tue
view. Questo rende più flessibile il codice e può essere riutilizzato in altri
casi. Un esempio semplicistico::

    # SBAGLIATO!
    def current_url_view_bad(request):
        return HttpResponse("Welcome to the page at /current/")

    # BUONO
    def current_url_view_good(request):
        return HttpResponse("Welcome to the page at %s" % request.path)

Altre Informazioni Riguardo la Richiesta
----------------------------------------

``request.META`` è un dizionario Python contenente tutti gli header della
richiesta HTTP -- incluso l'indirizzo IP dell'utente e l'user agent (solitamente
il nome e la versione del browser). Nota che la lista completa degli header
disponibili dipende da quali header invia l'utente e da quali header processa
il tuo server. Alcune chiavi comunemente disponibili in questo dizionario sono:

* ``HTTP_REFERER`` -- L'URL da cui si proviene, se esite. (Nota l'errore
  ortografico di ``REFERER``).
* ``HTTP_USER_AGENT`` -- La stringa che definisce lo user agent del browser
  dell'utente, se presente. Questa somiglia a qualcosa come:
  ``"Mozilla/5.0 (X11; U; Linux i686; fr-FR; rv:1.8.1.17) Gecko/20080829 Firefox/2.0.0.17"``.
* ``REMOTE_ADDR`` -- L'indirizzo IP del client, ad esempio ``"12.345.67.89"``.
  (Se la richiesta è passata tramite proxy, questa chive potrebbe restituire una
  lista di indirizzi IP separati da virgola, ad esempio ``"12.345.67.89,23.456.78.90"``).

Da notare che poiché ``request.META`` è un semplice dizionario Python, ottieni
l'eccezione ``KeyErrorse provi ad accedere ad una chiave che non esiste.
(Questo poiché gli HTTP headers sono dati *esterni* -- ovvero, sono passati dal
browser dell'utente -- non sono affidabili, ed è tuo compito di progettazione
far fallire la tua applicazione nel caso in cui un particolare header è vuoto o
non esiste). Puoi comunque usare una istruzione ``try``/``except`` o il metodo
``get()`` per gestire casi di chiavi indefinite::

    # SBAGLIATO!
    def ua_display_bad(request):
        ua = request.META['HTTP_USER_AGENT']  # Might raise KeyError!
        return HttpResponse("Your browser is %s" % ua)

    # BUONO (VERSIONE 1)
    def ua_display_good1(request):
        try:
            ua = request.META['HTTP_USER_AGENT']
        except KeyError:
            ua = 'unknown'
        return HttpResponse("Your browser is %s" % ua)

    # BUONO (VERSIONE 2)
    def ua_display_good2(request):
        ua = request.META.get('HTTP_USER_AGENT', 'unknown')
        return HttpResponse("Your browser is %s" % ua)

Ti incoraggiamo a scrivere piccole view che mostrino tutti i dati ``request.META``
per scoprire cosa conosci qui. Ecco come appare una view del genere::

    def display_meta(request):
        values = request.META.items()
        values.sort()
        html = []
        for k, v in values:
            html.append('<tr><td>%s</td><td>%s</td></tr>' % (k, v))
        return HttpResponse('<table>%s</table>' % '\n'.join(html))

Come esercizio, converti questa view per usare il sistema di template di Django
invece di scrivere brutalmente l'HTML. Prova inoltre ad aggiungere ``request.path`` e
altri metodi di ``HttpRequest`` visti nella sezione precedente.

Informazioni riguardo i Dati inviati
------------------------------------

Aldilà dei metadata di base della richiesta, gli oggetti ``HttpRequest`` hanno
due attributi che contengono informazioni riguardo le informazioni inviate
dall'utente: ``request.GET`` e ``request.POST``. Entrambi sono oggetti simili a
dizionari che ti permettono di accedere ai dati ``GET`` e ``POST``.

.. admonition:: Oggetti simili a dizionari

    Quando diciamo che ``request.GET`` w ``request.POST`` sono oggetti "simili a
    dizionari", vogliamo dire che hanno il comportamento standard dei dizionari
    Python, ma tecnicamente non sono dei veri dizionari. Per esempio,
    ``request.GET`` e ``request.POST`` hanno entrambi i metodi ``get()``, ``keys()``
    e ``values()``, e li puoi iterare usando con un ciclo for-in con
    ``for key in request.GET``.

    Perché la distinzione? Poiché entrambi ``request.GET`` e ``request.POST``
    hanno dei metodi addizionali che i normali dizionari non hanno. Li vedremo
    tra breve.

    Abbiamo incontrato il termine simile oggetti simili a file -- oggetti
    Python che hanno i metodi di base, come ``read()``, che agiscono come i
    "reali" oggetti file.

I dati ``POST`` sono generalmente inviati tramite il tag HTML ``<form>``, mentre
i dati ``GET`` possono derivare da un form o da una query inviata nell'URL della
pagina.

Esempio di un semplice Gestore di Form
======================================

Continuiamo con l'esempio già mostrato durante questo libro, autori ed
editori/case-editrici, creiamo una semplice view che permette all'utente di
cercare nel nostro database di libri per titolo.

In generale, ci sono due parti per lo sviluppo di un form: l'interfaccia utente
HTML ed il codice per la view di backend che elabora i dati inviati. La prima
parte è semplice; creiamo una view che mostra i form di ricerca::

    from django.shortcuts import render

    def search_form(request):
        return render(request, 'search_form.html')

Come imparato nel Capitolo 3, questi view possono stare ovunque nel tuo percorso
Python. Per comodità, inseriscile in ``books/views.py``.

Il template che lo forma, ``search_form.html``, dovrebbe somigliare a questo::

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>

L'URLpattern in ``urls.py`` dovrebbe somigliare a questo::

    from mysite.books import views

    urlpatterns = patterns('',
        # ...
        url(r'^search-form/$', views.search_form),
        # ...
    )

(Nota che stiamo importando il modulo ``views`` direttamente, invece di fare
cose come ``from mysite.views import search_form``, poiché ha una forma meno
lunga. Spiegheremo l'importanza di questo approccio in dettaglio nel Capitolo 8).

Or, se eseguiamo il ``runserver`` e visitiamo ``http://127.0.0.1:8000/search-form/``,
vedremo l'interfaccia per la ricerca. Molto semplice.

Provando ad inviare il form, viene restituito un errore 404 di Django. Il form
rimanda all'URL ``/search/``, che non è stato ancora implementato. Fixiamo il
problema creando una seconda funzione di view::

    # urls.py

    urlpatterns = patterns('',
        # ...
        (r'^search-form/$', views.search_form),
        (r'^search/$', views.search),
        # ...
    )

    # views.py

    def search(request):
        if 'q' in request.GET:
            message = 'You searched for: %r' % request.GET['q']
        else:
            message = 'You submitted an empty form.'
        return HttpResponse(message)

Per il momento, questa funzione semplicemente mostra all'utente il termine della
ricerca, per cui possiamo verificare che i dati inviati a Django sono corretti,
e quindi possiamo meglio studiare come avviene questo processo. In breve:

1. Il ``<form>`` definisce una variabile ``q``. Quando viene inviata, il valore
   di ``q`` viene inviato via ``GET`` (``method="get"``) all'URL ``/search/``.

2. La view Django che gestisce l'URL ``/search/`` (``search()``) ha l'accesso
   al valore di ``q`` presente in ``request.GET``.

Una cosa importante da sottolineare è che abbiamo esplicitamente verificato che
``'q'`` esiste in ``request.GET``. Come sottolineato nella sezione ``request.META``
qui sopra, non dobbiamo dare fiducia a nessun dato inviato dagli utenti o
assumere che abbiamo inviato qualcosa senza averne la certezza. Se non eseguiamo
questo controllo, l'invio di un form vuoto solleva l'eccezione ``KeyError``
nella view::

    # SBAGLIATO!
    def bad_search(request):
        # The following line will raise KeyError if 'q' hasn't
        # been submitted!
        message = 'You searched for: %r' % request.GET['q']
        return HttpResponse(message)

.. admonition:: Parametri nella query

    Poiché i dati ``GET`` vengono passati tramite stringhe di richieste (ad esempio,
    ``/search/?q=django``), puoi usare ``request.GET`` per accedere alle
    variabili. Nell'introduzione al sistema di URLconf di Django nel Capitolo 3,
    abbiamo confrontato gli 'URL carini' di Django a più tradizionali URL PHP/Java
    come ``/time/plus?hours=3`` ed abbiamo detto che ti mostreremo come crearli
    in questo Capitolo. Adesso sai come accedere ai parametri della query nelle
    tue view (come ``hours=3`` in questo esempio) -- usando ``request.GET``.

I dati ``POST`` lavorano allo stesso modo di quelli ``GET`` -- basta usare solo
``request.POST`` invece di ``request.GET``. Quale è la differenza fra ``GET`` e
``POST``? Usa ``GET`` quando l'azione di invio del form è solo una richiesta per
avere ('get') un dato. Usa ``POST`` in tutti i casi in cui l'invio del form avrà
un qualche effetto collaterale -- *cambiare* dati, o inviare una e-mail, o
qualcos'altro che sta al di sopra del semplice *mostrare* dati. Nel nostro
esempio di motore di ricerca di libri, useremo ``GET`` perché la query non
cambi alcun dato nel nostro server. (Vedi
http://www.w3.org/2001/tag/doc/whenToUseGet.html se vuoi saperne di più su ``GET``
e ``POST``).

Ora che abbiamo verificato che ``request.GET`` è stata passata correttamente,
andiamo ad eseguire una ricerca nel nostro database di libri con i dati
dell'utente(di nuovo, in ``views.py``)::

    from django.http import HttpResponse
    from django.shortcuts import render
    from mysite.books.models import Book

    def search(request):
        if 'q' in request.GET and request.GET['q']:
            q = request.GET['q']
            books = Book.objects.filter(title__icontains=q)
            return render(request, 'search_results.html',
                {'books': books, 'query': q})
        else:
            return HttpResponse('Please submit a search term.')

Ecco alcune cose da notare:

* Oltre a controllare che ``'q'`` esista in ``request.GET``, abbiamo anche
  controllato che ``request.GET['q']`` è un valore non vuoto prima di passarlo
  alla query del database.

* Usiamo ``Book.objects.filter(title__icontains=q)`` per richiedere alla tabella
  libri di ritornare tutti i libri che hanno un titolo che corrisponde a quello
  dato dall'utente. ``icontains`` è un tipo di ricerca (come spiegato nel
  Capitolo 5, appendice B), e l'istruzione può essere tradotta brutalmente come
  "Prendi i libri che contengono il titolo ``q``, senza distinzioni di maiuscolo
  o minuscolo".

  Questo è un modo molto semplice per fare una ricerca. Non raccomandiamo di
  usare una semplice query ``icontains`` su un database di produzione, poiché
  può essere davvero lento. (Nel mondo reale, vorremmo usare una ricerca con un
  certo meccanismo di ordine. Cerca su internet *open-source full-text search*
  per avere una ide delle possibilità).

* Passiamo ``books``, una lista di oggetti ``Book``, al template. Il codice del
  template ``search_results.html`` sarà qualcosa di simile::

      <p>You searched for: <strong>{{ query }}</strong></p>

      {% if books %}
          <p>Found {{ books|length }} book{{ books|pluralize }}.</p>
          <ul>
              {% for book in books %}
              <li>{{ book.title }}</li>
              {% endfor %}
          </ul>
      {% else %}
          <p>No books matched your search criteria.</p>
      {% endif %}

  Nota l'uso del filtro dei template ``pluralize``, che restituisce una "s" quando
  appropriato, in base al numero di libri trovati.

Migliorare il nostro esempio di Gestione di un Form
===================================================

Come nei precedenti capitoli, ti abbiamo mostrato il modo più semplice per far
funzionare il tutto. Ora sottolineeremo alcuni problemi e ti mostreremo come
migliorarlo.

In primis, la gestione della nostra view ``search()``  di una query vuota è
scadente -- mostriamo solo un messaggio ``"Please submit a search term."``,
richiedendo all'utente di pigiare sul pulsante per tornare indietro. Questo è
orrido e non professionale, e se hai mai implementato qualcosa del genere nella
foresta, i tuoi privilegi Django sarebbero revocati.

Sarebbe meglio mostrare di nuovo il form, con un errore che permetta all'utente
di riprovare immediatamente. La via migliore per farlo è renderizzare ancora il
template, ad esempio così:

.. parsed-literal::

    from django.http import HttpResponse
    from django.shortcuts import render
    from mysite.books.models import Book

    def search_form(request):
        return render(request, 'search_form.html')

    def search(request):
        if 'q' in request.GET and request.GET['q']:
            q = request.GET['q']
            books = Book.objects.filter(title__icontains=q)
            return render(request, 'search_results.html',
                {'books': books, 'query': q})
        else:
            **return render(request, 'search_form.html', {'error': True})**

(Nota che abbiamo incluso ``search_form()`` per cui puoi vedere entrambe le view
in un solo posto).

Abbiamo migliorato ``search()`` renderizzando di nuovo il template ``search_form.html``,
se la query è vuota. E poiché abbiamo bisogno di mostrare un messaggio di errore
nel template, gli passiamo una variabile. Possiamo quindi modificare
``search_form.html`` affinché controlli la variabile ``error``:

.. parsed-literal::

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        **{% if error %}**
            **<p style="color: red;">Please submit a search term.</p>**
        **{% endif %}**
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>

Possiamo ancora usare il template con dalla view originale, ``search_form()``,
poiché ``search_form()`` non passa la variabile ``error`` al template -- per cui
il messaggio di errore non viene mostrato in questo caso.

Con questo cambiamento, abbiamo migliorato l'applicazione, ma ecco la domandona:
è realmente necessaria una view ``search_form()`` dedicata? Per come è adesso,
una richiesta all'URL ``/search/`` (senza alcun parametro ``GET``) mostra un
form vuoto (ma con un errore). Possiamo rimuovere la view ``search_form()``,
con il suo URLpattern, cambiando semplicemente ``search()`` per nascondere il
messaggio di errore quando qualcuno visita ``/search/`` senza parametri ``GET``::

    def search(request):
        error = False
        if 'q' in request.GET:
            q = request.GET['q']
            if not q:
                error = True
            else:
                books = Book.objects.filter(title__icontains=q)
                return render(request, 'search_results.html',
                    {'books': books, 'query': q})
        return render(request, 'search_form.html',
            {'error': error})

In questa view aggiornata, se un utente visita ``/search/`` senza parametri ``GET``,
vedrà il form di ricerca con nessun messaggio di errore. Se un utente invia il
form con un valore vuoto di ``'q'``, vedrà un form di ricerca *con* un messaggio
di errore. E, infine, se un utente invia un form con un valore non vuoto di
``'q'``, vedrà effettivamente i risultati.

Possiamo dare un ultimo miglioramento all'applicazione, rimuovendo alcune
ridondanze. Ora che abbiamo unito le due view e gli URL in uno e ``/search/``
gestisce entrambi i form e mostra gli stessi risultati, il ``<form>`` in
``search_form.html`` non dovrebbe avere un URL scritto a mano. Invece di questo::

    <form action="/search/" method="get">

Potremmo usare questo::

    <form action="" method="get">

``action=""`` significa "Invia il form alla pagina corrente". Con questi
cambiamenti, non dovrai più ricordare di cambiare ``action`` tutte le volte che
usi la view ``search()`` in un altro URL.

Semplice Convalida (dei Dati)
=============================

Il nostro esempio di motore di ricerca rimane ancora ragionevolmente semplice,
in particolare in termini di convalida dei dati; stiamo semplicemente
controllando che il nostro termine di ricerca non sia vuoto. Molti form HTML
includono metodi di convalida più complessi di questo. Abbiamo tutti visto nel
web errori come:
* "Please enter a valid e-mail address. 'foo' is not an e-mail address".
* ("Digitare un indirizzo email valido. 'foo' non è un indirizzo
  e-mail);
* "Please enter a valid five-digit U.S. ZIP code. '123' is not a ZIP code".
* ("Digitare un valido U.S. ZIP code di 5 numeri. '123' non è uno ZIP code
  valido");
* "Please enter a valid date in the format YYYY-MM-DD".
* "Digitare una data valida nel formato YYYY-MM-DD";
* "Please enter a password that is at least 8 characters long and contains
  at least one number".
* "Digitare una password che è lunga almeno 8 caratteri e contiene almeno un
  numero".

.. admonition:: Una nota riguardo la convalida JavaScript

    Questo va al di là dello scopo di questo libro, ma puoi usare JavaScript per
    convalidare i dati a livello client, direttamente sul browser. Ma stai
    attento: anche se lo fai, *devi* convalidare i dati anche livello server.
    Alcune persone possono avere JavaScript disattivo, e alcuni utenti malevoli
    possono inviare dati puri, non convalidati direttamente al gestore del form
    per vedere se riescono a causare problemi.

    Non c'è nulla che puoi fare di diverso, oltre convalidare *sempre* i dati
    inviati da utente a livello server (in Django, a livello di view). Devi
    pensare alla convalida JavaScript come un bonus per l'usabilità, non come
    unica via per la convalida.

Modifichiamo la nostra view ``search()`` per controllare che il termine di
ricerca sia inferiore o uguale a 20 caratteri (banalmente, per evitare che
lunghe query rallentino la ricerca). Come possiamo farlo? La via migliore è
quella di mettere la logica direttamente nella view, come in questo modo:

.. parsed-literal::

    def search(request):
        error = False
        if 'q' in request.GET:
            q = request.GET['q']
            if not q:
                error = True
            **elif len(q) > 20:**
                **error = True**
            else:
                books = Book.objects.filter(title__icontains=q)
                return render(request, 'search_results.html',
                    {'books': books, 'query': q})
        return render(request, 'search_form.html',
            {'error': error})

Ora, se volessimo provare ad inviare una ricerca con un termine più lungo di 20
caratteri, semplicemente non verrà effettuata la ricerca; riceveremo un
messaggio di errore. Ma quel messaggio di errore in ``search_form.html`` dice
attualmente ``"Please submit a search term."`` -- perciò dobbiamo cambiarlo in
maniera accurata per rispondere ad entrambi i casi:

.. parsed-literal::

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        {% if error %}
            <p style="color: red;">Please submit a search term 20 characters or shorter.</p>
        {% endif %}
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>

C'è qualcosa di brutto qui. Il nostro messaggio di errore globale può causare
confusione. Perché viene mostrato un messaggio di errore per un form inviato
senza termini di ricerca non dice nulla riguardo al limite dei 20 caratteri? I
messaggi di errore dovrebbero essere specifici, senza possibili ambiguità o
confusione.

Il problema è che stiamo usando un semplice valore booleano per ``error``,
mentre potrebbe essere più utile avere una *lista* dei messaggi di errore. Ecco
come risolvere il problema:

.. parsed-literal::

    def search(request):
        **errors = []**
        if 'q' in request.GET:
            q = request.GET['q']
            if not q:
                **errors.append('Enter a search term.')**
            elif len(q) > 20:
                **errors.append('Please enter at most 20 characters.')**
            else:
                books = Book.objects.filter(title__icontains=q)
                return render(request, 'search_results.html',
                    {'books': books, 'query': q})
        return render(request, 'search_form.html',
            {**'errors': errors**})

Ora, dobbiamo fare una piccola modifica al template ``search_form.html`` per
mostrare correttamente la lista ``errors`` piuttosto che il valore booleano ``error``:

.. parsed-literal::

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        **{% if errors %}**
            **<ul>**
                **{% for error in errors %}**
                **<li>{{ error }}</li>**
                **{% endfor %}**
            **</ul>**
        **{% endif %}**
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>

Creare un Form di Contatto
==========================

Anche se abbiamo giocato con il nostro esempio del motore di ricerca diverse
volte in questo libro e lo abbiamo migliorato, rimane fondamentalmente semplice:
un singolo campo ``'q'``. Poiché è così semplice, non abbiamo usato alcuna
libreria dei Form di Django per lavorarci. Per form più complessi, è necessario
avere un trattamento complesso -- e ora svilupperemo qualcosa di più complesso:
un form di contatto.

Questo form deve permettere agli utenti di un sito web di inviare un qualche
feedback, e contenere un casella opzionale per avere un indirizzo e-mail di
ritorno. Dopo che il form viene inviato ed i dati sono convalidati, esso spedirà
automaticamente il messaggio via e-mail al team del sito web.

Iniziamo partendo dal nostro template, ``contact_form.html``.

.. parsed-literal::

    <html>
    <head>
        <title>Contact us</title>
    </head>
    <body>
        <h1>Contact us</h1>

        {% if errors %}
            <ul>
                {% for error in errors %}
                <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <form action="/contact/" method="post">
            <p>Subject: <input type="text" name="subject"></p>
            <p>Your e-mail (optional): <input type="text" name="email"></p>
            <p>Message: <textarea name="message" rows="10" cols="50"></textarea></p>
            <input type="submit" value="Submit">
        </form>
    </body>
    </html>

Abbiamo definito 3 campi: il soggetto, un indirizzo email ed un messaggio. Il
secondo è opzionale, ma gli altri due campi sono richiesti. Nota che stiamo
usando ``method="post"`` qui al posto di ``method="get"`` perché questo form ha
un effetto collaterale -- invia e-mail. Inoltre, abbiamo copiato il codice di
errore da mostrare dal template precedente ``search_form.html``.

Se continuiamo giù per la strada stabilita dalla nostra view ``search()`` della
sezione precedente, una versione semplice della nostra via ``contact()``
potrebbe essere la seguente::

    from django.core.mail import send_mail
    from django.http import HttpResponseRedirect
    from django.shortcuts import render

    def contact(request):
        errors = []
        if request.method == 'POST':
            if not request.POST.get('subject', ''):
                errors.append('Enter a subject.')
            if not request.POST.get('message', ''):
                errors.append('Enter a message.')
            if request.POST.get('email') and '@' not in request.POST['email']:
                errors.append('Enter a valid e-mail address.')
            if not errors:
                send_mail(
                    request.POST['subject'],
                    request.POST['message'],
                    request.POST.get('email', 'noreply@example.com'),
                    ['siteowner@example.com'],
                )
                return HttpResponseRedirect('/contact/thanks/')
        return render(request, 'contact_form.html',
            {'errors': errors})


(Se stai seguendo l'esempio cardine, potresti voler inserire questa view nel
file ``books/views.py``). Non ha nulla a che fare con l'applicazione dei libri,
perciò dove metterla? E' una tua scelta; a Django non importa, fino a che sei in
grado di far puntare la tua view al tuo URLconf. La nostra personale preferenza
sarebbe quella di creare un directory separata, ``contact``, allo stesso livello
della directory ``books``. Questa dovrebbe contenere due file vuoti ``__init__.py``
e ``views.py``).

Un paio di cose nuove sono successe qui:

* Stiamo controllando che ``request.method`` è ``'POST'``. Questo è vero solo
  nel caso in cui viene inviato il form; non sarà vero se il form viene
  solamente visto (In quest'ultimo caso, ``request.method`` sarà impostato a
  ``'GET'``, perché la comune navigazione viene riconosciuta come ``GET``, non
  ``POST``). Questo permette di isolare bene il caso di "mostrare form" dal caso
  "elaborare form".

* Invece di ``request.GET``, usiamo ``request.POST`` per accedere ai dati inviati
  dal form. Questo è necessario perché l'HTML ``<form>`` in ``contact_form.html``
  usa ``method="post"``. Se questa view viene visualizzata via ``POST``, allora
  ``request.GET`` sarà vuoto.

* Questa volta, abbiamo *due* campi richiesti, ``subject`` (oggetto) e ``message``
  (messaggio), per cui dobbiamo controllarli entrambi. Nota che stiamo usando
  ``request.POST.get()`` e restituendo una stringa vuota come valore di default;
  questo è un buon modo per gestirli semplicemente entrambi nel caso di chiavi o
  dati mancanti.

* Anche se il campo ``email`` non è richiesto, lo controlliamo quando viene
  inviato. Il nostro algoritmo di controllo è debole -- stiamo solo controllando
  che la stringa contenga solo un carattere ``@``. Nel mondo reale, potresti
  voler usare un controllo più robusto (e Django te lo fornisce, come ti
  mostreremo tra breve).

* Stimo usando la funzione ``django.core.mail.send_mail`` per inviare una e-mail.
  Questa funzione richiede 4 argomenti: l'oggetto, il corpo, l'indirizzo da cui
  si spedisce ed una lista degli indirizzi destinatari. ``send_mail`` è un utile
  wrapper della classe ``EmailMessage`` di Django, che fornisce impostazioni
  avanzate come allegati, e-mail multiple e pieno controllo degli header delle
  email.

  Nota che per inviare una e-mail con ``send_mail()``, il tuo server deve essere
  configurato per inviare mail, e Django deve essere informato riguardo l'email
  server. Leggi http://docs.djangoproject.com/en/dev/topics/email/ per i
  dettagli.

* Dopo che l'e-mail è stata spedita, re-indirizziamo l'utente ad una pagina di
  "successo" ritornando un oggetto ``HttpResponseRedirect``. Ti lasceremo
  l'implementazione della pagina di "successo" (è un semplice view/URLconf/template),
  ma ti spiegheremo perché effettuiamo un redirect invece, per esempio, di
  chiamare semplicemente ``render()`` con un template.

  La ragione: se l'utente pigia "Aggiorna" su una pagina carica via ``POST``, la
  richiesta viene ripetuta. Questo può portare a comportamenti indesiderati,
  come record duplicati aggiunti al database -- o, nel nostro esempio, ad e-mail
  inviate due volte. Se l'utente è invece re-indirizzato ad un'altra pagina dopo
  la richiesta ``POST``, allora non c'è modo per lui di ripetere la richiesta.

  Devi *sempre* fare un redirect per ogni richiesta ``POST`` che abbia successo.
  E' una best practice dello sviluppo Web.

Questa view funziona, ma le funzioni di convalida sono un po' brutali. Immagina
di dover processare un form con dozzine di campi; vuoi davvero scrivere una
istruzione ``if`` per ognuno di essi?

Un altro problema è la difficoltà nel *mostrare di nuovo il form*. Nel caso di
errori di convalida, è una best-practice mostrare di nuovo il form *con* i dati
precedentemente inviati, in modo che l'utente possa capire dove ha sbagliato (ed
inoltre re-inserire i dati corretti nei campi segnalati). Potremmo *manualmente*
passare i dati via ``POST`` al template, ma dovremmo modificare l'HTML in
maniera che ogni campo possa avere un particolare valore:

.. parsed-literal::

    # views.py

    def contact(request):
        errors = []
        if request.method == 'POST':
            if not request.POST.get('subject', ''):
                errors.append('Enter a subject.')
            if not request.POST.get('message', ''):
                errors.append('Enter a message.')
            if request.POST.get('email') and '@' not in request.POST['email']:
                errors.append('Enter a valid e-mail address.')
            if not errors:
                send_mail(
                    request.POST['subject'],
                    request.POST['message'],
                    request.POST.get('email', 'noreply@example.com'),
                    ['siteowner@example.com'],
                )
                return HttpResponseRedirect('/contact/thanks/')
        return render(request, 'contact_form.html', {
            'errors': errors,
            **'subject': request.POST.get('subject', ''),**
            **'message': request.POST.get('message', ''),**
            **'email': request.POST.get('email', ''),**
        })

    # contact_form.html

    <html>
    <head>
        <title>Contact us</title>
    </head>
    <body>
        <h1>Contact us</h1>

        {% if errors %}
            <ul>
                {% for error in errors %}
                <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <form action="/contact/" method="post">
            <p>Subject: <input type="text" name="subject" **value="{{ subject }}"**></p>
            <p>Your e-mail (optional): <input type="text" name="email" **value="{{ email }}"**></p>
            <p>Message: <textarea name="message" rows="10" cols="50">**{{ message }}**</textarea></p>
            <input type="submit" value="Submit">
        </form>
    </body>
    </html>

Qui c'è un sacco di fuffa, che danno diverse opportunità all'errore umano.
Speriamo che tu stia iniziando a vedere di usare alcune di librerie di più alto
livello per gestire i compiti relativi ai form ed alla loro convalida.

La tua prima Classe Form
========================

Django include alcune librerie relative ai form, chiamate ``django.forms``, che
gestiscono molte problematiche che abbiamo esplorato in questo capitolo -- dal
mostra il form HTML alla convalida dei dati. Entriamo nel dettglio e vediamo
come rivedere la nostra applicazione di form di contatto usando il framework dei
form di Django.

.. admonition:: La libreria "newforms" di Django

    Girando per la comunità Django, potresti vedere qualcosa chiamato
    ``django.newforms``. Quando alcuni parlano di ``django.newforms``, intendono
    ciò che adesso è ``django.forms`` -- la libreria trattata da questo capitolo.

    La ragione di questo nome è storica. Quando Django è stato rilasciato al
    pubblico, aveva un complicato e confuso sistema di gestione dei Form,
    ``django.forms``. E' stato completamente riscritto, e le nuove versioni lo
    hanno chiamato ``django.newforms``, per qualcuno potrebbe usare il vecchio
    sistema. Quando Django 1.0 è stato rilasciato, il vecchio ``django.forms``
    è scomparso, e ``django.newforms`` è diventato ``django.forms``.

Il modo principale per usare il framework dei form è definire una classe ``Form``
per ogni ``<form>`` HTML con cui stai lavorando. Nel nostro caso, abbiamo un
solo ``<form>``, per cui dobbiamo creare una classe ``Form``. Questa classe può
stare ovunque tu voglia -- incluso direttamente nel tuo file ``views.py`` -- ma
una convenzione adottata dalla comunità è tenere le classi ``Form`` separate in
un file chiamato ``forms.py``. Crea questo file nella stessa directory in cui
sta ``views.py``, ed incolla il seguente codice::

    from django import forms

    class ContactForm(forms.Form):
        subject = forms.CharField()
        email = forms.EmailField(required=False)
        message = forms.CharField()

E' molto intuitivo, e simile alla sintassi dei Modelli di Django. Ogni campo del
form è rappresentato da un tipo di classe ``Field`` -- `CharField`` e ``EmailField``
sono gli unici campi usati qui -- come attributi della classe ``Form``. Ogni
campo è richiesto di default, per cui, per rendere opzionale ``email``, bisogna
specificare ``required=False``.

Saltiamo nella shell interattiva di Python e vediamo cosa può fare questa classe.
La prima cosa da fare è mostrare l'HTML stesso::

    >>> from contact.forms import ContactForm
    >>> f = ContactForm()
    >>> print f
    <tr><th><label for="id_subject">Subject:</label></th><td><input type="text" name="subject" id="id_subject" /></td></tr>
    <tr><th><label for="id_email">Email:</label></th><td><input type="text" name="email" id="id_email" /></td></tr>
    <tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" /></td></tr>

Django aggiunge un label ad ogni campo, usando i tag ``<label>`` per una
questione di accessibilità. L'idea è rendere il comportamento di default più
ottimale possibile.

Questo output di default è formato da tag ``<table>``, ma ci sono alcuni output
integrati::

    >>> print f.as_ul()
    <li><label for="id_subject">Subject:</label> <input type="text" name="subject" id="id_subject" /></li>
    <li><label for="id_email">Email:</label> <input type="text" name="email" id="id_email" /></li>
    <li><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></li>
    >>> print f.as_p()
    <p><label for="id_subject">Subject:</label> <input type="text" name="subject" id="id_subject" /></p>
    <p><label for="id_email">Email:</label> <input type="text" name="email" id="id_email" /></p>
    <p><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></p>

Nota che i tag di chiusura ed apertura ``<table>``, ``<ul>`` e ``<form>`` non
sono inclusi nell'output, per cui puoi aggiungere righe facoltative e
personalizzazioni particolari se necessario.

Questi metodi sono solo scorciatoie per i casi comuni in cui è necessario
"mostrare l'intero form". Puoi inoltre mostrare l'HTML di un particolare campo::

    >>> print f['subject']
    <input type="text" name="subject" id="id_subject" />
    >>> print f['message']
    <input type="text" name="message" id="id_message" />

La seconda cosa che può fare un oggetto ``Form`` è convalidare i dati. Per
convalidare i dati, crea un nuovo oggetto ``Form`` e passagli un dizionario di
dati mappato con i nomi di ogni singolo campo::

    >>> f = ContactForm({'subject': 'Hello', 'email': 'adrian@example.com', 'message': 'Nice site!'})

Una volta associato data con una istanza di ``Form``, hai creato un form
"bound" (N.d.T. ovvero un oggetto form che contiene i reali valori)::

    >>> f.is_bound
    True

Chiama il metodo ``is_valid()`` su ogni ``Form`` creato per trovare i dati
validi. Abbiamo passato un valore valido per ogni campo, per cui ``Form`` è
intermente valido::

    >>> f.is_valid()
    True

Se non passi il campo ``email``, tutto rimane valido, perché abbiamo specificato
``required=False`` per quel campo::

    >>> f = ContactForm({'subject': 'Hello', 'message': 'Nice site!'})
    >>> f.is_valid()
    True

Ma, se lasci fuori ``subject`` o ``message``, il ``Form`` non è più valido::

    >>> f = ContactForm({'subject': 'Hello'})
    >>> f.is_valid()
    False
    >>> f = ContactForm({'subject': 'Hello', 'message': ''})
    >>> f.is_valid()
    False

Puoi anche specificare il campo per ottenere relativi messaggi di errori::

    >>> f = ContactForm({'subject': 'Hello', 'message': ''})
    >>> f['message'].errors
    [u'This field is required.']
    >>> f['subject'].errors
    []
    >>> f['email'].errors
    []

Ogni istanza ``Form`` creata ha un attributo ``errors`` che ti da la possibilità
di mostrare una lista di messaggi di errori::

    >>> f = ContactForm({'subject': 'Hello', 'message': ''})
    >>> f.errors
    {'message': [u'This field is required.']}

Finalmente, per ogni istanza ``Form`` che viene trovata valida, è disponibile un
attributo ``cleaned_data``. Questo è un dizionario dei dati inviati, "puliti".
Il framework dei form di Django non solo convalida i dati, ma li pulisce
convertendoli in appropriati valori Python.

    >>> f = ContactForm({'subject': 'Hello', 'email': 'adrian@example.com', 'message': 'Nice site!'})
    >>> f.is_valid()
    True
    >>> f.cleaned_data
    {'message': u'Nice site!', 'email': u'adrian@example.com', 'subject': u'Hello'}

Il nostro modulo di contatto si occupa solo di stringhe, che sono "pulite" in
oggetti Unicode -- ma se usiamo ``IntegerField`` o ``DateField``, il framework
dei form assicura che ``cleaned_data`` usi i giusti interi Python o gli oggetti
``datetime.date`` per i campi opportuni.

Inserire Oggetti Form nelle View
================================

Con alcune conoscenze di base riguardo le classi ``Form``, puoi già capire come
usare questa infrastruttura per sostituire parte della nostra view ``contact()``.
Ecco come possiamo riscrivere ``contact()`` per usare il framework dei form::

    # views.py

    from django.shortcuts import render
    from mysite.contact.forms import ContactForm
    from django.http import HttpResponseRedirect
    from django.core.mail import send_mail

    def contact(request):
        if request.method == 'POST':
            form = ContactForm(request.POST)
            if form.is_valid():
                cd = form.cleaned_data
                send_mail(
                    cd['subject'],
                    cd['message'],
                    cd.get('email', 'noreply@example.com'),
                    ['siteowner@example.com'],
                )
                return HttpResponseRedirect('/contact/thanks/')
        else:
            form = ContactForm()
        return render(request, 'contact_form.html', {'form': form})

    # contact_form.html

    <html>
    <head>
        <title>Contact us</title>
    </head>
    <body>
        <h1>Contact us</h1>

        {% if form.errors %}
            <p style="color: red;">
                Please correct the error{{ form.errors|pluralize }} below.
            </p>
        {% endif %}

        <form action="" method="post">
            <table>
                {{ form.as_table }}
            </table>
            {% csrf_token %}
            <input type="submit" value="Submit">
        </form>
    </body>
    </html>

Guarda quanta fuffa siamo stati in grado di rimuovere! Il framework dei form
integrato in Django si occupa del mostrare l'HTML, della convalida, della pulizia
dei dati e del mostrare di nuovo il form con degli errori.

Poiché abbiamo creato un form POST (che ha l'effetto di modificare dati),
dobbiamo preoccuparci del Cross-site request forgery. Fortunatamente, non dovrai
preoccupartene troppo, perché Django include un sistema semplice da usare per
proteggerti. In breve, tutti i form POST che si trovano all'interno di URL,
dovrebbero usare il tag dei template ``{% csrf_token %}``. Puoi trovare più
dettagli su ``{% csrf_token %}`` al :doc:`chapter16` e :doc:`chapter20`.

Prova ad avviare il tutto localmente. Il caricamento del form, l'invio senza
riempire alcun campo, l'invio con un indirizzo e-mail invalido, e poi l'invio
con i dati corretti (Ovviamente, a seconda delle configurazioni del suo server
e-mail, potresti ottenere un errore quando ``send_mail()``, ma questo è un
altro problema).

Cambiare il modo in cui sono renderizzati i Campi
=================================================

Probabilmente, hai già notato che quando l'utente renderizza questo form
localmente, il campo ``message`` è mostrato come un ``<input type="text">``,
mentre potrebbe essere un ``<textarea>``. Possiamo fixare il problema cambiando
le impostazioni *widget* del campo:

.. parsed-literal::

    from django import forms

    class ContactForm(forms.Form):
        subject = forms.CharField()
        email = forms.EmailField(required=False)
        message = forms.CharField(**widget=forms.Textarea**)

Il framework dei form separa la logica della presentazione di ogni campo in un
insieme di widget. Ogni tipo campo ha un widget di default, ma puoi facilmente
cambiare il comportamento predefinito o dare un widget personalizzato.

Pensa alle classi ``Field`` come rappresentazione della *logica di convalida*,
mentre i widget rappresentano la *logica della presentazione*.

Impostare una lunghezza Massima
===============================

Uno degli aspetti più comuni della convalida richiede il controllo che un campo
sia di una certa dimensione. Per un buon controllo, miglioreremo il nostro
``ContactForm`` per limitare il ``subject`` (oggetto) a 100 caratteri. Per farlo,
basta dare un ``max_length`` a ``CharField``, in questo modo:

.. parsed-literal::

    from django import forms

    class ContactForm(forms.Form):
        subject = forms.CharField(**max_length=100**)
        email = forms.EmailField(required=False)
        message = forms.CharField(widget=forms.Textarea)

E' valido anche un argomento opzionale ``min_length``.

Impostare un Valore Iniziale
============================

Per migliorare a questo form, andiamo ad aggiungere un *valore iniziale* per il
campo ``subject``: ``"I love your site!"`` ("Amo il tuo sito!" - Un piccolo
suggerimento non può ferire nessuno). Per farlo, possiamo usare l'argomento
``initial`` quando creiamo una istanza di ``Form``:

.. parsed-literal::

    def contact(request):
        if request.method == 'POST':
            form = ContactForm(request.POST)
            if form.is_valid():
                cd = form.cleaned_data
                send_mail(
                    cd['subject'],
                    cd['message'],
                    cd.get('email', 'noreply@example.com'),
                    ['siteowner@example.com'],
                )
                return HttpResponseRedirect('/contact/thanks/')
        else:
            form = ContactForm(
                **initial={'subject': 'I love your site!'}**
            )
        return render(request, 'contact_form.html', {'form': form})

Ora, il campo ``subject`` sarà mostrato con una istruzione già compilata.

Nota che c'è differenza fra passare un dato *iniziale* e passare un dato che
*riempie* il form. La più grande differenza è che se stiamo solo passando un
valore *iniziale*, allora il form non sarà *realmente costruito*, ovvero non
vedremo alcun messaggio di errore.

Regole personalizzate di Convalida
==================================

Immagina di aver lanciato il nostro form di contatto e le e-mail stanno iniziando
ad arrivare. C'è solo un problema: alcuni dei messaggi inviati sono formati solo
da una o due parole, che non sono lunghe abbastanza per dargli un senso.
Decidiamo quindi di adottare una nuova politica di convalida: ci vogliono almeno
quattro parole.

Questi sono numeri per vedere come creare una convalida personalizzata
all'interno del nostro Django form. Se la nostra regola è qualcosa che potremmo
usare ancora ed ancora, allora possiamo creare un tipo di campo detto 'custom'.
Molte convalide personalizzate si occupano di un solo elemento, e possono essere
quindi incluse direttamente nella classe ``Form``.

Aggiungiamo quindi la convalida addizionale per il campo ``message``, per cui
aggiungiamo un metodo ``clean_message()`` alla nostra classe ``Form``:

.. parsed-literal::

    from django import forms

    class ContactForm(forms.Form):
        subject = forms.CharField(max_length=100)
        email = forms.EmailField(required=False)
        message = forms.CharField(widget=forms.Textarea)

        def clean_message(self):
            message = self.cleaned_data['message']
            num_words = len(message.split())
            if num_words < 4:
                raise forms.ValidationError("Not enough words!")
            return message

Il sistema di form di Django cerca automaticamente ogni metodo che inizia con
``clean_`` e finisce con il nome di un campo. Se esiste un metodo con questo
nome, viene invocato durante la convalida.

Più in particolare, il metodo ``clean_message()`` sarà chiamato *dopo* la
logica di convalida di un dato campo (in questo caso, la logica di convalida
richiesta per ``CharField``). Poiché i dati del campo sono già stati
parzialmente elaborati, vengono messi in ``self.cleaned_data``. Inoltre, non
devi preoccuparti di controllare che il valore esiste ed è non vuoto; tutto
questo è fatto dal controllore predefinito.

Per contare il numero di parole, usiamo nativamente una combinazione di
``len()`` e ``split()``. Se l'utente ha inserito troppe poche parole, solleviamo
un errore ``forms.ValidationError``. La stringa collegata a questa eccezione
verrà mostrata all'utente come oggetti nella lista degli errori.

E' importante esplicitare il ritorno del valore pulito del campo alla fine del
metodo. Questo ci permette di modificare il valore (o convertirlo in un tipo
Python differente) con il nostro metodo di convalida. Se dimentichi l'istruzione
return, allora viene restituito ``None`` ed il valore originale viene perso.

Specificare label
=================

Di default, i label (etichette) di Django sono HTML auto-generato creato
sostituendo i trattini bassi (underscore '_') con degli spazi e capitalizzando
la prima lettera -- per cui il campo ``email`` diventa ``"Email"`` (ti suona
familiare? E' lo stesso algoritmo usato per calcolare il valore di default
``verbose_name`` per i campi. Ne abbiamo parlato nel Capitolo 5).

Ma, come nei modelli Django, possiamo personalizzare i label con un campo scelto.
Basta usare ``label``, in questo modo:

.. parsed-literal::

    class ContactForm(forms.Form):
        subject = forms.CharField(max_length=100)
        email = forms.EmailField(required=False, **label='Your e-mail address'**)
        message = forms.CharField(widget=forms.Textarea)

Personalizzare il Design del Form
=================================

Il nostro template ``contact_form.html`` usa ``{{ form.as_table }}`` per
mostrare il form, ma vorremmo mostrare il form in altri modi per avere un
controllo modulare di tutto.

Il modo più veloce per personalizzare la rappresentazione del form è usare i CSS.
Le liste degli errori, in particolare, possono essere oggetto di miglioramenti
visuali ed il codice auto-generato di queste liste usa sempre il codice
``<ul class="errorlist">``, che si presta a questo tipo di modifiche. Ecco delle
modifiche CSS per rendere migliori gli errori::

    <style type="text/css">
        ul.errorlist {
            margin: 0;
            padding: 0;
        }
        .errorlist li {
            background-color: red;
            color: white;
            display: block;
            font-size: 10px;
            margin: 0 0 3px;
            padding: 4px 5px;
        }
    </style>

Se è spesso utile avere un form generato in HTML per noi, in alcuni casi
potresti aver bisogno di sovrascrivere il rendering di default. ``{{ form.as_table }}``
e simili sono utili scorciatoie usate dagli sviluppatori per costruire le
applicazioni, ma tutto ciò che riguarda il mostrare un form può essere
sovrascritto, spesso con il template stesso, e probabilmente troverai un tuo
modo per farlo.

Ogni widget dei campi (``<input type="text">``, ``<select>``, ``<textarea>``,
etc..) può essere renderizzato individualmente accedendo al ``{{ form.fieldname }}``
nel template, e tutti gli errori associati ad un particolare campo sono
disponibili in ``{{ form.fieldname.errors }}``. Sapendo questo, possiamo
costruire un template personalizzato per il nostro form di contatto con il
seguente codice::

    <html>
    <head>
        <title>Contact us</title>
    </head>
    <body>
        <h1>Contact us</h1>

        {% if form.errors %}
            <p style="color: red;">
                Please correct the error{{ form.errors|pluralize }} below.
            </p>
        {% endif %}

        <form action="" method="post">
            <div class="field">
                {{ form.subject.errors }}
                <label for="id_subject">Subject:</label>
                {{ form.subject }}
            </div>
            <div class="field">
                {{ form.email.errors }}
                <label for="id_email">Your e-mail address:</label>
                {{ form.email }}
            </div>
            <div class="field">
                {{ form.message.errors }}
                <label for="id_message">Message:</label>
                {{ form.message }}
            </div>
            <input type="submit" value="Submit">
        </form>
    </body>
    </html>

``{{ form.message.errors }}`` mostra un ``<ul class="errorlist">`` se sono
presenti degli errori, mentre una stringa vuota se il campo è valido (o non
è stato ancora inviato). Possiamo inoltre trattare ``form.message.errors`` come
un Booleano o iterarlo come se fosse una lista. Per esempio::

    <div class="field{% if form.message.errors %} errors{% endif %}">
        {% if form.message.errors %}
            <ul>
            {% for error in form.message.errors %}
                <li><strong>{{ error }}</strong></li>
            {% endfor %}
            </ul>
        {% endif %}
        <label for="id_message">Message:</label>
        {{ form.message }}
    </div>

Nel caso di errori di validazione, viene aggiunta una classe "errors" al
``<div>`` che lo contiene e viene mostrata la lista degli errori in una lista
non ordinata.

Cosa c'è adesso?
================

Questo capitolo conclude il materiale introduttivo in questo libro -- il
cosiddetto "core curriculum". Nelle prossime sezioni del libro, dal capitoli 8
al 12, andremo a vedere nel dettaglio usi più avanzati di Django, incluso la
produzione di una applicazione Django (Capitolo 12).

Dopo questi 7 capitolo, dovresti saperne abbastanza per iniziare a scrivere i
tuoi progetti Django. Il resto del materiale in questo libro ti sarà utile per
riempire i buchi quando ne avrai bisogno.

Partiamo con il `Capitolo 8`_, tornando indietro e guardando più da vicino view
e URLconf (introdotti prima nel `capitolo03`).

.. _Capitolo 8: chapter08.html
