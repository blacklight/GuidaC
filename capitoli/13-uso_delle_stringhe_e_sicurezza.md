# Uso delle stringhe e sicurezza del programma

Nei paragrafi precedenti abbiamo preso in esame alcune funzioni sulle stringhe
che possono rivelarsi potenzialmente dannose per la stabilità e la sicurezza di
un'applicazione. In questo paragrafo esamineremo i rischi concreti connessi ad
una cattiva gestione delle stringhe.

Esempio pratico:

```c
#include <stdio.h>
#include <string.h>
 
int main()  {
    char s1[2];
    char *s2 = "Questa e' una stringa di prova";
     
    strcpy (s1,s2);
    return 0;
}
```

Eseguendo un codice del genere molto probabilmente l'applicazione andrà in
crash. Se siamo su un sistema Unix il kernel ci risponderà con un bel
segmentation fault, su Windows ci comparirà una finestra che ci avverte che
l'applicazione ha tentato la scrittura su un'area di memoria non valida. Quello
che abbiamo fatto è tentare di copiare in un buffer più byte di quelli che il
buffer stesso può contenere, e in modo non controllato (la strcpy non effettua
una copia controllata, non si ferma se i limiti di capienza della stringa
vengono raggiunti). Il risultato è che l'applicazione va in crash, in quanto,
attraverso la strcpy, è andata a scrivere su una zona di memoria al di fuori di
quella della stringa stessa, andando a sovrascrivere l'indirizzo di ritorno
della funzione con un indirizzo non valido. L'indirizzo viene letto dalla CPU,
che tenta di leggere l'istruzione a quell'indirizzo. Indirizzo che nella maggior
parte dei casi non sarà un indirizzo di memoria valido, quindi provocherà il
crash del programma.

Ma il crash del programma, nonostante sia un danno non da poco, non è nemmeno il
minore dei danni. Esempio pratico con un'applicazione:

```c
#include <stdio.h>
#include <string.h>
 
int main(int argc, char **argv)  {
    char str[16];
     
    strcpy (str,argv[1]);
    return 0;
}
```

Attenzione all'uso di strcpy in questa applicazione. La funzione copia il primo
argomento passato al programma nella stringa str, che può tenere 16 caratteri,
senza fare ulteriori controlli sulla lunghezza effettiva della stringa da
copiare. Proviamo ad avviare l'applicazione con il nostro debugger preferito (in
questo caso userò Gdb) per vedere cosa succede in memoria quando passo al
programma un argomento molto lungo:

```
(gdb) run `perl -e 'print "A" x32'`
Starting program: /home/blacklight/prog/c/5 `perl -e 'print "A" x32'`
Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
```

Il comando `perl -e 'print "A" x32'` non fa altro che richiamare l'interprete
Perl (un linguaggio di programmazione), stampando la lettera "A" 32 volte (un
modo per evitare di scrivere 32 volte "A", giusto una comodità). Il programma,
tentando di copiare un buffer troppo grande in una stringa che non è in grado di
contenere tanti byte, va in crash. Ma vediamo cosa succede a livello dei
registri:

```
(gdb) i r
eax 0xbfab7890 -1079281520
ecx 0xfffff098 -3944
edx 0xbfab8818 -1079277544
ebx 0xb7edeffc -1209143300
esp 0xbfab78b0 0xbfab78b0
ebp 0x41414141 0x41414141
esi 0xbfab7934 -1079281356
edi 0xbfab78c0 -1079281472
eip 0x41414141 0x41414141
eflags 0x210282 [ SF IF RF ID ]
cs 0x73 115
ss 0x7b 123
ds 0x7b 123
es 0x7b 123
fs 0x0 0
gs 0x33 51
```

