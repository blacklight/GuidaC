# Strumenti necessari per la programmazione in C

* Un editor di testo
* Un compilatore
* Un linker

o, in alternativa,

* Un ambiente di sviluppo integrato

Come editor di testo vanno bene anche l'EDIT del DOS o il Notepad su Windows,
oppure, nel caso si desideri un editor più avanzato, si può ricorrere a
EditPlus, Notepad++ o simili. Su sistemi Unix-like come Linux o *BSD le scelte
sono molte: dagli editor storici e inossidabili, Emacs e Vi/Vim, a editor più
friendly per gli utenti alle prime armi (KWrite, KEdit, Kate, Gedit...). Di
compilatori è possibile trovarne molti in rete, anche freeware (il compito del
compilatore è quello di tradurre il vostro programma scritto in C in linguaggio
macchina, creando quindi un file eseguibile). Sui sistemi Unix lo standard è il
gcc, il compilatore C della GNU che trovate pre-installato in molte
installazioni standard. Su Windows potete scaricare un porting gratuito di gcc
per sistemi MS come DJGPP, o Cygwin, un software che consente di utilizzare su
Windows molte applicazioni native per sistemi Unix, fra cui gcc.

In alternativa alla combo editor+compilatore è possibile far ricorso ad un
ambiente di programmazione integrato (o IDE, ossia un programma che ha già
incorporato editor e compilatore); su Windows c'è Visual C++, sviluppato dalla
Microsoft di cui è disponibile una versione express scaricabile gratuitamente.
In alternativa ci sono  Rhide (un IDE basato sul porting di gcc DJGPP ed
eseguibile in modalità MS-DOS, che però fa sentire tutti i suoi anni), Dev-C++
(il cui sviluppo è stato sfortunatamente abbandonato da anni), o Code::Blocks
(un IDE multipiattaforma sviluppato in C++ considerato da molti l'evoluzione del
defunto progetto Dev-C++), tutti basati su porting di gcc. Per ambienti
Unix-like come Linux o *BSD c'è KDevelop (per ambienti KDE), o Anjuta (per
ambienti Gnome o Gtk-oriented), o ancora Code::Blocks (è multipiattaforma,
quindi utilizzabile sia su sistemi Windows che Unix-like).

Sia su Windows che su Unix è inoltre possibile usare Eclipse, un ambiente di
sviluppo scritto in Java da IBM, relativamente oneroso da un punto di vista
computazionale ma estremamente completo e complesso, diventato in pochi anni uno
standard di sviluppo a livello professionale con molti programmatori che hanno
sviluppato un gran numero di estensioni, plugin e supporto per molti linguaggi.
Nonostante Eclipse sia originariamente nato per sviluppare in Java è possibile
utilizzarlo anche per sviluppare in C o C++.

Tutti gli esempi di codice riportati in questa guida sono stati compilati e
testati con gcc su sistemi Unix-like, ma a parte casi particolari (ovviamente
roba come i raw socket, o inotify, o la gestione dei file a basso livello, tutte
cose peculiari dei sistemi Unix-like, ovviamente avranno problemi a girare su un
sistema Microsoft o su un sistema embedded) dovrebbero funzionare su qualsiasi
compilatore aderente all'ANSI C (praticamente qualsiasi compilatore moderno).
