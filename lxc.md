## LXC

LXC sta per Linux Containers, ed è una tecnologia di 
virtualizzazione, utile nel momento in cui dobbiamo virtualizzare 
un sistema operativo che può usare il nostro stesso kernel, 
abbiamo in questo modo significativi vantaggi in termini di 
velocità e prestazioni in genere. Vediamo come configurare ed 
usare LXC.

Una volta installato con ad esempio:

```sh
sudo apt-get install lxc
```
una volta installato è utile controllare se il kernel utilizzato 
sia compatibile con tutto, altrimenti dobbiamo porre rimedio, il 
comando per controllare la configurazione è:

```sh
 lxc-checkconfig 
 # mostra la lista di varie feature e mi dice se 
 # sono abilitate o meno, è meglio assicurarsi che siano abilitate 
 # tutte, altrimenti dobbiamo cambiare la configurazione del 
 # nostro kernel
```
per vedere una lista di immagini disponibili da scaricare 
eseguiamo:

```sh
 # lxc-create -t download -n my-container
```
