# Controllare il flusso di un programma

I programmi visti finora eseguono tutti un blocco di istruzioni all'interno del
main(), o comunque all'interno di una funzione, ed escono. Abbiamo visto che è
anche possibile interagire con il programma, ma ci manca ancora qualcosa: ci
mancano gli strumenti per gestire il flusso di un programma, che esamineremo in
questo paragrafo.

## Costrutti if-else

I cicli if-else (in inglese "se-altrimenti") sono la struttura per il controllo
del programma più semplice messa a disposizione dai linguaggi di programmazione:
questa struttura definisce il codice da eseguire se una data condizione si
verifica e quello da eseguire se questa condizione non si verifica. La sua
sintassi è la seguente:

```c
if (condizione) {
    codice
        codice
}

else {
    codice
        codice
}
```
Esempio: prendiamo un frammento di codice che stabilisce se un numero intero n è
positivo o negativo facendo uso del costrutto if-else:

```c
int n;  // Dichiaro n

........

if (n>0) {
    printf ("n è positivo\n");    // Se n è maggiore di zero, allora è
    positivo
} else {
    printf ("n è negativo\n");    // Altrimenti, è negativo
}
```

Se un'istruzione if o else (o qualsiasi altro costrutto che vedremo in questo
paragrafo) contiene una sola istruzione (come nel caso di sopra) si possono
omettere le parentesi graffe {}:

```c
int n;

........

if (n>0)
    printf ("n è positivo\n");
    else
    printf ("n è negativo\n");
```

Dopo un'istruzione if non sempre è necessaria un'istruzione else: ecco un modo
abbastanza interessante per scrivere il frammento di codice riportato sopra:

```c
if (n>0)  {
    printf ("n è positivo\n");
    return 0;  // Esco dalla funzione
} else
    printf ("n è negativo\n");
    // Questa istruzione verrà eseguita se e soltanto se
    // n è negativo, perchè se è positivo ricade nel costrutto
    // if di sopra, che esce dalla funzione
```

Se qualcuno di voi ha programmato in Pascal, in BASIC, in Bash o in linguaggi
simili avrà notato che il costrutto if del C (e dei linguaggi da esso derivati,
C++, Java, Perl) manca della keyword then ("allora") usata in questi linguaggi,
in quanto ridondante e inutile (bastano le parentesi graffe per stabilire dove
il costrutto inizia e dove finisce).

## Operatori di confronto

Abbiamo incontrato, negli esempi sopra, il simbolo di maggiore > , usato per
stabilire se un valore è maggiore di un altro. Ovviamente, abbiamo anche il
simbolo di minore < usato per il caso contrario. Ecco i principali operatori di
confronto usati nel C:

Operatore  Significato
---------- ------------
>          Maggiore
<          Minore
>=         Maggiore o uguale
<=         Minore o uguale
!=         Diverso
==         Uguale (Attenzione: è diverso da =)

Il simbolo == sta per "uguale" come confronto. Se ad esempio vogliamo sapere se
una variabile vale 3, scriveremo:

```c
if (a==3)       // NON a=3!!!
```

è invece un errore comune scrivere, nei confronti,

```c
if (a=3)
```

attenzione: la scrittura di sopra fa semplicemente l'assegnamento di un valore
alla variabile a. Sappiamo che il ciclo if è verificato se la condizione al suo
interno è vera, viene ignorato quando la condizione è falsa. Il C prende come
convenzione _vero_ qualsiasi valore diverso da zero, falso qualsiasi valore
uguale a zero. Il codice di sopra non fa altro che assegnare un valore alla
variabile a ed entrare nel ciclo se il valore di a è diverso da zero (come in
quest'esempio), ignorarlo in caso contrario. Il che è leggermente diverso dal
fare un confronto, come volevamo noi...

In definitiva, l'uguale singolo = viene usato per gli assegnamenti (ad esempio
"a=2") mentre quello doppio == per i confronti (nel Pascal invece si usa = per i
confronti e := per le assegnazioni).

## Operatori logici

