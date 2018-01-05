# Funzioni e procedure

Ogni linguaggio di programmazione ad alto livello mette a disposizione del
programmatore gli strumenti delle funzioni e delle procedure, tanto più il C,
linguaggio procedurale per eccellenza.

Abbiamo già incontrato nel corso di tutorial un esempio di funzione: il main().
Il main() altro non è che una funzione speciale che viene eseguita all'inizio
del programma. Ma ovviamente è possibile definire anche altre funzioni (avevo
già accennato che tutto ciò che si fa in C si fa tramite le funzioni. Anche la
printf() che abbiamo usato nei paragrafi precedenti non è altro che una funzione
definita in stdio.h).

## Definizione intuitiva di funzione

Per capire meglio come lavorano le funzioni in C, ci aiuteremo con la
definizione matematica di funzione. Sappiamo che una funzione matematica è
scritta in genere nella forma _y=f(x)_, ossia ad ogni valore della variabile
indipendente x (che può essere o una variabile scalare, quindi una variabile a
cui corrisponde un solo valore reale, o un vettore di variabili) corrisponde uno
ed un solo valore della variabile dipendente y. Prendiamo ad esempio la funzione
_f(x)=x+2_: ad ogni valore della x corrisponde uno ed un solo valore della
funzione f(x), se x è 0, f(x) è 2, se x è 1, f(x) è 3, e così via.

È possibile anche che in una funzione ci sia più di una variabile indipendente:
ad esempio, _f(x,y)=x+y_.

Le "variabili indipendenti" delle funzioni nelle funzioni C sono i parametri,
ossia i valori che si danno in input alla funzione (anche se è possibile creare
funzioni senza alcun parametro), mentre il "risultato" della funzione (la
"variabile dipendente") si ottiene usando la keyword return che abbiamo già
incontrato. Ecco la struttura di una funzione in C:

```
tipo_ritornato nome_funzione(parametro1, parametro2 ... parametron) {
    codice
    codice
    // ......
}
```
## Esempi d'uso di funzioni e standard di utilizzo

Ecco un piccolo esempio:

```c
int square(int x) {
    return x*x;
}
```

