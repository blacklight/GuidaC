# Direttive per il preprocessore

Ogni compilatore traduce le istruzioni di un file sorgente in linguaggio
macchina. Il programmatore generalmente non è consapevole del lavoro del
compilatore: si fornisce delle istruzioni di un linguaggio di alto livello per
evitare le complessità gestionali del linguaggio macchina. Ma, comunque, è
importante poter comunicare con il compilatore. Il C fa uso del preprocessore
per estendere la sua potenza e la sua notazione, consentendo al programmatore
un'interazione con il compilatore. L'identificatore delle righe che riguardano
le direttive ad esso è #, che nel C ANSI può essere anche preceduto da spazi
mentre nel C Tradizionale deve trovarsi all'inizio della riga. Le direttive non
fanno comunque parte della grammatica del linguaggio, ampliano solo l'ambiente
di programmazione. Per lo standard ANSI, le direttive sono le seguenti:

```
#define  #error  #include  #elif    #if
#line    #else   #ifdef    #pragma  #endif
#ifndef  #undef  #warning
```

## La direttiva #include

Solitamente anche nei programmi più banali si usa la direttiva **#include** per,
appunto, includere nel sorgente file esterni o librerie.

Per includere una libreria si usano le parentesti angolari < e >, mentre per
includere un file esterno o magari nella stessa cartella del programma si usano
i doppi apici ".

Un esempio di inclusione di una libreria e un file che si trova nella cartella
superiore di dove si trova il sorgente in cui la includiamo:

```c
#include <stdio.h>
#include "../file1.h"
```

In questo caso il preprocessore quando incontrerà queste righe le sostituirà con
il contenuto del file richiamato.

In Unix solitamente i file d'intestazione specificati nelle parentesi angolari
si trovano nel percorso **/usr/include/**.

Nei file inclusi possono naturalmente anche esserci altre direttive al
preprocessore che verranno poi a loro volta "lavorate".

## La direttiva #define

La direttiva **#define** si usa, appunto, per definire qualcosa ad esempio:

```c
#define scrivi printf
```

In questo caso la definizione è **scrivi** che va a sostituire la parola
**printf** quindi nel corso del programma al posto di:

```c
printf("Ciao preprocessore!");
```

Si potrà scrivere

```c
scrivi("Ciao preprocessore!");
```

Comunque il define può anche definire numeri, simboli o altro. Vari esempi di
define:

```c
#define EQ  ==
#define OK  printf("OK\n");
#define DEBUG 1
```

Ogni tanto a un programmatore in C può scappare di mettere un solo = nelle
uguaglianze così con la definizione _EQ ==_ si potrà scrivere così:

```c
if ( a EQ b ) ...
```

evitando errori logici.

Un'altra cosa da notare è la definizione DEBUG molto utile nelle fasi di test di
un programma che si può usare nel controllo del flusso tramite sempre direttive
al preprocessore che vedremo adesso.

## Controllo del flusso

Con le direttive al preprocessore si può esegure anche un flusso del controllo (
_if_, _else_ ) utilizzando le direttive **#if**, **#else**, **#elif**,
**#endif** e **#ifdef**.

Iniziamo a spiegarli dai primi cioè **#if**, **#else**, **#elif** e **#endif** che corrispondo
al controllo del flusso normalmente utilizzato: **if**, **else**, **else if** mentre
l'ultimo **#endif** è "originale" del preprocessore.

Esempio:

```c
#include <stdio.h>
#define A 2
#define B 4

int main()  {
#if A == 2 || B == 2
    printf("A o B sono uguali a 2\n");
#elif A == 4 && B == 4
    printf("A e B sono uguali a 4\n");
#elif A != B
    printf("A è diversa da B\n");
#else
     printf("A e B sono di un valore non definito\n");
#endif

     return 0;
}
```

Si possono notare le seguenti cose:

* Le variabili su cui eseguire controlli devono essere definite tramite
  **#define**;
* Anche nel controllo del flusso tramite direttive al preprocessore si possono
  eseguire controlli con || ( OR ), && ( AND ) e != ( NOT ).
* La direttiva **#endif** "dice" al preprocessore che il controllo del flusso è
  finito.

Per eseguire un debug con questo sistema si potrebbe inserire qualcosa tipo:

```c
#if DEBUG 1
    printf("x = %d\n", x);
    printf("y = %s\n", y);
    ...
#endif
```

Ma ora vedremo con la direttiva **#ifdef** cosa si può fare, in pratica
"**ifdef**" sta per "**se definito**" quindi si può tramite essa controllare se
una variabile è stata definita o meno e con l'aggiunta delle direttive
**#undef** e **#ifndef** vedremo cosa si può fare con l'esempio seguente:

```c
#include <stdio.h>
#define  NUMERO 4
     
int main(void) {
#ifndef NUMERO
    #define NUMBER 4
#ifdef NUMBER
    #undef NUMBER
    #define NUMERO 4
#endif
    return 0;
}
```

Innanzitutto chiariamo cosa vuol dire _ifndef_ e _undef_, la prima equivale a
"**se non è definito**" ( _if not defined_ ) mentre la seconda equivale a
"**togli la definizione**" ( _undefine_ ).

Nell'esempio sopra definiamo **NUMERO** dopodichè all'interno del corpo main
iniziamo col verificare se non è definito numero, se ciò è vero definiamo
NUMBER, se invece è definito NUMBER togliamo la definizione di NUMBER e
definiamo NUMERO. Dopodichè si esce dal programma.

La direttiva **#undef** diciamo che è inutile nei piccoli programmi, ma risulta
utilissima nei programmi di grandi dimensioni composti magari da molti file e da
molte persone che ci lavorano e senza andare in giro o sfogliare tra i file se
una cosa è stata definita o meno questa semplice direttiva ci facilita la vita.

L'uso di #ifdef è utilissimo nel caso in cui si vogliano usare dei file header.
Infatti, un file header potrebbe essere incluso in due diversi file sorgenti che
si vanno a compilare insieme, e questo potrebbe generare ambiguità ed errori in
fase di compilazione (funzioni o dati che risulterebbero dichiarati due volte).
Per evitare questo problema si usano proprio le direttive al preprocessore.
Nell'header che andremo a creare avremo una cosa del genere:

* Se la variabile _NOMEHEADER_H non è definita
* Definisci la variabile
* Dichiara tutto il contenuto dell'header
* Altrimenti, termina la dichiarazione dell'header

In codice:

```c
#ifndef  _MIOHEADER_H
#define  _MIOHEADER_H
 
// Qui metto tutte le mie funzioni e i miei dati
 
#endif
```

In pratica, una volta definita la macro _MIOHEADER_H il file header non verrà
più incluso in nessun altro file, risolvendo quindi gli eventuali problemi di
header definiti due o più volte.

## Macro predefinite

Nel C esistono 5 tipi di macro già definite sempre disponibili che non possono
essere ridefinite dal programmatore. Si possono vedere nello schema seguente:

```c
/* MACRO    ||  COSA CONTIENE */
__DATE__   /* Una stringa che contiene la data corrente */
__FILE__   /* Una stringa che contiene il nome del file */
__TIME__   /* Una stringa che contiene l'ora corrente */
__LINE__   /* Un intero che raprresenta il numero di riga corrente */
__STDC__   /* Un intero diverso da 0 se l'implementazione segue lo standard ANSI
              C */
