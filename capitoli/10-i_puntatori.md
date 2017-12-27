# I puntatori

La memoria RAM del calcolatore non è altro che un insieme di locazioni di
memoria; per poter localizzare ciascuna locazione, ognuna di esse è identificata
da un indirizzo univoco. Questo significa che:

* Per scrivere qualcosa in memoria centrale dobbiamo conoscere l'indirizzo del
  punto esatto in cui scrivere;
* Se conosciamo l'indirizzo di una data locazione possiamo leggere ciò che è
  contenuto al suo interno.

## Puntatori in C

Il C consente di gestire, oltre al contenuto delle variabili stesse, anche i
loro indirizzi (ovvero le loro locazioni in memoria) attraverso il meccanismo
dei puntatori.

Fino ad oggi abbiamo gestito le variabili all'interno dei blocchi di codice in
cui tali variabili erano visibili, quindi non c'è stata la reale necessità di
utilizzare l'indirizzo della locazione di memoria in cui i valori di tali
variabili erano stati memorizzati. L'uso però a volte diventa indispensabile
all'interno delle funzioni (anche nella scanf, come avevo anticipato, si usava
implicitamente un puntatore per stabilire l'area fisica di memoria in cui
salvare la variabile letta da input).

Un puntatore ha una sintassi simile:

```c
int a=3;  // Variabile
int *x;  // ''Puntatore'' ad una variabile di tipo int
 
x=&a;  // Il puntatore ''x'' contiene l'indirizzo della variabile ''a''
*x=4;  // In questo modo modifico il contenuto del valore puntato
       // da x, quindi modifico indirettamente il valore di a
```

&a identifica l'indirizzo in memoria al quale si trova la variabile a, indirizzo
che viene salvato nel puntatore x. Quest'uso dei puntatori dovrebbe farci
tornare alla mente la sintassi della scanf:

```c
int a;
 
printf ("Inserisci un valore intero: ");
scanf ("%d",&a);
```

Ora possiamo capire appieno la sintassi della scanf. È una funzione che non fa
altro che leggere, in questo caso, un valore intero da tastiera, e salvarlo
nell'indirizzo fisico di memoria in cui si trova la variabile a (&_a_).

Ovviamente, se voglio salvare un valore letto da tastiera in una variabile a cui
è già associato un puntatore, non ho bisogno di ricorrere alla scrittura di
sopra:

```c
#include <stdio.h>
 
int main()  {
    int a;
    int *x=&a;
                     
    printf ("Inserisci un valore intero: ");
                             
    // Salvo il valore immesso direttamente nell'allocazione
    // di memoria puntata da ''x'', ovvero nella variabile ''a''
    scanf ("%d",x);
                                                     
    printf ("Valore salvato all'indirizzo: 0x%x: %d\n",x,a);

    return 0;
}
```

ritornerà come output qualcosa del tipo

    Valore salvato all'indirizzo: 0xbfc16a24: 4

dove 0xbfc16a24 è, in questo caso, l'indirizzo fisico di memoria (in formato
esadecimale) in cui si trova la variabile intera a (e quindi il valore 4, in
questo caso). Ricordiamo che sulle macchine a 32 bit (quindi tutte le macchine
Intel dal 486 in su, escluse quelle a 64 bit come Itanium e simili) un indirizzo
di memoria è sempre grande 32 bit (come in questo caso), quindi in memoria un
puntatore occuperà _sempre_, indipendentemente dalla variabile a cui punta, 32
bit = 4 byte.

## Passaggio di puntatori alle funzioni

Vediamo ora un'applicazione pratica dell'uso dei puntatori. Abbiamo una classica
applicazione che effettua lo scambio di due numeri interi (ovvero, se ho due
variabili, a=3 e b=4, voglio ottenere a=4 e b=3). Il modo più immediato di
risolvere questo problema è quello di appoggiarsi ad una variabile temporanea:

```c
int a=4;
int b=3;
int tmp;
 
.....
 
tmp=a;   // tmp=4
a=b;     // a=3
b=tmp    // b=tmp=4
```

Vogliamo ora implementare questo codice in una funzione a parte che viene poi
richiamata dal nostro programma. Con le nostre conoscenze attuali scriveremo un
codice del genere:

```c
#include <stdio.h>
 
// Funzione per lo scambio
void swap(int a, int b)  {
    int tmp;
       
    tmp=a;
    a=b;
    b=tmp;
}
 
int main()  {
    int a=4;
    int b=3;
     
    printf ("a=%d, b=%d\n",a,b);
    swap(a,b);
    printf ("a=%d, b=%d\n",a,b);

    return 0;
}
```

compilandolo avremo una sorpresa inaspettata: i valori sono rimasti immutati.
Questo perché alla funzione swap non passiamo le variabili _fisicamente_, ma
passiamo i loro valori. Quando invochiamo una funzione, l'atto della chiamata
crea in memoria una nuova area dello stack associata alla funzione appena
chiamata. In questo stack vengono _copiati_ i valori degli argomenti passati. La
funzione quindi non opera fisicamente sulle variabili passate, ma opera
piuttosto su _copie_ di esse. Quando la funzione termina l'area dello stack
corrispondente viene distrutta, e con essa anche le copie delle variabili al suo
interno, quindi non è possibile tener traccia delle modifiche.

La soluzione è proprio quella di _ricorrere_ ai puntatori, ovvero non passare
alla funzione copie delle variabili, ma gli indirizzi fisici in cui esse si
trovano, in modo che la funzione agirà direttamente su quegli indirizzi:

```c
#include <stdio.h>
 
// Funzione per lo scambio
void swap(int *a, int *b)  {
    int tmp;
       
    // tmp conterrà il valore della variabile intera puntata da a
    tmp=*a;

    // a conterrà il valore della variabile intera puntata da b
    *a=*b;

    // b conterrà il valore salvato in tmp
    *b=tmp;
}
 
int main()  {
    int a=4;
    int b=3;
         
    printf ("a=%d, b=%d\n",a,b);
           
    // Non passo le variabili alla funzione ma i loro indirizzi in memoria
    swap(&a,&b);
               
    printf ("a=%d, b=%d\n",a,b);

    return 0;
}
```

e ora il nostro codice funziona a dovere.

## Puntatori e array

Nel paragrafo precedente abbiamo visto gli array come oggetti a sé stanti,
diversi da qualsiasi altro tipo di variabile e di dato che abbiamo incontrato.
Ai fini del calcolatore però un array viene trattato esattamente alla stregua di
un puntatore, un puntatore all'area di memoria dov'è contenuto il primo elemento
dell'array stesso. Esempio:

```c
#include <stdio.h>
 
int main()  {
    int v[] = {4,2,8,5,2};
             
    // Queste due scritture sono equivalenti
    printf ("Primo elemento dell'array: %d\n",v[0]);
    printf ("Primo elemento dell'array: %d\n",*v);

    return 0;
}
```

questo vuol dire che possiamo accedere a qualsiasi elemento dell'array
specificando o il suo indice tra parentesi quadre o sommandolo al valore del
puntatore al primo elemento:

```c
#include <stdio.h>
 
int main()  {
    int v[] = {4,2,8,5,2};
             
    // Queste due scritture sono equivalenti
    printf ("Secondo elemento dell'array: %d\n",v[1]);
    printf ("Secondo elemento dell'array: %d\n",*(v+1));

    return 0;
}
```
questo perché quando viene instanziato un array non viene fatto altro che creare
un puntatore ad una certa area della memoria centrale, per poi riservare tanto
spazio in memoria quanto specificato dalla dimensione dell'array (nell'esempio
di sopra lo spazio di 5 variabili int, una variabile int in genere è grande 4
byte su una macchina a 32 bit quindi vengono riservati 5\*4=20 byte a partire
dall'indirizzo del primo elemento).

## Passaggio di array a funzioni

Tale caratteristica si rivela conveniente per molti aspetti. L'aspetto
principale consiste nel poter passare un array ad una funzione come se fosse un
puntatore. Esempio:

```c
#include <stdio.h>
 
int print_array (int *v, int dim)  {
    int i;
       
    for (i=0; i<dim; i++)
        printf ("Elemento [%d]: %d\n",i,v[i]);
}
 
int main()  {
    int v[] = { 3,5,2,7,4,2,7 };
       
    print_array(v,7);
}
```

## Allocazione dinamica della memoria

