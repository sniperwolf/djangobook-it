====================
Chapter 4: Templates
====================

Nel capitolo precedente, abbiamo inserito nella nostra view del codice HTML e lo
abbiamo integrato con il metodo per la data/ora. Vale a dire, l'HTML è stato
scritto direttamente nel nostro codice Python::

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Anche se questa tecnica era utile per spiegare come funzionano le view, non è
una buona idea scrivere HTML direttamente sulla view. Ecco perché:

* Qualsiasi modifica al progetto della pagina richiede una modifica al codice
  Python. La progettazione di un sito tende a cambiare molto più spesso di
  quanto debba cambiare il codice Python, per cui sarebbe davvero conveniente
  poter fare modifiche senza bisogno di modificare il codice backend.

* Scrivere codice Python e progettare HTML sono due discipline diverse, e la
  maggior parte degli ambienti di sviluppo web professionale ha diviso tali
  responsabilità a persone separate (o anche reparti separati). Designers e
  programmatori HTML/CSS non dovrebbero essere obbligati a modificare il
  codice Python per svolgere il loro lavoro.

* È molto più efficiente quella situazione in cui i programmatori possono
  lavorare solo sul codice Python ed i designer possono lavorare sui template
  allo stesso tempo, piuttosto che quella nella quale una persona debba
  attendere le modifiche dell'altro prima di iniziare a lavorare.

Per queste ragioni, è molto pulito e più gestibile separare la progettazione
della pagina dal codice Python stesso. Possiamo farlo con il
*sistema di template* di Django, di cui parleremo in questo capitolo.

Nozioni di base sul Sistema di Template
======================================

Un template di Django è una stringa di testo che serve a separare la
presentazione di un documento dai propri dati. Un template definisce un
segnaposto e varie parti di logica di base (tag template) che regolano le
modalità di visualizzazione del documento. Di solito, i modelli vengono
utilizzati per la produzione di codice HTML, ma i modelli di Django sono
ugualmente in grado di generare qualsiasi formato basato su testo.