Questa funzione calcola il quadrato di un numero intero x. La variabile int x è
il parametro che passo alla funzione. Ho stabilito all'inizio, dichiarando la
funzione come int, che il valore ritornato dalla funzione (la "variabile
dipendente") deve essere di tipo int. Attraverso la direttiva return stabilisco
quale valore deve ritornare la funzione (in questo caso il quadrato del numero
x, ossia x\*x). In matematica, una funzione del genere la potrei scrivere come
_f(x)=x^2_.

Questa funzione la posso richiamare all'interno del main() o di qualsiasi altra
funzione del programma. Esempio:

```c
int y;          // Dichiaro una variabile int
y = square(2);  // Passo alla funzione square il valore 2,
                // in modo che calcoli il quadrato di 2
printf ("Quadrato di 2: %d\n",y);
```

Ovviamente, posso dichiarare un'infinità di funzioni in questo modo. Ecco ad
esempio una funzione che calcola la somma di due numeri:

```c
int somma(int a, int b)  {
    return a+b;
}
```

Invocazione:

```c
int c;
c = somma(2,3); // c vale 5
```

La maggior parte delle funzioni matematiche sono dichiarate nel file math.h (ci
sono ad esempio funzioni per calcolare il seno, il coseno o la tangente di un
numero reale, il logaritmo, la radice quadrata, la potenza n-esima...), quindi
se vi interessa fare un programma di impostazione matematica date un'occhiata a
questo file per capire quale funzione usare.

Ovviamente, è anche possibile creare funzioni senza alcun parametro in input.
Esempio (banale):

```c
int ritorna_zero() {
    return 0;
}
```

Vediamo ora come inserire una funzione nel nostro programma. Le funzioni in C
possono andare in qualsiasi parte del codice.  Tuttavia l'ANSI C, per evitare
confusione, ha imposto che le funzioni debbano essere _implementate_ prima del
punto in cui vengono richiamate. Una scrittura del genere ad esempio è da
considerarsi errata:

```c
int main()  {
    foo();

    return 0;
}

int foo()  {
    return 0;
}
```

in quanto la funzione foo() viene richiamata dal main prima di essere
dichiarata.  gcc solleverà un warning:

    warning: implicit declaration of function ‘foo’

mentre un compilatore C++ come g++, generalmente più fiscale su questo tipo di
scritture, solleverà un vero e proprio errore:

    error: ‘foo’ was not declared in this scope

Ovviamente l'errore sparisce se si piazza la dichiarazione di foo() prima del
main.

Non sempre tuttavia è possibile avere funzioni dichiarate prima del punto in cui
vengono usate. Si pensi al caso in cui si usino funzioni da librerie esterne,
dove il codice della funzione è presente chissà dove, spesso direttamente in
formato binario dentro una libreria dinamica. In tal caso l'ANSI C impone di
assicurarsi che prima del punto in cui la funzione viene usata sia presente
almeno un suo _prototipo_. Il prototipo di una funzione è semplicemente la
funzione dichiarata attraverso tipo di ritorno, nome e lista di argomenti. Un
prototipo serve a dire al compilatore “più avanti nel codice verrà richiamata
questa funzione che ha questo nome, prende questi parametri e ritorna questo
valore, ignora per ora il suo contenuto, che sarà pescato a tempo di linking”.

Il codice errato visto sopra diventerebbe corretto se specificassimo il
prototipo della funzione _foo()_ prima del main, anche se la funzione vera e
propria viene implementata dopo:

```c
int foo();

int main()  {
    foo();
    return 0;
}

int foo()  {
    return 0;
}
```

Nei file header vengono generalmente piazzati i prototipi delle funzioni, non le
funzioni vere e proprie. Queste sono infatti presenti, in genere, nei file di
libreria inclusi implicitamente o esplicitamente a tempo di compilazione (ad
esempio aprendo il file stdio.h troveremo il prototipo di printf(), non il suo
codice, che è presente già compilato nella libc). Tuttavia nel caso in cui sia
presente solo il prototipo di una funzione e non la sua implementazione avremmo
un errore a livello di linking (attenzione, non a livello di compilazione, il
compilatore non fa altro che dire “richiama la funzione avente questo
prototipo”, ma se l'implementazione non c'è da nessuna parte il linker non sa a
che indirizzo in memoria mandare quella chiamata). Se dal codice di sopra
rimuovessimo l'implementazione di foo() lasciando solo il prototipo all'inizio
gcc (o meglio ld, ovvero il linker richiamato da gcc) darebbe il seguente errore
di linking:

    test.c:(.text+0x7): undefined reference to `foo'
    collect2: ld returned 1 exit status

L'alternativa, ovviamente, è piazzare l'implementazione della funzione prima che
venga usata. In tal caso non è necessario il prototipo.

```c
/* square.c */

#include <stdio.h>

int square(int x)  {    // Implementazione della funzione square()
    return x*x;
}

int main()  {
    int y;                        // Variabile intera
    y = square(3);                // Ora y vale 9
    printf ("Quadrato di 3: %d\n",y);

    // Più brevemente, potremmo anche scrivere:
    // printf ("Quadrato di 3: %d\n",square(3));
    // senza neanche "scomodare" la variabile di “appoggio” y

    return 0;
}
```

Nei programmi di grandi dimensioni in genere, come accenato, si usa piazzare il
prototipo della funzione in un file header (con estensione .h),
l'implementazione in un file .c e poi il programma vero e proprio nel file
main.c. Esempio:

```c
/* Questo è il file square.h */
int square(int x);

/* Questo è il file square.c */

int square(int x)  {
    return x*x;
}

/* Questo è il file main.c */

#include <stdio.h>
#include "square.h"

// Ovviamente includo il file square.h

int main()  {
    printf ("Quadrato di 4: %d\n",square(4));
    return 0;
}
```

Quando vado a compilare questo programma devo fare una cosa del genere:

    gcc -o square main.c square.c

## Procedure

Un discorso simile a quello delle funzioni vale anche per le procedure; le
procedure non solo altro che funzioni "speciali", funzioni che non hanno un
valore ritornato: eseguono un pezzo di codice ed escono. Per concludere una
procedura non è necessario il return (in quanto non ritorna alcun valore): al
massimo ci possiamo mettere un return;. Per dichiarare una procedura userò la
keyword void:

```c
void hello() {
    printf ("Hello world!\n");
    return;                       // Questa riga è opzionale
}
```

Il “return;” alla fine è facoltativo (è ovvio che una funzione void non ritorna
nulla), ma è indispensabile nel caso in cui voglio che la funzione, se si
presentano certe condizioni, termini prematuramente.

Quando voglio chiamare questa procedura all'interno di una qualsiasi funzione,
basterà fare così:

```c
hello();
```

Esempio:

```c
#include <stdio.h>

void hello();             // Prototipo della procedura

int main() {
    hello();              // Stampo la scritta "Hello world!"
                          // attraverso la procedura hello()
    return 0;
}

void hello() {            // Implementazione della procedura
  printf ("Hello world!\n");
}
```

Anche alle procedure posso passare qualche parametro. Esempio:

```c
void stampa_var(int x) {
    printf ("Valore della variabile passata: %d\n",x);
}
```

Invocazione:

```c
stampa_var(3);  // L'output è: "Valore della variabile passata: 3
```

Nota tecnica: attenzione a non fare cose del genere!

```c
int square(int x);
double square(double x);
```

Quando vado a chiamare la funzione:

```c
square(3);
```

il compilatore non sa che funzione chiamare e va nel pallone. Proprio per
evitare ambiguità del genere, la maggior parte dei compilatori danno un errore
(o almeno un warning) quando nel programma compaiono scritture del genere
(tuttavia, nel C++ cose del genere sono possibili, con l'overloading delle
funzioni, ossia con la dichiarazione di più funzioni con lo stesso nome MA con
la lista dei parametri differente. In ogni caso, una scrittura come quella di
sopra darà problemi anche in C++, in quanto entrambe le funzioni hanno un solo
parametro e il compilatore, nel momento dell'invocazione, non sa quale funzione
chiamare).

## Funzioni statiche

Le funzioni statiche hanno proprietà molto simili alle variabili statiche.  Tali
funzioni, al pari delle corrispettive variabili, sono

* Istanziate in memoria quando il programma viene creato, e distrutte quando il
  processo corrispondente è terminato;
* Visibili e utilizzabili _solo_ all'interno del file che le ha dichiarate.

La seconda proprietà impone delle limitazioni d'uso delle funzioni statiche, in
modo da rendere più modulare il programma, più protetto ed evitare che qualsiasi
file del programma possa richiamare qualsiasi funzione del programma.

Esempio:

```c
/* file: foo.c */

#include <stdio.h>

static void foo1() {
    printf ("Sono una funzione statica\n");
}

void foo2() {
    printf ("Richiamo una funzione statica\n");
    foo1();   // Chiamata valida. La funzione foo1() è contenuta
              // nello stesso file della funzione foo2()
}

/* file: main.c */

#include <stdio.h>

static void foo1();
void foo2();

int main() {
    foo2();     // Chiamata valida. La funzione foo2() è visibile
                // al main e non è una funzione statica
    foo1();     // ERRORE! foo1() è statica e non visibile qui
                // Errore del linker:
                // undefined reference to `foo1'
    return 0;
}
```

Il meccanismo della visibilità delle funzioni e delle variabili è ancora un po'
primitivo nel C, basato sul concetto di staticità, mentre verrà decisamente
approfondito in linguaggi a oggetti come C++, Java e Smalltalk.

## Funzioni globali/locali

C'è, infine, un'altro metodo, per creare una funzione, sconsigliato in quanto
rende meno modulare, leggibile o mantenibile il codice (ma comunque possibile).

Questo metodo consiste nel creare una funzione locale ad un'altra funzione.
Ovvero una funzione visibile e richiamabile solo all'interno della funzione in
cui è stata dichiarata.

Un esempio della sua creazione è:

```c
#include <stdio.h>

int main()  {
    void hello_local_function(void) {
        printf("Local Function is Ready!\n");
    }

    printf("Richiamo la funzione interna...\n");
    hello_local_function();
    printf("Esco.\n\n");

    return 0;
}
```

## Definizione di macro

Le funzioni che abbiamo imparato a dichiarare finora sono funzioni vere e
proprie, residenti in un segmento di memoria del processo, con un proprio
indirizzo di inizio e di fine, che il compilatore converte in codici operativi
CALL a basso livello. Ma non è l'unico modo di creare funzioni in C. Tale
linguaggio permette infatti di dichiarare anche “pseudo-funzioni”, chiamate
_macro_, che di fatto non vengono allocate in memoria quando il processo viene
eseguito ma vengono macinate dal precompilatore quando incontrare, a ogni
chiamata viene sostituito (“espanso”) il codice desiderato. Quest'approccio è
possibile tramite la direttiva al preprocessore #define, che abbiamo già
incontrato per la dichiarazione delle costanti. Esempio di dichiarazione di una
macro che calcola il quadrato di un numero attraverso la direttiva #define:

```c
#include <stdio.h>

#define     SQUARE(x)   (x*x)

int main()  {
    printf (“%d\n”, SQUARE(3));
    return 0;
}
```

Semplicemente prima della compilazione vengono esaminate tutte le occorrenze di
SQUARE(qualsiasi cosa) all'interno del sorgente, e gli viene sostituita la
sequenza (x\*x), dove x è l'argomento passato a SQUARE. Possiamo capire meglio
come si comporta il compilatore richiamando gcc con l'opzione -E su questo file,
che serve ad eseguire solo le fasi di precompilazione e stampare il risultato su
stdout senza eseguire la compilazione vera e propria:

```
$ gcc -E macro.c

...
int main() {
     printf ("%d\n", (3*3));
      return 0;
}
```

Questa scrittura è decisamente più primitiva della definizione di funzioni vere
e proprie, oltre a essere scarsamente leggibile nel caso di dichiarazioni di
funzioni particolarmente complesse, ma è da preferire per motivi di
ottimizzazione nel caso di routine relativamente semplici richiamate spesso
all'interno del programma. La chiamata di una funzione è infatti relativamente
onerosa da un punto di vista computazionale. A basso livello quando viene
richiamata una funzione la CPU “congela” lo stato del processo corrente,
salvando in memoria lo stato dei suoi registri e l'indirizzo a cui ci si trova,
per effettuare una chiamata attraverso il codice operativo CALL alla nuova
funzione, e ripescando dalla memoria lo stato dei registri prima della chiamata
quando la funzione termina. Tutto ciò può essere pesante nel caso in cui si
voglia un programma dalle elevate prestazioni che richiama anche milioni di
volte una determinata funzione. In casi come questo un fattore di ottimizzazione
può essere sostituire al codice della funzione una macro dichiarata via #define,
che viene sostituita al momento della precompilazione dal suo relativo
contenuto, risparmiando il tempo inevitabile del _context switch_ dovuto alla
chiamata a una funzione vera e propria.
