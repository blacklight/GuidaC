# Interfacciamento tra C e MySQL

È possibile interfacciarsi con un database MySQL tramite alcune funzioni C,
messe a disposizione dagli sviluppatori dello stesso DBMS.

## Applicazione pratica

Per vedere come è possibile interfacciarsi con un database MySQL tramite il C,
prendiamo subito in esame un esempio pratico. Abbiamo un database MySQL chiamato
"esami", che gestisce gli esami tenuti in una certa facoltà. Il database
contiene queste tabelle:

```
---CORSO---
+----------+----------+------+-----+---------+----------------+
| Field    | Type     | Null | Key | Default | Extra          |
+----------+----------+------+-----+---------+----------------+
| codcorso | int(11)  | NO   | PRI | NULL    | auto_increment |
| nomeC    | char(40) | YES  |     | NULL    |                |
| coddoc   | int(11)  | NO   | MUL |         |                |
+----------+----------+------+-----+---------+----------------+
 
---DOCENTE---
+----------+----------+------+-----+---------+----------------+
| Field    | Type     | Null | Key | Default | Extra          |
+----------+----------+------+-----+---------+----------------+
| coddoc   | int(11)  | NO   | PRI | NULL    | auto_increment |
| nomeD    | char(20) | YES  |     | NULL    |                |
| cognomeD | char(20) | YES  |     | NULL    |                |
+----------+----------+------+-----+---------+----------------+
 
---ESAME---
+----------+---------+------+-----+---------+-------+
| Field    | Type    | Null | Key | Default | Extra |
+----------+---------+------+-----+---------+-------+
| coddoc   | int(11) | NO   | PRI | 0       |       |
| codcorso | int(11) | NO   | PRI | 0       |       |
| matr     | int(11) | NO   | PRI | 0       |       |
| voto     | int(11) | YES  |     | NULL    |       |
+----------+---------+------+-----+---------+-------+
 
---STUDENTE---
+------------+----------+------+-----+---------+----------------+
| Field      | Type     | Null | Key | Default | Extra          |
+------------+----------+------+-----+---------+----------------+
| matr       | int(11)  | NO   | PRI | NULL    | auto_increment |
| nomeS      | char(20) | YES  |     | NULL    |                |
| cognomeS   | char(20) | YES  |     | NULL    |                |
| anno_corso | int(1)   | YES  |     | NULL    |                |
+------------+----------+------+-----+---------+----------------+
```

La nostra applicazione dovrà interfacciarsi con questo database in modo da poter
prelevare informazioni al suo interno. Innanzitutto, prima di effettuare
qualsiasi operazione sul database, bisogna inizializzare il descrittore del
database, usato all'interno dell'applicazione, tramite la funzione
`mysql_init()` (definita, come tutte le funzioni che operano su database MySQL,
in `mysql/mysql.h`), che ha questa sintassi:

```c
MYSQL *mysql_init(MYSQL *mysql)
```

dove MYSQL è un tipo di dato primitivo usato dalla libreria MySQL e, in questo
caso, rappresenta il descrittore del nostro database. La funzione ritorna NULL
quando non è possibile creare il descrittore. Esempio di utilizzo:

```c
#include <mysql/mysql.h>
 
.......
 
MYSQL db;
 
if (!mysql_init(&db))  {
    printf ("Errore nell'inizializzazione del database\n");
    exit(1);
}
```

A questo punto bisogna collegarsi fisicamente al database sull'host in
questione, usando la funzione `mysql_real_connect`, che ha la seguente sintassi:

```c
MYSQL *mysql_real_connect(MYSQL *mysql,
                          const char *host,
                          const char *user,
                          const char *passwd,
                          const char *db,
                          unsigned int port,
                          const char *unix_socket,
                          unsigned long client_flag)
```

dove `*mysql` rappresenta l'indirizzo del descrittore del database che abbiamo
inizializzato prima, `host` l'IP o il nome dell'host sul quale è ospitato il
database, `user` l'username con cui accedere al database e `passwd` la sua
password corrispondente, `db` l'eventuale database a cui collegarsi (se sulla
macchina esistono più istanze di MySQL, altrimenti si può tranquillamente
lasciare a NULL), `port` l'eventuale porta a cui collegarsi (se il database è in
ascolto su una porta diversa da quella di default, altrimenti si può
tranquillamente lasciare a 0), `unix_socket` l'indirizzo dell'eventuale socket
da utilizzare per la connessione (in genere si può lasciare a NULL),
`client_flag` un intero che rappresenta eventuali informazioni aggiuntive da
passare al db (in genere si lascia a 0, per esigenze particolari sul sito
developer di MySQL c'è una voce dedicata ai possibili valori che può assumere
questo flag, nel caso di esigenze particolari). Nel nostro caso di esempio, ci
collegheremo al server MySQL presente sul nostro host locale (localhost),
sfruttando il descrittore db creato prima, lo username 'root' e la password
'prova':

```c
char *db_host="localhost";
char *db_user="root";
char *db_pass="prova";
 
if (!mysql_real_connect(&db, db_host, db_user, db_pass, NULL, 0, NULL, 0))
    printf ("Errore di connessione al database su %s\n",db_host);
else
    printf ("Connessione avvenuta con successo al database su
            %s\n",db_host);
```

A questo punto selezioniamo il database da utilizzare sull'host a cui ci siamo
collegati. Il nostro database era quello dedicato agli esami della facoltà,
chiamato "esami". Per selezionare un database da un descrittore già aperto
usiamo la funzione `mysql_select_db`:

```c
if (mysql_select_db(&db,db_name))
    printf ("Errore di connessione al database %s\n",db_name);
else
    printf ("Connessione avvenuta con successo al database
            %s\n",db_name);
```

