# Raw socket

I socket standard usati in C sono relativamente comodi da usare in quanto
automatizzano tutti i meccanismi implementati dal protocollo TCP/IP, lasciando
allo sviluppatore solo la responsabilità del livello applicativo. Tuttavia in
alcuni contesti si vuole avere il controllo completo di ciò che viene inviato
sulla rete.  Applicazioni tipiche sono l'IP spoofing (invio di un pacchetto ad
un host con un altro IP) e i conseguenti attacchi Smurf. In questi casi può
risultare comodo costruirsi il pacchetto inviato sull'interfaccia di rete pezzo
per pezzo. Per fare ciò il C mette a disposizione i _raw socket_, dei socket su
cui è possibile inviare pacchetti _grezzi_ creati dallo sviluppatore (ovviamente
delle profonde conoscenze dei protocolli di rete e di trasporto sono richieste).
Vediamo subito un esempio pratico con un'applicazione che crea un pacchetto da
zero che pinga localhost e lo invia su raw socket:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <linux/ip.h>

#define ICMP_ECHO 8
#define IPLEN sizeof(struct iphdr)
#define ICMPLEN sizeof(struct icmphdr)

typedef unsigned char u8;
typedef unsigned short u16;
typedef unsigned long u32;

struct icmphdr {
    u8 type;
    u8 code;
    u16 checksum;
    u16 id;
    u16 sequence;
};

unsigned short csum (u16 *buf, int nwords) {
    unsigned long sum;

    for (sum = 0; nwords > 0; nwords--)
        sum += *buf++;

    sum = (sum >> 16) + (sum & 0xffff);
    sum += (sum >> 16);
    return ~sum;
}

main()  {
    int i,sd,one,len;
    unsigned char buff[BUFSIZ],in[BUFSIZ];
    char data[56];
    char *tmp;
     
    struct sockaddr_in sin;
    struct iphdr *ip = (struct iphdr*) malloc(IPLEN);
    struct icmphdr *icmp = (struct icmphdr*) malloc(ICMPLEN);
     
    srand ((unsigned) time(NULL));
    sd=socket (PF_INET, SOCK_RAW, IPPROTO_ICMP);
     
    sin.sin_family=AF_INET;
    sin.sin_port=0;
    sin.sin_addr.s_addr=inet_addr("127.0.0.1");

    memset (buff,0,sizeof(buff));
     
    for (i=0; i<56; i++)
        data[i]=i;
     
    ip->version=4;
    ip->ihl=5;
    ip->tos=0;
    ip->tot_len=IPLEN+ICMPLEN+sizeof(data);
    ip->id=0;
    ip->frag_off=0;
    ip->ttl=64;
    ip->protocol=IPPROTO_ICMP;
    ip->check=0;
    ip->saddr=inet_addr("127.0.0.1");
    ip->daddr=inet_addr("127.0.0.1");
    ip->check = csum ((u16*) buff, ip->tot_len >> 1);

    icmp->type=ICMP_ECHO;
    icmp->code=0;
    icmp->checksum=0;
    icmp->id=1;
    icmp->sequence=1;

    tmp = (char*) malloc(ICMPLEN+sizeof(data));
    memcpy (tmp, icmp, ICMPLEN);
    memcpy (tmp+ICMPLEN, data, sizeof(data));
    icmp->checksum=csum((u16*) tmp, ICMPLEN+sizeof(data) >> 1);

    memcpy (buff, ip, IPLEN);
    memcpy (buff+IPLEN, icmp, ICMPLEN);
    memcpy (buff+IPLEN+ICMPLEN, data, sizeof(data));

    one=1;

    if (setsockopt (sd, IPPROTO_IP, IP_HDRINCL, &one, sizeof (one)) < 0)
        printf ("Warning: Cannot set HDRINCL!\n");
     
        if (sendto (sd, buff, ip->tot_len, 0,
            (struct sockaddr *) &sin, sizeof (sin)) < 0)  {
            printf ("Error in send\n");
            exit(1);
        }
        else
            printf ("Send OK\n");
}
```

Vediamo i componenti notevoli:

```c
#include <linux/ip.h>
```

Questa dichiarazione è necessaria per poter usare la struttura _iphdr_,
contenente tutti i campi di un header IP, che semplifica notevolmente il lavoro.
Successivamente dichiaro la struttura di un header ICMP (_icmphdr_).

```c
unsigned short csum (u16 *buf, int nwords)  {
    unsigned long sum;
     
    for (sum = 0; nwords > 0; nwords--)
            sum += *buf++;
    sum = (sum >> 16) + (sum & 0xffff);
    sum += (sum >> 16);
    return ~sum;
}
```

Questa è la funzione per il calcolo del checksum di un header (complemento a 1
della somma dei complementi a 1 dell'header diviso in word da 16 bit). In
seguito inizializzo il socket come socket raw

```c
sd=socket (PF_INET, SOCK_RAW, IPPROTO_ICMP);
```

riempio gli ultimi 56 byte del pacchetto con byte casuali (struttura classica di
un pacchetto ping)

```c
for (i=0; i<56; i++)
      data[i]=i;
```

quindi riempio gli header IP e ICMP:

```c
ip->version=4;
ip->ihl=5;
ip->tos=0;
ip->tot_len=IPLEN+ICMPLEN+sizeof(data);
ip->id=0;
ip->frag_off=0;
ip->ttl=64;
ip->protocol=IPPROTO_ICMP;
ip->check=0;
ip->saddr=inet_addr("127.0.0.1");
ip->daddr=inet_addr("127.0.0.1");
ip->check = csum ((u16*) buff, ip->tot_len >> 1);
     
icmp->type=ICMP_ECHO;
icmp->code=0;
icmp->checksum=0;
icmp->id=1;
icmp->sequence=1;
```

A questo punto effettuo il calcolo del checksum ICMP

```c
tmp = (char*) malloc(ICMPLEN+sizeof(data));
memcpy (tmp, icmp, ICMPLEN);
memcpy (tmp+ICMPLEN, data, sizeof(data));
icmp->checksum=csum((u16*) tmp, ICMPLEN+sizeof(data) >> 1);
```

in quanto dovrò calcolare il checksum sull'header ICMP e sulla parte di dati.
Ora copio le strutture così riempite in un buffer

```c
memcpy (buff, ip, IPLEN);
memcpy (buff+IPLEN, icmp, ICMPLEN);
memcpy (buff+IPLEN+ICMPLEN, data, sizeof(data));
```

setto l'opzione `IP_HDRINCL` sul socket (necessaria per iniettare pacchetti raw,
richiede i privilegi di root)

```c
one=1;
 
if (setsockopt (sd, IPPROTO_IP, IP_HDRINCL, &one, sizeof (one)) < 0)
    printf ("Warning: Cannot set HDRINCL!\n");
```

quindi effettuo l'invio tramite sendto:

```c
if (sendto (sd, buff, ip->tot_len, 0,
    (struct sockaddr *) &sin, sizeof (sin)) < 0)  {
    printf ("Error in send\n");
    exit(1);
} else
    printf ("Send OK\n");
```
