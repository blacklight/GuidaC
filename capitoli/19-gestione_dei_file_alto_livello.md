# Gestione dei file ad alto livello

> “Everything is a file!”

Questa è la frase più comune tra i sistemisti Unix quando cercano di illustrarti
questo o quel dettaglio di un socket o di un dispositivo. Sui sistemi Unix ogni
entità è un file, un socket di rete o locale, un processo, un dispositivo, una
pipe, una directory, un file fisico vero e proprio...tutto è un file perché a
livello di sistema posso scrivere e leggere su tutte queste entità con le stesse
primitive (write, read, open, close). Queste sono quelle che vengono chiamate
primitive a basso livello per la manipolazione dei file, a basso livello perché
implementate a livello di sistema e non a livello della libreria C ad alto
livello.

Ma facciamo un passo indietro. Noi siamo abituati a vedere, nella vita
informatica quotidiana, un file come un'entità che contiene un certo tipo di
dato. Una canzone, un'immagine, un filmato, un file di testo, la nostra tesi di
laurea...tutte queste cose, in apparenza così diverse da loro, vengono trattate
a livello informatico come una sola entità magica, ovvero come file.

Finora abbiamo visto come scrivere applicazioni che rimangono residenti nella
memoria centrale del computer, nascono quando li eseguiamo, vengono caricati
nella memoria centrale, eseguono un certo numero di operazioni e poi spariscono.
Delle applicazioni del genere non sono poi molto diverse da quelle che può
effettuare una semplice calcolatrice se ci pensiamo...una vecchia calcolatrice
non ha memoria, non si ricorda i calcoli che abbiamo fatto e non ha traccia dei
numeri che abbiamo digitato la settimana scorsa. La grande potenza dei computer,
che ne ha decretato il successo già negli anni '50, è invece la capacità di
poter memorizzare dati su dispositivi fissi e permanenti, non volatili come le
memorie centrali, e per memorizzare questi dati c'è bisogno di ricorrere a
queste entità astratte che sono i file.

Ma come fa un linguaggio di programmazione, come il C, a interagire con queste
entità? Come ho già detto, una strada è quella delle primitive a basso livello,
implementate a livello di kernel. Queste primitive hanno il vantaggio di essere
estremamente lineari (come dicevo prima con la stessa primitiva posso scrivere
su entità diverse a livello logico) e veloci. Veloci perché implementate a basso
livello, marchiate a fuoco nel kernel stesso, che al momento della chiamata non
le deve quindi andare a pescare da una libreria esterna. Il difetto, però, è
quello della portabilità. Le funzioni write, read & co. non sono ANSI-C, perché
funzionano su un kernel Unix, ma non su altri tipi di sistemi. Per rendere
ANSI-C anche l'accesso ai files Kernighan e Ritchie hanno ideato delle primitive
ad alto livello, indipendenti dal tipo di sistema su cui sono compilate.

## Apertura dei file in C

Cominciamo a capire come un sistema operativo, e quindi anche un linguaggio di
programmazione, vede un file. Un file è un'entità identificata in modo univoco
da un nome e una posizione sul filesystem. Non posso interagire direttamente con
l'entità presente sul filesystem, ma ho bisogno di farlo da un livello di
astrazione leggermente più alto: quello dell'identificatore. Quando apro un file
all'interno di una mia applicazione in C non faccio altro che associare a quel
file un identificatore, che altro non è che una variabile o un puntatore di un
tipo particolare che mi farà da tramite nei miei accessi al file. In ANSI-C
questa variabile è di tipo FILE, un'entità definita in stdio.h, e per associarla
ad un file ho bisogno di ricorrere alla funzione fopen (sempre definita in
stdio.h, come tutte le funzioni che operano su entità di tipo FILE). La funzione
fopen è così definita:

```c
FILE* fopen(const char* filename, const char* mode);
```

dove *filename è il nome del nostro file (può essere sia un percorso relativo
che assoluto, ad es. mio_file.txt oppure /home/pippo/mio\_file.txt), mentre
invece *mode mi indica il modo in cui voglio aprire il mio file. Ecco le
modalità possibili:

* r Apre un file di testo per la lettura;
* w Crea un file di testo per la scrittura;
* a Aggiunge a un file di testo;
* rb Apre un file binario per la lettura;
* wb Apre un file binario per la scrittura;
* ab Aggiunge a un file binario;
* r+ Apre un file di testo per la lettura\\scrittura;
* w+ Crea un file di testo per la lettura\\scrittura;
* a+ Aggiunge a un file di testo per la lettura\\scrittura;
* r+b Apre un file binario per la lettura\\scrittura;
* w+b Crea un file binario per la lettura\\scrittura;
* a+b Aggiunge a un file binario per la lettura\\scrittura;

Quando non è possibile aprire un file (es. il file non esiste o non si hanno i
permessi necessari per scrivere o leggere al suo interno) la funzione fopen
ritorna un puntatore NULL. È sempre necessario controllare, quando si usa fopen,
che il valore di ritorno non sia NULL, per evitare di compiere poi operazioni di
lettura o scrittura su file non valide che rischiano di crashare il programma.

Ecco un esempio di utilizzo di fopen per l'apertura di un file in lettura:

```c
#define FILE_NAME               "prova.txt"
 
........
 
FILE *fp;
 
fp = fopen (FILE_NAME,"r");
 
if (!fp)  {
    printf ("Impossibile aprire il file %s in lettura\n",FILE_NAME);
    return;
}
```

È poi buona norma eliminare il puntatore al file quando non è più necessario.
Questo si fa con la funzione fclose, così definita:

```c
int fclose(FILE *fp);
```

La funzione fclose ritorna 0 quando la chiusura va a buon fine, -1 negli altri
casi (ad esempio, il puntatore che si prova a eliminare non è associato ad alcun
file).

## Scrittura su file testuali - fprintf e fputs

Vediamo ora come posso scrivere e leggere su file. In questo campo le funzioni
si dividono in due tipi: quelle per scrivere e leggere su file dati binari e
quelle per il testo semplice (ASCII).

Vediamo prima le funzioni ASCII. Le funzioni ASCII per scrivere e leggere su
file non sono altro che specializzazioni delle corrispettive funzioni per
leggere e scrivere su stdin/stdout. Abbiamo quindi fprintf, fscanf, fgets e
fputs.

L'uso di fprintf è del tutto analogo a quello di printf, e prende come argomenti
un file descriptor (puntatore alla struttura FILE) e una stringa di formato con
eventuali argomenti, in modo del tutto analogo a una printf. Esempio:

```c
#define MY_FILE mio_file.txt
.....
 
FILE *fp;
 
fp = fopen (MY_FILE,"w");
 
if (!fp)  {
    printf ("Errore: impossibile aprire il file %s in scrittura\n",MY_FILE);
    return;
}
 
// Scrivo su file
fprintf (fp,"Questa è una prova di scrittura sul file %s\n",MY_FILE);
```

Analogalmente, si può usare anche la fputs() per la scrittura di una stringa su
file, ricordando che la fputs prende sempre due argomenti (il file descriptor e
la stringa da scrivere su file):

```c
#define MY_FILE mio_file.txt
 
.....
 
FILE *fp;
 
fp = fopen (MY_FILE,"w");
 
if (!fp)  {
    printf ("Errore: impossibile aprire il file %s in scrittura\n",MY_FILE);
    return;
}
 
/* Scrivo su file */
fputs (fp,"Questa è una prova di scrittura\n");
```

Tramite la fprintf posso scrivere su file anche dati che poi posso andare a
rileggere dopo, creando una specie di piccolo 'database di testo'. Esempio:

```c
#include <stdio.h>
#include <stdlib.h>
 
#define USER_FILE               "user.txt"
 
typedef struct  {
    char user[30];
    char pass[30];
    char email[50];
    int age;
} user;
 
int main(void)  {
    FILE *fp;
    user u;
     
    if (!(fp=fopen(USER_FILE,"a")))  {
        printf ("Errore: impossibile aprire il file %s in modalità append\n",
        USER_FILE);
        exit(1);
    }

    printf ("=> Inseririmento di un nuovo utente <==\n\n");
     
    printf ("Username: ");
    scanf ("%s",u.user);
     
    printf ("Password: ");
    scanf ("%s",u.pass);
     
    printf ("Email: ");
    scanf ("%s",u.email);
     
    printf ("Età: ");
    scanf ("%d",&u.age);
     
    /* Scrivo i dati su file */
    fprintf (fp,"%s\t%s\t%s\t%d\n",u.user,u.pass,u.email,u.age);
     
    printf ("Dati scritti con successo sul file!\n");
    fclose (fp);

    return 0;
}
```

Questo produrrà un file di questo tipo:

```
username1       password1       email1  età1
username2       password2       email2  età2
.......
```

Ovvero una riga per ogni utente, dove ogni campo è separato da un carattere di
tabulazione.

## Lettura di file testuali - fscanf e fgets

Per leggere dati di testo semplici, come accennato prima, la libreria stdio.h
mette a disposizione la funzione fscanf, la cui sintassi è molto simile a quella
di scanf:

```c
int fscanf (FILE *fp, char *format_string, void *arg1, ..., void *argn);
```

La funzione fscanf, esattamente come scanf, ritorna il numero di oggetti letti
in caso di successo, -1 in caso di errore. Quindi possiamo struttura il nostro
algoritmo in questo modo: "finché fscanf ritorna un valore > 0, scrivi i valori
letti"

```c
#include <stdio.h>
#include <stdlib.h>
 
#define USER_FILE               "user.txt"
 
typedef struct  {
    char user[30];
    char pass[30];
    char email[50];
    int age;
} user;
 
int main(void)  {
    FILE *fp;
    user u;
    int i=0;
     
    if (!(fp=fopen(USER_FILE,"r")))  {
        printf ("Errore: impossibile aprire il file %s in modalità read-only\n",
                USER_FILE);
        exit(1);
    }
           
    while (fscanf(fp,"%s\t%s\t%s\t%d\n", u.user,u.pass,u.email,&u.age)>0)  {
        printf ("Username: %s\n",u.user);
        printf ("Password: %s\n",u.pass);
        printf ("Email: %s\n",u.email);
        printf ("Età: %d\n\n",u.age);
        i++;
    }
           
    printf ("Utenti letti nel file: %d\n",i);
    fclose (fp);
}
```

Ci sono modi alternativi per effettuare quest'operazione. Ad esempio, si
potrebbero contare gli utenti semplicemente contando il numero di righe nel
file, in modo del tutto indipendente dal ciclo di fscanf principale. Si tratta
semplicemente di introdurre una funzione del genere:

```c
...
 
int countLines (char *file)  {
    FILE *fp;
    char ch;
    int count=0;
     
    if (!(fp=fopen(file,"r")))
        return -1;
     
    while (fscanf(fp,"%c",&ch)>0)
        if (ch=='\n')
            count++;
     
    return count;
}
 
...
 
i=countLines(USER_FILE);
printf ("Numero di utenti letti: %d\n",i);
```

o ancora usando, invece di ciclare controllando il valore di ritorno di fscanf,
si può ciclare finché non viene raggiunta la fine del file. Per far questo si
ricorre in genere alla funzione feof, funzione che controlla se si è raggiunta
la fine del file puntato dal file descriptor in questione. In caso affermativo,
la funzione ritorna un valore diverso da 0, altrimenti ritorna 0

```c
...
 
int countLines (char *file)  {
    FILE *fp;
    char ch;
    int count=0;
     
    if (!(fp=fopen(file,"r")))
        return -1;
     
    while (!feof(fp)) {
        if ((ch = getc(fp)) == '\n')
            count++;
    }
     
    return count;
}
 
...
 
i=countLines(USER_FILE);
printf ("Numero di utenti letti: %d\n",i);
```

