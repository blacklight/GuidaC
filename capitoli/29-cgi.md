# CGI in C

Utilizzando il meccanismo dei CGI (**Common Gateway Interface**) è possibile
innescare vere e proprie applicazioni che hanno la libertà di svolgere qualsiasi
funzione eseguibile sul web server da un programma, per poi restituire un
risultato in forma di pagina HTML. Il C consente di fare operazioni del genere,
in modo forse meno avanzato rispetto a linguaggi dedicati come PHP o Perl ma
estremamente flessibile, date le sue caratteristiche.

## Pagine statiche e pagine dinamiche

Sono dette pagine statiche quella pagine presenti sul web server che non
richiedono alcuna elaborazione da parte del browser se non quella di prendere la
pagina così com'è e inviarla al browser. Generalmente queste pagine sono scritte
in linguaggi detti di "markup", gli esempi più palesi sono HTML e XML. Per
pagina dinamica si intende una pagina non presente fisicamente sul disco rigido
del Web Server, ma costruita al volo, per mezzo di un'applicazione (interfaccia
CGI) o uno script dedicato (in PHP o ASP). Il meccanismo dei CGI estende e
generalizza l'interazione request/response, cuore del protcollo HTTP. Ora
descriviamo passo passo il meccanismo dei CGI, il processo si può dividere in 4
fasi:

1. _Invio della request_ - Il browser (client HTTP) effettua una request a un
   server HTTP identificato dal seguente indirizzo o URL:

        http://www.nomesito.it/cgi-bin/hello.cgi?

    Possiamo identificare il server `HTTP:http://www.nomesito.it` e il
    riferimento alla procedura CGI:

        cgi-bin/hello.cgi

    La directory `/cgi-bin` è una sottodirectory della directory del web server
    che contiene le applicazioni CGI.

2. _Attivazione del CGI_ - Il server HTTP (es. Apache, Netscape Server o
   Microsoft IIS) riceve la URL, la interpreta e lancia il processo (o thread)
   che esegue il CGI.

3. _Risposta del CGI_ - Il risultato della computazione deve dar luogo a una
   pagina HTML di risposta, che il CGI invia verso il suo Standard Out (per i
   CGI lo STDOUT viene intercettato dal server HTTP) tenendo conto di quale deve
   essere il formato di una response HTTP.

4. _Risposta del server HTTP_ - Sarà poi il server HTTP ad inviare la response
   verso il client che aveva effettuato la request.

Passiamo adesso a vedere come si scrive un CGI in C, prendendo come spunto
un'applicazione che stampa all'interno di una pagina web 'hello world':

```c
//Il CGI hello.c
#include <stdio.h>
 
int main(int argc, char *argv[])  {
    printf("Content-type: text/html\n\n"); /*informazione necessaria per la
                                             response*/
     
    /*Inviamo su STDOUT i tag HTML*/
    printf("<html>\n"
           "<head>\n"
           "<title>Hello World!</title>\n"
           "</head>\n"
           "<body>\n"
           "<h1><p
           align=\"center\">Hello
           World</p></h1>\n"
           "</body>\n"
           "</html>\n");
     
    return 0;
}
```

Compilando questo programma all'interno della directory /cgi-bin del nostro
server web

    gcc -o hello.cgi hello.c

otteniamo un eseguibile CGI che possiamo richiamare all'interno del nostro
browser nel modo visto sopra

    http://www.miosito.it/cgi-bin/hello.cgi

L'esame del codice non è nulla di assurdo, tenendo sempre presente che lo STDOUT
di una CGI viene rediretto direttamente al client HTTP. Degna di nota è questa
riga:

```c
printf("Content-type: text/html\n\n");
```

Il suo scopo è quello di specificare al client HTTP il tipo di contenuto che si
sta per inviare (in questo caso del testo HTML). Senza questa specifica il
client non sa come comportarsi e stamperà una pagina bianca.

