# Tipi di dato derivati, enumerazioni e strutture

I tipi di dato su cui abbiamo operato finora erano tipi di dati _semplici_.
Abbiamo infatti operato su variabili sia scalari sia vettoriali, ma tutte
identificate univocamente da un tipo (una variabile int, una variabile float, un
array di int, un array di char...). Il C, al pari degli altri linguaggi di
programmazione ad alto livello, consente anche al programmatore di definire dei
propri tipi di dato e di operare su tipi di dato composti (appunto le strutture,
chiamate record nell'informatica teorica), ovvero tipi di dato composti da tipi
di variabili eterogenei.

## Definire propri tipi - L'operatore typedef

Accennavamo prima alla possibilità di poter definire propri tipi di dato in C, a
seconda delle esigenze del programmatore. Il C mette a disposizione l'operatore
_typedef_ per definire nuovi tipi a partire dai tipi primitivi già esistenti.

Sia ben inteso, non è indispensabile la definizione di nuovi tipi di dato in un
programma. Si possono benissimo manipolare dati primitivi anche in un programma
di grandi dimensioni, o usare strutture specificando in modo esplicito
l'etichetta _struct_, come vedremo in seguito, ma l'uso di typedef rende la
scrittura del programma più intuitiva (se un tipo di variabile la uso solo per
la temperatura posso chiamare il suo tipo _temp_, il che rende il suo uso più
intuitivo rispetto a un semplice float) e probabilmente più leggibile.

Esempio di utilizzo dell'operatore typedef: voglio creare un nuovo tipo di
variabile solo per misurare gli angoli in gradi, a partire dal tipo float.
Ricorrerò ad una scrittura del tipo

```c
typedef float degree;
```

Ora posso sfruttare il nuovo tipo nel programma:

```c
degree alpha=90;
 
printf ("L'angolo alpha è di %f gradi\n",alpha);
```

Un altro utilizzo molto comodo è per la definizione di nuovi dati di tipo
vettoriale. Ad esempio, so che un codice fiscale è sempre composto da 17
caratteri. Posso creare un nuovo tipo di dato dedicato alla memorizzazione dei
codici fiscali in questo modo:

```c
typedef char CF[17];

.........
 
CF c = "AAABBBCCCDDDEEEFF";
```

Le dichiarazioni dei nuovi tipi in genere vanno messe in modo da essere visibili
a tutte le funzioni del programma, quindi o al di fuori del main o in un file
header importato dall'applicazione.

## Enumerazioni

Le enumerazioni in C si dichiarano attraverso la keyword _enum_, e hanno
l'obiettivo di dichiarare nuovi tipi di dato con un dominio limitato, dove al
primo valore dell'enumerazione viene associato il valore 0, al secondo il valore
1 e così via.

Ad esempio, in C non ho di default un tipo di dato per poter operare su tipi
booleani. Posso però costruirmi un tipo di dato booleano grazie ad un
enumerazione:

```c
typedef enum { false, true } boolean;
```

In questo caso il primo campo dell'enumerazione è false, a cui viene attribuito
il valore 0, e il secondo è true, a cui quindi viene attribuito il valore 1.
Ora, grazie alla specifica typedef, posso usare questo tipo di dato all'interno
del mio codice:

```c
boolean trovato=false;
 
.........
 
if (valore1==valore2)
    trovato=true;
```

Altro esempio di enumerazione:

```c
typedef enum  {
    Lunedi,
    Martedi,
    Mercoledi,
    Giovedi,
    Venerdi,
    Sabato,
    Domenica
} giorni;
```

In questo caso Lunedi=0, Martedi=1, ..., Domenica=6. All'interno del mio codice
posso istanziare una variabile di questo tipo e sfruttarla così:

```c
giorno g1=Lunedi;
giorno g2=Martedi;
......
```

## Dati strutturati

Nella realtà di tutti i giorni abbiamo a che fare con entità descritte da più di
una caratteristica, e anche con tipi diversi di caratteristiche. Per soddisfare
questa esigenza, il C mette a disposizione i _tipi strutturati_. Esempio classico
di tipo strutturato: un'automobile è descritta da una targa, dall'anno di
immatricolazione, dalla casa produttrice e dal modello. Ecco come implementare
queste caratteristiche in C, creando il tipo di dato strutturato 'automobile':

```c
typedef struct  {
    char targa[16];
    char marca[16];
    char modello[16];
    int anno_imm;
} automobile;
```

Usando la keyword typedef, posso usare questo tipo di dato all'interno del mio
programma direttamente così:

```c
automobile a;
```

In alternativa, potevo specificare la struttura senza typedef:

```c
struct automobile  {
    char targa[16];
    char marca[16];
    char modello[16];
    int anno_imm;
};
// Ricordate sempre il ; finale
```

In questo caso, posso usare il tipo di dato strutturato all'interno del mio
programma ma specificando anche il fatto che faccio uso di un tipo di dato
strutturato dichiarato in precedenza:

```c
struct automobile a;
```

Per comodità e maggiore leggibilità del codice, in questo luogo useremo la prima
scrittura (quella con il typedef).

Per accedere ai dati contenuti all'interno di una struttura posso sfruttare
un'istanza della struttura stessa (ad esempio, nel caso di sopra, una variabile
di tipo 'automobile') e specificare il componente a cui voglio accedere separato
da un punto '.'. Esempio:

```c
typedef struct  {
    char targa[16];
    char marca[16];
    char modello[16];
    int anno_imm;
} automobile;
 
........
 
automobile a;
 
printf ("Inserisci la targa: ");
scanf ("%s",a.targa);
 
printf ("Inserisci la marca: ");
scanf ("%s",a.marca);
 
printf ("Inserisci il modello: ");
scanf ("%s",a.modello);
 
printf ("Inserisci l'anno di immatricolazione: ");
scanf ("%d",&a.anno_imm);
 
printf ("Targa: %s\n",a.targa);
printf ("Marca: %s\n",a.marca);
printf ("Modello: %s\n",a.modello);
printf ("Anno di immatricolazione: %d\n",a.anno_imm);
```

È possibile anche definire array di tipi strutturati, in questo modo:

```c
// Array di 10 automobili
automobile a[10];
int i;
 
for (i=0; i<10; i++)  {
    printf ("Automobile n.%d\n\n",i+1);
     
    printf ("Inserisci la targa: ");
    scanf ("%s",a.targa);
     
    printf ("Inserisci la marca: ");
    scanf ("%s",a.marca);
     
    printf ("Inserisci il modello: ");
    scanf ("%s",a.modello);
     
    printf ("Inserisci l'anno di immatricolazione: ");
    scanf ("%d",&a.anno_imm);
}
```

e ovviamente vale lo stesso discorso fatto con gli array di tipi primitivi per
quanto riguarda l'inizializzazione dinamica:

```c
// Puntatore alla struttura automobile
automobile *a;
int i,n;
 
printf ("Inserire i dati di quante automobili? ");
scanf ("%d",&n);
 
// Inizializzazione dinamica del vettore di automobili
a = (automobile*) malloc (n*sizeof(automobile));
 
for (i=0; i<n; i++)  {
    printf ("Automobile n.%d\n\n",i+1);
     
    printf ("Inserisci la targa: ");
    scanf ("%s",a.targa);
     
    printf ("Inserisci la marca: ");
    scanf ("%s",a.marca);
     
    printf ("Inserisci il modello: ");
    scanf ("%s",a.modello);
     
    printf ("Inserisci l'anno di immatricolazione: ");
    scanf ("%d",&a.anno_imm);
}
```

Posso anche dichiarare puntatori a strutture e accedere alle strutture stesse
tramite questi puntatori. In questo caso, invece del punto '.' per accedere ad
un certo elemento della struttura il C propone un operatore apposito,
l'operatore '->':

```c
// Puntatore a struttura
automobile *a;
 
printf ("Inserisci la targa: ");
scanf ("%s",a->targa);
 
printf ("Inserisci la marca: ");
scanf ("%s",a->marca);
 
printf ("Inserisci il modello: ");
scanf ("%s",a->modello);
 
printf ("Inserisci l'anno di immatricolazione: ");
scanf ("%d",&a->anno_imm);
 
printf ("Targa: %s\n",a->targa);
printf ("Marca: %s\n",a->marca);
printf ("Modello: %s\n",a->modello);
printf ("Anno di immatricolazione: %d\n",a->anno_imm);
```