Vediamo ora i principali operatori logici usati dal C. Facciamo prima un ripasso
di logica: date due o più proposizioni logiche è possibile fare 4 operazioni
fondamentali fra loro: la congiunzione (AND), la disgiunzione (OR), la
disgiunzione esclusiva (XOR) e la negazione (NOT). Quando parliamo di
proposizioni logiche parliamo di una qualsiasi affermazione che può essere vera
o falsa. La congiunzione (AND) di due proposizioni è vera se e soltanto se
entrambe le proposizioni sono vere. Ad esempio, in logica posso dire "fuori
piove E Marco è uscito" solo se fuori piove E Marco è uscito, ossia solo se
entrambi gli eventi sono veri. Con la disgiunzione (OR) basta invece che solo
uno dei due eventi sia vero per rendere l'operazione vera. La disgiunzione
esclusiva (XOR) invece richiede che un evento sia vero e l'altro sia falso per
essere vera. La negazione (NOT) è, lo dice il nome stesso, la negazione di una
proposizione. Se la proposizione è vera, la proposizione negata è falsa. Se
"fuori piove" è una proposizione vera, "fuori non piove" è una proposizione
falsa. Per maggiori delucidazione, ecco le tabelle di verità (le tabelle delle 4
operazioni logiche fondamentali), dove 0 sta per falso e 1 per vero (così come
la vede la macchina. a e b sono le due proposizioni logiche su cui voglio
operare):

 a   b   a AND b
--- --- ---------
 1   1      1
 1   0      0
 0   1      0
 0   0      0

 a   b   a OR b
--- --- ---------
 1   1      1
 1   0      1
 0   1      1
 0   0      0

 a   b   a XOR b
--- --- ---------
 1   1      0
 1   0      1
 0   1      1
 0   0      0

 a   NOT a
--- -------
 1   0
 0   1

A cosa ci possono servire questi rudimenti di logica per la programmazione in C?
È presto detto. Sappiamo che un computer ragiona con una logica binaria; nel
processore tutte le istruzioni che noi mettiamo in una programma diventano, a
livello logico-elettronico, delle semplici operazioni logiche, AND, OR, XOR e
NOT. In particolare, in C useremo perlopiù tali operatori per descrivere meglio
le condizioni all'interno di certi confronti. Ecco come si scrivono in C le
operazioni logiche:

Operazione  Scrittura in C
----------- ---------------
AND         &&
OR          ||
XOR         ^
NOT         !

Vediamo qualche applicazione pratica: un frammento di codice che stabilisce se
un numero è compreso fra 0 e 10. Senza operatori logici lo scriveremo così:

```c
if (n>0)  {
    if (n<10)
        printf ("n è compreso fra 0 e 10\n");
    else
        printf ("n è maggiore di 10\n");
} else
    printf ("n è minore di 0\n");
```

Con l'operatore logico AND scriveremo così:

```c
if ((n>0) && (n<10))  {
    printf ("n è compreso fra 0 e 10\n");
}
```

