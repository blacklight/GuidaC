# Argomenti passati al main

Sappiamo che molte applicazioni accettano una lista di parametri passati
dall'utente in input. Ad esempio, il comando dir del DOS è in grado di accettare
alcuni parametri per configurarne il funzionamento (ad esempio dir /h e
simili...). Idem per net send (net send indirizzo\_host messaggio) e per, ad
esempio, il comando ls di Unix (ls -l -h). È possibile passare questi parametri
ad un programma sfruttando gli argomenti del main. Il main è una funzione come
tutte le altre, e quindi può anche ricevere argomenti in input. In particolare,
è possibile leggere i parametri eventuali passati ad un programma tramite l'uso
di argv, un vettore di stringhe da passare al main. Il numero di argomenti
passati viene invece salvato nella variabile intera argc. Esempio di utilizzo:

```c
#include <stdio.h>
 
main(int argc, char **argv)  {
    printf ("Nome dell'eseguibile in esecuzione: %s\n",argv[0]);
}
```

La prima stringa del vettore argv contiene infatti il nome dell'eseguibile
(quindi la variabile argc è sempre settata almeno a uno). Gli eventuali
argomenti successivi passati al programma vengono salvati in
argv[1],...,argv[n]. Esempio pratico:

```c
#include <stdio.h>
 
main(int argc, char **argv)  {
    int i;
     
    printf ("Argomenti passati al programma:\n");
     
    for (i=1; i<argc; i++)
        printf ("%s\n",argv[i]);
}
```

Se compiliamo questo eseguibile come 'stampa\_arg' e lo invochiamo con gli
argomenti "Ciao mondo come stai", così (in ambiente Unix):

    ./stampa_arg Ciao mondo come stai

avremo come output qualcosa del tipo

    Argomenti passati al programma: Ciao mondo come stai
