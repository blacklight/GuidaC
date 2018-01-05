# Uso delle variabili

In tutti i linguaggi di programmazione le variabili rivestono un ruolo
fondamentale. Le variabili dell'informatica sono una sorta di "contenitori" che
al loro interno possono contenere numeri interi, numeri a virgola mobile,
caratteri di testo ecc.

## Tipi di variabili

La dichiarazione di una variabile in C (ricordando che in C, a differenza di
linguaggi come Perl, Python, PHP o i linguaggi per la shell, è _indispensabile_
dichiarare una variabile prima di poterla utilizzare) è qualcosa del tipo

    tipo nome_variabile;

Possiamo anche assegnarle un valore iniziale, in questo modo:

    tipo nome_variabile = valore_iniziale;

Il tipo di variabile caratterizza la variabile stessa. Ecco i principali tipi
ammessi dal C:

--------------------------------------------------------------------------------
Tipo        Uso tipico                                    Dimensione in bit[^1]
----------- -------------------------------------------- -----------------------
char        Caratteri di testo ASCII, valori binari                 8
            generici da 1 byte

short int   Numeri interi piccoli (da -32768 a 32767)               16

unsigned    Numeri positivi interi piccoli (da 0 a 65535)           16
short
int

int         Numeri interi (da -2147483648 a 2147483647)             32

unsigned    Numeri interi positivi (da 0 a 4294967295)              32
int

long int    Numeri interi[^2]                                       32

long long   Numeri interi grandi (da -9.22\*10^18 a                 64
int         9.22\*10^18)

unsigned    Numeri interi grandi positivi (da 0 a                   64
long        1.84\*10^19)
int

float       Numeri a virgola mobile (precisione singola)            32

double      Numeri a virgola mobile (doppia precisione,             64
            notazione scientifica).
--------------------------------------------------------------------------------

[^1]: in riferimento ad una architettura x86.
[^2]: la dimensione coincide con quella di un normale int su una macchina x86.

Esempio:

```c
int a;          // Dichiaro una variabile intera chiamata a
                // senza inizializzarla
int b = 3;      // Dichiaro una variabile intera b che vale 3
char c = 'q';   // Dichiaro una variabile char che contiene il
                   carattere q
float d = 3.5;  // Dichiaro una variabile float d che vale 3.5
a = 2;          // Adesso a vale 2
int e = a+b;    // e vale la somma di a e b, ossia 5
```

## Operazioni elementari sulle variabili

È possibile fare con le variabili ogni tipo di operazione matematica elementare:
addizione (+), sottrazione (-), moltiplicazione (\*), divisione (/), resto della
divisione (%). Diamo però un'occhiata a questo codice:

```c
int a = 2;      // Variabile int
float b = 3.5;  // Variabile float
int c = a+b;    // Attenzione...
```

Qui effettuiamo un'operazione fra una variabile int e una variabile float e
salviamo il risultato in una variabile int. Quello che si ha è una perdita di
precisione del risultato in questo caso, in quanto la parte decimale viene
troncata nel salvataggio a int (quindi c varrà 5). In casi come questi in cui si
opera su quantità non omogenee il compilatore se la può cavare riconoscendo da
solo il tipo di variabile di output, ma per maggiore comprensione e pulizia è
sempre opportuno specificare esplicitamente sia il formato di “output” della
variabile nel caso in cui si operi su quantità non omogenee fra loro, sia
eventualmente come prendere le singole variabili (parte intera, forzare il
casting a decimale, ecc.). Tale operazione è detta di _casting_. Esempio del
codice di sopra con l'operatore di casting:

```c
int a = 2;
float b = 3.5;
int c = (int) (a+b);  // Converto il risultato in int. c vale 5 in questo
                      // caso
```

Oppure:

```c
int a = 2;
float b = 3.5;
float c = (float) (a+b);  // Converto il risultato in float. c vale 5.5 in
                          // questo caso
```

A differenza delle variabili "matematiche", in C una scrittura del genere è
concessa:

```c
int a = a+2;    // Aggiorno il valore di a
```

Oppure, in modo più sintetico:

```c
int a += 2;
```

La scrittura a += 2 sta per a = a+2 (sono concesse scritture come += -= \*= /=
%=).

La scrittura a++ è invece un incremento della variabile a, ed equivale a a=a+1
(così come la scrittura a-- equivale a a=a-1).

Meglio soffermarci un attimo su quest'aspetto. In C sono concesse sia scritture
come a++ sia come ++a, ed entrambe incrementano la variabile a di un'unità. Qual
è la differenza tra le due scritture? Una scrittura del tipo a++ viene chiamata
_post-incremento_. Ciò vuol dire che, sulla maggior parte dei compilatori, viene
prima eseguita l'istruzione in cui si trova quest'operazione, quindi, alla fine
dell'istruzione, la variabile viene incrementata. Una scrittura del tipo ++a
viene invece chiamata _pre-incremento_. Quando il compilatore incontra
un'operazione di pre-incremento in genere incrementa prima il valore della
variabile, quindi esegue l'istruzione all'interno della quale è collocata.

