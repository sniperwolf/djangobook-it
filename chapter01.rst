=================================
Capitolo 1: Introduzione a Django
=================================

Questo libro parla di Django, un framework di sviluppo web che consente di
risparmiare tempo e rende lo sviluppo web una gioia. Usando Django, è possibile
costruire e mantenere applicazioni web di alta qualità con il minimo sforzo.

Se utilizzato bene, rende lo Sviluppo web un eccitante atto creativo;
se usato male, può essere un ripetitivo, fastidio frustrante. Django permette
di concentrarsi sulle cose divertenti -- il punto cruciale della propria
applicazione web -- mentre allevia il dolore quando hai a che fare con i bit
ripetitivi. Per far questo, fornisce astrazioni ad alto livello di modelli di
sviluppo web comuni, scorciatoie per le attività di programmazione frequenti e
convenzioni chiare per risolvere i problemi. Allo stesso tempo, Django cerca di
stare fuori dalla vostra strada, consentendo comunque di lavorare al di fuori
del campo di applicazione, se ciò si rende necessario.

L'obiettivo di questo libro è quello di fare di te un esperto di Django.
L'obiettivo è duplice. In primo luogo, si spiega, in profondità, cosa fa e come
usare Django per creare applicazioni web. In secondo luogo, si discute concetti
di livello superiore, se del caso, rispondendo a domande del tipo "Come posso
applicare questi strumenti in modo efficace nei miei progetti?" Con la lettura
di questo libro, imparerete le competenze necessarie per sviluppare siti web
potenti rapidamente, con codice che è pulito e facile da mantenere.

Che cos'è un framework web?
============================

Django è un membro di spicco di una nuova generazione di *web framework* -- ma
che cosa significa questo termine, esattamente?

Per rispondere a questa domanda, consideriamo la progettazione di
un'applicazione web scritta in Python *senza* un framework. In questo libro,
prenderemo come approccio il far vedere come creare qualcosa nel modo di base,
*senza* scorciatoie, nella speranza che si capisca il perché le scorciatoie sono
così utili (è anche importante sapere come fare le cose senza scorciatoie,
perché esse non sono sempre disponibili. E, soprattutto, sapere *perché* le cose
funzionano ci rende degli sviluppatori migliori).

Uno dei più semplici modi per costruire una applicazione web in Python da zero,
è quello di utilizzare lo standard Common Gateway Interface (CGI), tecnica
popolare intorno al 1998. Ecco una spiegazione a grandi linee su come funziona:
basta creare uno script Python che genera HTML, quindi salvare lo script in un
server web con un trasferimento e visitare la pagina nel browser digitando
"cgi". Tutto qui.

Ecco un esempio di Python script CGI che permette di visualizzare i dieci libri
pubblicati più recentemente in un database. Non preoccuparti dei dettagli della
sintassi, basta avere un'idea di cosa si sta scrivendo::

    #!/usr/bin/env python

    import MySQLdb

    print "Content-Type: text/html\n"
    print "<html><head><title>Books</title></head>"
    print "<body>"
    print "<h1>Books</h1>"
    print "<ul>"

    connection = MySQLdb.connect(user='me', passwd='letmein', db='my_db')
    cursor = connection.cursor()
    cursor.execute("SELECT name FROM books ORDER BY pub_date DESC LIMIT 10")

    for row in cursor.fetchall():
        print "<li>%s</li>" % row[0]

    print "</ul>"
    print "</body></html>"

    connection.close()

In primo luogo, per soddisfare i requisiti di CGI, questo codice stampa la riga
"Content-Type", seguito da una riga vuota. Poi stampa qualche HTML di base, si
connette a un database ed esegue una query per recuperare i nomi degli ultimi
dieci libri. Con un ciclo su quei libri, viene generato un elenco HTML dei
titoli. Infine, viene stampato il codice HTML di chiusura e viene chiusa la
connessione al database.