Si notino le due linee vuote; sono assolutamente necessarie, per le convenzioni
del protocollo HTTP, ma sono spesso dimenticate. Il segnale di STDOUT viene
quindi intercettato dal browser, e viene generata una pagina web con i contenuti
specificati.

Prendiamo ora in esame un rudimentale orologio che invia al client HTTP una
pagina web contenente l'ora del server sfruttando la libreria time.h:

```c
/* Ora quasi esatta */
#include <stdio.h>
#include <time.h>
 
int main(int argc, char *argv[])  {
    time_t bintime;
    struct tm *curtime;
     
    printf("Content-type: text/html\n\n");

    printf ("<html>\n");
    printf("<head>\n");
    printf("<title>Orologio</title>\n");
    printf("</head>\n");
    printf("<body>\n");
    printf("<h1>\n");
    time(&bintime);
    curtime = localtime(&bintime);
    printf("Data e ora: %s\n", asctime(curtime));
    printf("</h1>\n");
    printf("</body>\n");
    printf ("</html>\n");
     
    return 0;
}
```

## Richieste GET e POST

L'utilità principale delle CGI, e dei linguaggi web-oriented come PHP e ASP, è
quella di poter anche interagire con l'utente finale, leggendo dati inseriti da
un utente via form o tramite altre vie. Per far questo il protocollo HTTP
prevede due strade: il metodo _GET_ e il metodo _POST_.

### GET

Nel metodo GET i dati inseriti dall'utente o previsti dal programmatore vengono
caricati nell'URL, e il loro contenuto, a livello del sistema server, finisce in
una variabile d'ambiente chiamata `QUERY_STRING`. Immaginiamo ad esempio di
avere il seguente codice HTML (ricordate che la scelta del metodo, GET o POST,
va fatta a livello del codice del form HTML):

```html
<form method=GET action=/cgi-bin/cgi1>
    Come ti chiami? <input type="text" name="nome"><br>
    <input type="submit" value="Clicca">
</form>
```

È un semplice form HTML che chiede all'utente di turno come si chiama e invia la
stringa inserita dall'utente all'eseguibile 'cgi1' tramite metodo GET (per
convenzione gli eseguibili CGI si mettono nella directory `/cgi-bin` del
server).  Se salviamo questa pagina come 'user.html' dopo aver cliccato sul
tasto 'Clicca' la richiesta verrà inoltrata tramite metodo GET a cgi1, che
quindi verrà richiamato nel seguente modo:

    http://www.miosito.org/cgi-bin/my_cgi?nome=Nome_inserito_dall_utente

Nel caso ci fossero stati più campi oltre al nome (ad esempio un campo
'password') avremmo avuto una cosa del genere:

```
http://www.miosito.org/cgi-bin/cgi1?
nome=Nome_inserito_dall_utente&password=Password_inserita
```

In pratica quando inviamo una richiesta tramite GET l'eseguibile CGI che viene
richiamato (o lo script PHP/ASP) viene richiamato passando nell'URL una
struttura del genere:

```
http://www.miosito.org/cgi-bin/my_cgi?
campo1=val1&campo2=val2&campo3=val3........
```

Ora immaginiamo che il nostro eseguibile cgi1 debba leggere il nome inserire
dall'utente e generare per lui una pagina HTML di benvenuto (es. 'Benvenuto
pippo!'). Ecco un potenziale codice C come potrebbe essere:

```c
#include <stdio.h>
#include <stdlib.h>
 
// Funzione che converte eventuali caratteri speciali
// all'interno della stringa inserita dall'utente in
// caratteri ASCII leggibili
// Prende come parametri la stringa sorgente, la stringa
// di destinazione e la lunghezza della stringa da 'uncodare'
void unencode (char *src, char *dest, int len);
 
// Funzione per il prelevamento di
// un campo da una query
// Prende come parametri la query in cui cercare
// e il nome del campo da cercare (in questo caso 'nome')
char* get_field(char *query, char *field);
 
main()  {
    char *query,*nome;
    int len;
     
    // Genero la pagina HTML
    printf ("Content-type: text/html\n\n");
    printf ("<html>\n"
            "<head>\n"
            "<title>Pagina di benvenuto</title>\n"
            "</head>\n"
            "<body>\n");
     
    // Se la richiesta GET non contiene niente, la pagina è stata richiamata
    // in modo errato, quindi esco
    if ((query=getenv("QUERY_STRING"))==NULL)  {
        printf ("<h3>Pagina richiamata in modo errato</h3>\n"
                "</body></html>\n");
        exit(1);
    }
     
    // Controllo la lunghezza della query e
    // genero una stringa lunga quanto la query
    // che conterrà il nome inserito dall'utente
         
    // Ricordiamo che query ora sarà una stringa
    // del tipo 'nome=pippo'
    len=strlen(query);
    nome = (char*) malloc(len*sizeof(char));
     
    // Ora nome conterrà il campo 'nome' della query
    nome=get_field (query,"nome");
     
    printf ("<h3>Benvenuto %s!</h3>\n"
            "</body></html>\n",nome);
     
    exit(0);
}
 
char* get_field(char *query, char *field)  {
    int i,j,len,pos;
    char *tmp,*input;
     
    // len è pari alla lunghezza della querry+1
    len = strlen(query)+1;
     
    // tmp sarà il pattern di ricerca all'interno della query
    // Nel nostro caso andrà a contenere la stringa 'nome='
    tmp   = (char*) malloc( (strlen(field)+1)*sizeof(char) );
     
    // input è lunga quanto la query, e andrà a contenere
    // il campo da noi ricercato
    input = (char*) malloc(len*sizeof(char));
     
    // tmp <- nome=pippo
    sprintf (tmp, "%s=", field);
     
    // Se all'interno della query non c'è il campo richiesto, esco
    if (strstr(query,tmp)==NULL)
        return NULL;
     
    // Cerco la posizione all'interno della query
    // in cui è stato trovato il campo nome
    pos = ( (int) strstr(query,tmp) - (int) query) + (strlen(field)+1);
     
    // Controllo quanto è lungo il pattern nome=blablabla
    // Questo ciclo termina quando viene incontrato un '&' all'interno
    // della query (ovvero quando comincia un nuovo campo) o quando la stringa è
    // terminata alla fine i conterrà il numero di caratteri totali nel pattern
    // di ricerca
    for (i=pos; ; i++)  {
        if (query[i]=='\0' || query[i]=='&') break;
    }
     
    // Salvo il contenuto della query che mi interessa in input
    for (j=pos; j<i; j++)
        input[j-pos]=query[j];
     
    // 'unencodo' input, rendendo eventuali caratteri speciali umanamente
    // leggibili
    unencode(input,input,len);
     
    // Ritorno input
    return input;
}
 
void unencode(char *src, char *dest, int len)  {
    int i,code;
     
    // Ciclo finché non ho letto tutti i caratteri specificati
    for (i=0; i<len; i++, src++, dest++)  {
        // Se il carattere corrente di src è un '+', lo converto in uno spazio
        // ' '
        if (*src=='+')  *dest=' ';
         
        // Se il carattere corrente è un '%'
        else if (*src=='%')  {
            // Se il carattere successivo non è un carattere valido,
            // il carattere di destinazione sarà un '?',
            // altrimenti sarà il carattere ASCII corrispondente
            if (sscanf(src+1, "%2x", &code) != 1) code='?';
            *dest = (char) code;
             
            // Leggo il prossimo carattere
            src += 2;
        }
         
        // Se è un carattere alfanumerico standard e non un carattere speciale,
        // allora il carattere di destinazione è uguale a quello sorgente
        else  *dest=*src;
    }
     
    // Termino la stringa di destinazione
    dest[len]='\0';
}
```

### POST