Da notare il registro EIP. Tale registro contiene, nelle architetture
Intel-based, l'indirizzo in memoria della prossima istruzione da eseguire.
L'indirizzo è stato sovrascritto da una sequenza di 0x41. E 0x41, in
esadecimale, corrisponde al carattere ASCII "A". In pratica il nostro buffer
lungo è andato a sovrascrivere il registro EIP, cambiando il valore
dell'indirizzo della prossima istruzione da pescare in memoria. In questo caso,
la sequenza di 0x41 non rappresenta un indirizzo di memoria valido, o almeno un
indirizzo nel quale il programma può accedere, ragion per cui il programma
crasha. Ma, oltre ad una semplice sequenza di "A", possiamo anche inserire un
buffer costruito apposta, che inietta nel registro un indirizzo valido che punta
ad un codice arbitrario. Siamo quindi nella situazione di un _buffer overflow_
sfruttato in modo da poter eseguire codice arbitrario sul sistema, codice che
può mirare ad aggiungere un nuovo utente con certi privilegi su quel sistema, ad
ottenere i privilegi di amministratore in modo indebito o ad aprire una shell
remota o locale in modo indebito. In ogni caso, quando un attaccante ha
sfruttato un codice vulnerabile iniettando del codice arbitrario al suo interno
ha il controllo totale della macchina, anche se indebito. I bollettini di
sicurezza in giro per il web pullulano di bug del genere trovati ancora oggi in
molte applicazioni, e dovuti proprio all'uso errato di funzioni come quelle che
abbiamo visto sopra, bug che in genere sono corretti il più in fretta possibile
dopo la scoperta per evitare che i danni ai sistemi che usano quelle
applicazioni diventino maggiori. Non vedremo in questa sede, per evitare di
divagare troppo nel discorso, in che modo sfruttare tali vulnerabilità per
acquisire il controllo di un sistema, ma per ora ci basta sapere che usando
certe funzioni la cosa è possibile e sapere in che modo funziona.

In conclusione, le funzioni potenzialmente vulnerabili a buffer overflow e da
usare con cautela sono:

* scanf
* gets
* strcpy
* strcat
* sprintf

Queste funzioni vanno usate solo quando si è sicuri al 100% delle dimensioni del
buffer di destinazione. In alternativa, è più sicuro usare funzioni come

* fgets
* strncpy
* strncat
* snprintf

Soffermiamoci un attimo sulla fgets (ne faremo solo una trattazione sommaria per
le stringhe in questa sede, mentre la studieremo in modo più approfondito nel
capitolo sui file). Abbiamo visto prima che, per la lettura di una stringa da
input, sia l'uso di scanf che di gets è pericoloso. Per leggere stringhe la cosa
migliore è fare ricorso a questa funzione, che prende come primo argomento la
stringa di destinazione, come secondo argomento il numero massimo di caratteri
da leggere da input e come terzo argomento il descrittore da cui leggere (nel
nostro caso lo standard input, identificato da stdin). Esempio di uso:

```c
char str[16];
 
printf ("Inserisci una stringa: ");
fgets (str,sizeof(str),stdin);
str[strlen(str)-1]=0;
 
printf ("Stringa inserita: %s\n",str);
```

La notazione sizeof(str) dice di leggere da input al massimo tanti caratteri
quanti sono quelli supportati dalla dimensione di str (ovvero 16 in questo
caso), mentre stdin, costante definita in stdio.h, identifica lo standard input.
La scrittura `str[strlen(str)-1]=0;` serve perché la funzione fgets salva nella
stringa anche la pressione del carattere invio. Questa scrittura setta il
carattere NULL un byte prima, in modo da rimuovere il carattere invio dalla
stringa.

Attenzione anche ad evitare scritture del genere:

```c
char *str = "Ciao";
printf (str);
```

Se non viene specificato esplicitamente il formato della stringa da stampare
nella printf l'applicazione può potenzialmente essere vulnerabile a _format
string overflow_, una vulnerabilità scoperta abbastanza recentemente che consente
di scrivere dati arbitrari sullo stack. Seguendo questi passi per evitare buffer
overflow e format string overflow si può essere sicuri almeno a un 70% di
scrivere applicazioni relativamente sicure.