Cominciamo con un template semplice di esempio. Questo template descrive una
pagina HTML che ringrazia una persona per aver fatto un ordine con una società.
Pensa a come una lettera tipo::

    <html>
    <head><title>Ordering notice</title></head>

    <body>

    <h1>Ordering notice</h1>

    <p>Dear {{ person_name }},</p>

    <p>Thanks for placing an order from {{ company }}. It's scheduled to
    ship on {{ ship_date|date:"F j, Y" }}.</p>

    <p>Here are the items you've ordered:</p>

    <ul>
    {% for item in item_list %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>

    {% if ordered_warranty %}
        <p>Your warranty information will be included in the packaging.</p>
    {% else %}
        <p>You didn't order a warranty, so you're on your own when
        the products inevitably stop working.</p>
    {% endif %}

    <p>Sincerely,<br />{{ company }}</p>

    </body>
    </html>

Questo modello è un codice HTML di base con alcune variabili e tag template
che analizziamo riga per riga::

* Qualsiasi testo circondato da una coppia di parentesi graffe (ad esempio,
  ``{{ person_name }}``) è una *variabile*. Questo significa dire a Django
  "inserisci il valore della variabile che ha il nome incluso nelle parentesi."
  (Come possiamo specificare i valori delle variabili? Ci arriveremo fra un
  attimo).

* Qualsiasi testo che è circondato da parentesi graffe e segni di percentuale
  (ad esempio, {% se ordered_warranty%}) è un *tag template*. La definizione di
  tag è abbastanza ampia: esso dice al sistema di template di "fare qualcosa".

  Questo modello di esempio contiene un tag ``for`` (``{% for item in item_list
  %}``) e un tag ``if`` (``{% if ordered_warranty %}``).

* Ogni tag ``for`` funziona come una dichiarazione , ``for`` ti permette di fare
  iterazioni su un array. Un tag ``if``, come ci si potrebbe aspettare, agisce
  come un ``if`` logico di Python. In questo caso particolare, il tag controlla
  se il valore della variabile ``ordered_warranty`` restituisce ``True``. Se è
  ``True``, il sistema di template mostra tutto quello che c'è tra il
  ``{% if ordered_warranty %}`` e ``{% else %}``. In caso contrario, il sistema
  di template mostra tutto quello che c'è tra ``{% else %}`` e ``{% endif %}``.
  Nota che ``{% else %}`` è opzionale.

* Infine, il secondo paragrafo di questo template contiene un esempio di *filtro*,
  che è il modo più conveniente per alterare la formattazione di una variabile.
  In questo esempio, ``{{ ship_date|date:"F j, Y" }}``, stiamo passando la
  variabile ``ship_date`` al filtro ``date``, dando ``date`` al filtro con un
  parametro ``"F j, Y"``.  Il formato del filtro ``date`` nel formato specificato
  dai parametro. I filtri vengono collegati mediante un carattere pipe ((``|``),
  come negli ambienti Unix.

Ogni template Django ha accesso a diversi tag e filtri integrati, molti dei
quali sono discussi nelle sezioni che seguono. L'appendice E contiene una lista
completa dei tag e filtri, ed è una buona idea familiarizzare con quella lista
in modo da sapere cosa è possibile fare. E 'anche possibile creare i propri
filtri e tag; lo vedremo che nel capitolo 9.

Utilizzare il Sistema di Template
=================================

Tuffiamoci nel sistema di template di Django in modo da vedere come funziona --
ma *senza* integrarlo con le cose che abbiamo creato nel capitolo precedente.
Il nostro obiettivo è quello di mostrare come il sistema funziona
indipendentemente dal resto di Django. (Detto in altro modo: di solito lo
utilizzerai all'interno di una view di Django, ma vogliamo mettere in chiaro
che il sistema di template è solo una libreria Python che si può usare *ovunque*,
non solo in Django).

Ecco il modo più semplice per utilizzare il sistema di template di Django in
codice Python:

1. Creare un oggetto ``Template`` scrivendo il codice come se fosse una stringa;
2. Chiamare il metodo ``render()`` dell'oggetto ``Template`` con un dato insieme
   di variabili (il *contesto*, context) usando la funzione ``render()``. Questo
   restituisce un template completamente tradotto in una stringa, con tutte le
   variabili e tag template esaminati in base al contesto.

In codice, ecco come sembra::

    >>> from django import template
    >>> t = template.Template('My name is {{ name }}.')
    >>> c = template.Context({'name': 'Adrian'})
    >>> print t.render(c)
    My name is Adrian.
    >>> c = template.Context({'name': 'Fred'})
    >>> print t.render(c)
    My name is Fred.

Le seguenti sezioni descrivono ogni passaggio in modo molto più dettagliato.

Creazione degli Oggetti Template
-------------------------

Il modo più semplice per creare un oggetto ``Template`` è istanziarlo
direttamente. La classe ``Template`` sta nel modulo ``django.template`` ed il
suo costruttore accetta un argomento, il codice del template puro. Usiamo
l'interprete interattivo di Python per vedere come funziona tutto questo in
codice.

Dalla directory di progetto ``mysite`` creata da ``django-admin.py
startproject`` (come descritto nel Capitolo 2), digita ``python manage.py shell``
per avviare l'interprete interattivo.

.. admonition:: Uno speciale prompt di Python

    Se hai utilizzato Python prima, ci si potrebbe chiedere perché stiamo
    digitando ``python manage.py shell`` invece del solo ``python``. Entrambi i
    comandi avviano l'interprete interattivo, ma il comando ``manage.py shell``
    ha una differenza fondamentale: prima di avviare l'interprete, vengono
    passate le impostazioni di Django. Molte parti di Django, tra cui il sistema
    di template, si basano sulle impostazioni, e non saremmo in grado di usarli
    a meno che il contesto sa quali impostazioni da utilizzare.

    Se sei curioso, ecco come funziona dietro le quinte. Django cerca una
    variabile d'ambiente chiamata ``DJANGO_SETTINGS_MODULE``, che dovrebbe
    essere impostato sul percorso di importazione del vostro ``settings.py``.
    Ad esempio, ``DJANGO_SETTINGS_MODULE`` potrebbe essere impostato su
    ``'mysite.settings'``, supponendo che ``mysite`` sia il tuo percorso di
    Python.

    Quando si esegue ``python manage.py shell``, il comando si prende cura di
    interpretare ``DJANGO_SETTINGS_MODULE`` per noi. Incoraggiamo l'utilizzo del
    ``python manage.py shell`` in questi esempi per ridurre al minimo la
    quantità di lavoro e di configurazione che bisogna fare.

Passiamo in rassegna alcuni principi fondamentali del sistema di template::

    >>> from django.template import Template
    >>> t = Template('My name is {{ name }}.')
    >>> print t

Se stai seguendo la shell integrativa, vedrai qualcosa di simile a questo:

    <django.template.Template object at 0xb7d5f24c>

Quel ``0xb7d5f24c`` sarà diverso ogni volta, e non è rilevante: è una cosa Python
(l'"identità" dell'oggetto ``Template`` in Python, se proprio vuoi saperlo).

Quando si crea un oggetto ``Template``, il sistema compila per prima il codice
del template in una forma ottimizzata interna, pronta per il rendering. Ma se
il tuo codice di template include eventuali errori di sintassi, viene chiamata
la funzione ``Template()`` che causerà un'eccezione `TemplateSyntaxError``::

    >>> from django.template import Template
    >>> t = Template('{% notatag %}')
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
      ...
    django.template.TemplateSyntaxError: Invalid block tag: 'notatag'

Il termine "block tag" qui si riferisce al ``{% notatag %}``. "Block tag" e
"template tag" sono sinonimi.

Il sistema genera un'eccezione ``TemplateSyntaxError`` per uno qualsiasi dei
seguenti casi:

* Tag non validi
* Argomenti non validi per tag validi
* Filtri non validi
* Argomenti non validi per filtri validi
* Sintassi del template non valida
* Tag non chiusi (per i tag che richiedono tag di chiusura)

Il rendering di un template
---------------------------

Una volta che si dispone di un oggetto ``Template``, è possibile passare dati e dare
un *contesto* (context). Un contesto è semplicemente un insieme di nomi di
variabili di template ed i relativi valori associati. Un modello utilizza questo
meccanismo per popolare le sue variabili e valutare i casi.

Un contesto è rappresentata in Django dalla classe ``Context``, che sta nel
modulo ``django.template``. Il suo costruttore accetta un argomento opzionale:
un dizionario, che mappa i nomi a dei valori. Chiamare il metodo ``render()``
dell'oggetto ``Template`` con il contesto per "riempire" il template::

    >>> from django.template import Context, Template
    >>> t = Template('My name is {{ name }}.')
    >>> c = Context({'name': 'Stephane'})
    >>> t.render(c)
    u'My name is Stephane.'

Una cosa che dobbiamo sottolineare è che il valore di ritorno di ``t.render(c)``
è un oggetto Unicode -- non una normale stringa Python. Si può dire questo per
via della ``u`` davanti alla stringa. Django utilizza oggetti Unicode invece di
stringhe normali in tutta la struttura. Se si già capito cosa significa questo,
sarai grato per le cose raffinate che Django fa dietro le quinte per farlo
funzionare. Se non hai capito, non preoccuparti, per ora, è sufficiente sapere
che il supporto Unicode di Django rende relativamente indolore per le tue
applicazioni supportare una vasta gamma di set di caratteri al di là del "A-Z"
di base della lingua inglese.

.. admonition:: Dizionari e Contesti

   Un dizionario Python fa dei collegamenti tra chiavi note e valori variabili.
   Un ``Context`` è simile ad un dizionario, ma un ``Context`` fornisce delle
   funzionalità aggiuntive, di cui ci occuperemo nell capitolo 9.

I nomi delle variabili devono iniziare con una lettera (A-Z o a-z) e possono
contenere altre lettere, numeri, caratteri di sottolineatura e punti. (I punti
sono un caso particolare ci arriveremo in un attimo)(i nomi delle variabili sono
case sensitive).

Ecco un esempio di compilazione e di rendering di un contesto, utilizzando un
template simile a quello dell'esempio visto all'inizio di questo capitolo::

    >>> from django.template import Template, Context
    >>> raw_template = """<p>Dear {{ person_name }},</p>
    ...
    ... <p>Thanks for placing an order from {{ company }}. It's scheduled to
    ... ship on {{ ship_date|date:"F j, Y" }}.</p>
    ...
    ... {% if ordered_warranty %}
    ... <p>Your warranty information will be included in the packaging.</p>
    ... {% else %}
    ... <p>You didn't order a warranty, so you're on your own when
    ... the products inevitably stop working.</p>
    ... {% endif %}
    ...
    ... <p>Sincerely,<br />{{ company }}</p>"""
    >>> t = Template(raw_template)
    >>> import datetime
    >>> c = Context({'person_name': 'John Smith',
    ...     'company': 'Outdoor Equipment',
    ...     'ship_date': datetime.date(2009, 4, 2),
    ...     'ordered_warranty': False})
    >>> t.render(c)
    u"<p>Dear John Smith,</p>\n\n<p>Thanks for placing an order from Outdoor
    Equipment. It's scheduled to\nship on April 2, 2009.</p>\n\n\n<p>You
    didn't order a warranty, so you're on your own when\nthe products
    inevitably stop working.</p>\n\n\n<p>Sincerely,<br />Outdoor Equipment
    </p>"

Analizziamo il codice un'istruzione alla volta:

* In primo luogo, abbiamo importato le classi ``Template`` e ``Context``, che
  stanno nel modulo ``django.template``.

* Salviamo il testo del nostro template nella variabile ``raw_template``. Si
  noti che usiamo segni di triple virgolette per indicare stringhe che si
  espandono su più righe. Al contrario, le stringhe racchiuse tra virgolette
  singole non possono essere scritte su più righe.

* Quindi, creiamo un oggetto template, ``t``, passando ``raw_template`` al
  costruttore della classe ``Template``.

* Importiamo il modulo ``datetime`` dalla libreria standard di Python, perché ne
  avremo bisogno nella seguente dichiarazione.

* Quindi, creiamo un oggetto ``Context``, ``c``. Il costruttore ``Context``
  prende un dizionario Python, che mappa i nomi delle variabili di valori. Qui,
  per esempio, si precisa che ``person_name`` è ``'John Smith'``, l'azienda è
  ``'Outdoor Equipment'``, e così via.

* Infine, chiamiamo il metodo ``render()`` sul nostro oggetto template,
  passandogli il contesto. Questo restituisce il modello renderizzato -- vale a
  dire, che sostituisce le variabili del template con i valori attuali delle
  variabili, e viene rieseguito per ogni tag dei template.

  Nota che il paragrafo "You didn't order a warranty" è stato mostrato perché la
  variabile ``ordered_warranty`` è risultata ``False``. Da notare anche la data,
  ``April 2, 2009``, che viene mostrata in base alla stringa di formato
  ``'F j, Y'``. (Parleremo delle stringhe di formato per i filtri data fra un po ')

  Se sei nuovo in Python, potresti chiederti perché questo output include
  caratteri speciali come l'accapo (``'\n'``) anziché effettivamente vedere
  l'accapo. Questo avviene per una sottigliezza nell'interprete interattivo di
  Python: la chiamata a ``t.render(c)`` restituisce una stringa, e per
  impostazione predefinita, l'interprete interattivo mostra la rappresentazione
  della stringa, piuttosto che il valore stampato della stringa. Se vuoi vedere
  la stringa con interruzioni di riga visualizzati come veri a capo, piuttosto
  che caratteri ``'\n'``, bisogna utilizzare l'istruzione di stampa ``print``
  ``print t.render(c)``.

Queste sono le nozioni fondamentali sull'utilizzo del sistema di template di
Django: basta scrivere una stringa di template, creare un oggetto ``Template``,
creare un ``Context`` e chiamare il metodo ``render()``.

Più contesti, stesso modello
----------------------------

Una volta che si ha un oggetto ``Template``, è possibile rendere più contesti
attraverso di esso. Per esempio::

    >>> from django.template import Template, Context
    >>> t = Template('Hello, {{ name }}')
    >>> print t.render(Context({'name': 'John'}))
    Hello, John
    >>> print t.render(Context({'name': 'Julie'}))
    Hello, Julie
    >>> print t.render(Context({'name': 'Pat'}))
    Hello, Pat

Ogni volta che si utilizza lo stesso oggetto template per rendere più contesti
come questo, è più efficiente creare l'oggetto ``Template`` *una volta*, e
quindi chiama ``render()`` su di esso più volte::

    # Cattivo
    for name in ('John', 'Julie', 'Pat'):
        t = Template('Hello, {{ name }}')
        print t.render(Context({'name': name}))

    # Buono
    t = Template('Hello, {{ name }}')
    for name in ('John', 'Julie', 'Pat'):
        print t.render(Context({'name': name}))

L'analisi del template di Django è abbastanza veloce. Dietro le quinte, la
maggior parte del parsing avviene tramite una chiamata ad una singola
espressione regolare. Questo è in netto contrasto con i motori di template
basati su XML, che non sopportano un parser XML e tendono ad essere più lenti di
ordini di grandezza rispetto al motore di rendering di template incluso in Django.

Contesto variabile Ricerca
--------------------------

Negli esempi visti finora, abbiamo passato valori semplici nei contesti -- per
lo più stringhe, oltre a un esempio ``datetime.date``. Tuttavia, il sistema di
template gestisce con eleganza strutture di dati più complesse, come ad esempio
liste, dizionari e oggetti personalizzati.

La chiave per navigare strutture dati complesse nei template di Django è il
carattere punto (``.``). Utilizza un punto per accedere alle chiavi di un
dizionario, gli attributi, i metodi o agli indici di un oggetto.

Questo si spiega meglio con alcuni esempi. Per esempio, supponiamo di stare
passando un dizionario Python per un template. Per accedere ai valori chiave di
quel dizionario bisogna usare un punto::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'Sally is 43 years old.'

Allo stesso modo, i punti permettono anche l'accesso agli attributi dell'oggetto.
Ad esempio, un oggetto Python ``datetime.date`` ha ``year``, ``month`` e ``day``
e degli attributi, ed è possibile utilizzare un punto per accedere a questi
attributi in un template di Django::

    >>> from django.template import Template, Context
    >>> import datetime
    >>> d = datetime.date(1993, 5, 2)
    >>> d.year
    1993
    >>> d.month
    5
    >>> d.day
    2
    >>> t = Template('The month is {{ date.month }} and the year is {{ date.year }}.')
    >>> c = Context({'date': d})
    >>> t.render(c)
    u'The month is 5 and the year is 1993.'

Questo esempio utilizza una classe personalizzata e mostra che le variabili
consentono inoltre l'accesso ad ogni attributo su oggetti arbitrari::

    >>> from django.template import Template, Context
    >>> class Person(object):
    ...     def __init__(self, first_name, last_name):
    ...         self.first_name, self.last_name = first_name, last_name
    >>> t = Template('Hello, {{ person.first_name }} {{ person.last_name }}.')
    >>> c = Context({'person': Person('John', 'Smith')})
    >>> t.render(c)
    u'Hello, John Smith.'

I punti possono fare riferimento anche ai *metodi* degli oggetti. Ad esempio,
ogni stringa di Python ha i metodi ``upper()`` e ``isdigit()``, che si possono
chiamare nei template usando la stessa sintassi del punto::

    >>> from django.template import Template, Context
    >>> t = Template('{{ var }} -- {{ var.upper }} -- {{ var.isdigit }}')
    >>> t.render(Context({'var': 'hello'}))
    u'hello -- HELLO -- False'
    >>> t.render(Context({'var': '123'}))
    u'123 -- 123 -- True'

Da notare che *non* si devono includere le parentesi nelle chiamate del metodo.
Inoltre, non è possibile passare argomenti ai metodi, si possono solo chiamare
metodi che non hanno argomenti richiesti. (Spiegheremo questa filosofia più
avanti in questo capitolo).

Infine, i punti sono anche utilizzati per accedere agli indici delle liste, ad esempio::

    >>> from django.template import Template, Context
    >>> t = Template('Item 2 is {{ items.2 }}.')
    >>> c = Context({'items': ['apples', 'bananas', 'carrots']})
    >>> t.render(c)
    u'Item 2 is carrots.'

Indici negativi nelle liste non sono ammessi. Ad esempio, la variabile
``{{ items.-1 }}`` causerebbe un ``TemplateSyntaxError``.

.. admonition:: Liste Python

   Ricorda: le liste Python hanno indici che partono da 0. Il primo elemento è
   nella posizione di indice 0, il secondo è a indice 1, e così via.

Le ricerche con il punto possono essere riassunti in questo modo: quando il
sistema di template incontra un punto in un nome di variabile, svolge le
seguenti ricerche, in questo ordine:

* Dizionario di ricerca (ad esempio, ``foo["bar"]``)
* Attributo di ricerca (ad esempio, ``foo.bar``)
* Metodo di chiamata (ad esempio, ``foo.bar()``)
* Elenco-indice di ricerca (ad esempio, ``foo[2]``)

Il sistema utilizza il primo tipo di ricerca che funziona. E' la logica del
corto circuito.

Le ricerche con il punto possono essere nidificate su più livelli di profondità.
Per esempio, il seguente esempio utilizza ``{{ person.name.upper }}``,, che si
traduce in una ricerca nel dizionario (``person['name']``) e poi una chiamata di
metodo (``upper()``)::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name.upper }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'SALLY is 43 years old.'

Comportamento delle chiamate ad un metodo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Le chiamate ad un metodo sono leggermente più complesse rispetto agli altri tipi
di ricerca. Qui ci sono alcune cose da tenere a mente:

* Se, durante il metodo di ricerca, un metodo solleva un'eccezione, l'eccezione
  viene propagata, a meno che l'eccezione ha un attributo ``silent_variable_failure``
  il cui valore è ``True``. Se l'eccezione ha un attributo di questo tipo, la
  variabile sarà resa come una stringa vuota, per esempio::

        >>> t = Template("My name is {{ person.first_name }}.")
        >>> class PersonClass3:
        ...     def first_name(self):
        ...         raise AssertionError, "foo"
        >>> p = PersonClass3()
        >>> t.render(Context({"person": p}))
        Traceback (most recent call last):
        ...
        AssertionError: foo

        >>> class SilentAssertionError(AssertionError):
        ...     silent_variable_failure = True
        >>> class PersonClass4:
        ...     def first_name(self):
        ...         raise SilentAssertionError
        >>> p = PersonClass4()
        >>> t.render(Context({"person": p}))
        u'My name is .'

* Una chiamata al metodo funziona solo se il metodo non ha argomenti richiesti.
  In caso contrario, il sistema si sposta al prossimo tipo di ricerca (ricerca
  elenco-indice).

* Ovviamente, alcuni metodi hanno effetti collaterali, e sarebbe sciocco, e
  forse anche un buco di sicurezza, consentire al sistema di template di
  accedervi.

  Supponiamo, ad esempio, che si dispone di un oggetto BankAccount che ha un
  metodo ``delete()``. Se un modello include qualcosa come ``{{ account.delete }}``,
  dove ``account`` è un oggetto ``BankAccount``, l'oggetto viene eliminato
  quando sul template viene eseguito il rendering!

  Per evitare questo, impostare sul metodo  l'attributo ``alters_data``::

      def delete(self):
          # Delete the account
      delete.alters_data = True

  Il template di sistema non esegue alcun metodo contrassegnato in questo modo.
  Continuando l'esempio precedente, se un modello include ``{{ account.delete }}``
  e il metodo ``delete()`` è la ``alters_data=True``, il metodo ``delete()`` non
  viene eseguito quando il template viene eseguito il rendering. Invece, fallirà
  silenziosamente.

Come vengono gestite le variabili non valide
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Per impostazione predefinita, se una variabile non esiste, il sistema di
template la traduce in una stringa vuota, in mancanza di niente. Per esempio::

    >>> from django.template import Template, Context
    >>> t = Template('Your name is {{ name }}.')
    >>> t.render(Context())
    u'Your name is .'
    >>> t.render(Context({'var': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'NAME': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'Name': 'hello'}))
    u'Your name is .'

Il sistema non sta silenzio, ma solleva un'eccezione perché è destinato
persistere un errore umano. In questo caso, tutte le ricerche falliscono perchè
i nomi delle variabili sono il caso o il nome sbagliato. Nel mondo reale, è
inaccettabile per un sito Web di diventare inaccessibile a causa di un piccolo
errore di sintassi nel template.

Giocare con l'oggetto Context
-----------------------------

Per la maggior parte del tempo, avrai a che fare con le istanze di oggetti
``Context`` usando un dizionario completamente popolato da ``Context()``. Ma è
possibile aggiungere ed eliminare elementi da un oggetto ``Context`` anche una
volta che è stata creata l'istanza utilizzando la sintassi dizionario standard
di Python::

    >>> from django.template import Context
    >>> c = Context({"foo": "bar"})
    >>> c['foo']
    'bar'
    >>> del c['foo']
    >>> c['foo']
    Traceback (most recent call last):
      ...
    KeyError: 'foo'
    >>> c['newvariable'] = 'hello'
    >>> c['newvariable']
    'hello'

Template, Etichette e Filtri
============================

Come abbiamo già detto, è possibile usare diversi tag e filtri integrati nel
sistema di template. Le sezioni che seguono, forniscono una panoramica dei tag
e dei filtri più comuni.

Tag
---

if/else
~~~~~~~

Il tag ``{% if %}`` valuta una condizione o una variabile, e se quella risulta
"True" (cioè, esiste, non è vuoto, e non è un valore booleano false), il sistema
visualizza tutto tra ``{% if %}`` e ``{% endif %}``, ad esempio::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% endif %}

Un tag ``{% else %}`` è opzionale::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% else %}
        <p>Get back to work.</p>
    {% endif %}

