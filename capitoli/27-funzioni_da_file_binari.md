# Uso di funzioni da file binari esterni - dlopen, dlsym

Può capitare spesso di voler usare nei propri sorgenti funzioni definite in file
binari esterni, ad esempio altri file eseguibili o file di libreria di cui non
sono disponibili file header corrispondenti. Lo standard POSIX, quello che
definisce il C standard per sistemi Unix-like, fornisce in `dlfcn.h` un insieme
di funzioni che consentono, dato il nome dell'eseguibile e della funzione da
richiamare da lì, di ottenere un puntatore a quella funzione, che può essere
usato tranquillamente all'interno del proprio sorgente come se fossimo
all'interno di quel file binario. Vediamo subito un esempio. Immaginiamo che
l'eseguibile _prime.exe_ nella stessa directory del nostro sorgente abbia al suo
interno una funzione chiamata `is_prime` che dato un intero calcola se questo è
primo in un tempo estremamente basso. Vogliamo usare quella funzione nel nostro
sorgente. Una strada sarebbe quella di piazzare all'inizio del nostro sorgente
un prototipo di quella funzione e compilare il nostro sorgente insieme a
quell'eseguibile, ma non è una via molto pulita né molto intelligente (non
sempre possiamo sapere a priori come è fatto il prototipo della funzione, e se
il file binario in questione è da 100 MB non è molto saggio compilarlo insieme
al nostro sorgente, ottenendo magari un eseguibile da qualche centinaio di MB
solo per poter usare una funzione che controlla se un numero è primo).
Attraverso le funzioni `dlopen` e `dlsym` di `dlfcn.h` possiamo rispettivamente
aprire il file binario _prime.exe_, ottenere un puntatore che punta alla
funzione `get_prime` al suo interno, e a questo punto richiamare la funzione
usando il puntatore a funzione appena creato. Codice:

```c
#include <stdio.h>
#include <dlfcn.h>

int main()  {
    void *dl;
    double (*is_prime)(double) = NULL;

    if (!(dl = dlopen("./get_prime.exe", RTLD_LAZY)))  {
        fprintf (stderr, "dlopen error: %s\n", dlerror());
        return 1;
    }

    dlerror();

    if (!( *(void**) &get_prime = dlsym(dl, "is_prime")))
    {
        fprintf (stderr, "dlsym error: %s\n", dlerror());
        return 2;
    }

    printf (“31 is %s\n”, (*is_prime)(31) ?  “prime” : “not prime”);

    dlclose(dl);
    return 0;
}
```

Ricordare di compilare tutti i sorgenti che fanno uso di `dlfcn.h` passando a
gcc l'opzione `-ldl`.