Con una pagina "una tantum", come questa, il metodo scritta-da-zero non è
necessariamente un male. Per prima cosa, questo codice è semplice da comprendere
-- anche uno sviluppatore alle prime armi è in grado di leggere queste 16 righe
di Python e di capire tutto ciò che fa, dall'inizio alla fine. Non c'è
nient'altro da imparare, nessun altro codice da leggere. E 'anche semplice da
implementare: basta salvare il codice in un file che termina con ".cgi",
caricare il file su un server web e visitare la pagina con un browser.

Ma nonostante la sua semplicità, questo approccio presenta una serie di problemi
e fastidi. Ponetevi queste domande:

* Che cosa accade quando più parti dell'applicazione hanno bisogno di
  connettersi al database? Di certo, il codice relativi al database non dovrebbe
  essere duplicato in ogni singolo script CGI. La cosa pragmatica, sarebbe
  quella di un refactoring in una funzione condivisa.

* Cosa può succedere se uno sviluppatore non si preoccupa davvero della stampa
  della linea di "Content-Type" o dimentica di chiudere la connessione al
  database? Questo tipo di testo standard riduce la produttività dei
  programmatori e introduce l'opportunità di creare errori. Questi setup e
  manipolazioni sarebbe meglio lasciarli gestire ad alcune infrastrutture comuni.

* Che cosa succede quando questo codice viene riutilizzato in più ambienti,
  ognuno con un database diverso e password diverse? A questo punto, qualche
  configurazione specifica dell'ambiente diventa essenziale.

* Cosa succede quando un web designer, che non ha alcuna esperienza di codifica
  Python, vuole ridisegnare la pagina? Un carattere sbagliato potrebbe mandare
  in crash l'intera applicazione. Idealmente, la logica della pagina - il
  recupero di titoli di libri dal database - dovrebbe essere separato dalla
  visualizzazione della pagina HTML, in modo che un progettista possa modificare
  quest'ultimo senza influire sul precedente.

Questi problemi sono proprio ciò che un web framework intende risolvere.
Esso fornisce un'infrastruttura di programmazione per le applicazioni,
in modo da potersi concentrarsi sulla scrittura di codice pulito e mantenibile
senza dover reinventare la ruota. In poche parole, questo è quello che fa
Django.

Il design pattern MVC
=====================

Tuffiamoci in un rapido esempio che dimostra la differenza tra il precedente
approccio e l'approccio di un web framework. Ecco come si potrebbe scrivere il
codice CGI precedente con Django. La prima cosa da notare è che lo abbiamo
diviso in quattro file di Python (``models.py``, ``views.py``, ``urls.py``) e
un modello HTML (``latest_books.html``)::

    # models.py (the database tables)

    from django.db import models

    class Book(models.Model):
        name = models.CharField(max_length=50)
        pub_date = models.DateField()


    # views.py (the business logic)

    from django.shortcuts import render
    from models import Book

    def latest_books(request):
        book_list = Book.objects.order_by('-pub_date')[:10]
        return render(request, 'latest_books.html', {'book_list': book_list})


    # urls.py (the URL configuration)

    from django.conf.urls.defaults import *
    import views

    urlpatterns = patterns('',
        (r'^latest/$', views.latest_books),
    )


    # latest_books.html (the template)

    <html><head><title>Books</title></head>
    <body>
    <h1>Books</h1>
    <ul>
    {% for book in book_list %}
    <li>{{ book.name }}</li>
    {% endfor %}
    </ul>
    </body></html>

Anche in questo caso, non preoccuparti per i particolari della sintassi,
basta avere un'idea del disegno complessivo. La cosa più importante da notare
qui è la *separazione degli argomenti*:

* Il file ``models.py`` contiene una descrizione della tabella nel database,
  rappresentato da una classe Python. Questa classe si chiama "*modello*" o
  "*model*" in inglese. Con esso, è possibile creare, recuperare, aggiornare e
  cancellare i record nel database utilizzando semplice codice Python piuttosto
  che scrivere istruzioni SQL ripetitive.

* Il file ``views.py`` contiene la logica di funzionamento della pagina.
  La funzione ``latest_books()`` è chiamato *vista* o *view* in inglese.