.. admonition:: Python "Truthiness"

   In Python e nel sistema di template Django, questi oggetti restituiscono
   ``False`` in un contesto booleano:

   * Una lista vuota (``[]``)
   * Una tupla vuota (``()``)
   * Un dizionario vuoto (``{}``)
   * Una stringa vuota (``''``)
   * Zero (``0``)
   * L'oggetto speciale ``None``
   * L'oggetto ``False`` (ovviamente)
   * Oggetti personalizzati che definiscono il proprio comportamento contesto
     booleano (uso avanzato Python)

   Tutto il resto restituisce ``True``.

Il tag ``{% if %}`` accetta ``and``, ``or``, or ``not`` per lavorare su più
variabili, o per negare le variabili date. Per esempio::

    {% if athlete_list and coach_list %}
        Both athletes and coaches are available.
    {% endif %}

    {% if not athlete_list %}
        There are no athletes.
    {% endif %}

    {% if athlete_list or coach_list %}
        There are some athletes or some coaches.
    {% endif %}

    {% if not athlete_list or coach_list %}
        There are no athletes or there are some coaches.
    {% endif %}

    {% if athlete_list and not coach_list %}
        There are some athletes and absolutely no coaches.
    {% endif %}

I tag ``{% if %}`` non permettono clausole ``and`` e ``or`` all'interno della
stessa istruzione, poiché essa potrebbe avere una logica ambigua. Per esempio,
questa istruzione è invalida::

    {% if athlete_list and coach_list or cheerleader_list %}