La scelta tra metodo GET e metodo POST è legata ad un preciso criterio di
programmazione, che prevede che un GET venga scelto se e soltanto se i campi
all'interno del form sono idempotenti tra di loro. Questa è la regola formale.
La regola empirica insegna che le richieste GET vanno usate solo per campi di
piccole dimensioni (ad esempio, form con checkbox, con variabili contenenti i
nomi di pagine esterne da richiamare all'interno del codice, con campi
contenenti piccole stringhe e così via). Non è una buona idea utilizzare
richieste GET, ad esempio, per inviare un messaggio postato da un utente in un
forum, in quanto verrà fuori un URL lunghissimo senza senso. È anche rischioso
usare GET per form di login, in quanto i dati di autenticazione passerebbero in
chiaro nell'URL. In tutti questi casi (e altri) è consigliabile l'uso del metodo
POST.

Il metodo POST genera una query string che è uguale in tutto e per tutto a
quella generata dal metodo GET (nel nostro caso, sempre nome=nome_inserito). La
differenza è il metodo GET prevede che la query venga inviata al server tramite
la variabile d'ambiente `QUERY_STRING`, mentre a livello client viene integrata
nell'URL stesso. Il metodo POST invece prevede che la query venga inviata dal
client al server all'interno del pacchetto HTTP stesso, e viene letta dal server
come se fosse un normale input (quindi con scanf, gets o fgets). Prima di
inviare la query vera e propria il client invia al server una stringa che
identifica la lunghezza della query che sta per essere inviata. Questa stringa
viene salvata dal server nella variabile d'ambiente `CONTENT_LENGTH`. In questo
modo il server riceve la lunghezza della query che sta per essere inviata,
prepara un buffer di dimensioni adatte e quindi legge la query con le funzioni
per la lettura dell'input già viste. Dopo la procedura rimane uguale (ovvero
lettura del contenuto di una variabile con un metodo come `get_field` e
decodifica dei caratteri con un metodo come unencode).

Ecco un esempio di codice HTML che invia i dati di un form tramite metodo POST
(esempio tipico, l'invio di un messaggio in un form che viene poi inviato ad un
eseguibile CGI e stampato su schermo):

```html
<form method="POST" action="/cgi-bin/cgi2">
  Inserisci qui il tuo messaggio:<br>
  <textarea cols=50 rows=4 wrap="physical" name="msg"/><br>
  <input type="submit" value="Invia"/>
</form>
```

Ed ecco una potenziale applicazione CGI per elaborarlo:

```c
#include <stdio.h>
#include <stdlib.h>
 
// Il contenuto delle funzioni get_field() e unencode()
// è lo stesso visto nel codice precedente
void unencode (char *src, char *dest, int len);
char* get_field(char *query, char *field);
 
main()  {
  char *query,*msg;
  int len;
 
  // Genero la pagina HTML
  printf ("Content-type: text/html\n\n"
          "<html>\n"
          "<head>\n"
          "<title>Messaggio inserito</title>\n"
          "</head>\n"
          "<body>\n");
 
  // Se la variabile d'ambiente CONTENT_LENGTH è nulla, oppure
  // la conversione in intero con sscanf non produce un intero valido, esco
  if (getenv("CONTENT_LENGTH") == NULL || sscanf ( (char*) getenv("CONTENT_LENGTH"), "%d", &len) != 1)  {
    printf ("Contenuto non valido\n"
            "</body></html>\n");
    exit(1);
  }
 
  // Query sarà grande tanto quanto il pacchetto che sta per inviarmi il client
  query = (char*) malloc (++len*sizeof(char));
 
  // Leggo la richiesta POST inviatami dal client
  // come se fosse un normale input con fgets
  fgets (query,len,stdin);
 
  // Leggo il campo
  msg = get_field (query,"msg");
 
  printf ("Messaggio inserito:<br>\n%s\n",msg);
  exit(0);
}
```

## Link esterni

È possibile usare librerie già pronte per l'uso di eseguibili CGI in C (come
`get_field`, `unencode` e altre), senza dover reinventare la ruota e riscrivere
funzioni da zero di volta in volta. Sul mio sito (http://0x00.ath.cx o
http://blacklight.gotdns.org) è possibile trovare la mia libreria, testata con
successo su sistemi Unix, che già contiene molte funzioni comode per la
scrittura di eseguibili CGI.
