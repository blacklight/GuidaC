# Gestione dei file - primitive a basso livello

Diverse versioni del C offrono un altro gruppo di funzione per la gestione dei
file. Vengono chiamate funzioni di _basso livello_, perché rispetto alle altre
corrispondono in modo diretto alle relative funzioni implementate nel kernel del
sistema operativo. Vengono impiegate nello sviluppo di applicazioni che
necessitano di raggiungere notevoli prestazioni. In questa sede esamineremo le
primitive a basso livello per la gestione dei file in ambiente Unix.

Si deve far attenzione a non usare i due tipi di funzione sullo stesso file; le
strategie di gestione dei file infatti sono differenti e usarle insieme può
generare effetti collaterali sul programma.

## File pointer e file descriptor

A differenza delle funzioni d'alto livello definite in stdio.h, che utilizzano
il concetto di _file pointer_, le funzioni di basso livello fanno uso di un
concetto analogo, il _file descriptor_ (conosciuto anche come "canale" o
"maniglia"). Il file descriptor è un numero intero associato dalla funzione di
apertura al file sul quale si opera. Per poter usufruire di queste funzioni è
necessario includere nel sorgente i seguenti headers:

* fcntl.h;
* sys/types.h;
* sys/stat.h.

Il file descriptor, a differenza del file pointer definito in stdio.h che altro
non è che un puntatore alla struttura FILE, è un numero intero che identifica in
modo univoco il file aperto all'interno della tabella dei files aperti del
sistema operativo. In questa tabella i primi 3 numeri (0,1,2) sono riservati ai
cosiddetti _descrittori speciali_:

* 0 - _stdin_ (Standard Input);
* 1 - _stdout_ (Standard Output);
* 2 - _stderr_ (Standard Error).

Su un sistema Unix posso quindi scrivere su stdout o stderr e leggere dati da
stdin come se fossero normali file, quindi usando le stesse primitive
(_everything is a file!_, è un motto comune tra i sistemisti Unix). Se apro un
altro file sul mio sistema Unix tale file assumerà quindi un identificatore pari
a 3 nella tabella dei file aperti, se ne apro un altro ancora avrà un
identificatore 4 e così via.

## open

La funzione di apertura si chiama _open_, ecco un esempio:

```c
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
 
main() {
    int fd;
     
    fd = open("nomefile", O_WRONLY);
    ...
```

Questa funzione associa fd (file descriptor) a nomefile e lo apre in modalità di
sola scrittura.

La open ritorna un valore intero, che è negativo nel caso in cui si è verificato
un errore, ad esempio il file non esiste o non si hanno i diritti di
lettura/scrittura.

La sintassi della funzione è questa:

```c
int fd, modo, diritti;
...
fd = open("nomefile", modo [diritti]);
```

### Modalità di apertura

modo rappresenta la modalità di apertura del file, può essere una o più delle
seguenti costanti simboliche (definite in fcntl.h):

* O\_RDONLY apre il file in sola lettura;
* O\_WRONLY apre il file in sola scrittura;
* O\_RDWR apre il file in lettura e scrittura.

(Per queste tre costanti simboliche, se il file non esiste la open ritorna
errore)

* O\_CREAT crea il file;
* O\_TRUNC distrugge il contenuto del file;
* O\_APPEND tutte le scritture vengono eseguite alla fine del file;
* O\_EXCL Se al momento dell'apertura il file già esiste, la open ritorna
  errore;

Per poter specificare più di una modalità di apertura si può usare l'operatore
di OR bit a bit, esempio:

```c
fd = open("nomefile", O_CREAT | O_WRONLY, 0640);
```

Crea il file e lo apre in modalità sola scrittura.

### Permessi

Ora vi starete chiedendo cos'è quel 0640, sono i diritti, o permessi, con i
quali il file deve essere creato. Sono codificati con una sintassi simile a
quella di Unix, che suddivide gli utenti in tre categorie:

* possessore del file;
* appartiene al gruppo collegato al file;
* non è collegato al file in alcun modo.