L'uso delle parentesi per dare uno specifico ordine alle operazioni di controllo
non è supportato. Se ti trovi a dover necessariamente usare le parentesi,
considera di ricreare l'esecuzione di logica al di fuori del template passando
il risultato come una variabile del template. Oppure, basta usare un tag
``{% if %}`` nidificato, in questo modo::

    {% if athlete_list %}
        {% if coach_list or cheerleader_list %}
            We have athletes, and either coaches or cheerleaders!
        {% endif %}
    {% endif %}

Molteplici usi di uno stesso operatore logico vanno bene, ma non è possibile
combinare diversi operatori. Per esempio, questo è valido::

    {% if athlete_list or coach_list or parent_list or teacher_list %}


Non vi è alcun tag ``{% elif %}``. Basta usare un tag if nidificato ``{% if %}``
per realizzare la stessa cosa::

    {% if athlete_list %}
        <p>Here are the athletes: {{ athlete_list }}.</p>
    {% else %}
        <p>No athletes are available.</p>
        {% if coach_list %}
            <p>Here are the coaches: {{ coach_list }}.</p>
        {% endif %}
    {% endif %}

Assicurarsi di chiudere ogni ``{% if %}`` con un ``{% endif %}``. In caso
contrario, Django solleva un ``TemplateSyntaxError``.

for
~~~

Il tag ``{% for %}``  consente di fare delle iterazioni su ogni elemento di una
sequenza. Come in Python per la dichiarazione, la sintassi è ``for X in Y``,
dove ``Y`` è la sequenza di un ciclo su e ``X`` è il nome della variabile da
utilizzare per un particolare ciclo. Ogni iterazione del ciclo, il sistema di
template renderà tutto ciò presente fra ``{% for %}`` e ``{% endfor %}``.

