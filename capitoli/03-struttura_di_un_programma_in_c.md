# Struttura di un programma in C e cenni su linguaggi compilati e interpretati

Un programma scritto in C ha una sua struttura particolare. In primis i file
sorgente, ossia i file che contengono il codice C, hanno estensione .c . Ci sono
poi i file header (con estensione .h), che sono i file che contengono i
prototipi per le funzioni e le variabili globali usate nel programma. Tali
funzioni vengono poi implementate nei file sorgenti (quelli con estensione .c).
Vedremo che, per usare funzioni di qualsiasi tipo in un programma C non
contenute esplicitamente nel sorgente corrente, è necessario richiamare il file
header che corrispondente tramite la direttiva #include.

I programmini contenuti in questo tutorial sono relativamente semplici, quindi
possono essere contenuti in un solo file sorgente (con estensione .c).

## Linguaggi compilati e interpretati

Il C è un linguaggio compilato, ovvero una volta scritto il file sorgente (o i
files sorgenti) occorre che questo venga passato al compilatore C assieme al
nome del file in cui si desidera piazzare l'output. Con gcc da riga di comando
la procedura è

    gcc -o file_eseguibile file1.c file2.c ... filen.c

Il compilatore per prima cosa esegue le direttive al preprocessore (quelle che
iniziano con #, come #include #define #if #endif #ifdef... alcune le vedremo nel
corso di questo tutorial). Se non ci sono errori nei sorgenti traduce il codice
C contenuto nei files sorgenti in linguaggio macchina (in quanto è questo
l'unico linguaggio davvero comprensibile al compilatore) In genere questo
processo genera un file oggetto per ogni file sorgente, con estensione .o o
.obj, dove viente piazzato il codice macchina associato a quel sorgente.  Infine
viene eseguita l'operazione di linking, ossia di “fusione” fra i diversi file
oggetto, inserimento dell'eventuale codice importato da librerie esterne, e
creazione dell'eseguibile vero e proprio.

Linguaggi come C, C++, Pascal, COBOL, Assembly ecc. sono linguaggi _compilati_,
ovvero l'output del processo di compilazione, che prende come input uno o più
file sorgenti, è un file eseguibile contenente codice macchina eseguibile
direttamente dalla macchina.

Il QBasic, Perl, Python, Ruby e i linguaggi per la shell come bash, zsh, csh
ecc.  sono invece linguaggi interpretati, ovvero non è possibile creare un file
eseguibile vero e propio con questi linguaggi (pur esistendo compilatori offerti
da terze parti, ad esempio, per il Perl, questi non sono ufficialmente
supportati dagli sviluppatori del linguaggio, che rimane un linguaggio
interpretato). Ogni volta che voglio eseguire un codice scritto in questi
linguaggi devo ricorrere all'interprete corrispondente, ovvero un software che a
tempo di esecuzione (_runtime_) traduca direttamente il codice scritto in quel
linguaggio in codice eseguibile.

La via di mezzo è Java, o i linguaggi della piattaforma .NET di Microsoft. Una
volta scritto un programma ho bisogno di compilarlo (ad esempio, con il comando
javac), e da questo processo ho un file con estensione, nel caso di Java,
.class, scritto in un paragonabile al codice macchina, ma che non è linguaggio
macchina. Questo linguaggio “intermedio” è comprensibile alla _macchina
virtuale_ del linguaggio, il che vuol dire che lo stesso codice intermedio è
comprensibile da una macchina virtuale installata su una macchina Windows, su un
sistema Unix-like, su una piattaforma mobile, e così via. A questo punto posso
eseguire il mio programma attraverso l'interprete java o la piattaforma .NET
stessa (di cui esiste un porting open source per sistemi Unix-like chiamato
_mono_), che esegue il codice contenuto nel file “intermedio” traducendolo in
una sequenza di istruzioni macchina vere e proprie. Questo rende tali tecnologie
facilmente esportabili teoricamente su ogni piattaforma hardware o software per
cui esiste la macchina virtuale.

Entrambi i paradigmi hanno i loro pregi e difetti. Con un linguaggio compilato
posso creare un file eseguibile vero e proprio, totalmente indipendente dal
linguaggio di partenza e senza che sulla macchina di “destinazione” ci sia
bisogno della presenza fisica del compilatore per poter eseguire quel codice, ma
la procedura di precompilazione-compilazione-linkaggio è spesso lenta
(soprattutto quando si tratta di compilare programmi nell'ordine delle migliaia
o milioni di righe di codice, spesso sparse in decine o centinaia di file
sorgenti). Inoltre, il file eseguibile che ho ottenuto dalla compilazione è
pensato e ottimizzato per la piattaforma hardware o software dove l'ho
compilato, non per un'altra. In poche parole, se compilo un file C su Linux, lo
stesso file eseguibile non funzionerà su Windows, e viceversa (a meno che io non
adotti meccanismi di _emulazione_, che ovviamente non sono l'obiettivo di questo
corso).

Un linguaggio interpretato, invece, permette di vedere in real-time se il
programma che si sta scrivendo contiene o no errori, senza a avviare la
procedura di compilazione. Inoltre un listato scritto, ad esempio, in Perl su un
sistema Linux o *BSD funzionerà anche se lo porto su un sistema Windows, a patto
che vi sia installato l'interprete Perl e che non richiami comandi tipici di
quel sistema operativo al suo interno.  Ma questi linguaggi hanno lo svantaggio
di non creare un file eseguibile, ossia di non creare un vero e proprio
programma da eseguire, e di essere dipendenti dalla presenza dell'interprete
sulla macchina dove si vuole eseguire il codice (non posso eseguire del codice
Perl o PHP su una macchina dove non è presente l'interprete in questione).
