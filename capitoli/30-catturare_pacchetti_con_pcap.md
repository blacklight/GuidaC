# Catturare pacchetti con le librerie PCAP

La libreria _PCAP_, disponibile per sistemi Unix e Microsoft e scaricabile
gratuitamente da qui, offre al programmatore l'opportunità di usare all'interno
del suo codice funzioni per la cattura e la gestione di tutti i pacchetti che
transitano su un'interfaccia di rete. Su questa libreria di basa il celeberrimo
_tcpdump_, il più classico degli sniffer/monitor di rete (e infatti la libreria
è messa a punto dagli stessi sviluppatori di tcpdump), e anche sniffer più
avanzati come Ethereal/Wireshark, e un porting in Java chiamato _Jpcap_ che
offre le funzionalità della libreria stessa anche nell'ambiente del linguaggio
Sun.

## Compilare e linkare programmi con le librerie PCAP

La procedura che vedremo qui mostra come compilare programmi con le librerie
PCAP in ambiente Unix e con gcc, ma in generale è valida per qualsiasi libreria
esterna installata sul sistema. Una volta installata la libreria è necessario
includere nei sorgenti che ne fanno uso l'header `pcap.h`, e quindi compilare e
linkare il programma con l'opzione `-lpcap` passata a gcc:

    gcc -o nome_programma sorgente.c -lpcap

Inoltre i programmi che fanno uso delle librerie PCAP, accedendo alle interfacce
di rete con i privilegi di superutente, hanno bisogno di essere avviati con i
privilegi di root su sistemi Unix, di amministratore su sistemi Windows.

## Trovare un'interfaccia di rete

La prima informazione che bisogna passare alle funzioni di PCAP è l'interfaccia
di rete su cui si intende effettuare lo sniffing o il monitoring. Ciò è
possibile con la funzione `pcap_lookupdev`, che trova automaticamente la prima
interfaccia di rete disponibile sul sistema:

```c
#include <stdio.h>
#include <stdlib.h>
#include <pcap.h>
 
main(int argc, char **argv)  {
    char *dev, errbuf[PCAP_ERRBUF_SIZE];
     
    dev = pcap_lookupdev(errbuf);
     
    if (!dev)  {
        printf ("Errore: nessuna interfaccia di rete trovata sul
                sistema: %s\n",errbuf);
        exit(1);
    }
     
    printf ("Dispositivo di rete %s disponibile sul
            sistema\n",dev);
}
```

La funzione `pcap_lookupdev` in pratica cerca il miglior dispositivo di rete
disponibile sul sistema e ritorna una stringa ad esso associata, altrimenti NULL
se non c'è nessun dispositivo di rete. La stringa `errbuf`, di lunghezza
`PCAP_ERRBUF_SIZE` definita in `pcap.h`, serve a contenere eventuali messaggi di
errore.

Per trovare invece tutte le interfacce di rete sul sistema possiamo usare la
funzione `pcap_findalldevs`, che prende come parametri un puntatore a puntatore
a un tipo di dato `pcap_if_t` e il solito buffer di errore. `pcap_if_t` non è
altro che un tipo di dato che identifica un'istanza della struttura `pcap_if`
così definita:

```c
struct pcap_if {
    struct pcap_if *next;
    char *name;         /* name to hand to "pcap_open_live()" */
    char *description;  /* textual description of interface, or
                           NULL */
    struct pcap_addr *addresses;
    bpf_u_int32 flags;  /* PCAP_IF_ interface flags */
};
```

Ecco quindi un codice per visualizzare tutte le interfacce di rete disponibili
su un sistema:

```c
pcap_if_t *ifc;
char errbuf[PCAP_ERRBUF_SIZE];
struct sockaddr_in *addr;
 
pcap_findalldevs (&ifc,errbuf);
 
printf ("Interfacce di rete trovate sul sistema:\n\n");
 
// Finché ci sono interfacce da visualizzare...
while (ifc->next)  {
    // ...stampo nome e descrizione
    printf ("%s: %s\n",ifc->name,ifc->description);
     
    // Finché ci sono indirizzi associati all'interfaccia di rete...
    while (ifc->addresses)  {
        // ...stampo gli indirizzi
        addr = (struct sockaddr_in*) ifc->addresses->addr;
        printf ("Indirizzo: %s\n",inet_ntoa(addr->sin_addr.s_addr));
         
        // Passo all'indirizzo successivo
        ifc->addresses=ifc->addresses->next;
    }
     
    // Passo all'interfaccia di rete successiva
    ifc=ifc->next;
}
```