Ad esempio, per visualizzare un elenco di atleti, come la variabile
``athlete_list`` è possibile utilizzare la seguente istruzione::

    <ul>
    {% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {% endfor %}
    </ul>

Aggiungendo ``reversed`` al tag, è possibile ciclare in maniera inversa la lista::

    {% for athlete in athlete_list reversed %}
    ...
    {% endfor %}

E' possibile creare tag ``{% for %}`` annidati in questo modo::

    {% for athlete in athlete_list %}
        <h1>{{ athlete.name }}</h1>
        <ul>
        {% for sport in athlete.sports_played %}
            <li>{{ sport }}</li>
        {% endfor %}
        </ul>
    {% endfor %}

Un uso comune è controllare la dimensione della lista prima di eseguire il loop,
e poi stampare un testo speciale se la lista è vuota::

    {% if athlete_list %}
        {% for athlete in athlete_list %}
            <p>{{ athlete.name }}</p>
        {% endfor %}
    {% else %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endif %}

Poiché questo modello è così comune, il tag ``for`` supporta una clausola
``{% empty %}`` opzionale che permette di definire cosa scrivere se la lista è
vuota. Questo esempio è equivalente alla precedente::

    {% for athlete in athlete_list %}
        <p>{{ athlete.name }}</p>
    {% empty %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endfor %}

Non vi è alcun modo per "rompere" un ciclo prima che esso sia finito. Se vuoi
farlo, è necessario modificare la variabile che stai iterando in modo da
includere solo i valori su cui si desidera lavorare. Allo stesso modo, non vi è
alcuna istruzione "continue" che fa ricominciare immediatamente il ciclo.
(Leggi la sezione "Filosofia e limitazioni", più avanti in questo capitolo,
che spiega la motivazione dietro questa decisione di progettazione).

All'interno di ogni ciclo ``{% for %}``, si ottiene l'accesso ad una variabile
chiamata ``forloop``. Questa variabile ha un paio di caratteristiche che ti
danno informazioni sullo stato di avanzamento del ciclo::

* ``forloop.counter`` è sempre impostato su un numero intero che rappresenta il
  numero di volte che il ciclo è stato eseguito. Questo è una sorta di indice,
  quindi la prima volta che avviene il ciclo, il ``forloop.counter`` viene
  impostato ad ``1``. Ecco un esempio::

      {% for item in todo_list %}
          <p>{{ forloop.counter }}: {{ item }}</p>
      {% endfor %}

* ``forloop.counter0`` è come ``forloop.counter``, tranne per il fatto che parte
  zero. Il suo valore sarà impostato a 0 la prima volta che viene eseguito il
  ciclo.

* ``forloop.revcounter`` è sempre impostato su un numero intero che rappresenta
  il numero di elementi rimanenti nel ciclo. La prima esecuzione del ciclo,
  ``forloop.revcounter`` viene impostato con il numero totale di elementi nel
  ciclo che stai eseguendo. All'ultima iterazione del ciclo,
  ``forloop.revcounter`` viene impostato su ``1``.

* ``forloop.revcounter0`` è come ``forloop.revcounter``, tranne per il fatto che
  parte zero. La prima iterazione del ciclo, ``forloop.revcounter0`` viene
  impostato con il numero di elementi del ciclo che stai eseguendo meno 1.
  L'ultima iterazione del ciclo, è impostata a ``0``.


* ``forloop.first`` è un valore booleano impostato su ``True`` se questa è la
  prima iterazione. Questo è comodo per creare caso particolari::

      {% for object in objects %}
          {% if forloop.first %}<li class="first">{% else %}<li>{% endif %}
          {{ object }}
          </li>
      {% endfor %}

* ``forloop.last`` è un valore booleano impostato su true se questa è l'ultima
  iterazione. Un uso comune per questo è di mettere i caratteri pipe tra una
  lista di link::

      {% for link in links %}{{ link }}{% if not forloop.last %} | {% endif %}{% endfor %}

  Il codice del modello precedente potrebbe produrre qualcosa di simile a
  questo::

      Link1 | Link2 | Link3 | Link4

  Un altro uso comune di questa variabile è usarla per mettere una virgola tra
  le parole in un elenco luoghi preferiti::

      {% for p in places %}{{ p }}{% if not forloop.last %}, {% endif %}{% endfor %}


* ``forloop.parentloop`` è un riferimento all'oggetto ``forloop`` del ciclo
  *genitore*, nel caso di cicli annidati. Ecco un esempio::

      {% for country in countries %}
          <table>
          {% for city in country.city_list %}
              <tr>
              <td>Country #{{ forloop.parentloop.counter }}</td>
              <td>City #{{ forloop.counter }}</td>
              <td>{{ city }}</td>
              </tr>
          {% endfor %}
          </table>
      {% endfor %}

La magica variabile ``forloop`` è disponibile solo all'interno dei cicli. Dopo
che il parser del template ha raggiunto ``{% endfor %}``, ``forloop`` scompare.

.. admonition:: Contesto/Context e variabile forloop

   All'interno del blocco ``{% for %}`` le variabili esistenti vengono spostate
   al di fuori per evitare di sovrascrivere la variabile ``forloop``. Django
   mette questo contesto in ``forloop.parentloop``. In genere non è necessario
   preoccuparsi di questo fatto, ma se si crea una variabile all'interno del
   template chiamata ``forloop`` (anche se si sconsiglia di farlo), essa viene
   mantenuta al di fuori del ciclo in o chiamati i ``forloop.parentloop`` mentre
   all'interno del ``{% for %}`` per blocco.

ifequal/ifnotequal
~~~~~~~~~~~~~~~~~~

Il sistema di template Django volutamente non è un linguaggio di programmazione
a tutti gli effetti e, quindi, non consente di eseguire comandi Python. (Leggi
"Filosofia e limitazioni" per saperne su più questa idea), tuttavia, è piuttosto
comune dover confrontare due valori e mostrare a qualcosa se sono uguali in un
template -- e Django fornisce un tag ``{% ifequal %}`` per questo scopo.

Il tag ``{% ifequal %}`` confronta due valori e mostra tutto quello che c'è tra
``{% ifequal %}`` e ``{% endifequal %}`` se i valori sono uguali.

Questo esempio confronta il template variabili ``user`` e ``currentuser``::

    {% ifequal user currentuser %}
        <h1>Welcome!</h1>
    {% endifequal %}

Gli argomenti possono essere stringhe hardcoded, con le virgolette singole o
doppie, per cui vale quanto segue::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% endifequal %}

    {% ifequal section "community" %}
        <h1>Community</h1>
    {% endifequal %}

Proprio come ``{% if %}``, il ``{% ifequal %}`` tag supporta un opzionale
``{% else %}``::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% else %}
        <h1>No News Here</h1>
    {% endifequal %}

Solo le variabili di template, quindi stringhe, numeri interi e numeri decimali,
sono consentite come argomenti del tag ``{% ifequal %}``. Questi sono esempi
validi::

    {% ifequal variable 1 %}
    {% ifequal variable 1.23 %}
    {% ifequal variable 'foo' %}
    {% ifequal variable "foo" %}

Eventuali altri tipi di variabili, come dizionari Python, elenchi o booleani,
non possono essere a livello di codice in ``{% ifequal %}``. Questi sono esempi
validi::

    {% ifequal variable True %}
    {% ifequal variable [1, 2, 3] %}
    {% ifequal variable {'key': 'value'} %}

Se hai bisogno di verificare se qualcosa è vero o falso, utilizzare il tag
``{% if %}`` invece di ``{% ifequal %}``.

Commenti
~~~~~~~~