```

## Operatori # e ##

Questo tipo di operatori sono disponibili solo nel C ANSI. L'operatore unario #
trasforma un parametro formale di una definizione di macro in una stringa ad
esempio:

```c
#define nomi(a, b) printf("Ciao " #a " e " #b "! Benvenuti!\n");
```

da richiamare nel corpo main con:

```c
nomi(HdS619, BlackLight);
```

Una volta espanso dal preprocessore questa linea diventerà:

```c
printf("Ciao " "HdS619"  " e " "BlackLight" "! Benvenuti!\n");
```

Ora invece vediamo l'operatore binario ## che serve a concatenare token. Ad
esempio:

```c
#include <stdio.h>
#define X(y) x ## y

X(3) = X(4) = X(12) = ...
```

verrà espanso in:

```c
x3 = x4 = x12 = ...
```

In pratica si può pensare che "colleghi" i due parametri **x** e **y**.

## Direttive #error e #warning

Le direttive **#error** e **#warning** servono rispettivamente per dare errori
nella compilazione oppure avvisi. Solitamente queste due direttive vengono usate
insieme a quelle che controllano il "flusso di compilazione" ( #else, #if,
#undef, ecc... ). La loro sintassi è la seguente:

```c
#error   Messaggio di errore
#warning Messaggio di avvertimento
```

Ad esempio si può controllare che un codice in C++ venga compilato solo da un
compilatore C++ e non da un compilatore C nel seguente modo (ricordando che i
compilatori C++ definiscono la macro _\_\_cplusplus_):

```c
#ifndef __cplusplus
    #error "Devi compilare questo codice con un compilatore C++"
#endif
// Codice C++ di seguito
```

Compilando questo codice ad esempio con g++ non si avranno errori, mentre se si
prova a compilare con gcc si avrà

    error: #error "Devi compilare questo codice con un compilatore C++"
