# Input da tastiera

Finora abbiamo preso in esame programmi che eseguono delle istruzioni ed escono.
Ma un programma non ha molto senso se non può interagire con l'utente che lo
usa. Il C mette a disposizione molte funzioni per l'I/O (Input/Output) da
tastiera (quelle che useremo sono definite perlopiù in in stdio.h). Abbiamo già
incontrato la printf() per l'output sul monitor, ora facciamo conoscenza con la
scanf(), per la lettura di valori dalla tastiera. Ecco la forma della scanf():

```c
scanf ("tipo_da_leggere",&variabile);
```

Ed ecco un piccolo esempio:

```c
int a;                  // Dichiaro una variabile int

printf ("Inserisci una variabile intera: ");
scanf ("%d",&a);     // Dico al programma di leggere
                     // il valore immesso
```

Ecco nel frammento di programma di sopra cosa succede: Attraverso la scanf()
dico al programma di leggere un valore intero dalla tastiera (già abbiamo visto
che la sequenza %d dice al programma che quella che si sta per leggere o
scrivere è una variabile intera) e di salvare questo valore all'indirizzo della
variabile a (capiremo meglio questo concetto quando parleremo dei puntatori),
ossia copio questo valore nella variabile intera a.

Ecco un programmino facile facile che somma fra loro due numeri reali presi da
tastiera:

```c
/* somma.c */

#include <stdio.h>

// Prototipo della funzione somma()
double somma(double a, double b);

int main() {
    double a,b;           // Dichiaro 2 variabili double

    printf ("Inserire il primo numero: ");
    // Leggo il primo valore double e lo salvo all'indirizzo di a
    scanf ("%f",&a);

    printf ("Inserire il secondo numero: ");
    // Leggo il secondo valore double e lo salvo all'indirizzo di b
    scanf ("%f",&b);

    // Stampo la somma fra a e b
    printf ("Somma fra %f e %f = %f\n", a, b, somma(a,b));
    return 0;
}

double somma(double a, double b)  {
    return a+b;
}
```

Ecco invece un programmino che stampa l'area e la lunghezza di una circonferenza
dato il raggio:

```c
/* circ.c */

#include <stdio.h>
#include <math.h>
// Includo il file math.h per poter usare
// la costante M_PI (pi greco)

double area(double raggio);
double circ(double raggio);

int main() {
    double r;             // Raggio

    printf ("Inserire il valore del raggio: ");
    scanf ("%f",&r);     // Leggo il valore del raggio

    printf ("Area: %f\n",area(r));
    printf ("Circonferenza: %f\n",circ(r));

    return 0;
}

double area(double raggio)  {
    return M_PI*raggio*raggio;    // pi*r²
}

double circ(double raggio)  {
    return 2*M_PI*raggio;         // 2pi*r
}
```

Ho incluso il file math.h perché in questo file è già definita la costante M\_PI
(pi greco) con 20 cifre di precisione dopo la virgola.
