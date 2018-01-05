# Stringhe

La gestione delle stringhe è alla base della programmazione in qualsiasi
linguaggio di programmazione. Ogni oggetto che viene stampato sullo schermo è
una stringa. I messaggi che abbiamo scritto finora su stdout con la printf non
sono altro che stringhe, lo stesso vale anche per le stringhe di formato della
scanf ecc.

In C una stringa non è altro che un array di elementi di tipo char. Linguaggi di
programmazione più moderni, come Java, Perl, Python, PHP e lo stesso C++,
tramite l'uso della classe 'string', consentono di usare le stringhe in modo più
avanzato, come tipi predefiniti all'interno del linguaggio stesso. La visione
del C (ovvero stringhe=array di tipo char) può essere più macchinosa e a volte
anche più pericolosa, ma mette in mano al programmatore la gestione di questo
tipo di entità al 100%.

## Dichiarazione di una stringa

Abbiamo visto che in C una stringa non è altro che un array di elementi di tipo
char. Questo ci fa pensare subito a un tipo di dichiarazione immediato (ma
alquanto scomodo):

```c
char my_string[] = { 'H','e','l','l','o' };
```

La dichiarazione vista sopra non è comodissima, ragion per cui il C consente di
dichiarare le stringhe direttamente così:

```c
char my_string[] = "Hello";
```

o ancora così, sfruttando una scrittura di tipo puntatore:

```c
char *my_string = "Hello";
```

Ovviamente possiamo anche dichiarare delle stringhe senza inizializzarle. In
questo caso le dichiariamo specificando il nome e la dimensione:

```c
char my_string[20];  // Stringa che può contenere 20 caratteri
```

e vale anche lo stesso discorso che abbiamo fatto con gli array per
l'inizializzazione dinamica di una stringa:

```c
char *my_string;
int n;
 
.......
 
printf ("Quanti caratteri deve contenere la tua stringa? ");
scanf ("%d",&n);
 
my_string = (char*) malloc (n*sizeof(char));
```

Per leggere una stringa invece possiamo ricorrere alla funzione scanf, passando
come stringa di formato '%s':

```c
char str[20];
 
........
 
printf ("Inserisci una stringa: ");
scanf ("%s",str);
```

Si noti che non ho usato la scrittura '&str' nella scanf, in quanto la stringa
già di suo rappresenta un puntatore (in quanto un array non è altro che, a
livello del compilatore, un puntatore al suo primo elemento, come abbiamo visto
prima).

Attenzione: l'uso della scanf per la lettura delle stringhe è potenzialmente
dannoso per la stabilità e la sicurezza di un programma. In seguito valuteremo
metodi per fare letture in tutta sicurezza. La stessa sequenza di escape "%s"
usata per leggere una stringa è dannosa, in quanto non controlla quanti
caratteri vengono effettivamente letti. Per ora ci basta pensare così per
comprendere il rischio: se la mia stringa l'ho dichiarata come una stringa da 20
caratteri, devo controllare che effettivamente non vengano inseriti più di 20
caratteri al suo interno. Se non faccio questo controllo, i caratteri rimanenti
verranno piazzati da qualche parte in memoria al di fuori della stringa, dando
un problema di overflow che nel migliore dei casi provocherà un crash del
programma, nel peggiore comprometterà la sicurezza del sistema lasciando che un
utente non autorizzato scriva in zone di memoria in cui non è autorizzato a
scrivere ed esegua codice arbitrario. Per questo invece della sequenza di escape
"%s" è preferibile usare nella scanf la sequenza "%ns", dove n è il numero di
caratteri che si vogliono leggere al più. Esempio:

```c
char str[20];

........

printf ("Inserisci una stringa: ");
scanf ("%20s",str);
```

Così facendo mi assicuro che dall'input non verranno letti più di 20 caratteri.

Attenzione anche alla printf. Una scrittura del genere è teoricamente corretta:

```c
char str[] = “Prova”;
printf (str);
```