Anche qui, la funzione feof si pone ad un livello di astrazione superiore a
quello del sistema operativo. Infatti i sistemi operativi usano strategie
differenti per identificare l'EOF (End-of-File). I sistemi Unix e derivati
memorizzano a livello di filesystem la dimensione di ogni file, mentre i sistemi
DOS e derivati identificano l'EOF con un carattere speciale (spesso identificato
dal caratteri ASCII di codice -1). La strategia dei sistemi DOS però si rivela
molto pericolosa...infatti, è possibile inserire il carattere EOF in qualsiasi
punto del file, e non necessariamente alla fine, e il sistema operativo
interpreterà quella come fine del file, perdendo tutti gli eventuali dati
successivi.

La funzione feof si erge al di sopra di questi meccanismi di basso livello,
rendendo possibile l'identificazione dell'EOF su qualsiasi sistema operativo.

Se conosco a priori la dimensione del buffer che devo andare a leggere dal file,
è preferibile usare la funzione fgets, che ha questa sintassi:

```c
char* fgets (char *s, int size, FILE *fp);
```

Ad esempio, ho un file contenente i codici fiscali dei miei utenti. Già so che
ogni codice fiscale è lungo 16 caratteri, quindi userò la fgets:

```c
#include <stdio.h>
 
#define CF_FILE "cf.txt"
 
int main(void )  {
    FILE *fp;
    char cf[16];
    int i=1;
     
    if (!(fp=fopen(USER_FILE,"r")) )  {
        printf ("Errore: impossibile aprire il file %s in modalità read-only\n",
                USER_FILE);
        exit(1);
    }
     
    while (!feof(fp))  {
        fgets (cf,sizeof(cf),fp);
        printf ("Codice fiscale n.%d: %s\n",i++,cf);
    }
}
```

## Scrittura di dati in formato binario - fwrite

Quelle che abbiamo visto finora sono funzioni per la gestione di file di testo,
ovvero funzioni che scrivono su file dati sotto forma di caratteri ASCII. A
volte però è molto più comodo gestire file in modalità binaria, ad esempio per
file contenenti dati di tipo strutturato, e quindi di dimensione fissata, poiché
per quanto grande possa essere il dato strutturato da gestire queste funzioni
consentono di gestirlo in una sola lettura e in una sola scrittura.

Per la scrittura di dati binari su file si usa la funzione fwrite, che ha questa
sintassi:

```c
size_t fwrite (void *ptr, size_t size, size_t blocks, FILE *fp);
```

dove \*ptr identifica la locazione di memoria dalla quale prendere i dati da
scrivere su file (può identificare una stringa, un intero, un array...), size la
dimensione della zona di memoria da scrivere su file, blocks il numero di
blocchi da scrivere su file (in genere 1) e *fp è puntatore a nostro file.
fwrite ritorna un valore > 0, che identifica il numero di byte scritti, quando
la scrittura va a buon fine, -1 in caso contrario.

Esempio di utilizzo:

```c
#include <stdio.h>
#include <stdlib.h>
 
#define USER_FILE               "user.dat"
 
typedef struct  {
    char user[30];
    char pass[30];
    char email[50];
    int age;
} user;
 
int main()  {
    FILE *fp;
    user u;
     
    if (!(fp=fopen(USER_FILE,"a")))  {
        printf ("Errore: impossibile aprire il file %s in modalità append\n",
                USER_FILE);
        return 1;
    }
     
    printf ("===Inseririmento di un nuovo utente===\n\n");
     
    printf ("Username: ");
    scanf ("%s",u.user);
     
    printf ("Password: ");
    scanf ("%s",u.pass);
     
    printf ("Email: ");
    scanf ("%s",u.email);
     
    printf ("Età: ");
    scanf ("%d",&u.age);
     
    // Scrivo i dati su file
    if (fwrite (&u, sizeof(u), 1, fp)>0)
        printf ("Dati scritti con successo sul file!\n");
    else
        printf ("Errore nella scrittura dei dati su file\n");
     
    fclose (fp);
    return 0;
}
```