L'altro enorme vantaggio di quest'ottica da parte del C (ovvero il considerare
gli array come semplici puntatori) risiede nel poter allocare dinamicamente
dello spazio in memoria. Non sempre sappiamo a priori quanto spazio può servire
in memoria per un array usato nel nostro programma. Ad esempio, nel caso in cui
si dà la possibilità all'utente di stabilire il numero di elementi da inserire o
quando vogliamo salvare dei dati in memoria senza sapere ancora la quantità di
dati da salvare (in questo caso dovremmo prima contare il numero di dati da
salvare, quindi allocare tanta memoria da poterli mantenere). In questi casi ci
viene in aiuto una delle caratteristiche più potenti del C, l'allocazione
dinamica della memoria, allocazione che è possibile attraverso la funzione
_malloc_, definita in stdlib.h. La malloc ha una sintassi simile:

```c
void* malloc (unsigned int size);
```

al posto di size specificheremo quanta memoria vogliamo allocare per la nostra
variabile o il nostro array. Come è possibile vedere il valore di ritorno di
questa funzione è _void\*_, ovvero ritorna l'indirizzo della zona di memoria
allocata in formato 'grezzo'. Per questo motivo è necessario _specializzare_ la
funzione attraverso un operatore di cast. Esempio chiarificatore:

```c
#include <stdio.h>
#include <stdlib.h>

int main()  {
    int *v;
    int i,n;

    printf ("Quanti elementi vuoi inserire nell'array? ");
    scanf ("%d",&n);

    v = (int*) malloc(n*sizeof(int));

    for (i=0; i<n; i++)  {
        printf ("Elemento n.%d: ",i+1);
        scanf ("%d",&v[i]);
    }

    for (i=0; i<n; i++)
        printf ("Elemento n.%d: %d\n",i+1,v[i]);

    free(v);
    return 0;
}
```

