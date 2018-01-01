## Libreria time.h

La libreria _time.h_ è dedicata alla gestione della data e dell'ora. Comprende
funzioni di tre tipologie: _tempi assoluti_, rappresentano data e ora nel
calendario gregoriano; _tempi locali_, nel fuso orario specifico; _variazioni
dei tempi locali_, specificano una temporanea modifica dei tempi locali, ad
esempio l'introduzione dell'ora legale. Contiene le dichiarazioni della funzione
**time()**, che ritorna l'ora corrente, e la funezione **clock()** che
restituisce la quantità di tempo di CPU impiegata dal proramma.

## time\_t

Il tipo time\_t, definito in _time.h_, non è altro che un _long int_ addibito al
compito di memorizzare ora e date misurate in numero di secondi trascorsi dalla
mezzanotte del 1° gennaio 1970, ora di Greenwich. Bisogna ammetere che è un modo
un po' bislacco di misurare il tempo, ma è così perché così si è misurato il
tempo sui sistemi Unix. Tuttavia, nonostante possa sembrare un modo strano di
rappresentare il tempo, questa rappresentazione è estremamente utile per fare
confronti tra date che, essendo tutte rappresentate in questo modo, si riducono
a semplici confronti tra numeri interi, senza che ci sia bisogno di confrontare
giorni, mesi e anni. Ma ci sono anche problemi legati a questa rappresentazione.
La rappresentazione del tipo time\_t infatti è una rappresentazione a 32 bit,
che ammette numeri negativi (ovvero numero di secondi prima del 1 gennaio 1970,
che consente la rappresentazione di date fino al 1900) e numeri positivi (numero
di secondi passati dal 1 gennaio 1970). Il bit più significativo del numero
binario identifica il segno (0 per i numeri positivi, 1 per quelli negativi). In
questo modo è possibile rappresentare fino a 2^32^ numeri positivi, ovvero numero
di secondi dopo la data per eccellenza, e questo è un problema perché con una
tale rappresentazione la data andrà in overflow intorno al 2038 (ovvero, in un
certo momento dopo le prime ore del 2038 si arriverà ad un punto in cui la cifra
più significativa del numero andrà a 1, quindi le date cominceranno a essere
contate dal 1900). Il bug del 2038 è molto conosciuto in ambiente Unix, e per
porre rimedio si sta da tempo pensando di migrare ad una rappresentazione della
data a 64 bit.

## struct tm

_tm_ è una struttura dichiarata sempre in _time.h_, contiene informazioni circa
l'ora e la data, questo è il conenuto:

```c
struct tm {
    int tm_sec   //secondi prima del completamento del minuto
    int tm_min   //minuti prima del completamento dell'ora
    int tm_hour  //ore dalla mezzanotte
    int tm_mday  //giorno del mese
    int tm_mon   //mesi passati da gennaio
    int tm_year  //anni passati dal 1900
    int tm_wday  //giorni passati da Domenica
    int tm_yday  //giorni passati dal 1 Gennaio
    int tm_isdst //''unknow'' (lol)
};
```

## Esempio

Ecco un piccolo programma che ci mostra a schermo ora e data.

```c
#include <stdio.h>
#include <time.h>
 
int main(int argc, char *argv[])
{
    time_t a;
    struct tm *b;
     
    time(&a);
    b = localtime(&a);
    printf("Ora esatta: %s\n", asctime(b));
     
    return 0;
}
```

Un altro modo per visualizzare la data attuale senza ricorrere a un membro della
struttura tm è il seguente:

```c
#include <stdio.h>
#include <time.h>
 
main()  {
    time_t ltime = time();
    printf ("%s\n",ctime(&ltime));
}
```

La funzione _ctime_ prende l'indirizzo di una variabile di tipo time\_t
inizializzata tramite _time_ e stampa il suo valore in formato ASCII. Il formato
standard è

```
Giorno della settimana (3 lettere) Mese (3 lettere) Giorno del mese (2 cifre)
hh:mm:ss Anno (4 cifre)
```

Un modo per stampare il tempo in un altro formato diverso da quello previsto da
ctime e asctime è quello di usare la funzione _strftime_, che prende come
parametri

* Una stringa nella quale salvare la data nel formato che si è scelto;
* La dimensione della stringa;
* Una stringa di formato (simile a printf) nel quale si specifica il formato in
  cui stampare la data;
* Un puntatore a struttura tm.

Esempio:

```c
#include <stdio.h>
#include <time.h>
 
main()  {
    time_t timer=time();
    struct tm *now=localtime(&timer);
    char timebuf[20];
     
    strftime (timebuf,sizeof(timebuf),"%d/%m/%Y,%T",now);
    printf ("%s\n",timebuf);
}
```

Stampa la data nel formato

    gg/MM/aaaa,hh:mm:ss

La funzione localtime prende come parametro un puntatore a variabile time\_t e
ritorna una struttura tm corrispondente a quel tempo.

