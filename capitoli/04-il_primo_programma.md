# Il primo programma

Il primo programmino in C sarà un programma abbastanza semplice, che stampa
sullo schermo della console "Hello world!" ed esce. Vediamo il codice:

```c
/* hello.c */

#include <stdio.h>

int main(void)  {
      printf ("Hello world!\n");
        return 0;
}
```

Una volta scritto questo codice con il nostro editor preferito, salviamolo come
hello.c e compiliamolo con il nostro compilatore. Se usiamo gcc basta dare da
riga di comando, nella directory in cui è stato salvato il file sorgente, il
seguente comando:

    gcc -o hello hello.c

Quando lo eseguiamo (ovviamente in modalità console) apparirà la scritta "Hello
world!". Ma vediamo cosa fa nel dettaglio...

Innanzitutto, la prima riga è un commento. I commenti in C iniziano con /* e
finiscono con */, ma la maggior parte dei compilatori riconoscono anche i
commenti in stile C++ (che iniziano con // e finiscono con la fine della riga).
Esempio:

```c
codice
codice  /* Questo è un commento in stile C */
codice  /* Anche questo è
           un commento in stile C */
codice  // Questo è un commento in stile C++
```

gcc e la maggior parte dei compilatori C moderni dovrebbero riconoscere senza
problemi anche i commenti C++ (che personalmente reputo più comodi e leggibili
nella maggior parte dei casi). Tuttavia nel caso in cui si voglia scrivere
codice aderente al 100% agli standard del linguaggio tali commenti sarebbero da
evitare. Questo è l'output di gcc nel caso in cui si compili un codice
contenente al suo interno commenti in stile C++ usando le opzioni del
compilatore -Wall -pedantic:

    warning: C++ style comments are not allowed in ISO C90

Un _warning_ è comunque un “avvertimento” del compilatore che non compromette la
fase di compilazione vera e propria, ovvero la creazione dell'eseguibile, a
differenza di un _errore_.

All'interno di un commento è possibile scrivere informazioni sul programma, o
commenti su un passaggio di codice eventualmente poco chiaro, o note sul
copyright e l'uso del programma (spesso piazzate in commenti in testa ai file
sorgenti), o annotazioni su modifiche da apportare in seguito a un certo
spezzone di codice. È anzi buona norma commentare il più possibile un programma,
specie se il proprio codice dovrà essere esaminato da qualcun altro a termine,
dalla comunità open source, dai propri studenti, dai propri compagni di studio
in un contesto accademico, o dai propri colleghi di lavoro in un contesto
lavorativo.

La prima vera e propria linea di codice è `#include <stdio.h>`: come ho
accennato nel paragrafo precedente, questa è una direttiva al preprocessore,
ovvero un'istruzione che dice al calcolatore che nel programma che segue si
useranno le funzioni definite nel file stdio.h (i file header dovreste trovarli
nella cartella _include_ del vostro compilatore). Il file stdio.h contiene le
funzioni principali per lo STanDard Input/Output, ossia le funzioni che
permettono, ad esempio, di scrivere messaggi in modalità testo, di leggere
valori dalla tastiera, di manipolare files e buffer ecc.

Se non includessimo questa istruzione non potremmo usare la funzione printf()
più avanti (o meglio gcc potrebbe riconoscere, nel caso di printf(), una
chiamata a una funzione nella libreria standard e compilare lo stesso, al
massimo sollevando un warning per l'uso di una funzione della libreria standard
senza aver incluso l'header corrispondente, ma è buona abitudine includere
_sempre_ tutti i file header contenenti le entità che si usano nei propri
sorgenti per evitare di incappare in errori).

A questo punto inizia il programma vero e proprio: viene eseguito tutto ciò che
si trova all'interno della funzione main() (la funzione principale di ogni
programma), che inizia con { e finisce con }.  L'int situato prima del main()
dice al chiamante (in questo caso il sistema operativo stesso) che la funzione
main() ritornerà un numero intero quando sarà terminata (nel nostro caso,
attraverso _return_ 0 ritorniamo questo valore).

A questo punto chiamiamo la funzione printf(), definita in stdio.h.  Questa
funzione stampa un messaggio sullo standard output (la console, il prompt dei
comandi, un emulatore di terminale, una console virtuale, a seconda di dove si
esegue). Il messaggio è racchiuso fra parentesi tonde e i doppi apici "". La
sequenza \\n è una sequenza di escape, che dice di andare a capo dopo aver
scritto ciò che è contenuto nella printf() (\\n sta per "new-line"). Ecco le
principali sequenze di escape usate nel C:

* \\n Va a capo (new line)
* \\t Va avanti di una tabulazione (tasto TAB)
* \\b Va indietro di un carattere (tasto BACKSPACE)
* \\a Fa emettere un BEEP allo speaker interno (ALARM)
* \\" Stampa i doppi apici ""
* \\' Stampa un apice singolo

Piccola nota: tutte le istruzioni del C finiscono con un punto e virgola; tale
convenzione è adottata da diversi linguaggi di alto livello (Perl, PHP, Java,
Pascal...) e non da altri (Python, Ruby, Basic...).

L'istruzione return 0, come ho già detto prima, dice al programma di ritornare
il valore 0 (intero) al sistema operativo e uscire. Il suo perché sarà più
chiaro quando tratteremo nello specifico le funzioni.
