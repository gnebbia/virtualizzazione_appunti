## Chroot

Una delle più basilare tecnologie di virtualizzazione che possono 
essere utilizzate sono i "chroot", questa tecnologia è molto 
primitiva. Molto semplice da applicare:

```sh
 export MY_CHROOT="/directory"
```
```sh
 mount proc $MY_CHROOT/proc -t proc
```
```sh
  echo "proc $MY_CHROOT/proc proc defaults 0 0" >> /etc/fstab 
 # rende le modifiche permanenti
```
```sh
 echo "sysfs $MY_CHROOT/sys sysfs defaults 0 0" >> /etc/fstab 
 # rende le modifiche permanenti
```
```sh
  mount sysfs $MY_CHROOT/sys -t sysfs
```
```sh
 cp /etc/hosts $MY_CHROOT/etc/hosts
```
```sh
 cp /proc/mounts $MY_CHROOT/etc/mtab
```
```sh
 chroot /directory/ /bin/bash
```
ora prima di eseguire un chroot possiamo costruire un semplice 
sistema linux, questo può essere fatto in vari modi, possiamo 
addirittura utilizzare gli strumenti messi a disposizione da 
varie distribuzioni per creare lo scheletro di una distro 
minimale, ad esempio per gentoo, posso scaricare uno stage3 dal 
sito ufficiale, per debian posso semplicemente utilizzare 
debootstrap.

Per poter avviare applicazioni grafiche possiamo semplicemente 
utilizzare un comando prima di entrare nel chroot, il comando è:

```sh
 xhost local:localuser
```
Attenzione non possiamo usare systemctl ed in genere systemd per 
manipolare i servizi all'interno del chroot, questo è un problema 
noto di systemd, systemd implementa una sua soluzione di 
virtualizzazione simile ad LXC (ma al momento meno intuitiva e 
semplice), chiamata systemd-nspawn, questo viene considerato una 
specie di chroot pompato.
