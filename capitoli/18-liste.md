# Liste

Una lista è un insieme _finito e ordinato_ di elementi di un certo tipo. In
informatica una lista si indica come un insieme di termini compresi tra
parentesi quadre []. Esempio, ['a','n','c']. Come tutti i tipi di dato astratti,
anche le liste sono definite in termini di

* Dominio-base dei suoi elementi (interi, caratteri, stringhe...)
* Operatori di costruzione della lista
* Operatori di selezione sulla lista

Il grosso vantaggio delle liste sugli array è il fatto che una lista si può
definire in modo estremamente dinamico, anche senza conoscere il numero di
elementi totale di partenza dei suoi elementi, e di gestire i collegamenti tra
un elemento e un altro in modo estremamente versatile. Ma andiamo con ordine.

## Liste come tipi di dato astratto

Pochi linguaggi offrono di default il tipo 'lista' preimpostato (LISP, Prolog).
Negli altri linguaggi, come C, è necessario costruirsi questo tipo in base alle
proprie esigenze.

Le caratteristiche generali di un tipo di dato astratto sono state illustrate
sopra. In modo più preciso, possiamo definire un tipo di dato astratto in
termini di

* Dominio base D;
* Insieme di funzioni $\Phi = \{ F_{1} , ... , F_{n} \}$ sul dominio D;
* Insieme di predicati $\Pi  = \{ F_{1} , ... , F_{n} \}$ sul dominio D.

Un tipo di dato astratto generico _T_ è quindi definibile come

$T = \{ D ,  \Phi , \Pi \}$

Nel caso di una lista, possiamo definire

* $D = \{ qualsiasi tipo di dato \}$
* $\Phi = \{ cons, head, tail, emptylist \}$
* $\Pi = \{ empty \}$

Le funzioni base sulla lista sono così definite:

* _cons_ è il costruttore della lista, ovvero la funzione che, dato una lista di
  partenza e un elemento appartenente al dominio da inserire in cima alla lista,
  costruisce la lista specificata.
* _head_ è la funzione che ritorna la 'testa' della lista, ovvero il suo primo
  elemento.
* _tail_ è la funzione che ritorna la 'coda' della lista, ovvero una lista uguale
  a quella di partenza ma privata del primo elemento.
* _emptylist_ è una Funzione che ritorna la costante 'lista vuota'. Per
  convenzione, definiamo una lista come vuota quando il valore della sua testa è
  NULL.

L'unico predicato elementare sul tipo astratto di lista è così definito:

* _empty_: Funzione che verifica se la lista è vuota o meno.

Qualche esempio:

* `cons (5, [3,6,2,3])` crea la lista `[5,3,6,2,3]`;
* `head ([7,3,5,6])` ritorna `7` (testa della lista);
* `tail ([7,3,5,6])` ritorna la lista `[3,5,6]` (coda della lista);
* `empty ([7,3,5,6])` ritorna `falso` (la lista non è vuota).

Quelle illustrate sono le operazioni di base che si possono effettuare su una
lista. Tutte le altre operazioni (inserimento ordinato di elementi, ribaltamento
degli elementi, stampa degli elementi presenti...) sono operazioni derivate
dalle primitive appena illustrate. Considerando che esiste il concetto di lista
vuota (per convenzione la lista avente NULL in testa) e che è possibile
costruire nuove liste usando il costruttore _cons_, si possono definire tutte le
eventuali funzioni derivate sulla base di quelle già definite tramite _algoritmi
ricorsivi_.

## Rappresentazione statica

La rappresentazione più ovvia del tipo astratto di lista è gestendo gli elementi
della lista in un array. La lista così costruita conterrà

* Un vettore di lunghezza massima prefissata;
* Una variabile _primo_, che identifica l'indice del primo elemento della lista;
* Una variabile _lunghezza_, che indica il numero di elementi contenuti nella
  lista;

L'inconveniente principale è il fatto che le dimensioni del vettore sono fisse.
Il tipo di dato lista è quindi strutturato così in questo caso:

```c
#define   N   100
 
typedef struct  {
    int primo,lunghezza;
    int elementi[N];
} list;
```

E le primitive che agiscono sulla lista sono così definite:

```c
// Ritorna una lista vuota
list emptylist()  {
    list l;
     
    // Convenzione: quando la lista è vuota l'indice del primo elemento
    // è un numero negativo
    l.primo=-1;
    l.lunghezza=0;
}
 
// Controlla se la lista è vuota
bool empty(list l)  {
    return (l.primo==-1);
}
 
// Ritorna il primo elemento della lista
int head (list l)  {
    if (empty(l)) abort();
    return l.elementi[l.primo];
}
 
// Ritorna la coda della lista
list tail(list l)  {
    list t=l;
     
    // Se la lista è vuota, esce
    if (empty(l))  abort();
     
    // Altrimenti, la lista t avrà come primo elemento
    // il primo di l incrementato di 1, e la lunghezza
    // di l decrementata di 1 (ovvero scarto la testa della lista)
    t.primo++;
    t.lunghezza--;
    return t;
}
 
// Crea una nuova lista, prendendo come parametri
// l'elemento da inserire in testa e una lista di partenza
// (eventualmente vuota)
list cons (int e, list l)  {
    list t;
    int i;
     
    // Inserisco e in testa alla lista
    t.primo=0;
    t.elementi[t.primo]=e;
    t.lunghezza=1;
     
    // Copio il vettore contenuto in l nella nuova lista
    for (i=1; i<=l.lunghezza; i++)  {
        t.elementi[i]=t.elementi[i-1];
        t.lunghezza++;
    }
}
```