Se devo semplicemente incrementare il valore di _a_, è indifferente usare l'una
o l'altra scrittura. Ma si osservi questo esempio di codice...

```c
int a=3;
int b=4;
int c;
```

un conto è scrivere

```c
c = (a++)+b;
// c vale 3+4=7
// a viene incrementata dopo l'istruzione
// ora a vale 4
```

un altro conto è scrivere

```c
c = (++a)+b;
// c vale 4+4=8
// a viene incrementata prima dell'istruzione, e vale 4
```

## Stampa dei valori delle variabili

È anche possibile usare le variabili in funzioni come la printf(). Prendete ad
esempio il seguente codice:

```c
int x = 3;
printf ("x vale %d",x);
```

L'output sarà:

    x vale 3

La stringa di formato %d dice al compilatore di stampare la variabile intera
posta fuori i doppi apici "". In questo caso stampa il valore di x, che è
proprio 3. Se invece si desidera stampare una variabile di tipo float:

```c
float x = 3.14;
printf ("x vale %f",x);
```

dove la scrittura %f dice al compilatore di stampare una variabile di tipo
float. Ecco i formati di stringa principali usati per stampare i principali tipi
di variabili:

--------------------------------------------------------------------------------
Stringa di formato Uso
------------------ -------------------------------------------------------------
%c                 Variabili char

%d, %i             Valori in formato decimale

%x %X              Valori in formato esadecimale

%o                 Valori in formato ottale

%l, %ld            Variabili long int

%u                 Variabili unsigned

%f                 Variabili float

%lf                Variabili double

%p                 Indirizzo esadecimale di una variabile

%s                 Stringhe di testo (le vedremo più avanti...)

%n                 Scrive i byte scritti finora sullo stack dalla funzione
                   printf() (molto sfruttata in contesti di format string
                   overflow)
--------------------------------------------------------------------------------

Esempio:

```c
/* variabili.c */

#include <stdio.h>

int main() {
    int a,b,c;    // Dichiaro 3 variabili int

    a = 3;
    b = 4;
    c = a+b;      // c vale 7

    printf ("c vale %d\n",c);

    a += 3;       // Ora a vale 6
    b++;          // Ora b vale 5
    c = a-b;      // Ora c vale -1

    printf ("Ora c vale %d\n",c);

    return 0;
}
```
Il fatto interessante è che possiamo eseguire opeazioni anche sulle variabili
char. Le variabili char, infatti, vengono considerate dal programma come codici
ASCII, ovvero ogni variabile ha il suo codice numerico (da 0 a 255) che può
essere visualizzato come carattere o lasciato come valore numerico a 8 bit
(attenzione, in linguaggi di livello più alto, come Java queste operazioni non
sono concesse, sia perché i caratteri non sono in formato ASCII ma Unicode, sia
perché un carattere viene strettamente distinto dal suo valore numerico). Ecco
un esempio:

```c
char c = 65;
// Equivale a scrivere char c = 'A', infatti
// 65 è il codice per la lettera A

char c += 4;    // Ora a vale E
printf ("a = %c\n",c);
```

## Variabili locali e globali

In C le variabili vanno dichiarate o all'inizio del programma o all'inizio della
funzione che le usa. Attenzione: è un errore dichiarare una variabile in altri
posti o usare variabili non dichiarate. Esempio:

```c
int main()  {
    printf ("Questa e' una prova\n");

    int b = 3;  // ERRORE! Non si può dichiarare una variabile dopo che la
                // funzione ha già eseguito un'istruzione
    c=4;        // ERRORE! c non è dichiarata

    return 0;
}
```

In C++ non si ha il primo errore, in quanto la dichiarazione di una variabile è
considerata un'istruzione vera e propria e può essere messa ovunque, ma in C c'è
l'errore. La maggior parte dei compilatori C moderni può autorizzare comunque la
dichiarazione di variabili nel mezzo del codice (ma scritture valide in C++ come
la dichiarazione di variabili direttamente in cicli for sono ancora vietate), ma
per una questione di compatibilità è sempre meglio andare sul sicuro e
dichiarare le variabili usate in una certa funzione all'inizio della funzione
stessa (nel nostro caso, subito dopo l'inizio del main, senza altre istruzioni
di mezzo).

Le variabili dichiarate all'inizio del programma (prima del main e di ogni
funzione) vengono dette globali e possono essere usate da ogni funzione del
programma (lo vedremo meglio quando parleremo delle funzioni), mentre le
variabili locali possono essere viste solo dalla funzione che le dichiara (in
C++ è anche possibile far vedere le variabili ad un solo blocco di codice).
Esempio:

```c
#include <stdio.h>

int var_globale = 3;    // Variabile globale

int main()  {
    int var_locale = 2;   // Variabile locale
    // .......

    var_globale += var_locale;    // È possibile perchè var_globale è una
                                  // variabile globale
    // .......
}
```

Nel paragrafo sulle funzioni capiremo meglio il meccanismo di visibilità delle
variabili globali. In genere, per questioni di modularità del codice e
visibilità, è consigliabile usare le variabili globali solo quando è
strettamente indispensabile. Questo perché, proprio in virtù delle sue
proprietà, una variabile globale è modificabile da ogni funzione, e questo
potrebbe portare a malfunzionamenti nel programma, nel caso in cui una funzione
(che possiamo vedere come un 'pezzo' del programma) si trovi a lavorare su una
variabile modificata intanto da un'altra funzione, o da un altro processo
operante nello stesso programma.

## Variabili static e auto

Le variabili globali in genere sono statiche, ossia vengono instanziate in
memoria quando il programma viene chiamato e distrutte quando il programma viene
chiuso. Le variabili locali invece in genere sono automatiche, ossia vengono
instanziate quando la funzione che le dichiara viene invocata e vengono
distrutte quando la funzione chiamante è terminata. È però possibile stabilire
se una variabile deve essere static e automatica attraverso le keyword static e
auto. Esempio:

```c
// .....

auto int x = 7;                 // Variabile automatica

int main()  {
    static float pi = 3.14;     // Variabile statica
    // .....
}
```

Se una variabile è dichiarata come statica, questa viene instanziata e
inizializzata quando il programma viene avviato invece di essere creata quando
una funzione chiamante la dichiara e distrutta quando tale funzione termina. In
quanto tale, inoltre, il suo valore è lo stesso per tutte le parti del
programma.

## Costanti: l'istruzione #define e la keyword const

È possibile dichiarare anche delle costanti in C, o variabili a sola lettura,
delle variabili cioè che possono venire lette ma su cui non è possibile
scrivere. I modi sono due:

* Attraverso l'istruzione #define:

    ```c
    #include <stdio.h>
    /* Definisco la costante PI, che vale 3.14 */
    #define  PI  3.14

    int main()  {
        float area, raggio;
        // .....
        area = raggio*PI*PI;
        // .....
    }
    ```

* Attraverso la keyword const:

    ```c
    // .....
    const float pi = 3.14;
    // .....
    area = raggio*pi*pi;
    // .....
    ```

L'istruzione #define è, come la #include, un'istruzione al preprocessore. In
poche parole, quando il compilatore incappa in una #define, legge il valore
assegnato alla costante (anche se non è propriamente una costante, in quanto non
viene allocata in memoria), cerca quella costante all'interno del programma e
gli sostituisce il valore specificato in real-time. Ad ogni occorrenza di PI,
quindi, il preprocessore sostituisce automaticamente 3.14, senza andare a
cercare il corrispondente valore della variabile in memoria centrale.

Con la const, invece, creo una vera e propria variabile a sola lettura in modo
pulito e veloce, e per dichiarre una costante è di gran lunga preferito
quest'ultimo metodo.

Ovviamente una scrittura come questa darà un errore (o un warning, a seconda dei
compilatori):

```c
const float pi = 3.14;
pi += 1;
```

in quanto non è possibile modificare una variabile di sola lettura. gcc dà
quest'errore:

    error: increment of read-only variable ‘pi’

e in genere anche tutti i compilatori C++ danno un errore se si tenta di
modificare una variabile in sola lettura. Alcuni compilatori C potrebbero essere
meno fiscali e sollevare semplicemente un warning, ma, come è ovvio, non cambia
il fatto che questa pratica sia assolutamente da evitare.

## Variabili register e volatile

Le variabili vengono in genere allocate nella memoria RAM (sullo stack le
variabili locali statiche, sullo heap quelle dinamiche, nel segmento di memoria
DATA quelle globali). Ma in C è anche possibile allocare una variabile in un
registro del processore (in genere l'accumulatore su architetture x86, EAX)
attraverso la keyword register:

```c
register int var_reg = 3;
```

Può essere buona norma usare l'operatore register per _suggerire_ al compilatore
di salvare quella variabile in un registro del processore (ovviamente se quel
registro dovesse essere richiesto dal programma per salvare un'altra variabile
la variabile dichiarata come register viene “sfrattata” in memoria centrale),
cosa molto utile, ad esempio, nel caso in cui si debba accedere ripetutamente a
una stessa variabile nel contesto di un ciclo con molte iterazioni, in quanto
l'accesso a una variabile memorizzata in un registro è molto più veloce di un
accesso in memoria centrale e quindi ripetute letture della stessa variabile
vengono effettuate in tempo minore.

Dichiarando invece una variabile come volatile, questa variabile può venir
modificata da alti processi o da altre parti del programma in qualsiasi momento:

```c
volatile int vol_var;
```