Per ogni categoria si possono specificare i permessi tramite la forma ottale,
costituita da 3 o 4 cifre comprese tra 0 e 7. Esempio:

    0640

Tralasciamo il significato della prima cifra a sinistra, che è opzionale. Ogni
cifra è da interpretare come una somma delle prime tre potenze di 2 (2^0^=1,
2^1^=2, 2^2^=4), ognuna delle quali corrisponde ad un permesso - andando da
sinistra verso destra, la seconda rappresenta il proprietario, la terza il grupp
e l'ultima tutti gli altri utenti; la corrispondenza è questa:

* 4 permesso di lettura;
* 2 permesso di scrittura;
* 1 permesso di esecuzione;
* 0 nessun permesso.

Dunque per ottenere un permesso di lettura e scrittura non occorre far altro che
sommare il permesso di lettura a quello di scrittura (4+2=6) e così via.

Un altro modo di vedere i permessi Unix di un file è tramite la rappresentazione
binaria. I permessi Unix visti sopra non sono altro che una rappresentazione in
modo ottale di un numero binario che se visto fa capire al volo quali sono i
permessi su un particolare file. Ecco come funziona:

```
 U   G   O
rwx rwx rwx
110 100 000
```

In questo caso l'utente (U) ha permessi di lettura e scrittura sul file. Il
gruppo (G) ha solo i permessi di lettura. Gli altri utenti non hanno alcun
permesso. Se convertiamo ogni gruppetto di 3 cifre in ottale otteniamo 0640, che
è effettivamente il permesso che vogliamo.

## close

La funzione _close_ serve a chiudere un file descriptor aperto dalla open:

```c
int fd;
...
close(fd);
```

## read e write

Le operazioni di lettura e scrittura sul file, utlizzando i file descriptor, si
possono effettuare usando le primitive _read_ e _write_.

Esempio di utlizzo di read:

```c
char buf[100];
int dimensione;
int fd;
int n;
...
dimensione = 100;
n = read(fd, buf, dimensione);
```

fd rappresenta il file descriptor da dove si desidera leggere, buf è il vettore
che conterrà i dati letti e dimensione è la dimensione in byte del vettore.

Il valore di ritorno indica il numero di byte letti da fd; questo valore può
essere inferiore al valore di buf, succede quando il puntatore raggiunge la fine
del file; un valore di ritorno uguale a 0 indica la fine del file, mentre invece
un valore minore di 0 indica un errore in lettura.

Esempio di utilizzo di write:

```c
char buf[100];
int dimensione = 100;
int n, fd;
...
n = write(fd, buf, dimensione);
```

fd è il file descriptor da dove si legge, buf è il vettore che contiene i dati
da scrivere e dimensione è la dimensione in byte dei dati da scrivere. Il valore
di ritorno della write indica il numero di byte scritti sul file; questo valore
può essere inferiore alla dimensione nel caso in cui il file abbia superato la
massima dimensione ammessa, o inferiore di 0 in caso di errore.

### Esempio pratico

Ecco un possibile esempio di lettura tramite le primitive appena viste dei
contenuti di un file passato via riga di comando alla nostra applicazione:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
 
