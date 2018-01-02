# Socket e connessioni di rete in C

Tutte le connessioni di rete, dalle applicazioni per la posta elettronica a
quelle per la gestione di una rete aziendale, a livello di protocollo non sono
altro che canali di comunicazione tra processi residenti su macchine diverse in
collegamento tra loro. La macchina che offre il servizio viene chiamata server,
quella che lo richiede client (nel caso particolare della posta elettronica il
server sarà il server POP3 o IMAP dal quale scarichiamo i messaggi, il client la
nostra applicazione, sia essa Outlook, Thunderbird o Eudora). Il canale che
viene creato tra il processo residente sul server e quello residente sul client
è il socket, una struttura FIFO (First In First Out) che a livello logico non è
molto diverso da una pipe. La differenza sta nel fatto che la pipe è un canale
che generalmente mette in comunicazione due processi residenti sulla stessa
macchina, mentre il socket è tra due processi residenti su macchine diverse, e
inoltre un socket è un canale di comunicazione bidirezionale (sullo stesso
socket il client può sia leggere informazioni provenienti dal server sia
scrivere informazioni), mentre invece una pipe mette a disposizione un canale
per la lettura e uno per la scrittura. A parte queste differenze, a livello
concettuale le due strutture per la comunicazione inter-processo sono
relativamente simili.

## Protocolli TCP e UDP

Negli esempi che prenderemo in esame faremo riferimento al protocollo TCP/IP, lo
standard su cui si poggia l'intera infrastruttura di internet. L'altro
protocollo (UDP), basato su datagrammi, consente l'invio di pacchetti di
dimensioni variabili ed è, per alcune applicazioni, relativamente più veloce, ma
non garantisce l'invio di pacchetti ordinati, né l'effettivo recapito a
destinazione dei pacchetti stessi (in quanto si tratta di un modello
fondamentalmente connectionless oriented, a differenza del TCP che è connection
oriented). Prendendo in esame in questa sede applicazioni tipicamente
internet-oriented, useremo la famiglia protocollare che in C identifica il
dominio di comunicazione internet (definito in `<sys/socket.h`> come AF\_INET,
per il formato di indirizzi IPv4, e AF\_INET6, per il formato IPv6). Ci sono
anche altri domini di socket (dominio Unix, AF\_UNIX, dominio Novell, AF\_IPX, e
dominio AppleTalk, per macchine Mac, AF\_APPLETALK), che però non prenderemo in
esame in questa sede.

Per gli esempi in C che esamineremo faremo riferimento alle librerie Unix per la
gestione dei socket. Con qualche piccola modifica, in ogni caso, il codice qui
scritto è operativo anche su sistemi Windows.

## Indirizzi IP e endianness

L'indirizzo IP identifica univocamente una macchina all'interno di una rete, e
consiste (almeno nella versione 4 del protocollo, versione universalmente usata
da anni in tutte le reti) in 4 gruppi di numeri che possono andare da 0 a 255
(in esadecimale 0,...,FF). Un indirizzo IP occupa quindi complessivamente 32 bit
(4 byte) in memoria.

Per poter utilizzare indirizzi IP in un'applicazione in C è necessario passare
la stringa che corrisponde all'IP alla funzione inet\_addr, definita in
`<arpa/inet.h>`. Esempio:

```c
#include <sys/types.h>
#include <arpa/inet.h>
 
.....
 
in_addr_t addr;
struct in_addr a;
 
// Nella mia applicazione, la variabile addr sarà associata all'IP di localhost
addr = inet_addr(“127.0.0.1”)
 
// Associo al membro s_addr della struttura in_addr la variabile appena
associata all'indirizzo
a.s_addr=addr;
 
printf (“Indirizzo IP associato a 0x%x: %s\n”,
        addr, inet_ntoa(a));
```

