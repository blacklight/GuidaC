# Monitorare modifiche ai file tramite inotify

Il kernel Linux mette a disposizione un mezzo estremamente potente per
monitorare le modifiche di qualsiasi tipo a file e directory: l'oggetto
_inotify_.  Le potenzialità e la comodità sono non indifferenti: la possibilità
è quella, ad esempio, di controllare se un log di sistema viene aggiornato con
dei nuovi eventi, e inviare ad esempio questi messaggi di log direttamente
all'email dell'amministratore. Oppure controllare se i file in una directory
riservata vengono modificati, e gestire l'evento come si vuole. L'header da
includere è

```c
#include <linux/inotify.h>
```

quindi la procedura è

1. Inizializzare inotify (`inotify_init()`);
2. Aggiungere un file o una directory su cui effettuare il watch
   (`inotify_add_watch()`);
3. Controllare in un ciclo se vengono effettuati cambiamenti sul file o la
   directory in questione;
4. Gestire i cambiamenti come si vuole;
5. Rimuovere il `watch_point` (`inotify_rm_watch()`).

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <time.h>
#include <linux/inotify.h>
 
#ifndef BUFSIZ
        #define BUFSIZ  8139
#endif
         
int main (int argc, char **argv)  {
    char buff[BUFSIZ];
    int fd, // Descrittore del file
        ifd,        // Descrittore dell'istanza di inotify
        wd, // Descrittore del watch
        n;  // Numero di byte letti
         
    time_t ltime;
    char *strtime;
     
    // Se non c'è nessun argomento passato, esco
    if (!argv[1])
        return 1;
     
    // Apro il file passato come primo argomento
    if ((fd=open(argv[1], O_RDONLY))<0)
        return 2;
     
    // Inizializzo inotify. inotify_init() ritornerà un nuovo descrittore, che
    // userò per gestire i watch point
    if ((ifd=inotify_init())<0)
        return 3;
     
    // Aggiungo un watch point che monitora le modifiche al file specificato
    // via argv[1]
    wd = inotify_add_watch (ifd,argv[1],IN_MODIFY);
     
    // Mi posiziono alla fine di argv[1]
    lseek (fd,0,SEEK_END);
     
    // Ciclo infinito
    while(1) {
        // Leggo dal descrittore di inotify. La read è bloccante, quindi procede
        // solo quando ci sono byte da ricevere da ifd, ovvero solo quando viene
        // effettuata una modifica su argv[1]
        read (ifd,buff,BUFSIZ);
             
        // Leggo l'ora e la data attuale
        ltime = time((unsigned) NULL);
        strtime = strdup(ctime(&ltime));
        strtime[strlen(strtime)-1]=0;
             
        // Controllo quanti byte sono stati aggiunti al file e
        while ((n=read(fd,buff,BUFSIZ))>0)
            printf ("[%s] %s modified: %s\n",strtime, argv[1], buff);
    }
     
    // Rimuovo il watch point (non arriverà mai qui)
    inotify_rm_watch (ifd,wd);
}
```

Quello che faccio è monitorare le modifiche effettuate al file passato via argv.
Quindi apro il file in questione in modalità lettura, posizionandomi alla sua
fine, e dico al mio oggetto inotify di monitorare le modifiche che vengono
effettuate a quel file. `IN_MODIFY` è solo uno dei possibili eventi che posso
monitorare usando inotify. L'elenco completo è il seguente:

```
IN_ACCESS         File was accessed (read) (*).
IN_ATTRIB         Metadata changed (permissions, timestamps,  extended
                  attributes,  etc.) (*).
IN_CLOSE_WRITE    File opened for writing was closed (*).
IN_CLOSE_NOWRITE  File not opened for writing was closed (*).
IN_CREATE         File/directory created in watched directory (*).
IN_DELETE         File/directory deleted from watched directory (*).
IN_DELETE_SELF    Watched file/directory was itself deleted.
IN_MODIFY         File was modified (*).
IN_MOVE_SELF      Watched file/directory was itself moved.
IN_MOVED_FROM     File moved out of watched directory (*).
IN_MOVED_TO       File moved into watched directory (*).
IN_OPEN           File was opened (*).
```

Ora mi ritrovo in mano con un nuovo descrittore, associato all'istanza di
inotify, e in un ciclo infinito leggo da quest'ultimo. Poiché la read ha effetto
bloccante, potrò andare avanti solo quando ci sono byte da leggere da quel
descrittore, ovvero quando è stata materialmente compiuta qualche modifica sul
file specificato. In questo caso, leggo la data e l'ora della modifica, leggo i
byte aggiunti e stampo le modifiche su stdout, o su un file di log, o dove
voglio.

Questa riga

```c
read (ifd,buff,BUFSIZ);
```

salva il risultato della read sul descrittore di inotify in un buffer in
memoria. La read su un descrittore di inotify produce in realtà un'istanza della
struttura `inotify_event`, definita in questo modo:

```c
struct inotify_event {
    int      wd;       /* Watch descriptor */
    uint32_t mask;     /* Mask of events */
    uint32_t cookie;   /* Unique cookie associating related
                          events (for rename(2)) */
    uint32_t len;      /* Size of 'name' field */
    char     name[];   /* Optional null-terminated name */
};
```

dalla quale posso ad esempio vedere il descrittore del mio watch point che è
stato invocato, la maschera che ho usato per filtrare gli eventi, il cookie
associato all'evento e, se esiste, la stringa `name`, che nel caso di una
directory `watched`, contiene il nome del file che è stato modificato, con la
relativa lunghezza della stringa salvata in `len`.