Di fatto è una scrittura altamente pericolosa in quanto vulnerabile a un tipo di
attacco chiamato _format string overflow_, in quanto nulla mi impedisce, se io
utente ho controllo sul contenuto di str, di inserire una stringa di formato che
mi consenta di leggere contenuto arbitrario da zone di memoria adiacenti, e quel
che è peggio scriverci su e dirottare l'esecuzione del processo dove voglio io.
Se devo stampare anche solo una stringa, è meglio usare la sequenza di escape
"%s" esplicitamente per evitare questo tipo di problemi:

```c
char str[] = “Prova”;
printf (“%s”, str);
```

Esercizio pratico: un programmino che prende in input una stringa e trasforma
tutti i suoi eventuali caratteri alfabetici maiuscoli in caratteri minuscoli:

```c
#include <stdio.h>
#include <string.h>
 
// Funzione per la conversione di tutti i caratteri
// maiuscoli in caratteri minuscoli
void toLower(char *s)  {
    int i;
     
    for (i=0; i<strlen(s); i++)
    // Se il carattere corrispondente della stringa è
    // un carattere maiuscolo, ovvero è compreso tra A e Z...
        if ( (s[i]>='A') && (s[i]<='Z'))
            s[i]+=32;
}
 
int main()  {
    char s[20];
     
    printf ("Inserisci una stringa: ");
    scanf ("%20s",s);
     
    toLower(s);
    printf ("Stringa convertita completamente in "
            "caratteri minuscoli: %s\n",s);
    return 0;
}
```

Da notare l'uso della funzione _strlen_, definita in string.h. Tale funzione
ritorna la lunghezza di una stringa, ovvero il numero di caratteri presenti fino
al carattere terminatore della stringa. Ogni stringa possiede infatti un
carattere terminatore per identificarne la fine (ovvero fin dove il compilatore
deve leggere il contenuto della stringa). Tale carattere è, per convenzione, il
carattere NULL, identificato dalla sequenza di escape '\\0' e associato al codice
ASCII 0. Ogni stringa quindi, anche se non è specificato, ha N+1 caratteri,
ovvero gli N caratteri che effettivamente la compongono e il carattere NULL che
ne identifica la fine:

```c
char *str = "Hello";
// In realtà a livello del compilatore 'str' è vista come
// 'H','e','l','l','o','\0'
```

Con le conoscenze che abbiamo in questo momento possiamo anche capire come è
scritto il codice della funzione strlen:

```c
unsigned int strlen(char *s)  {
    unsigned int len;
       
    for (len=0; s[len] != 0; len++);
        return len;
}
```

ovvero un ciclo for dove la variabile contatore viene incrementata finché il
carattere corrispondente all'indice all'interno della stringa non è uguale al
carattere NULL (appunto con codice ASCII uguale a 0). Il valore della variabile
contatore a questo punto rappresenta il numero effettivo di caratteri fino al
NULL, ovvero il numero effettivo di caratteri all'interno della string, valore
che viene ritornato dalla funzione. Scrivere un

```c
for (i=0; i<strlen(s); i++)
```

equivale quindi a dire "cicla finché la stringa s contiene dei caratteri, o
finché non viene raggiunta la fine della stringa".

Questa scrittura

```c
if ( (s[i]>='A') && (s[i]<='Z') )
      s[i]+=32;
```

equivale a dire "se il carattere attuale è maggiore o uguale ad A e minore o
uguale a Z, ovvero è una lettera maiuscola, somma al suo valore ASCII attuale il
valore 32". 32 è l'offset che nella tabella dei caratteri ASCII esiste tra i
caratteri maiuscoli e quelli minuscoli. Per verificare:

```c
printf ("%d\n",'a'-'A');
```

## Operare sulle stringhe - La libreria string.h

Abbiamo incontrato nel paragrafo precedente la funzione _strlen_, definita in
_string.h_. Questo header mette a disposizione molte funzioni per operare su
questi tipi di dati. Tenteremo di esaminare le più importanti nel corso di
questo paragrafo.

### strcmp

La funzione _strcmp_ (STRing CoMPare) confronta tra di loro i valori di due
stringhe, il suo prototipo è qualcosa di simile:

```c
int strcmp(const char *s1, const char *s2);
```

dove s1 e s2 sono le due stringhe da confrontare. La funzione ritorna

* Un valore > 0 se da un confronto byte a byte s1 ha più caratteri il cui codice
  ASCII è maggiore del corrispondente codice ASCII di s2
* 0 se le due stringhe sono uguali
* Un valore < 0 nei casi rimanenti

questa funzione è utilizzatissima per vedere se due stringhe hanno lo stesso
contenuto. Sono infatti completamente sbagliate scritture del genere:

```c
char *s1;
char *s2="pippo";
 
........
 
if (s1==s2)
      printf ("Ciao pippo\n");
```

questo perché la scrittura sopra non fa altro che vedere se il puntatore s1 è
uguale al puntatore s2, ovvero confronta gli indirizzi in memoria delle due
stringhe, ed effettua le operazioni richieste se gli indirizzi coincidono. Ciò
ovviamente non sarà mai verificato, dato che due variabili diverse in memoria
hanno anche indirizzi diversi, quindi il codice scritto sopra non funzionerà
mai. Per confrontare due stringhe è invece necessario ricorrere alla funzione
strcmp, ricordando che la funzione ritorna 0 quando il contenuto di due stringhe
è lo stesso. Ecco quindi la versione corretta del codice di sopra:

```c
char *s1;
char *s2="pippo";

........

if (!strcmp(s1,s2))
    // Equivale a scrivere
    // if (strcmp(s1,s2)==0)
    printf ("Ciao pippo\n");
```

### strncmp

La funzione _strncmp_ è molto simile a strcmp, con l'eccezione che confronta
solo i primi _n_ caratteri sia di s1 che di s2. La sua sintassi è qualcosa di
simile:

```c
int strncmp(const char *s1, const char *s2, size_t n);
```

dove _n_ è il numero di caratteri da confrontare.

### strcpy

La funzione _strcpy_ copia una stringa in un'altra. La sua sintassi è qualcosa di
simile:

```c
char *strcpy(char *dest, const char *src);
```

dove _dest_ è la stringa all'interno della quale viene copiato il nuovo valore e
_src_ è la stringa da copiare. Il valore di ritorno della funzione è un
puntatore a _dest_.

Quando si vuole copiare una stringa in un altra è infatti sconsigliabile usare
una scrittura del genere:

```c
char *s1="pippo";
char *s2;

s2=s1;  // ATTENZIONE!!
```

La scrittura di sopra infatti copia il puntatore al primo elemento della stringa
s1 nella stringa s2. Ciò vuol dire che ogni eventuale modifica di s1 modifica
anche s2, dato che entrambe le variabili agiscono sulla stessa zona di memoria,
e viceversa, il che è decisamente un effetto collaterale. La scrittura corretta
è qualcosa del tipo

```c
char *s1="pippo";
char s2[32];

strcpy(s2,s1);
```

in quanto la funzione strcpy genera in s2 una copia esatta di s1, che però,
essendo residente in una zona di memoria diversa, è completamente indipendente
da s1.

La funzione strcpy ha un codice simile:

```c
char* strcpy (char *s1, char *s2)  {
    int i;
     
    // Finché la stringa s2 ha dei caratteri...
    for (i=0; i<strlen(s2); i++)
        // ...copia il carattere in s1
        s1[i]=s2[i];
     
    return s1;
}
```

Questa funzione è però potenzialmente dannosa per la sicurezza dell'applicazione
e sconsigliata. È consigliato usare al suo posto la funzione _strncpy_ che
effettua una copia esattamente di _n_ caratteri della stringa, evitando che
eventuali byte di troppo vadano a sovrascrivere pericolosamente zone di memoria
adiacenti. Per maggiori approfondimenti rimando al capitolo "Uso delle stringhe
e sicurezza del programma".

### strncpy