La scrittura _sizeof(int)_ ritorna la dimensione di una variabile int sulla
macchina in uso, quindi _n\*sizeof(int)_ è il numero di byte effettivi da
allocare in memoria (ovvero nella malloc diciamo di allocare in memoria n
blocchi di dimensione _sizeof(int)_ l'uno che ospiteranno n variabili intere, e
salviamo l'indirizzo a cui comincia questa zona di memoria nel puntatore v).

È possibile anche allocare dinamicamente vettori multidimensionali. Esempio di
allocazione dinamica di una matrice:

```c
#include <stdio.h>
#include <stdlib.h>

int main()  {
    int **A;
    int i,j;
    int m,n;

    printf ("Numero di righe della matrice: ");
    scanf ("%d",&m);

    printf ("Numero di colonne della matrice: ");
    scanf ("%d",&n);

    A = (int**) malloc(m*n*sizeof(int));

    // Inizializzo anche tutti i sotto-vettori,
    // ovvero le righe della matrice
    for (i=0; i<m; i++)
        A[i] = (int*) malloc(n*sizeof(int));

    for (i=0; i<m; i++)
        for (j=0; j<n; j++)  {
            printf ("Elemento [%d][%d]: ",i+1,j+1);
            scanf ("%d",&A[i][j]);
        }

    for (i=0; i<m; i++)
        for (j=0; j<n; j++)
                printf ("Elemento [%d][%d]: %d\n",i+1,j+1,A[i][j]);

    // N.B.  E’ necessario deallocare tutte le righe
    // della matrice precedentemente allocate
    for (i=0; i<m; i++)
        free(A[i])

    free(A);

    return 0;
}
```

È possibile anche usare la funzione _realloc()_ per modificare la dimensione di
aree di memoria. La sintassi è la seguente:

```c
void* realloc (void* ptr, unsigned int new_size);
```

Esempio chiarificatore:

```c
#include <stdio.h>
#include <stdlib.h>

int main()  {
    int *v = NULL;
    int i, val;
    int size = 0;

    do  {
        printf ("Inserire un nuovo elemento nell'array "
                "(-1 per terminare): ");
        scanf ("%d", &val);

        v = (int*) realloc( v, (++size)*(sizeof(int)) );
        v[size-1] = val;
    } while (val != -1);

    printf ("Elementi nell'array: ");

    for ( i=0; i < size; i++ )
        printf ("%d, ", v[i]);

    free(v);
    return
        0;
}
```

Come nota segnaliamo un comportamento interessante della realloc(). Si noti che
non usiamo mai la malloc() per allocare inizialmente lo spazio di memoria, ma al
primo ciclo la realloc() verrà richiamata su int\* v che è ancora NULL. Quando
la realloc() viene richiamata su un puntatore che è NULL si comporta esattamente
come una malloc(), quindi al primo giro allochiamo _v_ come vettore contenente
un elemento intero, e a ogni ciclo aumentiamo la sua dimensione, finché l'utente
non inserisce -1.

## Deallocazione della memoria, memory leak e garbage collection

È *fondamentale* usare la funzione _free()_, sempre dichiarata in stdlib.h,
quando una certa area di memoria precedentemente allocata non serve più. La
malloc() (e, come vedremo fra poco, anche la realloc()) allocano infatti memoria
su una zona di memoria chiamata _heap_, mentre tutte le variabili che abbiamo
esaminato finora vengono generalmente allocate sullo _stack_ (caso di variabili
locali) o nel segmento _data_ (caso di variabili globali). Tutto ciò che è
allocato sullo stack viene allocato quando la funzione corrispondente viene
invocata e viene distrutto quando tale funzione termina. Tutto ciò che è sullo
heap invece viene allocato e rimane lì finché qualcuno non lo dealloca
esplicitamente (appunto, tramite la funzione free()). Se devo allocare una zona
di memoria grande un milione di byte, quella zona di memoria rimane lì segnalata
come allocata finché qualcuno non dice che non serve più, oppure finché il
processo non termina.  Questo porta a un problema noto come una delle più grandi
_maledizioni del programmatore_ che è il _memory leak_, ovvero l'aumento
esponenziale, nel caso di programmi molto complessi con grande uso
dell'allocazione dinamica della memoria, della quantità di memoria utilizzata,
che può arrivare a rallentare drammaticamente le prestazioni della macchina a
causa di continui _swap_ fra memoria centrale satura e hard disk o peggio al
crash del programma. I memory leak sono anche difficili da scovare nel caso di
progetti molto complessi.  Esistono tool come _valgrind_ che aiutano il
programmatore a capire se il proprio programma presenta usi cattivi della
memoria o meno, ed eventualmente dove sono localizzabili, ma è comunque molto
difficile nel caso di un progetto molto grosso tenere sotto controllo tutte le
allocazioni dinamiche e capire quale è l'origine del problema.

Esempio tipico di esecuzione di valgrind su un programma in cui tutta la memoria
allocata dinamicamente viene correttamente deallocata dalla free():

```
$ valgrind ./leak
...
==16227== HEAP SUMMARY:
==16227==     in use at exit: 0 bytes in 0 blocks
==16227==   total heap usage: 1 allocs, 1 frees, 100 bytes allocated
==16227==
==16227== All heap blocks were freed -- no leaks are possible
```

Esempio di esecuzione su un programma che invece presenta memory leak:

```
$ valgrind ./leak
...
==30694== HEAP SUMMARY:
==30694==     in use at exit: 100 bytes in 1 blocks
==30694==   total heap usage: 1 allocs, 0 frees, 100 bytes allocated
==30694==
==30694== LEAK SUMMARY:
==30694==    definitely lost: 100 bytes in 1 blocks
==30694==    indirectly lost: 0 bytes in 0 blocks
==30694==      possibly lost: 0 bytes in 0 blocks
==30694==    still reachable: 0 bytes in 0 blocks
==30694==         suppressed: 0 bytes in 0 blocks
==30694== Rerun with --leak-check=full to see details of leaked memory
```

I memory leak sono considerati errori di programmazione molto seri, in quanto
possono compromettere la stabilità del programma e dell'intero sistema
operativo, ma sono anche molto comuni (è facile allocare della memoria che non
servirà più dopo 3000 righe di codice e dimenticarsi di deallocarla). Il trucco
sta nel piazzare subito dopo una malloc() o una realloc() la free()
corrispondente, per essere sicuri di non dimenticarsela in seguito, e scrivere
il codice che usa quella memoria allocata in mezzo, fra l'allocazione e la
deallocazione.

Linguaggi di livello più alto come Java hanno integrato un meccanismo chiamato
_garbage collection_. Java infatti non richiede che il programmatore allochi
esplicitamente la memoria dinamica attraverso funzioni come la malloc() del C:
la memoria dinamica viene gestita automaticamente dalla virtual machine, e
periodicamente sulla memoria del processo operano algoritmi di _garbage
collection_, che servono a deallocare automaticamente zone di memoria allocate
in precedenza quando non servono più. Tali meccanismi volendo sono disponibili
anche in C, sollevando il programmatore dall'onere della deallocazione della
memoria e quindi dal rischio di memory leak. La libreria probabilmente più
famosa che implementa il meccanismo di garbage collection sulla memoria dinamica
è la _libgc_, disponibile per la maggior parte delle piattaforme moderne. La
libgc sostituisce alle funzioni "a rischio" memory leak se la memoria associata
non viene deallocata esplicitamente (malloc, realloc e, come vedremo più avanti,
strdup) le proprie “versioni” (GC\_malloc, GC\_realloc e GC\_strdup) su cui
operano algoritmi di garbage collection, sollevando quindi il programmatore
dalla responsabilità della free(). Osserviamo brevemente come scrivere un
sorgente che faccia uso delle funzioni di questa libreria, ricordando che la
prassi che seguiremo ora è simile a quella da seguire ogni qual volta si voglia
eseguire codice da librerie esterne nei propri programmi in C.

Innanzitutto occorre installare la libgc sul proprio sistema (attraverso il
proprio package manager preferito se si è su un sistema Unix-like, o dal file di
setup se si è su Windows). Alla fine di un'installazione terminata con successo
ci si dovrebbe ritrovare nella directory include del proprio compilatore la
directory gc con dentro il file _gc.h_, e nella directory di lib il file
_libgc.a_, o _libgc.so_, o _libgc.dll_ se si opera su Windows. Ora si può
scrivere del codice che faccia uso delle funzioni della libreria:

```c
#include <gc.h>

int main()  {
      int *v = GC_malloc( 100*sizeof(int) );
        return 0;
}
```

A questo punto compiliamo il sorgente in questo modo:

    gcc -I/usr/include/gc -o noleak noleak.c -lgc

L'opzione _-I_ serve a identificare una nuova directory in cui cercare i file
header inclusi (in questo caso, se il file _gc.h_ è incluso in
_/usr/include/gc_, diciamo al compilatore di cercare i file inclusi anche in
quella directory), mentre l'opzione _-lgc_ dice al compilatore di linkare
l'eseguibile usando la libreria _libgc_. Questo funziona nel caso in cui il file
di _libgc_ sia presente in una directory contenuta, nel caso di sistemi
Unix-like, in una directory standard in cui ricercare i file di libreria (ad
esempio _/usr/lib_ o _/usr/local/lib_). In caso contrario è necessario
specificare esplicitamente in che directory cercare i file di libreria usando
l'opzione _-L_:

    gcc -I/usr/include/gc -L/usr/lib -o noleak noleak.c -lgc

Pur non essendoci una free() associata alla malloc notiamo che eseguendo
_valgrind_ sull'eseguibile non viene rilevato nessun memory leak:

```
$ valgrind ./noleak
...
==3019== HEAP SUMMARY:
==3019==     in use at exit: 0 bytes in 0 blocks
==3019==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==3019==
==3019== All heap blocks were freed -- no leaks are possible
```

L'uso di questa libreria è tuttavia abbastanza controverso. È vero che è molto
comoda e solleva il programmatore dalla responsabilità della deallocazione della
memoria, ma non sempre ci si ritrova a programmare su sistemi dove questa
libreria è presente, e il programmatore dovrebbe imparare a gestire la memoria
indipendentemente dalla presenza o meno della _libgc_ sul suo sistema. Inoltre un
programma che usa la _libgc_ ha una _dipendenza_ in più, in quanto essendo una
libreria dinamica il suo programma funzionerà solo su sistemi dove è presente la
_libgc_. Questa è una questione che un programmatore dovrebbe sempre porsi ogni
qualvolta crede che il suo software abbia bisogno di appoggiarsi a una libreria
esterna. Non è mai una buona idea reinventare la ruota riscrivendo da zero
funzioni complesse magari presenti già, meglio implementate, in un'altra
libreria, ma ridurre il proprio software a un castello di carta di dipendenze
esterne e rendere difficile la vita all'utente che vuole installarlo sulla
propria macchina e dovrà prima scaricare qualche MB di dipendenze solo per farlo
girare non è altrettanto una buona idea.

## Funzioni che ritornano array

Come già visto gli array altro non sono che puntatori al primo elemento, quindi
una funzione che ritorni, ad esempio, un array di interi avrà semplicemente un
prototipo del genere:

```c
int* funzione (parametri ...);
```

Tuttavia se scriviamo un codice del genere

```c
int* foo()  {
    int i, v[10];

    for ( i=0; i < 10; i++ )
        v[i] = i;
    return v;
}

int main()  {
    int *v = foo();
    return 0;
}
```

Ci ritroviamo di fronte a una sorpresa. La sorpresa è già preannunciata da un
warning del compilatore

    warning: function returns address of local variable

Se proviamo a stampare dal main il contenuto di v dopo la chiamata a foo(), ci
ritroveremo quasi sicuramente di fronte a dei valori casuali, invece di avere un
array contenente gli elementi da 0 a 9 ordinati. Questo perché _v_ dentro foo() è
dichiarato come array statico, quindi allocato sullo stack della funzione foo(),
modificato, e ritornato. Tuttavia quando la funzione foo() termina anche il suo
stack viene distrutto, quindi il contenuto di v non è più reperibile
dall'esterno, e questo spiega perché andiamo a leggere dei valori casuali. Se
volessimo correttamente ritornare un array da una funzione dovremmo prima
allocarlo dinamicamente attraverso una malloc(), in modo che sia allocato sullo
heap che è una zona di memoria che a differenza dello stack non viene distrutta
quando una funzione termina ma rimane viva per tutto il processo, quindi
ovviamente dovremmo ricordarci di deallocare quello spazio quando non ci serve
più.

```c
int* foo()  {
    int i, *v = (int*) malloc( 10*sizeof(int) );

    for ( i=0; i < 10; i++ )
        v[i] = i;
    return v;
}

int main()  {
    int *v = foo();
    free(v);
    return 0;
}
```

## Puntatori a funzione

Le funzioni a basso livello non sono altro che sequenze di istruzioni binarie
piazzate nella memoria centrale, al pari di una qualsiasi variabile. È quindi
possibile anche costruire puntatori che puntino a funzioni, in quanto normali
aree di memoria. La sintassi è la seguente:

```c
tipo (*nome_ptr)(argomenti) = funzione
```

Per richiamare la funzione puntata, basta poi un

```c
(*nome_ptr)(argomenti)
```

Esempio:

```c
#include <stdio.h>

void foo()  {
    printf ("Ciao\n");
}

int main()  {
    void (*ptr)(void) = foo;
    printf ("foo si trova all'indirizzo 0x%.8x\n",ptr);
    (*ptr)();
    return 0;
}
```

o ancora

```c
#include <stdio.h>

int foo(int a, int b)  {
    return a+b;
}

int main()  {
    int a=2,b=3;
    int (*ptr)(int, int) = foo;
    printf ("foo si trova all'indirizzo 0x%.8x\n",ptr);
    printf ("%d+%d=%d\n",a,b,(*ptr)(a,b));
    return 0;
}
```

Questo tipo di scrittura è molto utile in un'ottica di modularità del programma.
Si può ad esempio lasciare all'utente, o al programmatore finale nel caso di
sviluppo di una libreria, la libertà di stabilire che azioni associare a un
determinato contesto. Ad esempio scegliere a runtime che algoritmo usare per
ordinare un insieme di dati, o per effettuare l'interpolazione o
l'approssimazione di un insieme di valori numerici. Si dichiara un puntatore a
funzione, a seconda delle scelte dell'utente si decide a quale funzione farlo
puntare, e si richiama direttamente il puntatore invece della funzione. Un
esempio può essere quello per gestire, ad esempio, l'evento _onClick_ in un form
HTML, a cui si associa una funzione JavaScript. La funzione associata è trattata
a basso livello semplicemente come un puntatore a funzione.

## Funzioni come parametri di altre funzioni

A questo punto nulla mi impedisce di passare come parametro di una funzione un
puntatore a funzione, e la funzione richiamata può richiamare la funzione
passata come argomento. Esempio:

```c
#include <stdio.h>

void print ()  {
    printf ("Ciao\n");
}

int foo(void (*f)(void))  {
    (*f)();
}

int main()  {
    foo(print);
    return 0;
}
```