Queste sono le funzioni primitive sulla lista. Grazie a queste è possibile
costruire ricorsivamente eventuali funzioni derivate. Esempio, una funzione che
stampi tutti gli elementi della lista:

```c
void showList(list l)  {
    // Condizione di stop: se la lista è vuota, ritorna
    if (empty(l))
        return;
     
    // Stampa il primo elemento della lista
    printf ("%d\n",head(l));
     
    // Richiama la funzione sulla coda di l
    showList(tail(l));
}
```

## Rappresentazione dinamica

Una rappresentazione di liste estremamente utile è quella dinamica. In questo
tipo di rappresentazione si perde ogni riferimento statico (vettori, buffer di
dimensione fissa). Ogni elemento della lista contiene il suo valore e un
riferimento all'elemento successivo nella lista stessa. Si crea quindi così una
lista _grafica_, con *nodi* (elementi della lista) e *archi* (collegamenti tra
gli elementi).

Un generico elemento della lista sarà quindi così costruito:

```c
// Creo una lista di interi
// Nel caso volessi riutilizzare il codice per una lista
// di un altro tipo, mi basterà modificare il tipo element
typedef element int;
 
typedef struct list_element  {
    element value;
    struct  list_element *next;
} node;
```

Il tipo element mi consente di scrivere del codice estremamente modulare, in
quanto semplicemente modificando il tipo potrò usare la stessa lista per
memorizzare interi, float, caratteri e quant'altro. Come è possibile notare
inoltre nel dichiarare la struttura _node_ ho usato un'etichetta
(list\_element).  Ciò è indispensabile in quanto all'interno della struttura c'è
un collegamento a un elemento della struttura stessa (il prossimo elemento della
lista). Ma poiché _node_ non è ancora stato dichiarato a quel punto, è
indispensabile mettere un'etichetta temporanea. A questo punto, con un nodo
della lista così definito potrò includere al suo interno il suo stesso valore e
il riferimento al prossimo elemento. Nel caso l'elemento in questione sia
l'ultimo della lista, si mette come suo successore, per convenzione, il valore
NULL.

Per una maggiore genericità del codice possiamo creare funzioni che operano sul
tipo element, in modo che se in futuro dovessimo usare lo stesso tipo di lista
creato per gestire degli interi per gestire delle stringhe basterà cambiare
queste funzioni che agiscono su element, e lasciare inalterate le funzioni che
operano sulla lista. Si comincia così a entrare nell'ottica della creazione di
_codice modulare_ ovvero codice che è possibile scrivere una volta e riusare più
volte. Vediamo le funzioni di base che possono agire sul tipo element (in questo
caso tipo int, volendo modificando il tipo basterà cambiare le funzioni):

```c
bool isLess (element a, element b)  { return (a<b); }
bool isEqual (element a, element b)  { return (a==b); }
element get (element e)  { return e; }
 
element readElement()  {
    element e;
    scanf ("%d",&e);
    return e;
}
 
void printElement (element e)  { printf ("%d",e); }
```

A questo punto è conveniente dichiarare il tipo lista

```c
typedef node* list;
```

appunto come puntatore a un elemento di tipo _node_.

Per il tipo lista le primitive saranno le seguenti:

```c
// Ritorna la costante 'lista vuota'
list emptylist()  { return NULL; }
 
// Controlla se una lista è vuota
bool empty(list l)  { return (l==NULL); }
 
// Ritorna la testa della lista
element head (list l)  {
    if (empty(l))  abort();
    return l->value;
}
 
// Ritorna la coda della lista
list tail (list l)  {
    if (empty(l))  abort();
    return l->next;
}
 
// Costruttore. Genera una lista dato un elemento
// da inserire in testa e una lista
list cons (element e, list l)  {
    list t;
    t = (list) malloc(sizeof(node));
     
    t.value=get(e);
    t.next=l;
    return t;
}
```

Con queste primitive di base è possibile costruire qualsiasi funzione che operi
sul tipo di dato 'lista'. Esempio, per la stampa degli elementi contenuti nella
lista:

```c
void printList(list l)  {
    // Condizione di stop: lista vuota
    if (l==NULL)
        return;
     
    printElement(l->head);
    printf ("\n");
     
    // Scarto l'elemento appena stampato e
    // richiamo la funzione in modo ricorsivo
    printList(l->tail);
}
```

E allo stesso modo si possono anche definire per la ricerca di un elemento nella
lista, per la lettura di un elemento all'indice _i_ della lista e così via.


