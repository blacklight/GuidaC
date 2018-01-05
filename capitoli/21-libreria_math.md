# Libreria math.h

Includendo nel proprio codice l'header math.h è possibile utilizzare svariate
funzioni e costanti matematiche. A gcc bisogna passare l'opzione _-lm_ per poter
compilare sorgenti che fanno uso di tali funzioni. Ecco le principali:

## Funzioni trigonometriche

* _cos_ Calcola il coseno di un numero reale (espresso in radianti);
* _sin_ Calcola il seno di un numero reale (espresso in radianti);
* _tan_ Calcola la tangente di un numero reale (espresso in radianti);
* _acos_ Calcola l'arcocoseno di un numero reale;
* _asin_ Calcola l'arcoseno di un numero reale;
* _atan_ Calcola l'arcotangente di un numero reale.

## Funzioni iperboliche

* _cosh_ Calcola il coseno iperbolico di un numero reale;
* _sinh_ Calcola il seno iperbolico di un numero reale;
* _tanh_ Calcola la tangente iperbolica di un numero reale.

## Funzioni esponenziali e logaritmiche

* _exp_ Calcola l'esponenziale di un numero reale;
* _log_ Calcola il logaritmo in base e di un numero reale;
* _log10_ Calcola il logaritmo in base 10 di un numero reale.

## Potenze e radici

* _pow_ Calcola una potenza. Prende come primo argomento la base e come secondo
  l'esponente;
* _sqrt_ Calcola la radice quadrata di un numero reale.

## Arrotondamento e valore assoluto

* _ceil_ Approssima per eccesso un numero reale al numero intero più vicino;
* _abs_ Calcola il valore assoluto di un numero reale;
* _floor_ Approssima per difetto un numero reale al numero intero più vicino.

## Costanti

L'header math.h mette anche a disposizione del programmatore alcune costanti
matematiche di uso comune con un numero notevole di cifre significative dopo la
virgola, senza che ci sia bisogno di definirle di volta in volta. Tra queste il
_pi greco_ (**M\_PI**) e il _numero di Nepero_ e (**M\_E**).

## Generazione di numeri pseudocasuali

In C è possibile generare numeri pseudocasuali in modo relativamente semplice, a
patto che si includa l'header stdlib.h. Si comincia inizializzando il seme dei
numeri casuali tramite la funzione **srand()**. In genere si usa come variabile
di inizializzazione del seme la data locale:

```c
#include <stdlib.h>
#include <time.h>
 
...
 
srand((unsigned) time(NULL));
```

Una volta inizializzato il seme uso la funzione **rand()** per ottenere un
numero pseudocasuale. Tale funzione ritorna però numeri estremamente grandi. Per
restringere l'intervallo possibile dei numeri pseudocasuali che voglio generare
basta calcolarne uno con _rand()_ e poi calcolarne il modulo della divisione per
il numero più alto dell'intervallo che voglio ottenere. Ad esempio, se voglio
ottenere numeri pseudocasuali in un intervallo da 0 a 9 basterà

```c
int rnd=rand()%10;
```