La funzione _strncpy_ ha una sintassi molto simile a strcpy, con la differenza
che copia solo i primi n caratteri della stringa sorgente nella stringa di
destinazione, per tenere il processo di copia sotto controllo ed evitare
problemi di sicurezza nell'operazione, come vedremo in seguito. La sua sintassi
è

```c
char *strncpy(char *dest, const char *src, int n);
```

La sintassi è la stessa di strcpy, a parte per n, che identifica appunto il
numero di caratteri di _src_ che verranno copiati in _dest_. L'uso di questa
funzione è preferibile a quello di strcpy quando possibile, proprio per evitare
problemi di sicurezza legati ad una copia non controllata.

### strcat

La funzione _strcat_ concatena l'inizio di una stringa alla fine di un'altra. La
sua sintassi è

```c
char *strcat(char *dest, const char *src);
```

dove _dest_ è la stringa alla cui fine viene concatenata la stringa _src_.
Esempio di utilizzo:

```c
#include <stdio.h>
#include <string.h>
 
int main()  {
    char s1[20];
    char *s2 = "pippo";
     
    // Copio all'interno di s1 la stringa "Ciao "
    // copiando esattamente il numero di byte che
    // mi servono, tramite l'operatore sizeof
    strncpy (s1, "Ciao ", sizeof("Ciao "));
     
    strcat (s1,s2);
    // s1 ora contiene "Ciao pippo"

    return 0;
}
```

La funzione strcat ritorna un puntatore a char che rappresenta un puntatore alla
zona di memoria dove è salvato _dest_.

Questa funzione è però potenzialmente dannosa per la sicurezza dell'applicazione
e sconsigliata. È consigliato usare al suo posto la funzione _strncat_ che
effettua una copia esattamente di n caratteri della stringa, evitando che
eventuali byte di troppo vadano a sovrascrivere pericolosamente zone di memoria
adiacenti. Per maggiori approfondimenti rimando al capitolo "Uso delle stringhe
e sicurezza del programma".

### strncat

La sua sintassi è molto simile a _strcat_, con la differenza che in strncat
vanno specificati anche il numero di caratteri di src da copiare in dest, in
modo da tenere la copia sotto controllo. La sua sintassi è

```c
char *strncat(char *dest, const char *src, int n);
```

dove n rappresenta il numero di caratteri di _src_ da copiare. Il suo uso,
quando possibile, è preferibile a quello di strcat.

### strstr

La funzione _strstr_ serve per verificare se esiste una sottostringa all'interno
della stringa di partenza. La sua sintassi è

```c
char *strstr(const char *haystack, const char *needle);
```

dove _haystack_ (lett. 'pagliaio') è la stringa all'interno della quale cercare,
_needle_ (lett. 'ago') è la stringa da cercare (da notare ancora una volta il
sottile umorismo degli sviluppatori del C). La funzione ritorna

* Un puntatore intero, che rappresenta la zona di memoria in cui è stata trovata
  la sottostringa, nel caso in cui la sottostringa dovesse essere trovata
* NULL nel caso in cui la sottostringa non dovesse essere trovata

Esempio:

```c
/*
* Questo programmino chiede in input all'utente due stringhe e
* verifica se la seconda stringa è localizzata all'interno della prima
*/

#include <stdio.h>
#include <string.h>

int main()  {
    char s1[32];
    char s2[32];
     
    printf ("Inserire la stringa all'interno della quale cercare: ");
    scanf ("%s",s1);
     
    printf ("Inserire la stringa da cercare: ");
    scanf ("%s",s2);
     
    if (strstr(s1,s2))
        // Equivale a scrivere
        // if (strstr(s1,s2) != 0)
        // ovvero se il valore di ritorno della funzione non è NULL
        printf ("Stringa \"%s\" trovata all'interno di
                \"%s\", in posizione %d\n",
                s2,s1,(strstr(s1,s2)-s1));
    else
        printf ("Stringa \"%s\" non trovata “ “all'interno di \"%s\"\n",s2,s1);

    return 0;
}
```

Si noti questa scrittura:

```c
strstr(s1,s2)-s1
```

