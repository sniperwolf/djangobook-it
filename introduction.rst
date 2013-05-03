============
Introduzione
============

Agli albori, gli sviluppatori web scrivevano ogni pagina a mano. L'aggiornamento
di un sito Web significava modificare l'HTML, un "restyling" significava ricreare
ogni singola pagina, uno alla volta.

Non appena i siti web sono cresciuti e più ambiziosi, fu subito chiaro che
quella situazione era noiosa, che richiedeva tempo, e, in ultima analisi, era
insostenibile. Un gruppo di intraprendenti hacker alla NCSA (National Center for
Supercomputing Applications, dove Mosaic, il primo browser grafico, è stato
sviluppato) hanno risolto questo problema facendo in modo che il server web
potessero eseguire programmi esterni che generano dinamicamente HTML. Hanno
chiamato questo protocollo Common Gateway Interface, o CGI, ed il Web è cambiato
per sempre.

E' difficile ora immaginare la portata innovativa di CGI: invece di trattare le
pagine HTML come file semplici sul disco, CGI fa pensare alle tue pagine come
risorse generate dinamicamente su richiesta. Lo sviluppo di CGI ha inaugurato la
prima generazione di siti Web dinamici.

Tuttavia, CGI ha i suoi problemi: gli script CGI devono contenere un sacco di
codice ripetitivo e "stereotipato", rendendo difficile il riuso del codice, e
può essere difficile da capire per gli sviluppatori che si approcciano per le
prime volte.

PHP ha risolto molti di questi problemi, ed ha assalto il mondo -- è ora di
gran lunga lo strumento più utilizzato per creare siti Web dinamici, e decine di
linguaggi e ambienti simili (ASP, JSP, ecc) hanno seguito la progettazione di
PHP da vicino. La grande innovazione di PHP è data dalla sua facilità d'uso:
il codice PHP è semplicemente incorporato nel semplice HTML e la curva di
apprendimento per chi già conosce HTML è estremamente rapida.

Ma PHP ha i suoi problemi, la sua stessa facilità d'uso incoraggia la scrittura
di codice ripetitivo e mal concepito. Peggio ancora, PHP non fa molto per
proteggere i programmatori da vulnerabilità di sicurezza, e, quindi, molti
sviluppatori PHP si sono trovati ad imparare concetti relativi alla sicurezza
una volta che fosse troppo tardi.

Queste e simili frustrazioni hanno portato direttamente allo nascita della
"terza generazione" dello sviluppo web, con i framework. Questi framework --
Django e Ruby on Rails sembrano essere i più popolari in questi giorni --
riconoscono che l'importanza del web si è intensificata negli ultimi tempi. Con
questa nuova esplosione del web, lo sviluppo è più ambizioso; gli sviluppatori
web devono essere in grado di fare di più ogni giorno che passa.

Django è stato inventato per rispondere a queste nuove ambizioni. Django
consente di creare siti dinamici, profondi ed interessanti in un tempo
estremamente breve. Django è progettato per farti concentrare sulle parti
interessanti e divertenti del tuo lavoro, mentre allevia il dolore quando hai a
che fare con roba ripetitiva. Per farlo, fornisce astrazioni ad alto livello di
modelli di sviluppo web comuni, scorciatoie per le attività di programmazione
frequenti e chiare convenzioni su come risolvere i problemi. Allo stesso tempo,
Django cerca di stare lontano dalla tua strada, consentendo di lavorare al di
fuori del campo di applicazione del framework, se necessario. Abbiamo scritto
questo libro perché crediamo fermamente che Django rende lo sviluppo web
migliore. E' progettato per farti lavorare velocemente su un tuo progetto Django,
e, in ultima analisi, ti insegnerà tutto ciò che c'è da sapere per progettare,
sviluppare e distribuire con successo un sito di cui sarete orgogliosi.

Siamo molto interessati a vostro feedback. Questo libro è `open source`__
(`tradotto in Italiano`__ da Fabrizio Fallico) e tutti sono invitati a
migliorarlo. Se si preferisce suggerire modifiche, scrivi due righe all'indirizzo
email feedback@djangobook.com (se hai consigli per la versione italiana, manda
una email a djangobook@sparkblog.org). In entrambi i casi, ci piacerebbe sentire
cosa hai da dire! Siamo felici che tu sia qui, e ci auguriamo che troverai
Django emozionante, divertente e utile come facciamo noi.

__ http://github.com/jacobian/djangobook.com
__ http://github.com/sniperwolf/djangobook.com