int main (int argc, char **argv)  {
    int fd;
    char buff;
     
    // Controllo se al programma è stato passato almeno un argomento
    if (argc==1)  {
        printf ("Uso: %s <file>\n",argv[0]);
        return 1;
    }
     
    // Provo ad aprire il file passato
    if ((fd=open(argv[1],O_RDONLY))<0)  {
        printf ("Errore nell'apertura di %s\n",argv[1]);
        return 2;
    }
     
    // Finché ci sono caratteri da leggere li leggo tramite read...
    while (read (fd,buff,sizeof(buff))>0)
        // ...e li scrivo su stdout tramite write
        // Notate che per la scrittura posso anche usare la write
        // usando come file descriptor 1, che identifica lo stdout
        write (1,buff,sizeof(buff));
     
    // Chiudo il file
    close (fd);
    return 0;
}
```

## lseek

Come per i file pointer, esiste una funzione che consente di muovere il
puntatore al file, per i file descriptor si chiama lseek (per i file pointer era
fseek). Esempio:

```c
long offset;
long n;
int start;
int fd;
...
offset = lseek(fd, n, mode);
```

dove, come sempre, fd è il file descriptor sul quale si muoverà il puntatore; n
è il numero di byte che copre lo spostamento (se negativo lo spolstamento avvine
all'indietro anziché in avanti); mode invece indica la posizione da quale
iniziare a muovere il puntatore: se vale 0 ci sid eve muovere dall'inizio del
file, se vale 1 dalla posizione corrente, mentre se vale 2 a partire dalla fine
del file. Il valore di ritorno di lseek contiene la posizione corrente del
puntatore (dopo lo spostemento ovviamente). Quindi:

```c
lseek(fd, 0L, 1) restituisce la posizione corrente
lseek(fd, 0L, 2) restituisce la dimensione del file in byte
```

## Redirezione

La redirezione è qualcosa che ad alto livello, dalla nostra shell Unix, si
traduce in qualcosa del tipo

    ./nome_eseguibile > mio_file.txt

Ovvero non stampo l'output dell'eseguibile su stdout o stderr, come sarebbe
previsto, ma lo re-direziono su un secondo file, magari un file di log. Come si
traduce questa caratteristica a basso livello? Semplice. Abbiamo visto che
stdin, stdout e stderr sono visti a basso livello come dei semplici file con dei
descrittori speciali (rispettivamente 0, 1 e 2). Posso chiudere, ad esempio, il
descrittore 1 (stdout) e fare in modo che venga sostituito da un descrittore
arbitrario, che in questo caso sarà il descrittore al nostro file di log. Per
chiudere il descrittore userò la primitiva già vista _close()_, mentre invece
per fare in modo che il descrittore del mio file di log sovrascriva il
descrittore di stdout userò la primitiva _dup()_ (_duplicate_), che prende come
unico argomento il descrittore del mio file e lo copia sul primo descrittore
disponibile. In questo caso il primo descrittore disponibile è quello di stdout,
lasciato vuoto dalla chiusura di 1, e quindi qualsiasi testo che è indirizzato
verso stdout verrà re-indirizzato verso il mio file arbitrario. Esempio pratico:

```c
#define  MSG  "Hello\n"
 
main()  {
    write (1,MSG,sizeof(MSG));
}
```

Questo codice effettuerà come prevedibile la stampa di un semplice messaggio su
stdout. Effettuando la redirezione di stdout su un file di log arbitrario
diventa:

```c
#define  MSG  "Hello\n"
#define  ERR  "Impossibile aprire il file di log\n"
 