strstr ritorna infatti l'indirizzo dell'area di memoria in cui si trova s2
all'interno di s1. Se a questo indirizzo sottraggo l'indirizzo di s1, ovvero
l'indirizzo del primo carattere di s1, ottengo la locazione effettiva della
sottostringa all'interno della stringa di partenza. Se ad esempio s1="Ciao
pippo" e s2="pippo", strstr(s1,s2)-s1 = 5.

## Altre funzioni sulle stringhe

### sprintf

La funzione _sprintf_, definita in stdio.h, è del tutto analoga alla printf. La
differenza è che la printf scrive dell'output formattato su standard output,
mentre la sprintf scrive dell'output formattato direttamente su una stringa, che
rappresenta il primo argomento della funzione. Esempio:

```c
#include <stdio.h>
 
int main()  {
    char s1[32];
    char s2="Ciao";
    char s3="pippo";
    int age=24;
     
    sprintf (s1, "%s %s ho %d anni", s2, s3, age);
    // Scrivo su s1 attraverso la sprintf
    // Ora s1 contiene la stringa "Ciao pippo ho 24 anni"
}
```

Anche la funzione sprintf è sulla lista di quelle da usare con cautela, e solo
quando si è sicuri che la stringa di destinazione è in grado di contenere tutti
i byte che si stanno per copiare al suo interno. Se non si ha questa sicurezza,
è preferibile usare _snprintf_ che controlla che non vengano copiati nella
stringa di destinazione più di n caratteri.

### snprintf

La funzione _snprintf_ è un'alternativa più sicura alla sprintf, e al suo
interno va specificato anche il numero massimo di caratteri da copiare. La sua
sintassi è quindi

```c
int snprintf(char *str, int size, const char *format, ...);
```

dove _size_ rappresenta il numero massimo di byte della stringa di formato da
copiare all'interno di _str_.

### sscanf

La funzione _sscanf_ è del tutto analoga alla scanf classica, solo che invece di
leggere i dati dalla tastiera li legge dall'interno di una stringa. Esempio,
ecco un uso classico di sscanf. Abbiamo una stringa che rappresenta una data, in
formato 'gg/mm/aaaa'. Vogliamo ottenere, dall'interno di questa stringa, il
giorno, il mese e l'anno e salvarli all'interno di 3 variabili intere. Con
sscanf la cosa è presto fatta:

```c
char *date = "13/08/2007";
int d,m,y;
 
sscanf (date,"%d/%d/%d",&d,&m,&y);
// d=13, m=8, y=2007
```

La funzione ritorna un intero che rappresenta il numero di campi letti
all'interno della stringa. Il controllo su questo valore di ritorno può tornare
utile per verificare se l'utente ha inserito la stringa nel formato giusto:

```c
#include <stdlib.h>
 
int main()  {
    char date[16];
    int d,m,y;
     
    printf ("Inserisci una data: ");
    scanf ("%16s",date);
     
    // Se non leggo almeno 3 interi nella stringa
    // inserita separati da '/'...
    if (sscanf(date,"%d/%d/%d",&d,&m,&y) != 3)  {
        printf ("Errore: data inserita non valida: %s\n",date);
        return 1;
    }
     
    printf ("Giorno: %d\n",d);
    printf ("Mese: %d\n",m);
    printf ("Anno: %d\n",y);

    return 0;
}
```

### gets

Un piccolo limite della lettura delle stringhe sta nel fatto che la lettura si
interrompe quando incontra uno spazio. Se ad esempio un'applicazione richiede
all'utente di inserire una stringa e l'utente inserisce "Ciao mondo", se la
lettura avviene tramite scanf molto probabilmente la stringa risultante dopo la
lettura sarà semplicemente "Ciao". Per evitare questo piccolo inconveniente si
ricorre alla funzione _gets_, nonostante il suo uso sia deprecato dai nuovi
standard C. Esempio di utilizzo:

```c
#include <stdio.h>
 
int main()  {
    char str[32];
     
    printf ("Inserisci una stringa: ");
    gets (str);
     
    // Se ora inserisco stringhe con degli spazi in mezzo vengono
    // salvate ugualmente nella
    // stringa finale, in quanto la gets legge tutti i caratteri fino
    // al fine linea
         
    printf ("Stringa inserita: %s\n",str);
    return 0;
}
```