Per verificare l'indirizzo e la netmask associate ad un'interfaccia di rete
conviene usare la funzione `pcap_lookupnet()`, che prende come argomenti

* Il nome dell'interfaccia di rete;
* Un puntatore ad una variabile a 32 bit che identifica la rete;
* Un puntatore ad una variabile a 32 bit che identifica la netmask;
* errbuf.

La funzione ritorna -1 nel caso non ci sia nessun indirizzo associato ad
un'interfaccia di rete. Esempio, per trovare l'indirizzo associato
all'interfaccia di rete eth0:

```c
bpf_u_int32 net,mask;
char errbuf[PCAP_ERRBUF_SIZE];
 
......
 
if (pcap_lookupnet("eth0",&net,&mask,errbuf)==-1)  {
    printf ("Nessun indirizzo associato a eth0: %s\n",errbuf);
    exit(1);
}
```

## Sniffing

La funzione messa a disposizione dalle PCAP per l'apertura di un dispositivo di
rete per lo sniffing è `pcap_open_live()`. Tale funzione prende come argomenti:

* Il nome del dispositivo di rete su cui effettuare lo sniffing;
* Il numero massimo di byte da catturare per ogni sessione;
* Un valore booleano (promisc) che se settato a 1 pone il dispositivo di rete in
  modalità promiscua. Se lasciato a 0 di default PCAP snifferà solo il traffico
  di rete diretto verso la propria interfaccia di rete;
* `to_ms`, che identifica il numero di secondi passati i quali la sessione di
  sniffing va in timeout. Se settato a 0 non ci sarà nessun timeout per la
  sessione di sniffing;
* errbuf.

La funzione ritorna un puntatore ad una variabile di tipo `pcap_t`, che per il
resto del listato sarà il descriptor della nostra sessione di sniffing, oppure
NULL in caso di errore. Esempio pratico:

```c
pcap_t *sniff;
char errbuf[PCAP_ERRBUF_SIZE];
 
......
 
if (!(sniff=pcap_open_live("eth0",1024,1,0,errbuf)))  {
    printf ("Errore nella creazione di una sessione di sniffing su eth0:
            %s\n",errbuf);
    exit(1);
}
 
printf ("Sessione di sniffing creata con successo\n");
```

Questo codice apre una sessione di sniffing sul dispositivo eth0 in modalità
promiscua, leggendo 1024 byte per volta, senza un timeout impostato per la
sessione e con un eventuale buffer di errore salvato in `errbuf`. Se
l'esecuzione del codice va a buon fine in `sniff` troveremo un descrittore per
la nostra sessione di sniffing da usare in seguito nel codice.

A questo punto è necessario compilare la sessione di sniffing specificando un
eventuale filtro. Il filtro servirà nel caso in cui non vogliamo sniffare tutto
il traffico di rete ma solo quello diretto o proveniente da una determinata
porta, solo il traffico TCP o solo quello UDP e così via. Per compilare la
sessione useremo la funzione `pcap_compile()` che prende i seguenti argomenti:

* Il descrittore della sessione di sniffing inizializzato precedentemente con
  `pcap_open_live()`;
* Un puntatore ad una variabile di tipo `bpf_program`, dove verrà memorizzata la
  versione compilata della nostra sessione;
* Una stringa di filtro;
* La variabile booleana optimize che stabilisce se il filtro andrà ottimizzato o
  meno;
* La netmask sulla quale verrà applicato il filtro (precedentemente
  inizializzata tramite `pcap_lookupnet()`).

La stringa di filtro sarà una stringa che identificherà il tipo di traffico da
filtrare. La sintassi dettagliata è illustrata qui. In generale, una stringa di
filtro è strutturata nel seguente modo per il filtraggio su una determinata
porta o protocollo:

    [proto] [src|dst] [port numero_porta]

Ad esempio

    tcp dst port 80

catturerà tutti e soli i pacchetti destinati alla porta 80 e scarterà gli altri.
Per il filtraggio sugli host la stringa di filtro sarà così costruita:

    [host] [src|dst indirizzo_host]

Ad esempio

    host dst 192.168.1.1

catturerà tutti e soli i pacchetti destinati all'host 192.168.1.1.

Nel caso non si voglia utilizzare un filtro e si vogliano sniffare tutti i
pacchetti basterà settare la `filter_string` a NULL. Esempio pratico:

```c
pcap_t *sniff;
bpf_u_int32 net,mask;
struct bpf_program filter;
 
// Questo per sniffare senza filtri
char *filter_string=NULL;
 
// Questo per filtrare, per esempio, solo il traffico destinato alla porta 80
char filter_string[] = "tcp dst port 80";
 
......
 
pcap_compile (sniff,&filter,filter_string,0,net);
```