Ora la connessione al database è avvenuta con successo, e il database è pronto
ad accettare le nostre richieste. Per fare una query SQL al database si usa la
funzione `mysql_real_query`, a cui bisogna passare i seguenti parametri:

* Indirizzo del descrittore del db;
* Query SQL, sotto forma di stringa;
* Lunghezza della query.

Esempio: vogliamo interrogare il database in modo da ottenere il numero di
matricola, il nome e il cognome di tutti gli studenti che hanno sostenuto almeno
un esame, con il nome dell'esame superato e il voto corrispondente. Si tratta di
fare un join tra 3 tabelle del db: studente (dal quale prelevo il nome, il
cognome e la matricola degli studenti), corso (dal quale prelevo il nome del
corso) e esame (dal quale prelevo il voto). Ovviamente la condizione di join è
che il codice del corso di esame sia uguale a quello di corso, e la matricola
dello studente sia uguale a quella di esame. Ordinando i risultati in modo
crescente secondo il codice del corso, la query diventa così:

```c
char *query =
    "select s.matr,s.nomeS,s.cognomeS,c.nomeC,e.voto "
    "from studente s,corso c,esame e "
    "where e.codcorso=c.codcorso "
    "and e.matr=s.matr "
    "order by c.codcorso";
```

Il codice per eseguirla diventa così:

```c
if ( mysql_real_query (&db, query, (unsigned int) strlen(query)) )  {
    printf ("Errore nell'esecuzione della query %s\n",query);
    exit(2);
}
```

Ora è possibile salvare il risultato della query in una variabile apposita (di
tipo predefinito `MYSQL_RES`), tramite la funzione `mysql_store_result`, quindi
contare il numero di campi letti attraverso `mysql_num_fields` e salvare il nome
di ogni campo (es. nomeS, cognomeS, voto...) in una variabile apposita (di tipo
`MYSQL_FIELD`) attraverso la funzione `mysql_fetch_fields`. Ecco quindi come
ottenere i nomi di tutti i campi letti dalla query all'interno del database e
stamparli su schermo uno per uno (la formattazione del testo non sarà ottimale,
ma è solo per capire come funziona il procedimento):

```c
MYSQL_RES *res;
MYSQL_FIELD *f;
int i;
 
.........
 
res = mysql_store_result(&db);
f = mysql_fetch_fields(res);
 
for (i=0; i<mysql_num_fields(res); i++)
    printf ("%s\t",f[i].name);
```

Ora stampiamo i contenuti effettivi di ogni riga della query. Per fare ciò,
usiamo un altro tipo di dato primitivo di MySQL (`MYSQL_ROW`) e usiamo la
funzione `mysql_fetch_row`. Per leggere tutte le righe date in output dalla
query il codice diventa quindi qualcosa del genere:

```c
MYSQL_ROW row;
 
.........
 
// Finché ci sono righe da leggere...
while ((row=mysql_fetch_row(res)))  {
    // ...per ogni riga letta...
    for (i=0; i<n; i++)
        //...stampane il contenuto i-esimo
        printf ("[%s]\t", row[i]);
    printf ("\n");
}
```

A questo punto il nostro interfacciamento con il database è completo, e
ripuliamo sia il risultato della query sia l'identificatore della connessione
con il database, attraverso le funzioni `mysql_free_result` e `mysql_close`:

```c
mysql_free_result (res);
mysql_close(&db);
```

Ecco il codice completo dell'esempio:

```c
#include <stdio.h>
#include <mysql/mysql.h>
 
main()  {
    char *db_host="localhost";
    char *db_user="root";
    char *db_pass="prova";
    char *db_name="esami";
    char *query =
        "select s.matr,s.nomeS,s.cognomeS,c.nomeC,e.voto "
        "from studente s,corso c,esame e "
        "where e.codcorso=c.codcorso "
        "and e.matr=s.matr "
        "order by c.codcorso";

    int i;
    unsigned int n;
     
    MYSQL db;
    MYSQL_RES *res;
    MYSQL_ROW row;
    MYSQL_FIELD *f;

    if (mysql_init(&db)==NULL)  {
        printf ("Errore nell'inizializzazione del database\n");
        exit(1);
    }
     
    if (!mysql_real_connect(&db, db_host, db_user, db_pass, NULL, 0, NULL, 0))
        printf ("Errore di connessione al database su %s\n",db_host);
    else
        printf ("Connessione avvenuta con successo al database su %s\n",db_host);
     
    if (mysql_select_db(&db,db_name))
        printf ("Errore di connessione al database %s\n",db_name);
    else
        printf ("Connessione avvenuta con successo al database %s\n",db_name);
     
    if ( mysql_real_query (&db, query, (unsigned int) strlen(query))) {
        printf ("Errore nell'esecuzione della query %s\n",query);
        exit(2);
    }
     
    res = mysql_store_result(&db);
    n = mysql_num_fields(res);
    f = mysql_fetch_fields(res);
     
    printf ("\n");
     
    for (i=0; i<n; i++)
        printf ("%s\t",f[i].name);
     
    printf ("\n");

    while ((row=mysql_fetch_row(res))) {
        for (i=0; i<n; i++)
            printf ("[%s]\t", row[i]);
        printf ("\n");
    }
     
    mysql_free_result (res);
    mysql_close(&db);
}
```

Ovviamente, le funzioni contenute in questo codice di esempio si possono
riutilizzare per effettuare delle query su qualsiasi db, e anche per effettuare
operazioni di creazione, inserimento e aggiornamento di record all'interno di un
database. La documentazione completa per le funzioni di interfacciamento tra
MySQL e C è reperibile sul sito degli sviluppatori MySQL.