## Lettura di dati in formato binario - fread

Per la lettura si ricorre invece alla funzione fread, che ha una sintassi molto
simile:

```c
size_t fread (void *ptr, size_t size, size_t blocks, FILE *fp);
```

Esempio di utilizzo:

```c
#include <stdio.h>
#include <stdlib.h>
 
#define USER_FILE               "user.dat"
 
typedef struct  {
    char user[30];
    char pass[30];
    char email[50];
    int age;
} user;
 
int main(void)  {
    FILE *fp;
    user u;
    int i=0;
     
    if (!(fp=fopen(USER_FILE,"r")))  {
        printf ("Errore: impossibile aprire il file %s in modalità read-only\n",
                USER_FILE);
        exit(1);
    }
     
    while (fread(&u,sizeof(u),1,fp)>0)  {
        printf ("Username: %s\n",u.user);
        printf ("Password: %s\n",u.pass);
        printf ("Email: %s\n",u.email);
        printf ("Età: %d\n\n",u.age);
        i++;
    }
     
    printf ("Utenti letti nel file: %d\n",i);
    fclose (fp);

    return 0;
}
```

## Posizionamento all'intero di un file - fseek e ftell

Vediamo ora altre due funzioni indispensabili per il posizionamento all'interno
di un file.

Un file è un'entità software memorizzata su un dispositivo ad accesso diretto,
come un hard disk o una chiave USB, e in quanto tale è possibile accedere ad
esso in qualsiasi punto dopo l'apertura. Ciò è possibile tramite la funzione
fseek:

```c
int fseek (FILE *fp, int offset, int whence);
```

dove \*fp è il puntatore al file in cui ci si vuole spostare, offset una
variabile intera che rappresenta lo spostamento in byte all'interno del file
(può essere positiva o anche negativa, nel caso di spostamenti all'indietro) e
whence rappresenta il punto da prendere come riferimento nello spostamento. In
stdio.h vengono definiti 3 tipi di whence:

* SEEK\_SET (corrispondente al valore 0), che rappresenta l'inizio del file;
* SEEK\_CUR (corrispondente al valore 1), che rappresenta la posizione corrente
  all'interno del file;
* SEEK\_END (corrispondente al valore 2), che rappresenta la fine del file.

Ad esempio, se come secondo argomento della funzione passo 3 e come terzo
argomento SEEK\_CUR, mi sposterò avanti di 3 byte a partire dalla posizione
attuale all'interno del file.

C'è poi la funzione ftell:

```c
int ftell (FILE *fp);
```

che non fa altro che ritornare la posizione attuale all'interno del file puntato
da fp (ovvero il numero di byte a cui si trova il puntatore a partire
dall'inizio del file).

Esempio pratico: un programmino per la ricerca di una parola all'interno di un
file

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
#define MY_FILE "file_to_search.txt"
 
int main()  {
    FILE *fp;
    char s[100];
    char *buff;
    int dim;
    int i=0;
     
    if (!(fp=fopen(MY_FILE,"r")))  {
        printf ("Errore nella lettura dal file %s\n",MY_FILE);
        exit(1);
    }

    printf ("Parola da cercare all'interno del file %s:", MY_FILE);
    scanf ("%s",s);
         
    dim=strlen(s);
    buff = (char*) malloc(dim*sizeof(char));
             
    while (!feof(fp))  {
        fscanf (fp,"%s",buff);

        if (!strcmp(s,buff))  {
            printf ("Parola trovata a %d byte dall'inizio\n", ftell(fp)-dim);
            i++;
        }
         
        /* Mi posiziono indietro nel file di dim+1 caratteri
         * a partire dalla posizione corrente */
        fseek (fp,-dim+1,SEEK_CUR);
    }
             
    printf ("%d occorrenze di %s trovate nel file\n",i,s);
    return 0;
}
```