Quest'uso di `pcap_compile()` compilerà la nostra sessione di sniffing puntata
da `sniff`, salverà la versione compilata su `filter` usando la stringa di
filtro `filter_string`, senza opzioni di ottimizzazione e usando la netmask
salvata in `net`.

Una volta creato e compilato il filtro è il caso di associarlo alla nostra
sessione di sniffing. Questo si fa con la funzione `pcap_setfilter()`, che
prende come argomenti.

* Il descrittore di tipo `pcap_t` della sessione;
* Il puntatore all'istanza di `bpf_program` nella quale è salvato il filtro
  appena compilato.

```c
pcap_setfilter (sniff,&filter);
```

Questa riga assocerà il descrittore della sessione creato in precedenza al
filtro appena creato. A questo punto tutto è pronto per cominciare il ciclo di
sniffing vero e proprio attraverso la funzione `pcap_loop()`, che prende come
argomenti

* Il descrittore di tipo `pcap_t` della sessione;
* Il numero di pacchetti da sniffare prima di uscire (0 per non imporre nessun
  limite);
* Il nome della funzione da richiamare quando giunge un pacchetto (la funzione
  che compierà le operazioni richieste su quel pacchetto);
* Eventuali argomenti aggiuntivi (generalmente settati a NULL).

Nel nostro caso:

```c
pcap_loop (sniff,0,pack_handle,NULL);
```

dirà al compilatore di creare un ciclo di sniffing associato al descrittore
`sniff`, senza imporre un limite massimo di pacchetti sniffati. Ogni volta che
un pacchetto transita sull'interfaccia di rete viene richiamata la funzione
`pack_handle()` per gestirla, funzione così definita:

```c
void pack_handle (u_char *args, const struct pcap_pkthdr *p_info, const u_char
                  *packet)  {
```

Questa è la sintassi standard di una funzione passata come argomento a
`pcap_loop()`. Il primo argomento punta all'ultimo argomento specificato in
`pcap_loop()`, ovvero agli eventuali argomenti aggiuntivi aggiunti (generalmente
NULL). Il secondo argomento è un puntatore alla struttura `pcap_pkthdr`, che
contiene informazioni circa il pacchetto appena sniffato. Questa struttura è
così definita:

```c
struct pcap_pkthdr {
    struct timeval ts; /* time stamp */
    bpf_u_int32 caplen; /* length of portion present */
    bpf_u_int32 len; /* length this packet (off wire) */
};
```

I campi disponibili sono

* Ora di cattura del pacchetto.
* Lunghezza della porzione catturata;
* Lunghezza totale del pacchetto.

L'ultimo argomento è un buffer contenente il contenuto vero e proprio del
pacchetto. In questo caso possiamo semplicemente scrivere all'interno della
nostra funzione un

```c
printf ("%s\n",packet);
```

Un programma costruito in questo modo, con una tale funzione, farà un dump di
tutti i pacchetti transitanti su un'interfaccia di rete su stdout. Possiamo fare
qualcosa di più elaborato conoscendo gli standard dei pacchetti TCP/IP. Ad
esempio nel caso di un'interfaccia ethernet risalire al MAC mittente e al MAC
destinatario del pacchetto, tenendo presente che queste informazioni occupano i
primi 12 byte del pacchetto, è un gioco da ragazzi:

```c
printf ("MAC sorgente: %.2x:%2x:%.2x:%.2x:%.2x:%.2x\n",
        packet[0],packet[1],packet[2],packet[3],packet[4],packet[5]);
 
printf ("MAC destinatario: %.2x:%2x:%.2x:%.2x:%.2x:%.2x\n",
        packet[6],packet[7],packet[8],packet[9],packet[10],packet[11]);
```

Per maggiori informazioni sulla struttura dei pacchetti TCP/IP rimando alle
sezioni apposite nell'area reti.

## Packet injection

Tramite le PCAP è anche possibile fare packet injection, ovvero inserire su
un'interfaccia di rete pacchetti costruiti arbitrariamente. La funzione da usare
è in questo caso `pcap_inject()`, che prende i seguenti argomenti:

* Il descrittore della sessione di sniffing;
* Il buffer contenente il pacchetto costruito arbitrariamente;
* La lunghezza del pacchetto.

Si può quindi sniffare un pacchetto su un'interfaccia di rete, modificare il MAC
o l'IP mittente e inviare una risposta al destinatario, che crederà che quel
pacchetto venga dal mittente specificato. Tecnica ancora più efficace se
abbinata a tecniche di ARP poisoning.
