# Gli array

Gli array, o vettori, sono le strutture di dati più elementari in informatica,
del tutto simili ai vettori trattati dall'algebra lineare.

## Array monodimensionali

Si tratta di un insieme di variabili dello stesso tipo e accomunate dallo stesso
nome (il nome del'array). Ciò che distingue un elemento dell'array da un altro è
l'indice, ovvero il suo numero, la sua posizione all'interno dell'array.
Possiamo immaginare un array come una cassettiera: per sapere dove mettere le
mani per trovare qualcosa ci serve il numero del cassetto dove cercare (prima
cassetto, secondo cassetto...). Così, un array è una raccolta di variabili dello
stesso tipo sotto lo stesso nome dove ogni variabile è un "cassettino"
identificato da un numero. Ecco come si dichiara un array in C:

```c
tipo  nome_array[quantità];
```

Esempio:

```c
int mio_array[10];
```

dichiara un array di 10 variabili int (N.B. da 0 a 9, non da 1 a 10!) chiamato
mio\_array. Se voglio cambiare un valore qualsiasi di questo array, basterà fare
così:

```c
mio_array[0] = 3;    // Il primo valore ora vale 3
mio_arrar[1] = 2;    // Il secondo valore vale 2
.......
```

Ovviamente posso anche leggere da tastiera il valore di un elemento dell'array:

```c
printf ("Inserisci il valore del primo elemento: ");
scanf("%d",&mio_array[0]);    // Leggo il valore del primo elemento

printf ("Il primo elemento vale %d\n",mio_array[0]);\
```

Posso anche leggere tutti i valori e poi stamparli tramite un ciclo for:

```c
int main()  {
    int mio_array[10];
    int i;
     
    for (i=0; i<10; i++)  {             // Per i volte...
        printf ("Elemento n.%d: ",i);

        // Leggo un valore int dalla tastiera
        // e lo memorizzo nell'elemento numero
        // i dell'array.
        scanf("%d",&mio_array[i]);
    }
     
    for (i=0; i<10; i++)
        // Stampo tutti i valori contenuti nell'array
        printf ("Elemento n.%d: %d\n",i,mio_array[i]);

    return 0;
}
```

Un array può anche essere dichiarato in modo esplicito con il suo contenuto:

```c
int v[] = {2,4,6,2,6,5};
```

Vediamo ora un esempio più utile: un programma che calcola la media aritmetica
di 5 numeri:

```c
int main()  {
    float numeri[5];     // Array di 5 float
    float med=0;         // Media aritmetica
    int i;               // Variabile contatore
     
    for (i=0; i<5; i++)  {
        printf ("Valore n.%d: ",i);
        scanf ("%f",&numeri[i]);
        med += numeri[i];  // Sommo fra loro tutti i numeri
        nell'array
    }
     
    // Divido la somma dei numeri per la loro
    quantità (5)
    med /= 5;
     
    printf ("Media aritmetica: %f\n",med);
    return 0;
}
```

Ancora un altro esempio, assimilabile all'algebra lineare vera e propria: un
programmino che effettua il prodotto scalare tra due vettori (ricordo che dati
due vettori v1 e v2 entrambi di n elementi il loro prodotto scalare è un numero
uguale a v1[0]\*v2[0] + v1[1]\*v2[1] + ... + v1[n-1]\*v2[n-1]), dove gli
elementi di entrambi i vettori sono stabiliti dall'utente via input:

```c
#include <stdio.h>
 
// Dimensione dei due vettori
#define  N  5
 
int main()  {
    int v1[N],v2[N];
    int i;
    int prod=0;
     
    for (i=0; i<N; i++)  {
        printf ("Elemento %d del primo vettore: ",i+1);
        scanf ("%d",&v1[i]);
    }
     
    for (i=0; i<N; i++)  {
        printf ("Elemento %d del secondo vettore: ",i+1);
        scanf ("%d",&v2[i]);
    }
     
    for (i=0; i<N; i++)
        prod += (v1[i]*v2[i]);
     
        printf ("Prodotto scalare dei due vettori: %d\n", prod);
    return 0;
}
```

## Matrici e array pluridimensionali

Negli esempi riportati sopra sono sempre presi in esame array a una dimensione,
ovvero array dove ogni elemento è definito univocamente da un solo indice, che
identifica la loro posizione all'interno dell'array stesso. Il C, al pari degli
altri linguaggi di programmazione ad alto livello, mette anche a disposizione la
possibilità di usare array a più dimensioni. Ci soffermeremo in particolar modo
sugli array bidimensionali (dato che array di dimensioni superiori sono usati
molto di rado), meglio conosciuti come _matrici_.

Una matrice si dichiara esattamente come un array monodimensionale, ma
specificando sia il numero di righe che di colonne al suo interno:

```c
int matrix[2][2];  // Dichiara una matrice di interi 2x2
```

La lettura e la scrittura su questi elementi vengono effettuate in modo molto
simile agli array, ma con due indici, in modo da gestire sia il numero di righe
che di colonne della matrice:

```c
int matrix[2][2];
int i,j;
 
// Leggo i valori della matrice da input
for (i=0; i<2; i++)
    for (j=0; j<2; j++)  {
        printf ("Elemento [%d][%d]: ",i+1,j+1);
        scanf ("%d",&matrix[i][j]);
    }
 
// Stampo i valori della matrice
for (i=0; i<2; i++)
    for (j=0; j<2; j++)
        printf ("Elemento in posizione [%d][%d]: %d\n",i+1,j+1,matrix[i][j]);
```
