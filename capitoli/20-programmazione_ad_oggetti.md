# Cenni di programmazione a oggetti in C

La programmazione a oggetti è un paradigma proprio di linguaggi quali C++, Java
o Smalltalk, e prevede la manipolazione dell'informazione attraverso entità
astratte (gli oggetti) caratterizzate da attributi e su cui è possibile operare
solo attraverso metodi (funzioni) prestabiliti. È un paradigma molto utile per
modellare entità in un modo molto vicino alla visione umana di quelle entità. Ad
esempio, un mazzo di carte da poker può essere modellato come un oggetto
caratterizzato dagli attributi _numerocarte_ e _carte: (inteso a sua volta come
array di oggetti), e sul quale è possibile richiamare i metodi _mescola_,
_distribuisci_, _estrai\_carta_ e _reimmetti\_carta_.

Questo paradigma è molto semplice da sfruttare in linguaggi nativamente a
oggetti quali C++ o Java che prevedono già tutti i costrutti sintattici per
poterlo gestire (classi, ereditarietà, polimorfismo, ridefinizione degli
operatori...), ma con un po' più di impegno si può usare anche in C. Anzi, in
grandi progetti come le librerie Gtk in C o lo stesso kernel Linux questo è un
paradigma molto comune, in quanto consente di mantenere i sorgenti più ordinati,
meglio "ingegnerizzati", più facili da gestire e più vicini alla logica umana.

Si pensi ad esempio a come gestire l'oggetto "automobile" all'interno di un
sorgente. Un'automobile, nella nostra implementazione volutamente semplificata,
è un'oggetto caratterizzato dai seguenti attributi:

* velocità massima
* velocità attuale
* stato (accesa o spenta)

e su cui si può operare tramite i seguenti metodi:

* crea un nuovo oggetto automobile (costruttore)
* rimuovi dalla memoria un oggetto automobile (distruttore)
* accelera di tot km\/h
* decelera di tot km\/h
* metti in moto
* spegni

Vediamo come modellare quest'oggetto in C. Questo può essere il contenuto del
file "car.h", contenente la dichiarazione dell'oggetto e dei suoi metodi:

```c
/* car.h */

typedef enum  { off, on } car_status;

struct _car  {
    unsigned int max_speed;
    unsigned int cur_speed;
    car_status status;
};

/* Typedef creato per comodità, per non portarsi continuamente
   dietro degli struct _car*
 */
typedef struct _car* car;

car  car_create (unsigned int max_speed);
void car_destroy (car c);
void car_accelerate (car c, unsigned int kmh);
void car_decelerate (car c, unsigned int kmh);
void car_start (car c);
void car_stop (car c);
```

E questa una possibile implementazione dei metodi nel file “car.c”:

```c
/* car.c */

#include “car.h”

car car_create (unsigned int max_speed)  {
    car c = (car) malloc(sizeof(struct _car));
    car->status = off;
    car->cur_speed = 0;
    car->max_speed = max_speed;
}

void car_destroy (car c)  {
    if (c == NULL) return;
    free(c);
}

void car_accelerate (car c, unsigned int kmh)  {
    if (c == NULL) return;
    if (c->status == off) return;
    if (c->cur_speed + kmh > max_speed) return;
    c->cur_speed += kmh;
}

void car_decelerate (car c, unsigned int kmh)  {
    if (c == NULL) return;
    if (c->status == off) return;
    if ((int) (c->cur_speed - kmh) < 0) return;
    c->cur_speed -= kmh;
}

void car_start (car c)  {
    if (c == NULL) return;
    if (c->status == on) return;
    c->status = on;
    c->cur_speed = 0;
}

void car_stop (car c)  {
    if (c == NULL) return;
    if (c->status == off) return;
    c->status = off;
    c->cur_speed = 0;
}
```

E questa una possibile implementazione:

```c
/* main.c */

#include “car.h”

int main()  {
    car c = car_create(180);
    car_start(c);
    car_accelerate(10);
    car_decelerate(5);
    car_accelerate(20);
    car_decelerate(25);
    car_stop(c);
    car_destroy(c);
}
```

Si noti che attraverso questo paradigma si creano entità al di sopra dei tipi di
dato del C, e su cui per quanto sia "sintatticamente" ammissibile effettuare
ogni tipo di operazione, non lo è "semanticamente". Ad esempio, pur essendo la
velocità un parametro dichiarato come unsigned int le uniche operazioni sensate
su quest'attributo sono l'incremento o il decremento di un tot di velocità. Per
il compilatore può aver senso ad esempio prendere la velocità dell'auto e
sostituirgli la parte intera della sua radice quadrata, ma non ha senso da un
punto di vista logico per come è stato modellato l'oggetto (se le uniche azioni
possibili sono accelerare e decelerare, le uniche operazioni logicamente
ammissibili sull'attributo sono incremento e decremento, e non modificando
direttamente il campo della struttura ma usando i rispettivi metodi
_car\_accelerate_ e _car\_decelerate_). In linguaggi come C++ e Java non è
neanche possibile accedere ai dati “privati” di un oggetto dall'esterno. È
possibile eventualmente leggerne il valore o modificarlo attraverso i metodi
dell'oggetto stesso, ma attraverso un meccanismo di _visibilità_ (ovvero cosa
ogni entità esterna può toccare, leggere o modificare dell'oggetto) è possibile
settare in modo molto pulito i permessi. Il C, non essendo un linguaggio
nativamente a oggetti, è un po' più rude sotto questo punto di vista, ma dei
rudimentali meccanismi di visibilità e permessi si possono comunque implementare
usando variabili di flag (settate nei metodi che possono modificare i valori
dell'oggetto e non settate al di fuori) o stratagemmi simili.
