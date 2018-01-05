# Cenni di programmazione

## Il programma

Si definisce "programma" qualsiasi sequenza di istruzioni scritte in linguaggio
macchina (l'unico linguaggio comprensibile ad un calcolatore, le famose sequenze
di 0 e 1) atta ad essere elaborata da un calcolatore o comunque da una struttura
informatica.

Ogni volta che usiamo un calcolatore facciamo uso di programmi. Word e Outlook
sono programmi, così come Vim o Emacs. Windows o Linux stessi non sono altro che
programmi ("software di base"). Anche i virus sono dei programmi eseguibili.

Si pone qui il problema di come scrivere un programma. Per questa esigenza si fa
ricorso ai *linguaggi di programmazione*.

## Breve storia della programmazione

### I linguaggi a basso livello

In principio, quando i computer erano enormi mobili grandi quanto palazzine, era
il programmatore stesso a gestire il calcolatore attraverso istruzioni binarie,
nei primordi dell'informatica accendendo o spegnendo fisicamente valvole o
attaccando e staccando cavi. Programmare attraverso sequenze binarie è però
qualcosa di completamente innaturale per l'uomo.

Passiamo alla metà degli anni 50: i programmatori creano un sistema più "comodo"
per creare i loro progetti, l'Assembly. L'Assembly (o GLI Assembly, dato che la
sintassi di questo linguaggio cambia in funzione di molte variabili, quali il
sistema operativo in uso, l'architettura della macchina che si va a programmare
e l'assemblatore in uso) non è altro che una rappresentazione simbolica del
linguaggio macchina, dove ad ogni istruzione binaria corrisponde un'istruzione
mnemonica, relativamente più semplice da ricordare. Così la sequenza `1100 1101
0010 0001`, o `CD 21` (linguaggio macchina) diventa int `21h` (sintassi Assembly
Intel).

### I linguaggi a medio/alto livello

L'Assembly tuttavia è ancora qualcosa di molto primitivo. E' pur sempre un
linguaggio orientato alla macchina, non al problema, e, col passare degli anni i
progetti si fanno via via più grandi, e l'Assembly da solo non può gestirli.
Ecco quindi che compaiono sulla scena i primi linguaggi ad "alto livello", ossia
linguaggi più orientati al problema (e quindi all'utente) che all'architettura
intrinseca della macchina. Sono il COBOL, il BASIC e il FORTRAN, linguaggi molto
più semplici dell'Assembly ma non altrettanto potenti.

Anche questi linguaggi hanno le loro pecche: il COBOL (_COmmon Business-Oriented
Language_) è un linguaggio orientato principalmente al business e alla logica
per la gestione dei dati (molti sistemi informativi di banche o grosse aziende
sono stti sviluppati decenni fa in COBOL e sono ancora funzionanti e vegeti), ma
ha delle regole sintattiche troppo rigide per poter essere un linguaggio
_all-purpose_.

Il FORTRAN (_FORmula TRANslation_) è ottimo per la progettazione di applicazioni
a carattere scientifico, è ancora oggi usato in molti contesti scientifici o
ingegneristici (anche se piattaforme più moderne come _Matlab_ gli hanno eroso
molti contesti di utilizzo), ma non per la progettazione di codice di sistema.

Il BASIC (_Beginner's All-purpose Symbolic Instruction Code_), nonostante la sua
incredibile facilità di apprendimento, non è potente, non è un linguaggio
strutturato, e oggi ha scenari di impiego molto relativi.

Nasce quindi il PASCAL, un linguaggio ad alto livello dotato di una struttura e
di istruzioni vere e proprie per il controllo del flusso del programma, ma non
progettato per un vasto campo di azione, quindi poco efficiente per la scrittura
di codice di sistema. Al giorno d'oggi il PASCAL è usato perlopiù per scopi
didattici, spesso come primo approccio fra gli studenti e il mondo della
programmazione, grazie alla sua semplicità di apprendimento e alla sua sintassi
estremamente pulita.

### Il C

Arriviamo all'inizio degli anni 70, l'hardware diventa sempre più potente e la
richiesta di software cresce giorno dopo giorno, ma non esiste ancora un
linguaggio ad alto livello che soddisfi qualsiasi richiesta di software. Fino al
1972, "l'ora zero" del linguaggio C: in un laboratorio della AT&T Bell Dennis
Ritchie fa girare un primo prototipo del C su un DEC PDP-11 con sistema
operativo UNIX. Il C fu il risultato dello sviluppo di due linguaggi di
programmazione più vecchi: il B (sviluppato da Ken Thompson) e il BCPL
(sviluppato da Martin Richards). Per anni il linguaggio C fu strettamente legato
al sistema operativo UNIX (infatti, dopo la progettazione del C, tutte le
successive versioni di UNIX furono scritte in questo linguaggio, e ancora oggi i
sistemi Unix-based, fra cui Linux e *BSD, sono scritti in C). Nel 1989, alla
luce dei vari "stili" del C formatisi, l'ANSI (American National Standards
Institute) mise a punto l'ANSI C, una versione standard del C priva di ambiguità
che è ancora oggi il riferimento per la stragrande maggioranza dei compilatori.

La novità del C, ed anche il motivo di tutto il suo successo, è che il C è un
linguaggio di programmazione sviluppato dai programmatori stessi, e non da
un'istituzione governativa o da un'università, per i programmatori stessi:
questo rende il C il _linguaggio dei programmatori_. Unita a questa
caratteristica, c'è la versalità del C: un linguaggio usato tanto per semplici
programmini didattici, tanto per sviluppare codice di sistema o software per
sistemi embedded: è un linguaggio che si presta ad un'infinità di usi, grazie
anche ad una libreria davvero vastissima. Il C infatti, a differenza degli altri
linguaggi di programmazione, ha pochissime keyword (parole riservate), ma un
vasto insieme di funzioni presenti nella libreria standard (dalla gestione
dell'I/O alle funzioni matematiche, dalla manipolazione dei file alla gestione
della memoria) e un gran numero di librerie esterne sviluppate dai programmatori
e facilmente integrabili nei propri progetti (dagli strumenti per la creazione
di interfacce grafiche (GUI), alla gestione delle espressioni regolari, alla
programmazione di rete, e così via).

È un linguaggio ad alto livello che consente però l'interazione a livello molto
più basso con la macchina. È infatti possibile dal C allocare e deallocare
direttamente aree di memoria (questa è vista da molti come un sinonimo di
potenza del linguaggio, ma come è noto da un grande potere derivano grandi
responsabilità), accedere direttamente a indirizzi di I/O in modo da leggere e
scrivere dati su una periferica (questo rende il C il linguaggio privilegiato
per la scrittura dei driver), e perfino inserire nel proprio codice spezzoni di
codice _Assembly_ (in una sintassi detta inline Assembly). Per questi motivi
molti preferiscono definire il C un linguaggio a _medio livello_.

### L'evoluzione ad oggetti del C - il C++

Del 1982 è invece il C++ che, a differenza di quello che pensano in molti, non è
un nuovo linguaggio in senso stretto, ma un'estensione, un'evoluzione del C. Un
codice scritto in C verrà infatti compilato senza problemi anche da un
compilatore C++ (a meno che non ci siano, come vedremo in seguito nel corso
della guida, costrutti sintattici che vengono considerati come _warning_ in C ed
errori in C++). Tuttavia, il C++ mette a disposizione del programmatore
meccanismi di programmazione _orientata agli oggetti_, (OOP), che non è
strettamente qualcosa che si "aggiunge" al programma ma un modo diverso di
concepire l'applicazione. Gli oggetti sono creati dal programmatore e gestiti
dal linguaggio come nuovi tipi di dati astratto, contenenti un insieme di
attributi e su cui è possibile operare attraverso un insieme di metodi (ad
esempio un'automobile può essere modellata in un linguaggio come il C++ come un
oggetto caratterizzato dagli attributi _targa_, _cilindrata_, _velocità
massima_, _litri di carburante per km_, _colore_, _numero posti_, e su cui
possono operare i metodi _accendi_, _spegni_, _accelera_, _decelera_, _frena_,
_ripara_, _sterza_, ecc.). Gli oggetti sono rappresentati dalle classi, che sono
la loro rappresentazione astratta, ed ogni classe può ereditare oggetti da altre
classi o cambiare la loro visibilità all'interno del programma (esistono oggetti
privati, protetti e pubblici). La potenza della OOP permette al programmatore di
fare cose davvero interessanti, come ridefinire gli operatori, fare
l'overloading di funzioni, creare oggetti il cui tipo viene deciso solo al
momento dell'uso (_template_), gestire le eccezioni in modo potente (con i
blocchi try e catch), e così via.  Occorre tuttavia procedere per passi. Sempre
più corsi di studio, a qualsiasi livello, hanno adottato la filosofia di
insegnare ai propri studenti direttamente la programmazione attraverso il
paradigma a oggetti, in C++, Java, C# ecc., ma tale scelta forma programmatori
che non sono in grado di apprezzare le differenze fra il paradigma
procedurale/imperativo classico (quello del C) e quello a oggetti, e quindi non
possono comprendere i motivi che hanno spinto a quest'evoluzione, e dov'è che il
paradigma procedurale è da preferire a uno fortemente a oggetti. Studiare prima
il C consente invece di acquisire la flessibilità necessaria per entrare meglio
in quest'ottica, oltre a imparare il seguito il C++ senza apprendere tutto da
zero ma semplicemente concentrandosi sulle differenze strutturali, sintattiche e
di paradigma dei due linguaggi.  La programmazione oggi Nel 1991 fu concepito il
Java che sintatticamente è fortemente influenzato dal C/C++. Il Java si pone
come obiettivi quello di essere un linguaggio di programmazione fortemente a
oggetti (in Java si può dire che tutto viene trattato come un oggetto),
fortemente multipiattaforma (write once, run everywhere è uno dei motti dei
programmatori Java), e queste caratteristiche hanno decretato il suo successo
nel mondo informatico sempre più eterogeneo degli ultimi anni. Lo stesso
programma che scrivo in Java lo posso eseguire teoricamente senza problemi (a
patto di effettuare le opportune modifiche nel caso in cui voglio portare
un'applicazione sviluppata per un sistema fisso su una piattaforma mobile) su
Windows, su un sistema Unix, su un telefonino o smartphone, a patto che esista
una JVM (Java Virtual Machine) per quel sistema.  Pur essendo un linguaggio dal
punto di vista sintattico fortemente legato al C il suo gradino di apprendimento
è considerato meno ripido, dato che nasconde al programmatore molti costrutti di
basso livello che il programmatore C deve gestire esplicitamente (allocazione e
deallocazione dinamica della memoria, programmazione di rete o multiprocesso
relativamente complesse). Proprio per la sua ottica molto ad alto livello, con
un livello di astrazione e di trasparenza rispetto alla macchina molto alti,
Java è diventato un linguaggio estremamente utilizzato nella maggior parte dei
contesti accademici e lavorativi, ma come sempre a ogni scelta corrispondono dei
risvolti non positivi (essendo un linguaggio estremamente di alto livello Java
non consente al programmatore di effettuare operazioni di basso livello
possibili in C, come la gestione diretta delle locazioni di memoria, o
l'input-output diretto, o la scrittura di codice in kernel space, in quanto
tutte le applicazioni Java operano nella sandbox della macchina virtuale, e
inoltre essendoci per tutte le applicazioni una macchina virtuale di mezzo che
traduce a runtime il codce pseudo-compilato in codice eseguibile le prestazioni
di un programma scritto in Java sono generalmente minori di quelle di un
programma equivalente sviluppato in C o C++, in quanto il passaggio in più
dell'interpretazione a runtime costituisce un overhead temporale inevitabile).
Andando a linguaggi sempre più di alto livello troviamo Perl, Python, PHP e
Ruby, tutti linguaggi di scripting interpretati, scarsamente strutturati ed
estremamente semplici da apprendere.  Microsoft ha inoltre messo a punto negli
ultimi anni un framework (.NET) fortemente ispirato alla filosofia di virtual
machine di Java, con cui è possibile interagire attraverso diversi linguaggi
(C#, esplicitamente influenzato dalla sintassi di Java, J#, Visual
Basic.NET...).