Ossia: se n è maggiore di 0 E contemporaneamente n è minore di 10, allora n è
compreso fra 0 e 10. Facciamo ora un esempio con l'OR: un programma che
stabilisce se un numero è minore di 0 OPPURE maggiore di 10 (il contrario
dell'intervallo che abbiamo visto sopra):

```c
if ((n<0) || (n>10))
      printf ("n è minore di 0, oppure n è maggiore di 10\n");
```

Ossia: controlla se n è minore di 0 OPPURE è maggiore di 10. Ragionamento simile
anche per lo XOR. Lo XOR è un'operazione logica molto usata in Assembly, in
quanto fare lo XOR di un registro con se stesso equivale a svuotare il registro.
Il NOT viene invece usato per sostituire scritture ridondanti come n==0 o n!=0:
infatti una variabile negata è sempre 0:

```c
if (n)  // Equivale a scrivere if (n!=0)
    printf ("n è diverso da 0\n");
if (!n) // Se "NOT n". Equivale a scrivere if (n==0)
    printf ("n è uguale a 0\n");\
```
Gli operatori logici possono anche essere usati fra variabili, consentendo
quindi di effettuare operazioni logiche fra numeri a livello di bit. Occhio che
in questo caso l'AND si scrive come '&', l'OR come '|' e il NOT, che diventa
“operatore di complemento” (ovvero prende il complementare di ogni bit, ad
esempio trasformando 0110 in 1001) si sc rive come '~'.

```c
int a=0xa0a1a2a3;
int b = a & 0x0000ff00;  // Fa un AND che azzera tutti i byte tranne il
penultimo -> b = 0x0000a200

// O anche, esempio più immediato:

char a=3;         // a = 00000011
char b=5;         // b = 00000101
char c = a & b;  // c = 00000001 = 1

// O ancora:

char a=3;         // a = 00000011
char b=5;         // b = 00000101
char c = a | b;  // c = 00000111 = 7
char a=7;   // a = 00000111
char b=~a;  // b = 11111000
```

Un'altra operazione logica messa a disposizione dal C è lo SHIFT.

Immaginiamo di avere una variabile int i = 4; scritta in binario (facciamo per
comodità a 4 bit) sappiamo che equivale a 0100. Fare uno shift a sinistra di 1
bit (la scrittura in questo caso è <<) equivale a spostare tutti i bit di un
posto a sinistra: la nostra variabile binaria da 0100 diventa quindi 1000,
quindi i da 4 diventa per magia 8. Una cosa degenere in C si scrive così:

```c
int i = 4;
i = i << 1;     // Faccio lo shift a sinistra di 1 bit
```

C'è anche lo shift a destra, il simbolo è >>. Ad esempio, se facciamo uno shift
a destra di 1 bit di i, questa variabile da 0100 diventa 0010, quindi da 4
diventa 2:

```c
int i = 4;
i = i >> 1;     // Faccio lo shift a destra di 1 bit
```

Risulta immediato quanto può essere comodo lo switch per calcolare le potenze
del 2. Se voglio calcolare 2^n infatti posso scrivere semplicemente 1 << (n-1).
Pensateci un attimo e capirete perché.

## Costrutti switch-case

Le strutture switch-case sono un modo più elegante per gestire un numero
piuttosto alto di costrutti if-else. Prendiamo un programmino che riceve in
input un carattere e stabilisce se il carattere è 'a','b','c','d','e' oppure è
diverso da questi cinque. Con l'if-else scriveremmo una roba del genere:

```c
char ch;        // Carattere

printf ("Inserisci un carattere: ");
scanf ("%c", &ch);

if (ch=='a')
    printf ("Hai digitato a\n");
else {
    if (ch=='b')
        printf ("Hai digitato b\n");
    else {
        if (ch=='c')
            printf ("Hai digitato c\n");
        else {
            if (ch=='d')
                printf ("Hai digitato d\n");
            else {
                if (ch=='e')
                    printf ("Hai digitato e\n");
                else
                    printf ("Non hai digitato un carattere compreso fra ");
                    printf ("a ed e\n");
            }
        }
    }
}
```

Tale scrittura non è certo il massimo della leggibilità. Vediamo ora lo stesso
frammento di programma con una struttura switch-case:

```c
char ch;

printf ("Inserisci un carattere: ");
scanf ("%c",&ch);

switch(ch)  {                   // Ciclo switch per la variabile ch
    case 'a':             // Nel caso ch=='a'...
        printf ("Hai digitato a\n");
        break;              // Interrompe questo case

    case 'b':
        printf ("Hai digitato b\n");
        break;

    case 'c':
        printf ("Hai digitato c\n");
        break;

    case 'd':
        printf ("Hai digitato d\n");
        break;

    case 'e':
        printf ("Hai digitato e\n");
        break;

        // Nel caso il valore di ch non sia
        // uno di quelli sopra elencati...
    default:
            printf ("Non hai digitato un carattere compreso fra a ed e\n");
            break;
}  // Fine
```

Metodo molto più pulito ed elegante. La struttura di uno switch-case è la
seguente:

```c
switch(variabile)  {
    case val_1:
        codice
            break;
    case val_2:
        codice
            break;
        ...........
    case val_n:
            codice
                break;
    default: // La clausola di default non è obbligatoria
                codice
                break;
}
```

Ogni etichetta case va interrotta con la clausola break, che interrompe lo
switch-case e ripassa il controllo al programma.

## Cicli iterativi - Istruzione for

Immaginiamo di voler far ripetere al nostro programma un blocco di istruzioni
per un tot numero di volte. Immaginiamo ad esempio un programmino che stampi
diceci volte "Hello world!". Con le conoscenze che abbiamo finora, scriveremmo
un lavoro del genere:

```c
int main()  {
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
    printf ("Hello world!\n");
}
```
Il che è decisamente scomodo. Per evenualità di questo tipo ci viene in aiuto il
ciclo for, che ha la seguente sintassi:

```c
for (variabile_1=valore1, ..., variabile_n=valore_n; condizione; step)  {
      codice
}
```

Dove variabile\_1,...,variabile\_n sono le cosiddette _variabile contatori_,
condizione è una condizione booleana che stabilisce il numero di cicli da
eseguire (ovvero, finché la condizione è vera esegui il ciclo for) e step
l'eventuale incremento o decremento da far subire alle variabili contatore ad
ogni ciclo.

Esempio chiarificatore: ecco il programmino di sopra scritto con un ciclo for:

```c
int main()  {
    int i;                        // Variabile "contatore"

    for (i=0; i<10; i++)
        printf ("Hello world!\n");

    return 0;
}
```

Dove la variabile contatore è i, e viene inizialmente posta, all'interno del
ciclo for, uguale a 0. La condizione è i<10, ovvero _finché la variabile i è
minore di 10 esegui il ciclo_, lo step invece è i++, ovvero 'ad ogni ciclo
incrementa la variabile i (_finché, ovviamente, non varrà 10 e il ciclo può
ritenersi concluso_).

Ecco un altro esempio chiarificatore:

```c
int main()  {
    int i;

    for (i=0; i<10; i++)
        printf ("Valore di i: %d\n",i);

    return 0;
}
```

Ecco l'output di questo programmino:

```
Valore di i: 0
Valore di i: 1
Valore di i: 2
Valore di i: 3
Valore di i: 4
Valore di i: 5
Valore di i: 6
Valore di i: 7
Valore di i: 8
Valore di i: 9
```

Ovviamente, il ciclo for di sopra si può scrivere in moltissimi modi:

```c
for (i=1; i<=10; i++)
      printf ("Valore di i: %d\n",i);
```

In questo caso, i ha come valore iniziale 1 e il ciclo termina quando i è
esattamente uguale a 10. In questo caso l'output sarà:

```
Valore di i: 1
Valore di i: 2
Valore di i: 3
Valore di i: 4
Valore di i: 5
Valore di i: 6
Valore di i: 7
Valore di i: 8
Valore di i: 9
Valore di i: 10
```

Altro esempio:

```c
for (i=10; i>0; i--)
      printf ("Valore di i: %d\n",i);
```

In questo caso, i ha come valore iniziale 10, viene decrementata di un'unità ad
ogni loop e il ciclo termina quando i vale 0. L'output è il seguente:

```
Valore di i: 10
Valore di i: 9
Valore di i: 8
Valore di i: 7
Valore di i: 6
Valore di i: 5
Valore di i: 4
Valore di i: 3
Valore di i: 2
Valore di i: 1
```
Vedremo più avanti che i cicli for sono molto utili per manipolare gli array.
Piccola nota: è possibile usare i cicli for anche per eseguire un blocco di
istruzioni all'infinito:

```c
for (;;)
      printf ("Stampa questo all'infinito\n");
```

In questo caso, dato che non c'è nessuna variabile contatore che limita il
ciclo, le istruzioni all'interno del for verrano semplicemente eseguite
teoricamente all'infinito. Questo perché, nonostante l'istruzione for preveda 3
campi (variabili contatore con valori iniziali, condizione di break e step),
nessuno di questi 3 campi è strettamente obbligatorio.

## Cicli iterativi - Istruzione while

I cicli while, o di iterazione per vero, sono cicli che eseguono un blocco di
istruzioni finchè una condizione specificata risulta vera. La loro sintassi è la
seguente:

```c
while (espressione_booleana)  {
     codice
}
```

Esempio molto semplice:

```c
int i=0;

while (i<10)  {
      printf ("Valore di i: %d\n",i);
        i++;
}
```

Sotto un punto di vista pratico, questo frammento di codice è esattamente uguale
a quello esaminato sopra, nel paragrafo sul for. Semplicemente, controlla se la
variabile i è minore di 10: se lo è, allora esegue il blocco di istruzioni
all'interno del while (ovviamente, ad ogni loop la variabile i viene
incrementata di un'unità). Quando la condizione di partenza non è più vera,
allora il ciclo termina. Esempio un po' più complesso:

```c
int n = -1;

while (n!=0)  {
    printf ("Inserisci un numero (0 per finire): ");
    scanf ("%d",&n);

    printf ("Numero inserito: %d\n",n);
}
```

In questo caso, il programma mi chiederà di inserire un numero intero e stamperà
il numero che ho appena inserito: se il numero è proprio 0, allora il ciclo
termina (l'espressione while (n!=0) sta per "mentre n è diverso da 0"). Anche
attraverso i while è possibile creare cicli infiniti:

```c
while (1)
      printf (“Stampa questo all'infinito!\n”);
```

Il motivo è semplice. Il while viene eseguito finché l'espressione in parentesi
risulta vera (come abbiamo già visto, il C considera vero qualsiasi valore
intero diverso da zero), quindi un while del genere equivale concettualmente a
un "finché 1 è diverso da 0" (sempre vero).

Allo stesso modo si potrebbero creare dei (perfettamente inutili) cicli che non
verranno mai eseguiti:

```c
while (0)
      printf (“Questo non verra' mai eseguito\n”);
```

## Cicli iterativi - Istruzione do-while

Una caratteristica dei cicli while è quella che prima verificano la condizione,
poi eseguono il codice contenuto al loro interno. Se la condizione iniziale è
falsa a priori, il codice non verrà mai eseguito. Esempio:

```c
int n = -1;             // Variabile int

while (n>0)
      printf ("Questo codice non verrà mai eseguito\n");
```

L'istruzione printf() contenuta all'interno del while non verrà mai eseguita, in
quanto la condizione di partenza è falsa (il valore di n è minore di 0). Se
volessimo che il nostro programma esegua prima il codice e poi controlli la
verità della condizione dobbiamo usare un ciclo do-while. La sua struttora è la
seguente:

```c
do  {
      codice
        codice
          ......
} while(condizione_booleana);
```

Esempio:

```c
int n = -1;             // Variabile int

do  {
    printf ("Questo codice verrà eseguito una sola volta\n");
} while(n>0);
```

In questo caso il programma esegue prima l'istruzione printf(), quindi controlla
la condizione specificata. Dato che in questo caso la condizione è falsa, il
ciclo termina.

## Istruzione goto

L'istruzione goto ("vai a") è l'istruzione per i cicli più elementare, e deriva
direttamente dall'istruzione JMP (JuMP) dell'Assembly. La sua sintassi è la
seguente:

```c
etichetta:
codice
codice
......
goto etichetta;         // Salta all'etichetta specificata
```

Esempio: prendiamo il classico programmino che stampa 10 volte "Hello world!".
Con ll'istruzione goto verrebbe più o meno così:

```c
int main()  {
    int i=0;      // Variabile contatore

hello:          // Etichetta "hello". Ma posso chiamarla
    // in qualsiasi altro modo
    printf ("Hello world!\n");
    i++;          // Incremento la variabile contatore 

    if (i<10)
        goto hello; // Se i è minore di 10 salto
    all'etichetta "hello"

        return 0;
}
```
È possibile scrivere qualsiasi tipo di ciclo visto finora (for, while, do-while)
attraverso una sequenza di if e goto.

Tuttavia, l'istruzione goto oggigiorno è estremamente sconsigliata, in quanto
tende a creare il cosiddetto "codice a spaghetti", ossia un codice spezzettato,
pieno di salti e  difficile da leggere (è decisamente più intuitivo vedere come
è fatto un ciclo a primo occhio vedendo un for o un while che seguendo come
Pollicino una scia di goto che non si sa dove portano). In genere i cicli for,
while e do-while sono molto più leggibili di codici scritti con il goto.

## Istruzioni break e continue

È possibile manipolare i cicli attraverso le istruzioni break (che abbiamo già
incontrato quando abbiamo parlato delle strutture switch-case) e continue.
Un'istruzione break termina un ciclo, un'istruzione continue interrompe invece
l'iterazione corrente e va alla prossima. Esempio:

```c
int i=0;                // Variabile "contatore"

// Questo ciclo durerebbe teoricamente all'infinito
for (;;)  {
    printf ("Ora i vale %d\n",i);
    i++;

    if (i>5)
        break;              // Se i è maggiore di 5 interrompo il
    ciclo
}

printf ("Ora il ciclo è concluso!\n");
```

L'output sarà il seguente:

```
Ora i vale 0
Ora i vale 1
Ora i vale 2
Ora i vale 3
Ora i vale 4
Ora i vale 5
Ora i vale 6
Ora il ciclo è concluso!
```

La clausola break in questo caso interrompe il ciclo che altrimenti sarebbe
infinito dopo 6 iterazioni. È possibile usare queste clausole (tra l'altro
abbiamo già incontrare il break nello switch-case) in qualsiasi punto di un
ciclo per interromperlo o continuarlo, al verificarsi di determinate condizioni.
Vediamo invece la continue:

```c
int i;

for ( i=0; i < 5; i++ )  {
    if ( i == 2 )
        continue;

    printf (“i vale %d\n”, i);
}
```

L'output sarà

```
i vale 0
i vale 1
i vale 3
i vale 4
```

Questo perché nel caso i == 2 abbiamo usato la clausola _continue_, che dice di
interrompere l'iterazione corrente e andare alla prossima.