* Il file ``urls.py`` specifica quale vista richiamare per un determinato
  modello di URL. In questo caso, l'URL ``/latest/`` sarà gestita dalla funzione
  ``latest_books()``. In altre parole, se il dominio è example.com, ogni visita
  all'URL ``http://example.com/latest/`` chiamerà la funzione
  ``latest_books()``.

* Il file latest_books.html è un modello HTML che descrive la struttura della
  pagina. Utilizza un linguaggio specifico per i template (template language)
  con istruzioni logiche di base - ad esempio, ``{% for book in book_list %}``.

Messi insieme, tutti questi pezzi seguono vagamente uno schema chiamato
Model-View-Controller (MVC). In poche parole, MVC è il modo di sviluppare
software in modo che il codice per la definizione e l'accesso ai dati
(il modello) è separato dalla logica richiesta-routing (il controller),
che a sua volta è separato dall'interfaccia utente (la vista).

(Discuteremo dell'approccio MVC in modo più approfondito nel capitolo 5)

Uno dei principali vantaggi di questo approccio è che i componenti sono
*debolmente accoppiati*. Ogni pezzo che compone un'applicazione web
Django-alimentata ha un solo ed unico scopo fondamentale e può essere modificato
in maniera indipendente, senza influenzare gli altri pezzi. Ad esempio, uno
sviluppatore può modificare l'URL per una particolare vista o richiesta senza
influenzare l'implementazione sottostante. Un web designer può modificare il
codice HTML di una pagina senza dover toccare il codice Python che lo mostra.
Un amministratore di database può rinominare una tabella di database e
specificare il cambiamento in un unico luogo, piuttosto che dover cercare e
sostituire con una dozzina di file.

In questo libro, ogni componente della MVC ha il proprio capitolo. Il Capitolo 3
riguarda vista, il Capitolo 4 riguarda i modelli ed il Capitolo 5 riguarda i
modelli.

La Storia di Django
===================

Prima di tuffarci sul codice, dovremmo prenderci un momento per raccontare la
storia di Django. Abbiamo già detto che mostreremo come fare le cose *senza*
scorciatoie in modo da capire più a fondo le scorciatoie stesse. Allo stesso
modo, è utile per capire il *perché* Django è stato creato, dato che la
conoscenza della storia metterà in un ben chiaro contesto perché Django funziona
ed il modo in cui funziona.

Se hai mai provato a creare applicazioni web per un po', probabilmente hai
familiarità con i problemi di CGI come mostrato nell'esempio precedente. Il
percorso classico dello sviluppatore web è più o meno il seguente:

1. Scrivere un'applicazione web da zero;
2. Scrivere un'altra applicazione web da zero;
3. Realizzare che l'applicazione della fase 1 condivide del codice in comune con
   l'applicazione della fase 2;
4. Refattorizzare il codice in modo che l'applicazione 1 condivida il codice con
   l'applicazione 2;
5. Ripetere le fasi 2-4 diverse volte;
6. Realizzare che hai creato un framework.

Questo è esattamente come Django stesso è stato creato!

Django è cresciuto organicamente dalle applicazioni reali scritte da un team di
sviluppo web a Lawrence, Kansas, Stati Uniti d'America. Nasce nell'autunno del
2003, quando i web developer del quotidiano *Lawrence Journal-World*, Adrian
Holovaty e Simon Willison, hanno iniziato ad usare Python per creare
applicazioni.

Il team di World Online, responsabile per la produzione e la manutenzione di
alcuni siti di notizie locali, ha prosperato in un ambiente di sviluppo dettato
da scadenze di giornalismo. Per siti di questo genere -- including LJWorld.com,
Lawrence.com e KUsports.com -- i giornalisti (ed i content-manager), richiedono
un qualcosa che abbia la caratteristica di poter aggiungere o rimuovere intere
applicazioni ad un qualcosa di già esistente in maniera estremamente rapida,
spesso con solo giorni o ore di preavviso. Così, Simon e Adrian hanno sviluppato
un framework che facesse risparmiare tempo di sviluppo per necessità -- era
l'unico modo per costruire applicazioni mantenibili sotto certi termini estremi.

Nell'estate del 2005, dopo aver sviluppato questo framework ad un punto in cui è
stato reso estremamente efficiente ed in grado di alimentare la maggior parte
dei siti online del mondo, la squadra, che ora ha incluso Jacob Kaplan-Moss, ha
deciso di rilasciare il framework come software open source. Lo hanno pubblicato
nel luglio 2005 e la chiamarono Django, in onore del chitarrista jazz Django
Reinhardt.

Ora, molti anni dopo, Django è un progetto ormai consolidato, open source, con
decine di migliaia di utenti e collaboratori sparsi in tutto il pianeta.
I due sviluppatori originari (i "dittatori benevoli per la Vita", Adrian e
Jacob) ancora danno una guida centrale per la crescita del framework, ma oggi lo
sviluppo è molto più frutto di un lavoro di squadra collaborativo.

Questa storia è importante perché aiuta a spiegare due cose fondamentali. Il
primo sono i cosiddetti "Sweet spot" di Django, ovvero le parti "carine". Poiché
nato in un ambiente di notizie, offre diverse caratteristiche (come la sua
"zona" admin, trattata nel Capitolo 6) che sono particolarmente adatti per siti
web di "contenuto" -- siti web come Amazon.com, craigslist.org ed il
washingtonpost.com, che offrono dinamicamente le informazioni basandosi su
database. Non lasciare però che questa tendenza metta fuori discussione Django
per il tuo progetto, poiché il fatto che esso sia particolarmente buono per lo
sviluppo di questo genere di siti, non esclude che sia uno strumento efficace
per la costruzione di qualsiasi tipo di sito web dinamico.

(C'è differenza tra l'essere *particolarmente efficace* in qualcosa ed essere
*inefficace* in altre cose)

Il secondo punto da notare è come le origini di Django hanno plasmato la cultura
della sua comunità open source. Proprio perché Django è stato estrapolato dal
codice del mondo reale, piuttosto che essere un esercizio accademico o di un
prodotto commerciale, si rivela acutamente focalizzato sulla soluzione dei
problemi di sviluppo web che il team di Django stesso ha affrontato -- e
continua a subire. Come risultato, Django si migliora attivamente su base quasi
giornaliera. I "mantainer" del framework hanno tutto l'interesse a fare in modo
che Django sia una via per far risparmiare tempo agli sviluppatori, produrre
applicazioni che sono facili da mantenere e che si comportino bene sotto carico.
Se non altro, gli sviluppatori sono motivati dai propri desideri egoistici di
salvare tempi se stessi e di godere del proprio lavoro.

(Per dirla senza mezzi termini, mangiano il loro cibo per cani)

.. AH La seguente sezione è quel tipo di contenuto che solitamente appare nella
.. AH sezione Introduzione di un libro, ma la includiamo qui perché questo
.. AH capitolo funge da introduzione.

Come leggere questo libro
=========================

Nello scrivere questo libro, abbiamo cercato di trovare un equilibrio tra
leggibilità e riferimento, con una tendenza verso la leggibilità. Il nostro
obiettivo con questo libro, come affermato in precedenza, è quello di fare di te
un esperto di Django, e crediamo che il modo migliore per insegnare è attraverso
la prosa e un sacco di esempi, piuttosto che fornire un catalogo esaustivo, ma
insipido con le caratteristiche di Django.

(Come dice il proverbio, non si può pretendere di insegnare a qualcuno come
parlare una lingua semplicemente insegnando loro l'alfabeto)

Con questo mantra in testa, si consiglia di leggere i capitoli dall'1 al 12 in
ordine. Essi formano la base per usare Django, una volta che li hai letti, sarai
in grado di creare e distribuire siti web Django-powered. In particolare, i
capitoli dall'1 al 7 sono il "core curriculum", i capitoli dall'8 all'11 coprono
un uso più avanzato di Django, mentre il capitolo 12 riguarda la distribuzione.
I restanti capitoli, dal 13 al 20, si concentrano su specifiche caratteristiche
di Django e possono essere letti in qualsiasi ordine.

Le appendici sono lì come riferimento. Essi, insieme alla documentazione
gratuita presente sul sito web ufficiale http://www.djangoproject.com/, sono
probabilmente ciò che potrete sfogliare di tanto in tanto per ricordare la
sintassi o trovare rapidamente la sinossi di ciò che fanno alcune parti di
Django.

Conoscenze necessarie di programmazione
---------------------------------------

I lettori di questo libro dovrebbero conoscere le basi della programmazione
procedurale e orientata agli oggetti: strutture di controllo (ad esempio,
``if``, ``while``, ``for``), strutture di dati (liste, dizionari), variabili,
classi e oggetti.

L'esperienza nello sviluppo web è, come ci si potrebbe aspettare, molto utile,
ma non è necessaria per comprendere questo libro. In tutto il libro, cerchiamo
di promuovere le migliori pratiche di sviluppo web per i lettori che non hanno
questa esperienza.

Conoscenze di Python Richieste
------------------------------

Al suo nucleo, Django è semplicemente una raccolta di librerie scritte nel
linguaggio di programmazione Python. Per sviluppare un sito utilizzando Django,
è necessario scrivere codice Python che utilizza queste librerie. Imparare
Django, quindi, è una questione di imparare a programmare in Python e capire
come funzionano le librerie Django.

Se si ha un po' di esperienza con Python, non si dovrebbero avere problemi. Il
codice di Django non ha tanta "magia" (vale a dire, programmazione particolare
la cui implementazione è difficile da spiegare o capire). Per voi, imparare
Django sarà una questione di imparare le convenzioni e le API di Django.

Se non si ha esperienza con Python, state per godere molto. E 'facile da
imparare e una gioia da usare! Sebbene questo libro non includa un completo
tutorial di Python, mette in evidenza le caratteristiche e le funzionalità di
Python, ed in alcuni casi particolari si scrive in esso quando il codice non ha
subito senso. Tuttavia, si consiglia di leggere il tutorial ufficiale di Python,
disponibile online all'indirizzo http://docs.python.org/tut/. Si consiglia anche
la lettura del libro Dive Into Python, di Mark Pilgrim, disponibile
all'indirizzo http://www.diveintopython.net/ e pubblicato da Apress.

(N.d.T.: Dive Into Python è stato tradotto anche in italiano! E' possibile
trovarlo all'indirizzo http://it.diveintopython.net/)

Versione richiesta di Django
----------------------------

Questo libro copre Django 1.4.

Gli sviluppatori di Django mantengono la compatibilità all'indietro il più
possibile, ma di tanto in tanto introducono alcuni cambiamenti incompatibili
con delle vecchie versioni. Le variazioni di ogni release sono sempre coperte
nelle note di rilascio, che potete trovare qui:
https://docs.djangoproject.com/en/dev/releases/1.X

Come ottenere aiuto
-------------------

Uno dei maggiori vantaggi di Django è il suo genere e comunità di utenti
disponibili. Per informazioni di qualsiasi aspetto di Django -
dall'installazione, alla progettazione di applicazioni o dalla progettazione di
database alla distribuzione - non esitate a fare domande online.

* La mailing list django-users è dove migliaia di utenti di Django chiedono e
  rispondono alle domande. Iscriviti gratuitamente alla pagina
  http://www.djangoproject.com/r/django-users.

* Il canale Django IRC è dove gli utenti di Django si danno appuntamento e si
  aiutano su particolari problemi in tempo reale. Per unirsi il divertimento
  basta collegarsi al canale IRC #django su Freenode.

Cosa c'è adesso?
================

Nel `Capitolo 2`_, inizieremo con Django, coprendo la fase di installazione e
delle configurazioni iniziali.

.. _Capitolo 2: chapter02.html