int main()  {
    char *log="mylog.txt";
    int fd;
     
    if ((fd=open(log,O_WRONLY)<0)  {
        write (2,ERR,sizeof(ERR));
        return -1;
    }
                 
    // Chiudo stdout
    close(1);
                 
    // Duplico il mio descrittore del log
    // che andrà a sovrascrivere stdout
    dup(fd);
                 
    // A questo punto tutto ciò che doveva finire
    // su stdout verrà re-direzionato sul mio log
    write (1,MSG,sizeof(MSG));

    close(fd);
    return 0;
}
```

## Gestione del filesystem a basso livello

I kernel Unix mettono a disposizione del programmatore anche delle primitive per
la gestione del filesystem a basso livello, quali cancellazione e rinominazione
di files.

Per rinominare un file la primitiva è _rename()_, che, come prevedibile, prende
come argomenti

* Una stringa che identifica l'attuale nome del file;
* Una stringa che identifica il nuovo nome da assegnare.

Per la cancellazione la primitiva è _unlink()_, che prende come unico argomento
una stringa contenente il nome del file da cancelare.

## Gestione delle directory

I kernel Unix mettono a disposizione del programmatore anche primitive per la
gestione delle directory. Per modificare la directory in cui opera il programma
si usa la primitiva _chdir()_, che prende come unico argomento una stringa
contenente il nome della directory in cui spostarsi e ovviamente ritorna -1 in
caso di errore. Per ottenere invece l'attuale percorso della directory in cui si
trova il programma in un certo momento si usa la primitiva _getcwd()_, che prende
come argomenti

* Un buffer nel quale salvare il nome della directory corrente;
* La sua dimensione.

Esempio:

```c
#include <stdio.h>
#include <unistd.h>
#include <dirent.h>
 
int main(int argc, char **argv)  {
    char dir[MAXNAMLEN];
    char *new_dir;
     
    if (argc==1)  {
        printf ("Uso: %s <dir>\n",argv[0]);
        return -1;
    }
     
    new_dir=argv[1];
     
    // Cambio la directory corrente
    if (chdir(new_dir)<0)  {
        printf ("Errore - impossibile spostarsi in %s\n",new_dir);
        return -2;
    }
     
    // Ottengo il nome della directory attuale
    // e lo salvo in dir
    getcwd (dir,sizeof(dir));
     
    printf ("Directory attuale: %s\n",dir);
    return 0;
}
```

Questo codice semplicemente prende una directory come argomento da riga di
comando e prova a spostarsi in quella directory tramite _chdir()_, uscendo in
caso di errore. In caso di successo invece salva il percorso della directory
corrente in un buffer di dimensione _MAXNAMLEN_ (costante definita in _dirent.h_
che identifica la dimensione massima che può assumere il nome di una directory)
e lo stampa su stdout. In sostanza questo listato fa qualcosa di simile al
comando _cd_.

All'interno del file _dirent.h_ sono anche definite primitive per la lettura dei
file contenuti all'interno di una directory. Ciò che ci serve è un puntatore a
directory di tipo _DIR_ (non molto diverso in sostanza dal puntatore a file di
tipo _FILE_ definito in stdio.h che abbiamo visto in precedenza) e un puntatore
a una struttura di tipo _dirent_ che conterrà le informazioni sulla directory.
Il campo che ci interessa maggiormente in questo caso della struttura è
_d\_name_, che conterrà di volta in volta il nome di un file contenuto
all'interno della directory. Per l'apertura e la chiusura di un puntatore di
tipo DIR useremo le primitive _opendir()_ e _closedir()_, le cui sintassi non
sono molto diverse da quelle di una _fopen_ o di una _fclose_:

```c
include <dirent.h>
 
......
 
DIR *dir;
struct dirent *info;
 
if (!(dir=opendir("nome_dir")))
    // Errore
 
......
 
closedir(dir);
```

Così come _fopen_, _opendir_ ritorna NULL nel caso in cui non riesca ad aprire
la directory passata come argomento.

Per scannerizzare uno per uno gli elementi della directory si usa la primitiva
_readdir()_, che legge le informazioni di tutti i file contenuti nella
directory, uno dopo l'altro, e le salva in un puntatore a struttura _dirent_.
Quando la lettura è terminata readdir ritorna NULL, e si può prendere questa
come condizione di stop. A questo punto, con queste nozioni possiamo scrivere un
rudimentale programma che si comporta come il comando _ls_ in C, prendendo come
parametro da riga di comando il nome della directory di cui si vuole
visualizzare il contenuto:

```c
#include <stdio.h>
#include <dirent.h>
 
int main (int argc, char **argv)  {
      DIR *dir;
      struct dirent *info;
       
      if (argc==1)  {
          printf ("Uso: %s <dir>\n",argv[0]);
          return 1;
      }
       
      // Apro il descrittore della directory
      if (!(dir=opendir(argv[1])))  {
          printf ("Impossibile aprire la directory %s\n",argv[1]);
          return 1;
      }
       
      // Finché ci sono file all'interno della directory...
      while (info=readdir(dir))
          // ...stampa su stdout il loro nome
          printf ("%s\n",info->d_name);
       
      // Chiudi la directory
      closedir(dir);
      return 0;
}
```