Proprio come in HTML o Python, il linguaggio di template Django permette i
commenti. Per designare un commento, si usa ``{# #}``::

    {# This is a comment #}

Il commento non sarà considerato quando il template viene renderizzato.

Uutilizzando questa sintassi, i commenti non possono estendersi su più righe.
Questa limitazione migliora le prestazioni del parse del template. Nel seguente
template, l'output del rendering sarà esattamente lo stesso del modello (ad
esempio, il tag di commento non viene analizzato come un commento)::

    This is a {# this is not
    a comment #}
    test.

Se si desidera utilizzare i commenti su più righe, è necessario utilizzare il
tag ``{% comment %}``, in questo modo::

    {% comment %}
    This is a
    multi-line comment.
    {% endcomment %}

Filtri
------

Come spiegato in precedenza in questo capitolo, i filtri dei template sono
semplici modi di alterare il valore delle variabili prima di essere visualizzati.
I filtri utilizzano un carattere pipe, come questo::

    {{ name|lower }}

Visualizza il valore della ``{{ name }}`` variabile, dopo essere stata filtrata
attraverso il filtro ``lower``, che converte il testo in lettere minuscole.

I filtri possono essere *concatenati* -- cioè, essi possono essere utilizzati in
combinazione in modo che il risultato di un filtro venga applicato al successivo.
Ecco un esempio che prende il primo elemento in una lista e lo converte in
maiuscolo::

    {{ my_list|first|upper }}

Alcuni filtri richiedono degli argomenti. Un argomento viene passato al filtro
usando i due punti ed è si mette sempre tra virgolette. Per esempio::

    {{ bio|truncatewords:"30" }}

Questo esempio visualizza le prime 30 parole della variabile ``bio``.

I seguenti sono alcuni dei filtri più importanti. Leggi l'Appendice E per
scoprire il resto.

* ``addslashes``: Aggiunge una barra rovesciata prima di ogni backslash,
  apostrofo, o virgolette. Questo è utile se il testo prodotto è incluso in una
  stringa JavaScript.

* ``date``: Formatta un oggetto ``date`` o ``datetime`` in base a una stringa di
  formato passata come parametro, ad esempio::

      {{ pub_date|date:"F j, Y" }}

  Le stringhe di formato sono definite nell'Appendice E.

* ``length``: Restituisce la lunghezza del valore. Per un elenco, restituisce il
  numero di elementi. Per una stringa, restituisce il numero di caratteri.
  (Esperti di Python, prendiate atto che questo funziona su qualsiasi oggetto
  Python che sa come determinare la sua lunghezza - ovvero, ogni oggetto che ha
  un metodo ``__len__()``).

Filosofia e limitazioni
=======================

Ora che hai toccato con mano il linguaggio di template di Django, segnaliamo
alcuni dei suoi limiti intenzionali, insieme ad alcune filosofie che stanno
dietro al perché funziona nel modo in cui funziona.

Più di ogni altra componente delle applicazioni Web, la sintassi del template è
molto soggettiva, e le opinioni dei programmatori variano selvaggiamente. Il
fatto che solo Python abbia decine, se non centinaia, di implementazioni open
source di linguaggi di template rafforza questo punto. Ciascuno è stato
probabilmente creato perché il suo sviluppatore ha ritenuto tutte le lingue di
template esistenti inadeguati. (In realtà, si dice che sia un rito di passaggio
per ogni sviluppatore Python scrivere il proprio linguaggio di template! Se non
lo hai ancora fatto, considera di farlo. Si tratta di un esercizio divertente).

Tenendo questo a mente, si potrebbe essere interessati a sapere che Django non
richiede di utilizzare il linguaggio di template. Perché Django è destinato ad
essere un framework web "full-stack" (ovvero completo ed intercambiabile) che
mette a disposizione tutti i pezzi necessari agli sviluppatori web per essere
produttivi, molte volte *è più comodo* da usare sistema di template di Django di
altre librerie di modelli di Python, ma non è un requisito rigoroso. Come si
vedrà nel paragrafo "Uso dei template nelle View", è molto semplice usare un
altro linguaggio di template con Django.

Eppure, è chiaro che abbiamo una forte preferenza per il modo in cui la lingua
di template di Django funziona. Il template di sistema ha radici nel modo in cui
lo sviluppo web è stato fatto al World Online e all'esperienza combinata dei
creatori di Django. Ecco alcune di quelle filosofie:

* *Business logic should be separated from presentation logic*. Django's
  developers see a  template system as a tool that controls presentation and
  presentation-related logic -- and that's it. The template system shouldn't
  support functionality that goes beyond this basic goal.

* *La logica di business deve essere separata dalla logica di presentazione*.
  Gli sviluppatori di Django vedono il template di sistema come uno strumento
  che controlla la presentazione e la logica della presentazione correlata -- e
  questo è tutto. Il template di sistema non dovrebbe supportare funzionalità
  che vanno al di là di questo obiettivo di base.

  Per questo motivo, non è possibile chiamare direttamente codice Python
  all'interno dei modelli di Django. La "programmazione" è fondamentalmente
  limitata a ciò che i tag dei template possono fare. *E' possibile* scrivere
  dei tag per i template personalizzati che fanno cose arbitrarie, ma i tag
  "out-of-the-box" (ovvero predefiniti) dei template di Django volutamente non
  consentono l'esecuzione di codice Python arbitrario.

* *La sintassi dovrebbe essere disaccoppiato da HTML/XML*. Sebbene il sistema di
  template di Django viene utilizzato principalmente per la produzione di codice
  HTML, ed è destinato ad essere altrettanto utile per i formati non HTML, come
  il testo normale. Alcune altre lingue template sono basate su XML, mettendo
  tutta la logica all'interno dei tag dei template o degli attributi XML, ma
  Django evita deliberatamente questa limitazione. Essi richiedono XML valido
  per scrivere i template ed introduce quindi un mondo di errori umani da capire
  con duri messaggi di errore, ed inoltre l'utilizzo di un motore XML per
  analizzare i template comporta un livello inaccettabile a prescindere dal come
  è fatto il template.

* *Si assume che i designer siano a loro agio con il codice HTML*. Il template
  di sistema non è stato progettato in modo che siano visualizzati bene su
  editor WYSIWYG come Dreamweaver. Questo è una limitazione troppo severa e non
  avrebbe permesso alla sintassi di essere amichevole come è adesso. Django si
  aspetta autori di template che siano a loro agio direttamente con l'HTML.

* *Si assume che i designer non siano programmatori Python*. Gli autori sanno
  che i template per le pagine web sono spesso scritti da designer, non da
  programmatori, e quindi non devono conoscere Python.

  Tuttavia, il sistema intende inoltre ospitare piccole parti in cui i template
  siano creati da programmatori Python. Offre quindi un modo per estendere la
  sintassi del sistema di scrittura di codice Python. (Maggiori informazioni su
  questo nel capitolo 9).

* *L'obiettivo non è quello di inventare un linguaggio di programmazione*.
  L'obiettivo è di offrire solo le funzionalità di programmazione semplice, come
  salti e cicli, che sono essenziali per prendere decisioni relative alla
  presentazione di una informazione.

Utilizzo dei Template nelle View
========================

Hai imparato le nozioni di base sull'uso del sistema di template, ora usiamo
questa conoscenza per creare una vista. Ricordiamo la vista ``current_datetime``
in ``mysite.views``, che abbiamo iniziato nel capitolo precedente. Ecco come si
presenta::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Cambiamo questa visualizzazione per utilizzare il sistema di template di Django.
In un primo momento, si potrebbe pensare di fare qualcosa di simile a questo::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = Template("<html><body>It is now {{ current_date }}.</body></html>")
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Certo, utilizza il sistema di template, ma non risolve i problemi che abbiamo
sottolineato nell'introduzione di questo capitolo. Vale a dire, il template è
ancora incorporato nel codice Python, così la vera separazione fra dati e
presentazione non è raggiunta. Cerchiamo di rimediare spostando il template in
un *file separato*, che questa view carica.

Si potrebbe invece considerare di salvare prima il template da qualche parte sul
tuo filesystem e poi utilizzare le funzionalità di apertura file integrata in
Python per leggere il contenuto del template. Ecco come sembra il tutto,
prendendo per vero il fatto che il template sia stato salvato in questo modo
``/home/djangouser/templates/mytemplate.html``::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        # Simple way of using templates from the filesystem.
        # This is BAD because it doesn't account for missing files!
        fp = open('/home/djangouser/templates/mytemplate.html')
        t = Template(fp.read())
        fp.close()
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Tuttavia, questo approccio è poco elegante per questi motivi:

* Non gestisce il caso di un file mancante. Se il file ``mytemplate.html`` non
  esiste o non è leggibile, la chiamata ``open()`` genera un'eccezione
  ``IOError``.

* Il percorso del template potrebbe essere lungo. Se si sceglie di usare questa
  tecnica per ogni view, si dovrebbero duplicare le posizioni dei template. Per
  non parlare del fatto che comporta un bel po' di battitura!

* Comprende un sacco di codice standard noioso. Hai di meglio da fare che
  scrivere chiamate a ``open()``, ``fp.read()``, e ``fp.close()`` ogni volta che
  si carica un modello.

Per risolvere questi problemi, useremo quindi il *Template Loading* (caricamento
dei Template) e le *directory dei template*.

Template Loading
================

Django fornisce un'API comoda e potente per il caricamento dei template dal
filesystem, con l'obiettivo di eliminare la ridondanza sia nelle chiamate ai
template che nei template stessi.

Per poter utilizzare le API del Template Loading, in primo luogo è necessario
dire al framework dove si memorizzano i template. Il luogo per far ciò è nel
file di impostazioni -- il file ``settings.py`` che abbiamo menzionato
nell'ultimo capitolo, quando abbiamo introdotto l'impostazione ``ROOT_URLCONF``.

Continuando così, aprire il ``settings.py`` e trovare l'impostazione
``TEMPLATE_DIRS``. Per impostazione predefinita, è una tupla vuota,
probabilmente contenente alcuni commenti generati automaticamente::

    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
    )

Questa impostazione indica al meccanismo di caricamento di Django dove cercare i
modelli. Scegli una directory in cui vuoi memorizzare i template e aggiungilo a
``TEMPLATE_DIRS``, in questo modo::

    TEMPLATE_DIRS = (
        '/home/django/mysite/templates',
    )

Ci sono alcune cose da notare:

* È possibile specificare qualsiasi directory che si desidera, a condizione che
  la directory ed i template all'interno di tale directory siano leggibili
  dall'account utente in cui il server web viene eseguito. Se non hai idea del
  nome da dare ad una directory per i template, si consiglia di creare una
  cartella ``templates`` all'interno del progetto (cioè, all'interno della
  directory ``mysite`` creata nel capitolo 2).