Attenzione: anche la gets è nella lista delle funzioni a rischio. Anzi, si può
dire che un programma che usi tale funzione è SEMPRE a rischio overflow, ed è
mantenuta nella libreria standard del C solo per retrocompatibilità. Se si usa
un sorgente che fa uso di _gets()_ è il compilatore stesso a sollevare un
warning:

    warning: the 'gets' function is dangerous and should not be used.

Se si vuole leggere un'intera riga da input senza fermarsi al primo spazio è
meglio valutare delle alternative, fra cui:

* fgets(), che può essere vista come una versione “sicura” di gets() (la vedremo
  fra un attimo)
* la libreadline (vedremo anche questa fra un attimo)
* la scrittura "in casa" di una funzione che legga dell'input dallo stdin finché
  non incontra '\\n' (ovvero finché non viene premuto invio), usando
  l'allocazione dinamica della memoria:

    ```c
    #include <stdio.h>
    #include <stdlib.h>

    char* safe_gets ()  {
        char *s = NULL;
        char ch;
        unsigned int size = 0;

        // Finché leggo un carattere da input e questo
        // carattere è diverso da '\n'...
        while ((ch = getchar()) != '\n')  {
            // Creo un nuovo elemento in s e piazzo in coda
            // il nuovo carattere
            s = (char*) realloc( s, ++size );
            s[size-1] = ch;
        }

        // Termino la stringa correttamente con un '\0' in coda
        if ( size > 0 )
            s[size] = 0;

        return s;
    }

    int main()  {
        char* s = safe_gets();
        printf ("Hai inserito: %s\n", s);
        free(s);
        return 0;
    }
    ```

### fgets

La _fgets()_ è una funzione che opera nativamente sui file (vedremo in seguito
come) leggendo una riga da essi, ma può essere usata anche per leggere da stdin,
come una versione "safe" di _gets()_. Prende come parametri la stringa in cui
salvare la sequenza letta, il numero di byte da leggere, e il descrittore del
file da cui leggere (se si vuole leggere da tastiera, basta passare _stdin_).
Esempio:

```c
char nome[30];
printf (“Inserisci il tuo nome e il tuo cognome: “);
fgets ( nome, 30, stdin );
nome[strlen(nome)-1] = 0;
printf (“Ti chiami %s\n”, nome);
```

_fgets()_ legge fino alla fine della riga, includendo il carattere di newline
'\\n'. Se si vuole escludere il newline, basta piazzare il carattere terminatore
'\\0' al suo posto.

### libreadline

Una seconda via per la lettura da stdin, adottata anche dalle shell Unix (bash e
zsh in primis) e da moltissime altre applicazioni, è la libreria esterna
_libreadline_. Questa libreria è installata di default praticamente su tutti i
sistemi Unix-based, anche se su alcuni può essere necessario installare il
pacchetto _libreadline-dev_ per poter compilare i propri sorgenti con questa
libreria. _libreadline_ non solo offre il salvataggio di una qualsiasi sequenza
passata via stdin dentro una stringa, ma offre anche meccanismi ben più
avanzati, fra cui un sistema di history per salvare le stringhe lette e la
possibilità di poter editare la riga scritta sul terminale attraverso una specie
di _mini-editor_, che può supportare perfino degli elementari comandi di editing
in stile Vi o Emacs, a seconda dei gusti dell'utente (ad esempio per fare in
modo che tutte le applicazioni che usano la _libreadline_ supportino un editing
della riga passata in input in stile Vi, ad esempio premendo ESC da terminale
per andare in command mode, dw per cancellare una parola, fc per cercare il
carattere c, c$ per modificare tutto ciò che c'è dalla posizione attuale del
cursore fino alla fine della riga, e così via, basta piazzare nel file
_$HOME/.inputrc_ l'opzione set _editing-mode vi_).