In `<netinet/in.h>` è definita la costante INADDR\_ANY, che identifica un
qualsiasi indirizzo IP (usato nel codice dei server per specificare che
l'applicazione può accettare connessioni da qualsiasi indirizzo).

Attenzione, avrete notato l'uso di un membro della struttura in\_addr. Tale
struttura (relativamente scomoda e in sé per sé poco utile, ma preservata nella
gestione degli indirizzi in C per una compatibilità con il passato) è deputata a
contenere indirizzi di rete, ed è così definita:

```c
struct in_addr  {
    u_long   s_addr;
}
```

s\_addr conterrà l'indirizzo ottenuto con inet\_addr. Noterete poi l'uso della
funzione inet\_ntoa (Network to ASCII), che vuole come parametro un dato di tipo
in\_addr. Tale funzione è necessaria per ottenere una stringa ASCII standard a
partire da un indirizzo per un motivo particolare, legato alle convenzioni del
protocollo TCP/IP. In tale protocollo, infatti, si usa una convenzione di tipo
big endian (ovvero le variabili più grandi di un byte si rappresentano a partire
dal byte più significativo). Tale convenzione era in uso anche su altre
macchine, come i processori Motorola e i VAX, ma la maggioranza delle macchine
odierne usa lo standard little endian (prima i byte meno significativi e poi a
salire quelli più significativi) per rappresentare le informazioni in memoria o
nella CPU. Per leggere sulla mia macchina un informazione passata in formato di
rete e viceversa devo quindi fare ricorso a funzioni in grado di passare da una
convenzione all'altra. La funzione duale di inet\_ntoa sarà ovviamente
inet\_aton, che converte una stringa in formato host che rappresenta un IP
(quindi con numeri e punti) in formato binario di rete, per poi salvarla
all'interno di una struttura in\_addr passata come parametro alla funzione:

```c
int inet_aton(const char *cp, struct in_addr *inp);
```

Esistono anche funzioni per operare conversioni su tipi di dato diversi dalle
stringhe, quali htonl (da codifica Host Byte Order a codifica Network Byte
Order, Long), htons (da codifica Host a codifica Network, Short), ntohl (da
codifica Network a codifica Host, Long) e ntohs (da codifica Network a codifica
Host, Long).

## Porte

Per poter effettuare una connessione non basta un indirizzo IP e il protocollo
da usare, è necessario anche specificare la porta dell'host alla quale si
desidera collegare il socket, ovvero il servizio da richiedere. In definitiva,
quindi, per costruire un socket per la comunicazione tra un client e un server
ho bisogno di

* Protocollo per la comunicazione (TCP, UDP);
* Indirizzo IP di destinazione;
* Porta su cui effettuare il collegamento.

Per poter utilizzare un socket in un programma ho bisogno di far ricorso alle
strutture sockaddr, definite in `<sys/socket.h>`. La struttura sockaddr di
riferimento è strutturata in questo modo:

```c
struct sockaddr  {
    // Famiglia del socket
    short sa_family;
     
    // Informazioni sul socket
    char  sa_data[];
}
```

Nel nostro caso, in cui useremo dei socket per la comunicazione di applicazioni
via internet, useremo la struttura sockaddr\_in, convertendola in sockaddr,
quando richiesto, tramite operatori di cast:

```c
struct sockaddr_in  {
    // Flag che identifica la famiglia del socket,
    // in questo caso AF_INET
    short sa_family;
     
    // Porta
    short sin_port;
     
    // Indirizzo IP, memorizzato in una struttura
    // di tipo in_addr
    struct        in_addr sin_addr;

    // Riempimento di zeri
    char  sin_zero[8];
}
```

Ci sono caratteristiche in questa struttura quantomeno curiose e apparentemente
obsolete e ridondanti, ma conservate per tradizione e per compatibilità con il
passato. In primis il riferimento alla struttura in\_addr (vista prima) per
memorizzare l'indirizzo IP, quando si poteva tranquillamente ricorrere ad una
variabile long. Il riferimento a questa struttura all'interno di sockaddr è uno
dei più profondi misteri della tradizione Unix. In secundis, il riempimento
della struttura con una stringa (sin\_zero) che non fa altro che contenere degli
zeri, o comunque caratteri spazzatura. Ciò è necessario per rendere la
dimensione della struttura pari esattamente a 16 byte, in modo da poter
effettuare senza problemi il cast da sockaddr a sockaddr\_in e viceversa (in
quanto sono della stessa dimensione).

## Inizializzazione dell'indirizzo

Per inizializzare l'indirizzo all'interno della nostra applicazione dovremo
quindi far ricorso ad un membro della struttura sockaddr\_in, specificando al
suo interno famiglia protocollare (AF\_INET), porta e indirizzo IP. Per fare ciò
conviene creare una procedura esterna al main che faccia il tutto:

```c
void addr_init (struct sockaddr_in *addr, int port, long int ip)  {
    // Inizializzazione del tipo di indirizzo (internet)
    addr->sin_family=AF_INET;
     
    // Inizializzazione della porta (da host byte order
    // a network byte order
    addr->sin_port = htons ((u_short) port);
     
    // Inizializzazione dell'indirizzo (passando per la
    // struttura in_addr
    addr->sin_addr.s_addr=ip;
}
```

Per l'invocazione della procedura ricorreremo a qualcosa del genere:

```c
// Porta per il collegamento
#define PORT            3666;
 
// IP della macchina a cui collegarsi
#define IP              “192.168.1.1”
 
// Variabile sockaddr_in che identifica la macchina a cui ci vogliamo collegare
struct sockaddr_in      server;
 
.......
 
addr_init  (&server,PORT,inet_addr(IP));
```

## Creazione del socket e connessione

Per l'inizializzazione di un socket ricorreremo invece alla primitiva socket,
che prende come parametri il dominio a cui fare riferimento (nel caso di
un'applicazione internet AF\_INET), il tipo di socket (SOCK\_STREAM nel caso di
TCP, SOCK\_DGRAM nel caso di UDP) e il protocollo (impostando questo parametro a
zero il protocollo viene scelto in modo automatico). La funzione ritorna un
valore intero che identifica il socket (socket descriptor) e che verrà usato in
seguito per le connessioni, le letture e le scritture, allo stesso modo di un
descrittore per file, per processi o per pipe. La funzione ritorna invece -1 nel
caso in cui non sia stato possibile creare il socket. Esempio di chiamata:

```c
// Descrittore del socket
int sd;
 
.........
 
if ((sd=socket(AF_INET,SOCK_STREAM,0))<0)  {
    printf ("Impossibile creare un socket TCP/IP\n");
    exit(3);
}
```

Per la chiusura del socket ricorreremo invece alla primiva close, passandogli
come parametro il descrittore del nostro socket.

A questo punto è possibile connettersi all'host sfruttando il socket appena
creato, usando la primitiva connect. Tale primitiva richiede come parametri

* Il descrittore del socket da utilizzare;
* Un puntatore a sockaddr, contenente le informazioni circa dominio del socket,
  indirizzo IP di destinazione e porta (l'abbiamo creato in precedenza);
* La dimensione del puntatore a sockaddr.

La funzione, in modo analogo a socket, ritorna -1 nel caso la connessione non
sia andata a buon fine. Esempio pratico per il nostro caso:

```c
if (connect(sd, (struct sockaddr*) &server, sizeof(struct sockaddr))<0)  {
    printf ("Impossibile collegarsi al server %s sulla porta %d\n",
            inet_ntoa(server.sin_addr.s_addr),PORT);
    exit(4);
} else {
    printf (“Connessione effettuata con successo al server %s sulla porta
            %d\n”, inet_ntoa(server.sin_addr.s_addr),PORT);
}
```

In questo caso è richiesto l'operatore di cast esplicito, in quanto in
precedenza abbiamo creato una variabile di tipo sockaddr\_in ma la funzione
richiede una variabile di tipo sockaddr.

## Lettura e scrittura di informazioni sul socket

Creato il canale di comunicazione, per sfruttarlo all'interno della nostra
applicazione abbiamo bisogno di scrivere o leggere dati su di esso, in modo da
mettere in comunicazione le due macchine. Ciò è possibile grazie alle funzioni
recv e send, rispettivamente per leggere e scrivere sul socket.

La sintassi di queste due funzioni è praticamente uguale. Entrambe prendono 4
parametri:

```c
ssize_t recv(int s, void *buf, size_t len, int flags);
ssize_t send(int s, const void *buf, size_t len, int flags);
```

* Il socket da sfruttare (da cui leggere o su cui scrivere);
* Un puntatore ai dati interessati (una variabile su cui salvare i dati letti o
  la variabile da scrivere su socket);
* La dimensione dei dati (da leggere o da scrivere);
* Un eventuale flag (si lascia a 0 nella maggior parte dei casi).

Entrambe le funzioni ritornano il numero di byte letti o scritti, quindi si
possono fare dei cicli con queste funzioni del tipo "finché ci sono dati da
leggere o scrivere su socket, fai una certa cosa" sfruttando il fatto che quando
non ci sono più dati le funzioni ritornano zero.

## Lato server

Per inizializzare una comunicazione di rete su un client basta questa procedura:

* **addr\_init** (inizializzazione della variabile di tipo sockaddr\_in che
  identifica l'indirizzo e la porta);
* **socket** (creazione del socket per la comunicazione con il server);
* **connect** (connessione al server sfruttando il socket appena creato).

Su un server sono necessari un paio di passaggi in più. La procedura in genere è
questa (non solo in C ma per qualsiasi linguaggio di programmazione):

* **addr\_init** (inizializzazione della variabile di tipo sockaddr\_in);
* **socket** (creazione del socket);
* **bind** (creazione del legame tra il socket appena creato e la variabile;
* **sockaddr\_in** che identifica l'indirizzo del server);
* **listen** (mette il server in ascolto per eventuali richieste da parte dei
  client);
* **accept** (accettazione della connessione da parte di un client).

La sintassi di bind è la seguente:

```c
int bind(int sockfd, const struct sockaddr *my_addr, socklen_t addrlen);
```

dove _sockfd_ è l'identificatore del socket, _*my_addr_ il puntatore alla
variabile di tipo sockaddr che identifica l'indirizzo e _addrlen_ la lunghezza
di tale variabile. La funzione ritorna 0 in caso di successo, -1 in caso di
errore.

La sintassi di listen invece è la seguente:

```c
int listen(int sockfd, int backlog);
```

dove _sockfd_ è il descrittore del socket e _backlog_ il numero massimo di
connessioni che il server può accettare contemporaneamente. Anche questa
funzione ritorna 0 in caso di successo e -1 in caso di errore.

La sintassi di accept infine è la seguente:

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

dove _sockfd_ è il descrittore del socket, _*addr_ il puntatore alla variabile
di tipo sockaddr e _*addrlen_ il puntatore alla sua lunghezza. In caso di
successo accept ritorna un valore >0 che è il descrittore del socket accettato,
mentre ritorna -1 in caso di errore.

Attenzione: la accept ritorna un nuovo identificatore di socket, che è il socket
da utilizzare da quel momento in poi per le comunicazioni con il client.
Inoltre, alla accept va passato il puntatore alla variabile sockaddr che
identifica il client, non quello del server.

## Esempio pratico

Bando alle ciance, vediamo ora un semplice codice in C per l'invio di messaggi
sulla rete sfruttando i socket TCP che abbiamo appena esaminato. Il server
rimane in attesa di messaggi sulla porta 3666 e quando arrivano li scrive su
stdout, mentre il client si collega al server (il cui indirizzo è passato come
parametro da riga di comando) e gli invia un messaggio, passato anch'esso come
una lista di parametri da riga di comando.

Codice del client:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/socket.h>
#include <errno.h>

// Porta per la comunicazione
#define PORT            3666
 
// Inizializzazione della variabile sockaddr_in
void addr_init (struct sockaddr_in *addr, int port, long int ip)  {
    addr->sin_family=AF_INET;
    addr->sin_port = htons ((u_short) port);
    addr->sin_addr.s_addr=ip;
}

main(int argc, char **argv)  {
    int i,sd;
    int var1,var2,var3,var4;
    int sock_size=sizeof(struct sockaddr_in);
    int N,status;
    pid_t pid;
    struct sockaddr_in server,client;
     
    // Controllo che vengano passati almeno due argomenti
    if (argc<3) {
        printf ("%s <server> <msg>\n",argv[0]);
        exit(1);
    }

    // Controllo che l'IP del server passato sia un indirizzo IPv4 valido
    if (sscanf(argv[1],"%d.%d.%d.%d",&var1,&var2,&var3,&var4) != 4)  {
        printf ("%s non è un indirizzo IPv4 valido\n",argv[1]);
        exit(2);
    }
                                                                 
    // Inizializzazione dell'indirizzo
    addr_init (&server,PORT,inet_addr(argv[1]));

    // Creazione del socket
    if ((sd=socket(AF_INET,SOCK_STREAM,0))<0)  {
        printf ("Impossibile creare un socket TCP/IP\n");
        exit(3);
    }
                                                                 
    // Creazione della connessione
    if (connect(sd, (struct sockaddr*) &server, sock_size)<0) {
        printf ("Impossibile collegarsi al server %s sulla porta %d: errore %d",
                inet_ntoa(server.sin_addr.s_addr),PORT,errno);
        exit(4);
    }

    printf ("Connessione stabilita con successo con il server %s sulla porta %d\n",
            inet_ntoa(server.sin_addr.s_addr), ntohs(server.sin_port));

    // Il numero di parole contenute nel messaggio è pari ad argc-2,
    // ovvero argc-(nome del programma)-(IP del server) N=argc-2;

    // Dico al server che sto per inviargli N stringhe
    send (sd, (int*) &N, sizeof(int), 0);

    // Per i che va da i ad argc...
    for (i=2; i<argc; i++)  {
        // ...N è la lunghezza dell'i-esima stringa
        N=strlen(argv[i]);
         
        // Dico al server che sto per inviargli una stringa lunga N caratteri
        send (sd,(int*)&N,sizeof(int),0);
         
        // Invio al server la stringa
        send (sd,argv[i],N,0);
        printf ("Stringa %s lunga %d caratteri inviata con successo a %s",
                argv[i],N,inet_ntoa(server.sin_addr.s_addr));
    }

    // Chiusura della connessione
    close(sd);
    exit(0);
}
```

Codice del server:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>

// Porta su cui mettersi in ascolto
#define PORT            3666

// Numero massimo di connessioni accettabile
#define MAXCONN 5
 
void addr_init (struct sockaddr_in *addr, int port, long int ip)  {
    addr->sin_family=AF_INET;
    addr->sin_port = htons ((u_short) port);
    addr->sin_addr.s_addr=ip;
}

main()  {
    int sd,new_sd;
    struct sockaddr_in server,client;
    int sock_size=sizeof(struct sockaddr_in);
    int pid,status;
    int i,args,N;
    char *buff;

    // Inizializzazione dell'indirizzo
    // Con INADDR_ANY specifico che posso accettare connessioni da qualsiasi indirizzo
    addr_init (&server,PORT,INADDR_ANY);

    // Creazione del socket
    if ((sd=socket(AF_INET,SOCK_STREAM,0)) < 0)  {
        printf ("Impossibile inizializzare il socket TCP/IP %d\n",
                getsockname (sd, (struct sockaddr*) &server, &sock_size));
        exit(1);
    }

    // Lego il socket appena creato all'indirizzo del server
    if (bind(sd, (struct sockaddr*) &server, sizeof(server))<0)  {
        printf ("Impossibile aprire una connessione sulla porta %d\n"
                "La porta potrebbe essere già in uso da un'altra applicazione",
                PORT);
        exit(2);
    }

    printf ("Server in ascolto sulla porta %d\n",PORT);
             
    // Metto il server in ascolto
    if (listen(sd,MAXCONN)<0)  {
        printf ("Impossibile accettare nuove connessioni sul socket creato\n");
        exit(3);
    }

    printf ("Server in ascolto - accetta fino a un massimo di %d connessioni\n",MAXCONN);

    // Accetto connessioni finché ce ne sono
    while (1)  {
        // Accetto le connessioni da parte del client creando un nuovo socket
        if ((new_sd=accept(sd,(struct sockaddr*)&client,&sock_size)) < 0)  {
            printf ("Impossibile accettare una connessione dal client %s\n",
                    inet_ntoa(client.sin_addr.s_addr));
            exit(4);
        }

        printf ("Connessione stabilita con successo con il client %s sulla
        porta %d\n",inet_ntoa(client.sin_addr.s_addr),ntohs(client.sin_port));

        // Ricevo il numero di messaggi che il client ha da inviare
        recv (new_sd, (int*) &args, sizeof(int), 0);
                      
        printf ("Stringa ricevuta da %s: ", inet_ntoa(client.sin_addr.s_addr));

        // Finché il client ha stringhe da inviare...
        for (i=0; i<args; i++)  {
            // ...leggo la dimensione dell'i-esima stringa
            recv (new_sd, (int*) &N, sizeof(int), 0);
             
            // Alloco memoria per ricevere la stringa
            buff = (char*) malloc(N*sizeof(char));
             
            // Ricevo la stringa
            recv (new_sd,buff,N,0);
             
            // Scrivo su stdout la stringa appena ricevuta
            printf ("%s ",buff);
        }

        printf ("\n");
    }
}
```