* Se il tuo ``TEMPLATE_DIRS`` contiene solo una directory, non dimenticare
  comunque la virgola alla fine della stringa di directory!

  Male::

      # Virgola Mancante!
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates'
      )

  Bene::

      # Virgola correttamente inserita.
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates',
      )

  La ragione di questo è che Python richiede virgole all'interno di tuple in un
  singolo elemento per rendere non ambiguo il fatto che sia una tupla. Questo è
  un comune ai Principianti.

* Se sei in Windows, includere la lettera dell'unità e utilizzare le barre in
  stile Unix piuttosto che gli backslash, come segue::

      TEMPLATE_DIRS = (
          'C:/www/django/templates',
      )

* E' semplice usare percorsi assoluti (cioè percorsi di directory che iniziano
  alla radice del filesystem). Ma se si vuole essere un po' più flessibili e
  disaccoppiati, si può approfittare del fatto che i file delle impostazioni
  Django sono solo codice Python costruendo il contenuto del ``TEMPLATE_DIRS``
  dinamicamente. Per esempio::

      import os.path

      TEMPLATE_DIRS = (
          os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
      )

  Questo esempio utilizza la "magia" della variabile ``__file__`` di Python, che
  viene impostata automaticamente sul nome del modulo in esecuzione. Si prende
  il nome della directory che contiene ``settings.py`` (``os.path.dirname``), lo
  si unisce a quella dei modelli indipendentemente dal sistema in uso
  (``os.path.join``) e ci si assicura che tutto utilizza le barre invece che il
  backslash (nel caso di Windows).

  Parlando di codice dinamico nel file di impostazione, bisogna dire che è molto
  importante evitare errori Python nel file di impostazioni. Se si introduce un
  errore di sintassi, o di un errore di runtime, il tuo sito Django-powered
  probabilmente lo si bloccherà.

Con ``TEMPLATE_DIRS``, il passo successivo è quello di modificare il codice al
fine di utilizzare la funzionalità di Template loading inclusa in Django,
piuttosto che a livello di codice. Tornando alla nostra view ``current_datetime``,
cambiamo il tutto in questo modo::

    from django.template.loader import get_template
    from django.template import Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = get_template('current_datetime.html')
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

In this example, we're using the function
``django.template.loader.get_template()`` rather than loading the template from
the filesystem manually. The ``get_template()`` function takes a template name
as its argument, figures out where the template lives on the filesystem, opens
that file, and returns a compiled ``Template`` object.

Our template in this example is ``current_datetime.html``, but there's nothing
special about that ``.html`` extension. You can give your templates whatever
extension makes sense for your application, or you can leave off extensions
entirely.

To determine the location of the template on your filesystem,
``get_template()`` combines your template directories from ``TEMPLATE_DIRS``
with the template name that you pass to ``get_template()``. For example, if
your ``TEMPLATE_DIRS`` is set to ``'/home/django/mysite/templates'``, the above
``get_template()`` call would look for the template
``/home/django/mysite/templates/current_datetime.html``.