Una volta appurata la presenza della libreria sul proprio sistema è molto
semplice scrivere codice che la usi:

```c
#include <readline.h>
#include <history.h>

int main()  {
    char *line = readline(“Inserisci una stringa: “);
    return 0;
}
```

Ancora una volta, per la compilazione del sorgente che si appoggia a una
libreria esterna la procedura sarà

    gcc -I/usr/include/readline -o test_readline test_readline.c -lreadline

### atoi

La funzione _atoi_ (ASCII to int), definita in stdlib.h, è usata per convertire
una stringa in un valore intero. La sua sintassi è molto semplice:

```c
int atoi(const char *nptr);
```

e restituisce il valore convertito. Nel caso in cui la stringa non contenga un
valore numerico valido, la funzione ritorna zero. Esempio di utilizzo:

```c
#include <stdio.h>
#include <stdlib.h>
 
main()  {
      int n;
        char *s = "3";
         
          n=atoi(s);
            // Ora n contiene il valore intero 3
}
```

Della stessa famiglia sono le funzioni _atol_ (ASCII to long) e _atof_ (ASCII to
float).

## Gestione di stringhe binarie

Finora abbiamo esaminato stringhe di testo, ovvero contenenti caratteri ASCII
stampabili e la cui fine è identificata dal carattere con codice ASCII 0. Questo
non è affatto l'unico caso di sequenze di dati in cui ci si può imbattere, anzi
è molto comune il caso in cui si debbano gestire array di char che altro non
sono che sequenze di dati binari qualsiasi. In questo caso le funzioni e
l'approccio che abbiamo visto finora non funzionano più, in quanto la presenza
del carattere 0 può diventare "legale" anche nel mezzo della stringa, e non
identificare più la fine della stringa. Se ad esempio dichiarassimo una stringa
del genere

```c
char str[5] = "\x01\x00\x02\x03\x0a";
```

Ovvero contenente in esadecimale la sequenza { 1, 0, 2, 3, 10 } (questa sequenza
può essere stata letta pari pari da un socket di rete, da un dispositivo fisico
o da un file binario). Se volessimo calcolare la lunghezza di questa stringa via
strlen, ad esempio, vedremo che tale funzione ritornerà 1, in quanto dopo il
carattere 0x01 è piazzato il byte nullo, 0x00, che per le stringhe di testo
identifica la fine della stringa. Se copiassimo via strcpy o simili il contenuto
di quella stringa in un'altra stringa vedremo che la copia si ferma dopo il
primo carattere, per lo stesso motivo. Per fortuna il C mette a disposizione
anche funzioni per gestire stringhe binarie. Ma in questo caso, ovviamente, non
essendoci più un carattere a identificare la fine delle stringhe dovrà essere il
programmatore a sapere quanti byte gestire e quanto saranno grandi i suoi
buffer. Si noti che tutte queste funzioni prendono e ritornano argomenti che
sono _void\*_, _non char\*_, in quanto possono operare tanto su stringhe, tanto
su zone di memoria _raw_ di qualsiasi tipo (interi, float, tipi di dati
strutturati, ecc.).

### memset

Prototipo:

```c
void *memset(void *s, int c, size_t n);
```

Tale funzione riempie un'area di memoria _s_, riempiendo _n_ byte con un valore
costante _c_. Esempio:

```c
char seq[10];
memset ( seq, 0, 10 );
```

In questo caso abbiamo riempito di zeri i 10 byte di _seq_.

### memcpy

Prototipo:

```c
void *memcpy(void *dest, const void *src, size_t n);
```

Tale funzione copia _n_ byte di _src_ dentro _dest_.

### memmem

Prototipo:

```c
void *memmem(const void *haystack, size_t haystacklen,
                    const void *needle, size_t needlelen);
```

Tale funzione cerca la sequenza _needle_ di lunghezza _needlelen_ dentro
_haystack_, di lunghezza _haystacklen_, e ritorna un puntatore alla zona di
memoria in cui è stata trovata l'occorrenza, o NULL se non è stata trovata, in
modo simile a _strstr_.