If ``get_template()`` cannot find the template with the given name, it raises
a ``TemplateDoesNotExist`` exception. To see what that looks like, fire up the
Django development server again by running ``python manage.py runserver``
within your Django project's directory. Then, point your browser at the page
that activates the ``current_datetime`` view (e.g.,
``http://127.0.0.1:8000/time/``). Assuming your ``DEBUG`` setting is set to
``True`` and you haven't yet created a ``current_datetime.html`` template, you
should see a Django error page highlighting the ``TemplateDoesNotExist`` error.

.. figure:: graphics/chapter04/missing_template.png
   :alt: Screenshot of a "TemplateDoesNotExist" error.

   Figure 4-1: The error page shown when a template cannot be found.

This error page is similar to the one we explained in Chapter 3, with one
additional piece of debugging information: a "Template-loader postmortem"
section. This section tells you which templates Django tried to load, along with
the reason each attempt failed (e.g., "File does not exist"). This information
is invaluable when you're trying to debug template-loading errors.

Moving along, create the ``current_datetime.html`` file within your template
directory using the following template code::

    <html><body>It is now {{ current_date }}.</body></html>

Refresh the page in your Web browser, and you should see the fully rendered
page.

render()
--------

We've shown you how to load a template, fill a ``Context`` and return an
``HttpResponse`` object with the result of the rendered template. We've
optimized it to use ``get_template()`` instead of hard-coding templates and
template paths. But it still requires a fair amount of typing to do those
things. Because this is such a common idiom, Django provides a shortcut that
lets you load a template, render it and return an ``HttpResponse`` -- all in
one line of code.

This shortcut is a function called ``render()``, which lives in the
module ``django.shortcuts``. Most of the time, you'll be using
``render()`` rather than loading templates and creating ``Context``
and ``HttpResponse`` objects manually -- unless your employer judges your work
by total lines of code written, that is.

Here's the ongoing ``current_datetime`` example rewritten to use
``render()``::

    from django.shortcuts import render
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        return render(request, 'current_datetime.html', {'current_date': now})

What a difference! Let's step through the code changes:

* We no longer have to import ``get_template``, ``Template``, ``Context``,
  or ``HttpResponse``. Instead, we import
  ``django.shortcuts.render``. The ``import datetime`` remains.

* Within the ``current_datetime`` function, we still calculate ``now``, but
  the template loading, context creation, template rendering, and
  ``HttpResponse`` creation are all taken care of by the
  ``render()`` call. Because ``render()`` returns
  an ``HttpResponse`` object, we can simply ``return`` that value in the
  view.

The first argument to ``render()`` is the request, the second is the name of
the template to use. The third argument, if given, should be a dictionary to
use in creating a ``Context`` for that template. If you don't provide a third
argument, ``render()`` will use an empty dictionary.

Subdirectories in get_template()
--------------------------------

It can get unwieldy to store all of your templates in a single directory. You
might like to store templates in subdirectories of your template directory, and
that's fine. In fact, we recommend doing so; some more advanced Django
features (such as the generic views system, which we cover in
Chapter 11) expect this template layout as a default convention.

Storing templates in subdirectories of your template directory is easy.
In your calls to ``get_template()``, just include
the subdirectory name and a slash before the template name, like so::

    t = get_template('dateapp/current_datetime.html')

Because ``render()`` is a small wrapper around ``get_template()``,
you can do the same thing with the second argument to ``render()``,
like this::

    return render(request, 'dateapp/current_datetime.html', {'current_date': now})

There's no limit to the depth of your subdirectory tree. Feel free to use
as many subdirectories as you like.

.. note::

    Windows users, be sure to use forward slashes rather than backslashes.
    ``get_template()`` assumes a Unix-style file name designation.

The ``include`` Template Tag
----------------------------

Now that we've covered the template-loading mechanism, we can introduce a
built-in template tag that takes advantage of it: ``{% include %}``. This tag
allows you to include the contents of another template. The argument to the tag
should be the name of the template to include, and the template name can be
either a variable or a hard-coded (quoted) string, in either single or double
quotes. Anytime you have the same code in multiple templates,
consider using an ``{% include %}`` to remove the duplication.

These two examples include the contents of the template ``nav.html``. The
examples are equivalent and illustrate that either single or double quotes
are allowed::

    {% include 'nav.html' %}
    {% include "nav.html" %}

This example includes the contents of the template ``includes/nav.html``::

    {% include 'includes/nav.html' %}

This example includes the contents of the template whose name is contained in
the variable ``template_name``::

    {% include template_name %}

As in ``get_template()``, the file name of the template is determined by adding
the template directory from ``TEMPLATE_DIRS`` to the requested template name.

Included templates are evaluated with the context of the template
that's including them. For example, consider these two templates::

    # mypage.html

    <html>
    <body>
    {% include "includes/nav.html" %}
    <h1>{{ title }}</h1>
    </body>
    </html>

    # includes/nav.html

    <div id="nav">
        You are in: {{ current_section }}
    </div>

If you render ``mypage.html`` with a context containing ``current_section``,
then the variable will be available in the "included" template, as you would
expect.

If, in an ``{% include %}`` tag, a template with the given name isn't found,
Django will do one of two things:

* If ``DEBUG`` is set to ``True``, you'll see the
  ``TemplateDoesNotExist`` exception on a Django error page.

* If ``DEBUG`` is set to ``False``, the tag will fail
  silently, displaying nothing in the place of the tag.

Template Inheritance
====================

Our template examples so far have been tiny HTML snippets, but in the real
world, you'll be using Django's template system to create entire HTML pages.
This leads to a common Web development problem: across a Web site, how does
one reduce the duplication and redundancy of common page areas, such as
sitewide navigation?

A classic way of solving this problem is to use *server-side includes*,
directives you can embed within your HTML pages to "include" one Web page
inside another. Indeed, Django supports that approach, with the
``{% include %}`` template tag just described. But the preferred way of
solving this problem with Django is to use a more elegant strategy called
*template inheritance*.

In essence, template inheritance lets you build a base "skeleton" template that
contains all the common parts of your site and defines "blocks" that child
templates can override.

Let's see an example of this by creating a more complete template for our
``current_datetime`` view, by editing the ``current_datetime.html`` file::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>The current time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>It is now {{ current_date }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

That looks just fine, but what happens when we want to create a template for
another view -- say, the ``hours_ahead`` view from Chapter 3? If we want again
to make a nice, valid, full HTML template, we'd create something like::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>Future time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

Clearly, we've just duplicated a lot of HTML. Imagine if we had a more
typical site, including a navigation bar, a few style sheets, perhaps some
JavaScript -- we'd end up putting all sorts of redundant HTML into each
template.

The server-side include solution to this problem is to factor out the
common bits in both templates and save them in separate template snippets,
which are then included in each template. Perhaps you'd store the top
bit of the template in a file called ``header.html``::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>

And perhaps you'd store the bottom bit in a file called ``footer.html``::

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

With an include-based strategy, headers and footers are easy. It's the
middle ground that's messy. In this example, both pages feature a title --
``<h1>My helpful timestamp site</h1>`` -- but that title can't fit into
``header.html`` because the ``<title>`` on both pages is different. If we
included the ``<h1>`` in the header, we'd have to include the ``<title>``,
which wouldn't allow us to customize it per page. See where this is going?

Django's template inheritance system solves these problems. You can think of it
as an "inside-out" version of server-side includes. Instead of defining the
snippets that are *common*, you define the snippets that are *different*.

The first step is to define a *base template* -- a skeleton of your page that
*child templates* will later fill in. Here's a base template for our ongoing
example::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        {% block content %}{% endblock %}
        {% block footer %}
        <hr>
        <p>Thanks for visiting my site.</p>
        {% endblock %}
    </body>
    </html>

This template, which we'll call ``base.html``, defines a simple HTML skeleton
document that we'll use for all the pages on the site. It's the job of child
templates to override, or add to, or leave alone the contents of the blocks.
(If you're following along, save this file to your template directory as
``base.html``.)

We're using a template tag here that you haven't seen before: the
``{% block %}`` tag. All the ``{% block %}`` tags do is tell the template
engine that a child template may override those portions of the template.

Now that we have this base template, we can modify our existing
``current_datetime.html`` template to use it::

    {% extends "base.html" %}

    {% block title %}The current time{% endblock %}

    {% block content %}
    <p>It is now {{ current_date }}.</p>
    {% endblock %}

While we're at it, let's create a template for the ``hours_ahead`` view from
Chapter 3. (If you're following along with code, we'll leave it up to you to
change ``hours_ahead`` to use the template system instead of hard-coded HTML.)
Here's what that could look like::

    {% extends "base.html" %}

    {% block title %}Future time{% endblock %}

    {% block content %}
    <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>
    {% endblock %}

Isn't this beautiful? Each template contains only the code that's *unique* to
that template. No redundancy needed. If you need to make a site-wide design
change, just make the change to ``base.html``, and all of the other templates
will immediately reflect the change.

Here's how it works. When you load the template ``current_datetime.html``,
the template engine sees the ``{% extends %}`` tag, noting that
this template is a child template. The engine immediately loads the
parent template -- in this case, ``base.html``.

At that point, the template engine notices the three ``{% block %}`` tags
in ``base.html`` and replaces those blocks with the contents of the child
template. So, the title we've defined in ``{% block title %}`` will be
used, as will the ``{% block content %}``.

Note that since the child template doesn't define the ``footer`` block,
the template system uses the value from the parent template instead.
Content within a ``{% block %}`` tag in a parent template is always
used as a fallback.

Inheritance doesn't affect the template context. In other words, any template
in the inheritance tree will have access to every one of your template
variables from the context.

You can use as many levels of inheritance as needed. One common way of using
inheritance is the following three-level approach:

1. Create a ``base.html`` template that holds the main look and feel of
   your site. This is the stuff that rarely, if ever, changes.

2. Create a ``base_SECTION.html`` template for each "section" of your site
   (e.g., ``base_photos.html`` and ``base_forum.html``). These templates
   extend ``base.html`` and include section-specific styles/design.

3. Create individual templates for each type of page, such as a forum page
   or a photo gallery. These templates extend the appropriate section
   template.

This approach maximizes code reuse and makes it easy to add items to shared
areas, such as section-wide navigation.

Here are some guidelines for working with template inheritance:

* If you use ``{% extends %}`` in a template, it must be the first
  template tag in that template. Otherwise, template inheritance won't
  work.

* Generally, the more ``{% block %}`` tags in your base templates, the
  better. Remember, child templates don't have to define all parent blocks,
  so you can fill in reasonable defaults in a number of blocks, and then
  define only the ones you need in the child templates. It's better to have
  more hooks than fewer hooks.

* If you find yourself duplicating code in a number of templates, it
  probably means you should move that code to a ``{% block %}`` in a
  parent template.

* If you need to get the content of the block from the parent template,
  use ``{{ block.super }}``, which is a "magic" variable providing the
  rendered text of the parent template. This is useful if you want to add
  to the contents of a parent block instead of completely overriding it.

* You may not define multiple ``{% block %}`` tags with the same name in
  the same template. This limitation exists because a block tag works in
  "both" directions. That is, a block tag doesn't just provide a hole to
  fill, it also defines the content that fills the hole in the *parent*.
  If there were two similarly named ``{% block %}`` tags in a template,
  that template's parent wouldn't know which one of the blocks' content to
  use.

* The template name you pass to ``{% extends %}`` is loaded using the same
  method that ``get_template()`` uses. That is, the template name is
  appended to your ``TEMPLATE_DIRS`` setting.

* In most cases, the argument to ``{% extends %}`` will be a string, but it
  can also be a variable, if you don't know the name of the parent template
  until runtime. This lets you do some cool, dynamic stuff.

What's next?
============

You now have the basics of Django's template system under your belt. What's next?

Many modern Web sites are *database-driven*: the content of the Web site is
stored in a relational database. This allows a clean separation of data and logic
(in the same way views and templates allow the separation of logic and display.)

The next chapter `Chapter 5`_ covers the tools Django gives you to
interact with a database.

.. _Chapter 5: chapter05.html
