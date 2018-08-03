<!--
.. title: Virtualizzazione Appunti
.. slug: vt_technologies_appunti
.. date: 2018-07-06 14:20:00 UTC+02:00
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->




## Introduzione 

In informatica per "virtualizzazione" si intende l'atto di 
creare/gestire una versione virtuale di qualcosa, come un 
computer con hardware virtuale, sistemi operativi, dispositivi di 
storage, o risorse di rete.

Quando si parla di virtualizzazione hardware, la macchine `host` e' l'attule
macchine su cui ha luogo l'attivita' di virtualizzazione, mentre la macchina
`guest` e' la cosiddetta *virtual machine*.
Il software (o firmware) che crea la macchina virtual sulla macchina host e'
chiamato **hypervisor** o **virtual machine manager**.

Different types of hardware virtualization include:
I diversi tipi di virtualizzazione includono:

* **Full virtualization** -- almost complete simulation of the actual 
  hardware to allow software, which typically consists of a guest 
  operating system, to run unmodified. 
* **Partial virtualization** -- some but not all of the target 
  environment attributes are simulated. As a result, some guest 
  programs may need modifications to run in such virtual 
  environments. 
* **Paravirtualization** -- a hardware environment is not simulated; 
  however, the guest programs are executed in their own isolated 
  domains, as if they are running on a separate system. Guest 
  programs need to be specifically modified to run in this 
  environment. 

E' importante notare la differenza tra emulazione e virtualizzazione:

* Emulazione: Un pezzo di software o hardware (esistono device 
  per l'emulazione) imita un'altro pezzo di hardware (un'altra 
  CPU), quindi ad esempio ho un x86 che imita un PowerPC
* Virtualizzazione: Un pezzo di software (chiamato hypervisor o 
  VMM (virtual machine manager) ) imita un pezzo particolare del 
  computer o l'intero sistema

Per quanto riguarda la virtualizzazione, esistono due tipi di 
hypervisor:

* Type 1 (native or bare-metal hypervisors): Questi hypervisor 
  runnano direttamente sull'hardware del sistema host e non hanno 
  bisogno di software( o sistemi operativi supplementari) esempi 
  di questi sono: Citrix XenServer, Microsoft Hyper-V, Oracle VM 
  Server for x86 (or Sparc), VMware ESX/ESXi
* Type 2 (hosted hypervisor): Questi hypervisor runnano su un 
  sistema operativo convenzionale e hanno quindi bisogno di un 
  sistema operativo host su cui runnare, esempi di questi sono: 
  VirtualBox, Qemu, VMware

Comunque la distizione tra questi due tipi non è del tutto 
chiara, ad esempio esistono moduli in GNU/Linux e FreeBSD che 
rendono in modo efficace un sistema operativo host ad un 
hypervisor di tipo 1; però è da considerare che essendo questi 
ultimi (GNU/Linux e FreeBSD) OS general purpose, l'hypervisor 
dovrà comunque condividere risorse col resto dell'OS, quindi 
vengono da molti considerati hypervisors "type 2".

## Chroot

Una delle più basilare tecnologie di virtualizzazione che possono 
essere utilizzate sono i "chroot", questa tecnologia è molto 
primitiva. Molto semplice da applicare:

```sh
 # export MY_CHROOT="/directory"
```
```sh
 # mount proc $MY_CHROOT/proc -t proc
```
```sh
 # echo "proc $MY_CHROOT/proc proc defaults 0 0" >> /etc/fstab 
  
 # rende le modifiche permanenti
```
```sh
 # echo "sysfs $MY_CHROOT/sys sysfs defaults 0 0" >> /etc/fstab 
  
 # rende le modifiche permanenti
```
```sh
 # mount sysfs $MY_CHROOT/sys -t sysfs
```
```sh
 # cp /etc/hosts $MY_CHROOT/etc/hosts
```
```sh
 # cp /proc/mounts $MY_CHROOT/etc/mtab
```
```sh
 # chroot /directory/ /bin/bash
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
 # xhost local:localuser
```
Attenzione non possiamo usare systemctl ed in genere systemd per 
manipolare i servizi all'interno del chroot, questo è un problema 
noto di systemd, systemd implementa una sua soluzione di 
virtualizzazione simile ad LXC (ma al momento meno intuitiva e 
semplice), chiamata systemd-nspawn, questo viene considerato una 
specie di chroot pompato.

## Qemu

Qemu è sia un emulatore che un sistema di virtualizzazione con 
hypervisor "type 2".

### Installazione


prima di installarlo è nostro compito verificare se il sistema 
host possiede una CPU che supporta estensioni di virtualizzazione 
(Intel VT o AMD-V), per verificare la presenza di queste 
estensioni ci basta eseguire:

```sh
 egrep '(vmx|svm)' /proc/cpuinfo 
 # se l'output non è vuoto, 
 # allora la CPU possiede estensioni di virtualizzazione
```
possedere estensioni di virtualizzazione ci permette di avere una 
virtualizzazione più efficiente e veloce attraverso il modulo 
KVM, per installarlo con KVM eseguiamo:

```sh
 apt-get install kvm qemu-kvm libvirt-bin 
 # per installarlo con 
 # KVM
```
```sh
 # yum install qemu-kvm
```
```sh
 # yast -i kvm
```
mentre nel caso in cui avessimo una CPU senza estensioni di 
virtualizzazione allora installeremo

```sh
 # apt-get install qemu qemu-kvm libvirt-bin
```
```sh
 # yum install qemu
```
```sh
 # yast -i qemu
```
N.B.: QEMU can make use of KVM when running a target architecture 
that is the same as the host architecture. For instance, when 
running qemu-system-x86 on an x86 compatible processor, you can 
take advantage of the KVM acceleration - giving you benefit for 
your host and your guest system. 

### Dischi Immagine supportati da Qemu


Una volta installato, QEMU è pronto per runnare un OS guest 
(ospite) caricato da un immagine disco. Questo tipo di immagine 
rappresenta dati su un hard disk, possiamo pensare a questo come 
un hard disk virtuale. Per mettere in running un immagine disco, 
eseguiamo:

```sh
 qemu myImageDisc.img 
 # mette in running l'immagine di un OS
```
possiamo fare in modo che Qemu prenda il controllo del mouse, 
clickando nella finestra relativa, mentre per far rilasciare il 
mouse schiacciamo "Ctrl+Alt".

Qemu supporta diversi tipi di immagine, ma la sua immagine nativa 
e più flessibile è la "qcow2" che supporta:

```sh
 # il "copy on write"
```
```sh
 # encryption
```
```sh
 # compressione
```
```sh
 # snapshot di macchine virtuali
```
anche se Qemu supporta correntemente questi formati di immagine 
disco:

```sh
 # raw: questo è un formato binario semplice di un'immagine disco 
 # ed è molto portable
```
```sh
 # cloop: Compressed Loop format, usata principalmente per 
 # particolari immagini live come Knoppix e altri cd live
```
```sh
 # cow: immagine copy on write, supportata per ragioni storiche
```
```sh
 # qcow: immagina nativa di qemu, versione precedente, supportata 
 # per questioni di compatibilità
```
```sh
 # qcow2: immagine nativa di qemu
```
```sh
 # vmdk: immagine di VMware
```
```sh
 # vdi: immagine di Virtualbox
```
### Creazione di un'immagine disco (ovvero come fare il set up 

  di un sistema guest)

Per creare il nostro OS ospite (guest) dobbiamo prima creare 
un'immagine disco vuota. Qemu utilizza il comando "qemu-img" per 
creare e manipolare immagini disco, il formato di default con cui 
crea le immagini è quello "Raw", vediamo come fare:

```sh
 qemu-img create -f qcow2 miaImg.img 10G 
 # creiamo un'immagine 
 # di tipo qcow2, di nome "miaImg.img" e di dimensione massima 
 # pari a 10GB 
```
```sh
 qemu-img resize miaImg.img +10G 
 # aumenta di 10GB la dimensione 
 # dell'immagine menzionata, non è ancora possibile per le 
 # immagini qcow2 rimpicciolire le immagini
```
una volta creata l'immagine possiamo eseguire il boot di una ISO 
di un OS con il comando "qemu", che su alcuni sistemi può essere "
kvm":

```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img -cdrom 
  nomeIso.iso -boot d 
 # in questo caso stiamo inizializzando un 
 # sistema con 256MB di RAM, e utilizzando la iso menzionata come 
 # immagine montata al boot, ricorda che se qemu non viene 
 # trovato, dobbiamo usare kvm, solitamente "kvm" è 
 # un'abbreviazione di "qemu-system-[myArch] -enable-kvm", 
 # attenzione se l'architettura emulata ha la stessa architettura
```
```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img -cdrom 
  /dev/cdrom -boot d 
 # in questo caso viene preso proprio il 
 # contenuto del lettore cd/dvd del sistema host come boot per 
 # l'immagine, al posto di "/dev/cdrom" potremmo avere "/dev/sr0" 
 # o "/dev/dvd" a differenza della configurazione HW/SW del 
 # sistema host
```
N.B.: Si può passare in modalità full-screen con la combinazione "
Ctrl+Alt+f".

Vediamo altre opzioni di boot più complessa:

```sh
 # qemu-system-x86_64 -enable-kvm -cpu host -smp 2 -hda miaImg.img 
 # -cdrom W7ALLINONE.iso -boot d -m 1024 -k it -netdev 
 # user,id=user.0 -device e1000,netdev=user.0 -usb -usbdevice 
  tablet -vga qxl -display sdl -monitor stdio 
 # con smp 
 # specifichiamo il numero di processori, con -cpu host 
 # specifichiamo la cpu del sistema ospitante
```
```sh
 # qemu-system-x86_64 -enable-kvm -cpu host -smp 2 -hda miaImg.img 
 # -cdrom debian-8.2.0-amd64-netinst.iso -boot d -m 1024 -usb -vga 
  qxl 
 # per vga esistono diverse opzioni, se dovessimo avere 
 # problemi col video possiamo provare le altre, possiamo 
 # analizzarle attraverso "man qemu-system-x86_64" e poi cercando "
 # -vga"
```
```sh
 # qemu-system-ppc -hda miaImg.img -cdrom 
 # debian-8.2.0-powerpc-netinst.iso -boot d -m 1024 -usb 
  
 # emuliamo un powerpc in questo caso
```
### Quali CPU ho a disposizione ?


Per vedere quali CPU possiamo emulare da un terminale premendo:

```sh
 qemu-+TAB 
 # tabbando vedremo le varie architetture disponibili
```
una volta selezionata l'architettura generale possiamo anche 
utilizzare un'architettura specifica, utilizzando il flag "-cpu ?"
, ad esempio per vedere quali modelli specifici abbiamo per la i 
processori dell'architettura MIPS, eseguiamo:

```sh
 qemu-system-mips -cpu ? 
 # in questo caso possiamo vedere i vari 
 # modelli di CPU disponibili per l'architettura MIPS, una volta 
 # visualizzato il modello interessato possiamo selezionarlo con:
```
```sh
 # qemu-system-mips -cpu 4Km -hda miaImg.img -cdrom 
 # debian_8.0_mips.iso -boot d -m 1024
```
altre volte per alcune CPU tipo arm, dobbiamo specificare prima 
il tipo di macchina attraverso il comando "-machine" quindi 
eseguiremo:

```sh
 qemu-system-arm -machine ? 
 # visualizza le macchine disponibili
```
poi sarà possibile utilizzare -cpu ?, quindi eseguiremo:

```sh
 qemu-system-arm -machine NameOfTheMachine -cpu ? 
 # visualizza 
 # le cpu disponibili per l'architettura e la macchina menzionata
```
### Opzioni di Boot


Per il boot esistono diverse opzioni:

```sh
 -boot c 
 # fa il boot dal primo virtual hard drive
```
```sh
 -boot d 
 # da il boot dal primo CDROM drive virtuale
```
```sh
 -boot n 
 # fa il boot dalla virtual network
```
ATTENZIONE "UEFI": Per poter effettuare il boot di un sistema 
uefi doabbiamo installare il pacchetto "ovmf". E possiamo 
avviarlo selezionando come boot:

```sh
 # qemu-kvm -bios ./usr/share/qemu-ovmf/bios/bios.bin -m 1G -cdrom 
 # boot.iso
```
### Boot di un'immagine con OS già installato


una volta installato il sistema da ISO o da CD/DVD possiamo 
avviare il nostro sistema guest con:

```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img 
  -kernel-kqemu 
 # avvia il sistema operativo installato con qemu, 
 # NOTA BENE, quest'istruzione prevede una CPU con 32 bit
```
```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -name debianProva -hda 
  miaImg.img 
 # avvia il sistema operativo, e assegna un nome alla 
 # macchina virtuale che è "debianProva"
```
```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img 
  -kernel-kqemu 
 # avvia il sistema operativo installato con qemu, 
 # quest'istruzione è per i sistemi a 64 bit, in realtà su alcuni 
 # sistemi non c'è bisogno di usare l'opzione -kernel-kqemu
```
```sh
 # qemu-system-x86_64 -enable-kvm -m 256 -smp 2 -hda miaImg.img 
  
 # l'opzione "smp" permette di specificare il numero di 
 # processori da utilizzare
```
Qemu può utilizzare fino a 4 immagini contemporaneamente, in modo 
che 4 dischi virtuali vengono presentati contemporaneamente allo 
stesso OS guest, questo è molto utile ad esempio nei seguenti 
esempi:

```sh
 # un immagine disco pagefile o file di swap virtuale che può 
 # essere condiviso tra più macchine virtuali
```
```sh
 # un disco dati comune a più OS guest che può essere condiviso 
 # tra questi ultimi
```
```sh
 # dare spazio addizionale ad un OS guest senza riconfigurare o 
 # compromettere l'immagine principale
```
```sh
 # separare operazioni di I/O su un dispositivo di memoria, 
 # andando a salvare immagini diverse di Qemu su dispositivi di 
 # memoria diversi del sistema host
```
```sh
 # emulazione di un ambiente fisico con più dispositivi di memoria 
 # per ragioni di testing/learning
```
E' da ricordare che però solo un'istanza di QEMU può accedere ad 
un'immagine alla volta. Per usare più immagini con un sistema 
operativo guest eseguiamo:

```sh
 # qemu -m 256 -hda miaImg.img -hdb miaImg2.img -hdc miaImg3.img 
 # -hdd tempFiles.img -kernel-kqemu
```
NB: QEMU doesn't support both -hdc and -cdrom at the same time, 
as they both represent the first device on the second IDE channe

Per creare e lanciare una nuova immagine basata su un altro file 
immagine eseguiamo:

```sh
 # qemu-img create -f qcow2 -o backing_file=miaImg.img test01.img 
  
 # crea un nuovo file immagine che è un clone di un'altra 
 # immagine
```
```sh
 qemu -m 256 -hda test01.img -kernel-kqemu & 
 # lancia la nuova 
 # macchina virtuale
```
### Montare immagini sul sistema Host


a volta può essere utile montare immagini disco sul sistema host. 
Ad esempio se il sistema guest non ha un support network, l'unico 
modo per trasferire file da host a guest e viceversa sarà 
montando l'immagine sul sistema host. I sistemi Linux e UNIX 
possono montare immagini create nel formato "raw" usando un 
dispositivo di loopback. Da un utente root possiamo montare 
un'immagine raw con:

```sh
 # mount -o loop,offset=32356 /percorso/immagine.img 
  /mnt/mountpoint 
 # monta l'immagine "RAW" su un mountpoint 
 # specifico, attenzione l'offset dipende dalla partizione 
 # specifica che vogliamo montare dell'immagine.img, se abbiamo 
 # solo una partizione allora possiamo omettere l'opzione offset, 
 # inoltre questo comando può montare solo immagini RAW
```
possiamo determinare l'offset corretto con 

```sh
 # fdisk -l /percorso/immagine.img 
 
 #  in questo caso quello che 
 
 #  dobbiamo guardare è dove inizia la partizione interessata e la 
 
 #  dimensione di settore "sector size", se ad esempio la 
 
 #  partizione inizia al blocco 128 e il sector size è 512, allora 
 
 #  l'offset da mettere è 512*128=65536, quindi avremmo dovuto 
 
 #  mettere offset=65536
```

ricordiamo che l'offset serve a specificare la partizione 
interessata all'interno della nostra immagine virtuale, infatti 
non abbiamo bisogno di specificare l'offset se la nostra immagine 
ha solo una partizione.

ATTENZIONE: Mai montare un immagine mentre è in utilizzo da Qemu, 
quest'operazione la compromette.

Per montare immagini di tipo diverso dal "RAW", come ad esempio 
le "qcow2" allora dobbiamo usare qemu-nbd, dove "nbd" sta per "
Network Block Device", (questo metodo che verrà descritto in 
realtà funziona anche con le immagini "RAW", solo che siccome il "
mount" è più efficiente, con le RAW preferiamo eseguire quello) 
eseguiamo quindi:

```sh
 # modprobe nbd max_part=16
```
```sh
 # qemu-nbd -c /dev/nbd0 image.qcow2
```
```sh
 # sudo partprobe /dev/nbd0
```
```sh
 # fdisk /dev/nbd0 
 
 #  visualizziamo informazioni, in modo da capire 
 
 #  quale partizione ci può interessare
```

```sh
 # mount /dev/nbd0p1 /mnt/image
```

E' da ricordare che le partizioni gestite con LVM non possono 
essere montate con "mount", ma dobbiamo usare i relativi comandi.

### Copiare un immagine virtuale su un Dispositivo di Memoria reale

It may be desired to copy a diskimage to a physical device. An 
example may be if building a cluster, it might be easier to get 
everything ready in qemu, then write the final diskimage to all 
of the hard drives. Of course your image will need to contain all 
of required configuration and drivers for the new system to boot 
properly.

The diskimage will need to be in raw format, quindi prima dovremo 
convertire l'immagine in immagine RAW

```sh
 # qemu-img convert -O raw diskimage.qcow2 diskimage.raw 
 
 #  converte in formato RAW un'immagine qcow2
```
una volta che abbiamo l'immagine in formato raw possiamo 
eseguire:

```sh
 # dd if=diskimage.raw of=/dev/sdX 
 
 # copia l'immagine RAW su un dispositivo reale
```

un'alternativa più rapida è eseguire:

```sh
 # qemu-img convert -O raw diskimage.qcow2 /dev/sdX 
 
 #  questo converte e scrive direttamente su dispositivo fisico
```

### Informazioni su un'immagine virtuale

Per ottenere informazioni su un'immagine virtuale eseguiamo:

```sh
 # qemu-img info test.vmdk 
 
 #  in questo caso viene fatto il retrieving delle informazioni 
 
 #  sull'immagine virtuale
```

### Convertire Immagini

Per convertire immagini da un formato all'altro seguiamo questo 
formato di istruzione:

```sh
 # qemu-img convert -O formatoImmagineDesiderato nomeImmagineOriginale nomeImmagineConvertita
```
ad esempio:

```sh
 # qemu-img convert -O qcow2 test.vmdk test.qcow2 
 
 #  in questo caso convertiamo l'immagine di tipo vmdk in qcow2
```
```sh
 # qemu-img convert -O vdi test.qcow2 test.vdi 
 
 #  in questo caso un'immagine qcow2 viene convertita in "vdi" che è un formato 
 
 #  leggibile da virtualbox
```

### Console di Qemu


Qemu presenta una console molto utile per effettuare diverse 
operazioni, la console è accessibile tramite i tasti "
Ctrl+Alt+Shift+2" e con "Ctrl+Alt+Shift+1" ritorniamo alla 
macchina virtuale di nuovo, la console ci permette di effettuare 
diverse operazioni, ad esempio possiamo reperire molte 
informazioni attraverso il comando:

```sh
 # info opzione 
 
 #  dove la lista delle opzioni è reperibile attraverso il comando "help info"
```
ad esempio per vedere se il supporto kvm è abilitato possiamo 
eseguire:

```sh
 # info kvm
```
altri comandi utili sono:

```sh
 info snapshots 
 #  mostra informazioni sugli snapshot
```
altre operazioni interessanti sono:

```sh
 # screendump filename 
 
 #  esegue uno screenshot della macchina virtuale
```
```sh
 # sendkey ctrl-alt-f1 
 
 #  invia la sequenza specificata alla macchina virtuale, 
 
 #  altri tasti degni di nota sono :shift, 
 
 #  shift_r, altgr, esc, tab, backspace, ctrl_r, delete, menu
```

```sh
 # system_powerdown 
 
 #  esegue uno shutdown, mandando un segnale 
 
 #  ACPI, quindi il sistema si spegnerà safely
```

```sh
 # balloon value 
 
 #  cambia la quantità di RAM, il valore "value" è il valore
 
 #  in MB di ram da impostare
```
```sh
 quit 
 #  esce dalla Macchina virtuale immediatamente
```

```sh
 system_reset 
 #  analogo ad un tasto di reset su una macchina fisica
```

N.B.: In the virtual consoles, you can use Ctrl-Up, 
Ctrl-Down, Ctrl-PageUp and Ctrl-PageDown to move in the back log.

AGGIUNGERE:

```sh
 # aggiungere device usb
```
### Comandi Tastiera per Qemu (i.e., Qemu Keys)


During the graphical emulation, you can use special key 
combinations to change modes. The default key mappings are shown 
below, but if you use -alt-grab then the modifier is 
Ctrl-Alt-Shift (instead of Ctrl-Alt) and if you use -ctrl-grab 
then the modifier is the right Ctrl key (instead of Ctrl-Alt):

```sh
 Ctrl-Alt-f 
 #  Toggle full screen 
```
```sh
 Ctrl-Alt-+ 
 #  Enlarge the screen
```
```sh
 Ctrl-Alt-- 
 #  Shrink the screen
```
```sh
 Ctrl-Alt-u 
 #  Restore the screen’s un-scaled dimensions 
```
```sh
 # Ctrl-Alt-n 
 
 #  Switch to virtual console ’n’. Standard console 
 # mappings are:
```

  -- 1 Target system display 

  -- 2 Monitor 

  -- 3 Serial port 

```sh
 Ctrl-Alt 
 #  Toggle mouse and keyboard grab 
```

During emulation, if you are using the -nographic option, use :

```sh
 Ctrl-a h 
 #  Get terminal commands:
```
```sh
 Ctrl-a h Ctrl-a ? 
 #  Print help 
```
```sh
 Ctrl-a x 
 #  Exit emulator 
```
```sh
 Ctrl-a s 
 #  Save disk data back to file (if -snapshot) 
```
```sh
 Ctrl-a t 
 #  Toggle console timestamps 
```
```sh
 Ctrl-a b 
 #  Send break (magic sysrq in Linux) 
```
```sh
 Ctrl-a c 
 #  Switch between console and monitor 
```
```sh
 Ctrl-a Ctrl-a 
 #  Send Ctrl-a 
```

### Gestione Snapshot in Qemu

Per creare una copia di un'immagine virtuale che non influisce 
sulla copia originale eseguiamo:

```sh
 # qemu-img create -f qcow2 -b centos-cleaninstall.img snapshot.img 
 
 #  crea un'immagine chiamata snapshot.img che è una 
 
 #  copia dell'immagine chiamata centos-cleaninstall, il vantaggio 
 
 #  rispetto ad una semplice copia è che utilizza una tecnologia 
 
 #  chiamata Redirect-On-Write, quando l'immagine originale verrà 
 
 #  cambiata allora lo snapshot sarà inutilizzabile
```

Da una macchina virtuale in running possiamo aprire il terminale 
di Qemu con "Ctrl+Alt+2" ed eseguire:

```sh
 # savevm nomeSnapshot 
 
 #  salva la macchina virtuale con il nome snapshot
```

```sh
 # info snapshot 
 
 #  visualizza gli snapshot disponibili, ogni snapshot 
 
 #  è identificato da un id numerico ed un nome
```

```sh
 loadvm idSnapshotONomeSnapshot 
 #  Carica lo snapshot menzionato
```
```sh
 delvm idSnapshotONomeSnapshot 
 #  Cancella lo snapshot menzionato
```
```sh
 stop 
 #  Sospende l'esecuzione della macchina virtuale
```
```sh
 cont 
 #  Riprende l'esecuzione di una VM
```

### Boot diretto di kernel


E' possibile effettuare boot diretti di kernel, ad esempio per 
motivi di testing/debugging, vediamo un esempio, per effettuare 
un boot diretto di un kernel linux effettuiamo:

```sh
 # qemu-system-i386 -kernel arch/i386/boot/bzImage -hda root-2.4.20.img -append "root=/dev/hda" 
 
 #  usiamo l'opzione "-kernel" per lanciare il kernel menzionato, 
 
 #  l'opzione "-append" serve a fornire opzioni al lancio del kernel 
 
 #  è analogo ai parametri che passiamo ad un boot manager quando lancia un 
 
 #  kernel, è obbligatorio fornire comunque un hard disk virtuale, 
 
 #  per fare in modo che il kernel parta, in quanto il suo boot 
 
 #  sector è utilizzato per lanciare il kernel linux
```

L'opzione "-initrd" può essere usato per fornire un immagine 
INITRD (o INITRAMFS), ad esempio:

Per eseguire banalmente un kernel custom non necessariamente 
Linux eseguiamo:

```sh
 # qemu-system-i386 -kernel vmlinuz -initrd initrd.img -hda root-2.4.20.img -append "root=/dev/hda" 
 
 #  usiamo l'opzione " -kernel" per lanciare il kernel menzionato, l'opzione "-append" 
 
 #  serve a fornire opzioni al lancio del kernel è analogo ai 
 
 #  parametri che passiamo ad un boot manager quando lancia un 
 
 #  kernel, è obbligatorio fornire comunque un hard disk virtuale, 
 
 #  per fare in modo che il kernel parta, in quanto il suo boot 
 
 #  sector è utilizzato per lanciare il kernel linux
```
```sh
 # qemu-system-i386 -kernel myKernel.bin 
 
 #  viene eseguito il kernel menzionato, ovviamente al posto di "-i386" 
 
 #  dobbiamo mettere l'architettura adatta al kernel
```

### Selezione del firmware (Legacy BIOS o UEFI)

Di default qemu caricherà un'interfaccia firmware di tipo Legacy 
BIOS, possiamo comunque decidere quale interfaccia firmware 
utilizzare attraverso l'opzione "-bios", quest'opzione si rivela 
particolarmente utile nel momento in cui vogliamo emulare sistemi 
UEFI nel caso più comune, o comunque sistemi con interfaccia 
firmware totalmente diversa, pensato per altre piattaforme, il 
firmwareUEFI è emulabile attraverso il pacchetto "ovmf", quindi è 
necessario installare questo pacchetto o scaricare comunque il 
file relativo del firmware da internet, una volta ottenuto questo 
firmware eseguiamo:

```sh
 # qemu-system-x86_64 -bios OVMF.fd -hda disk.img -cdrom 
  GNULinux.iso -boot d 
 # esegue l'immagine di un sistema 
 # operativo avviandolo in modalità UEFI, cioè attraverso un 
 # interfaccia firmare di tipo UEFI
```
### Dispositivi USB in Qemu


Possiamo rendere disponibile un dispositivo USB identificato 
attraverso un "lsusb" sul sistema host, andando a specificare il "
Bus" attraverso "hostbus" e l'"ID" con "hostaddr", facendo:

```sh
 # qemu-system-x86_64 -m 1024 -name debianProva miaImg -usb 
 # -device usb-host,hostbus=2,hostaddr=13
```
Potrebbe essere necessario aggiungere i diritti di scrittura al 
device per poterlo utilizzare, una delle soluzioni è scrivere una 
semplice udev rule, che tenga conto del device.

### Scheda Audio in Qemu


Per montare una scheda audio nel nostro sistema possiamo 
utilizzare l'opzione "-soundhw", per vedere quali schede video 
abbiamo a disposizione possiamo eseguire:

```sh
 qemu-system-x86_64 -soundhw ? 
 # visualizza le schede audio 
 # disponibili
```
per avviare qemu con una scheda audio, ad esempio una Intel HD 
Audio (hda), allora eseguiamo:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2G -hda w7img.img -soundhw 
  hda 
 # avvia l'immagine di un sistema operativo con una scheda 
 # audio Intel HD Audio
```
per avviare qemu con sia scheda audio che un dispositivo usb 
eseguiamo:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2G -hda w7img.img -soundhw 
  hda -usb -device usb-host,hostbus=2,hostaddr=13 
 # esegue qemu 
 # con sia scheda audio che dispositivo usb
```
### Schede Video in Qemu


Per montare una scheda video diversa da quella di default 
(cirrus), possiamo visualizzare la lista delle disponibili 
eseguendo "man qemu-system-86_64" e poi premiamo "/" per cercare 
e inseriamo la stringa "-vga", come possiamo vedere abbiamo 
diverse opzioni, vediamo un esempio di impostazione di scheda 
video:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2G -hda w7img.img -vga qxl 
  
 # in questo caso imponiamo l'utilizzo di una scheda video qxl, 
 # questo tipo di scheda video è utile ad esempio su macchine 
 # virtuali GNU/Linux con alcune distro
```
### Altre Periferiche in Qemu


Per vedere quali device abbiamo a disposizione per una 
determinata architettura possiamo eseguire:

```sh
 qemu-system-x86_64 -device ? 
 # dove al posto di "
 # qemu-system-x86_64" possiamo mettere l'architettura che 
 # preferiamo
```
### Redirection di porte per collegamenti


Vediamo ora come è possibile redirigere porte per avere 
collegamenti tra host e guest machine, ad esempio potremo 
collegarci attraverso ssh, o sftp o attraverso qualsiasi metodo 
noi desideriamo, in pratica è doveroso sapere che qemu crea una 
rete virtuale in cui sono presenti solo macchina host che funge 
anche da server dhcp eccetera con IP di default 10.0.2.2, e 
macchina guest con IP di default 10.0.2.15, a questo punto se 
vogliamo ad esempio effettuare un collegamento tra host e guest, 
dobbiamo redirigere il traffico della macchina host di una porta 
a nostra scelta sulla porta su cui si aspetta la connessione il 
sistema guest. Ad esempio, vogliamo effettuare una connessione 
ssh tra host e guest, e sappiamo che il guest si aspetta una 
connessione su porta 22, allora noi lanceremo qemu redirigendo il 
traffico della porta 5555 (scelta a caso da noi) al sistema guest 
sulla porta 22 con:

```sh
 # qemu.system-x86_64 -enable-kvm -hda miaImg.img -m 2G -smp 2 
  -redir tcp:5555::22 
 # in questo caso redirigiamo tutto il 
 # traffico tcp che avviene in localhost sulla porta 5555 alla 
 # porta 22 dell'host, al posto di "tcp" possiamo inserire "udp" 
 # se è questo il protocollo interessato
```
ad esempio nel caso di una connessione netcat eseguiremo

```sh
 # ncat localhost 5555
```
una volta lanciato il sistema guest, ora possiamo da host 
effettuare:

```sh
 ssh nomeAccountValidoGuest@localhost -p 5555 
 # in questo caso 
 # ci connettiamo alla macchina guest dalla macchina host, per la 
 # macchina guest sarà una normale connessione in ssh alla porta 
 # 22
```
se invece volessimo collegarci dal sistema guest al sistema host 
eseguiamo dal sistema guest:

```sh
 ssh nomeAccountValidoHost@10.0.2.2 -p 22 
 # richiediamo una 
 # connessione alla porta 22
```
possiamo sempre verificare l'indirizzo ip del sistema host, 
eseguendo un comando come:

```sh
 route -n 
 # visualizza l'indirizzo ip del sistema host sotto la 
 # voce "Gateway"
```
### Qemu Redirection


  Qemu Monitor Redirection

Possiamo avere il monitor di qemu sul terminale in cui l'abbiamo 
lanciato attraverso l'opzione "monitor -stdio", in pratica il 
monitor di qemu che prima aprivamo con la combinazione "
Ctrl+Alt+2"

```sh
 # qemu-system-x86_64 -enable-kvm -hda VMs/debian.img -monitor 
  stdio 
 # avvia una macchina virtuale, e apre il monitor di qemu 
 # direttamente nel terminale da cui ho avviato qemu, quindi non 
 # dovrò premere "Ctrl+Alt+2" per accedere al terminale di qemu
```
  Qemu Text Redirection

Possiamo anche redirigere il testo all'interno del nostro 
terminale attraverso il comando:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2048 -smp 2 -hda VMs/lfs.img 
 # -kernel /boot/vmlinuz-3.2.0-4-686-pae -append /boot/initrd.img 
 # "root=/dev/sda2 console=tty0 console=ttyS0 rw" -serial 
 # mon:stdio
```
oppure addirittura non aprire una finestra per qemu, andando a 
redirigere tutto il testo all'interno del terminale, eseguiamo:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2048 -smp 2 -hda VMs/lfs.img 
 # -kernel /boot/vmlinuz-3.2.0-4-686-pae -append /boot/initrd.img 
 # "root=/dev/sda2 console=tty0 console=ttyS0 rw" -serial 
  mon:stdio -nographic 
 # l'opzione "-nographic" fa in modo di non 
 # creare una nuova finestra per qemu e redirige tutto all'interno 
 # del terminale da cui ho lanciato qemu
```
nel caso volessimo redirigere tutto il testo all'interno della 
nostra finestra terminale, ma anzichè dargli in input un kernel, 
dandogli in pasto solo un'immagine da cui parte un sistema 
operativo, dobbiamo in pratica impostare le opzioni di boot 
all'interno del sistema operativo questo è possibile andando a 
modificare il file /etc/default/grub ed andando ad aggiungere la 
stringa:

GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0 rw"

una volta apportata questa modifica possiamo eseguire:

```sh
 # qemu-system-x86_64 -enable-kvm -m 2048 -smp 2 -hda 
  VMs/debian.img -serial mon:stdio -nographic 
 # l'opzione "
 # -nographic" fa in modo di non creare una nuova finestra per 
 # qemu e redirige tutto all'interno del terminale da cui ho 
 # lanciato qemu
```
Questa tecnica mi è tornata molto utile nel kernel development, 
in quanto non ho tutto l'overhead portato da spice, per il copy & 
paste, ovviamente non potrò avviare il server xorg, per 
effettuare copy & paste dall'interfaccia grafica ho infatti 
bisogno di spice.

### Initramfs e Qemu sono amici


E' molto comodo per qemu quando si vuole lanciare un kernel, e un 
disco, utilizzare al posto di un'immagine disco solo un 
initramfs, in modo da avere più flessibilità, l'initramfs avrà 
uno script chiamato "/init" che eseguirà tutte le 
inizializzazioni o in genere le operazioni da effettuare; ad 
esempio per emulare sistemi arm, una volta creato un initramfs 
possiamo eseguire:

```sh
 # QEMU_AUDIO_DRV=none \ qemu-system-arm -m 256M -nographic -M 
 # vexpress-a9 -kernel zImage -append "console=ttyAMA0 
 # rdinit=/bin/sh" -dtb vexpress-v2p-ca9.dtb -initrd 
 # initramfs.cpio.gz
```
4 Docker

Docker is a Container managing software. Containers or in general 
Container based virtualization uses the kernel on the host's OS 
to tun multiple guest istances. Each guest istance is called a 
container, and each container has its own:

```sh
 # root fs
```
```sh
 # processes
```
```sh
 # memory
```
```sh
 # devices
```
```sh
 # network ports
```
An example is for example even if we have to run multiple 
versions of java for different applications, each one using a 
different virtual machine.

The advantages of using Containers vs VMs are:

```sh
 # containers are more lightweight
```
```sh
 # no need to install guest OS
```
```sh
 # less cpu, ram, storage space required
```
```sh
 # more containers per machine than VMs
```
```sh
 # greater protability
```
## Docker Installation


Docker engine is the program that enables containers to be built, 
shipped and run, docker engine uses Linux kernel namespaces and 
control groups, namespaces give us the isolated workspace. Una 
volta installato e avviato il demone, con:

$> systemctl start docker

oppure se non abbiamo il demone possiamo avviare l'applicazione 
con "sudo docker -d &" questa è la modalità demone.

Possiamo provarne il funzionamento andando ad avviare:

$> sudo docker run hello-world

questa istruzione dovrebbe dirci che l'immagine hello world non 
esiste, abbiamo dovuto usare sudo, per poter avviare docker con 
altri utenti dobbiamo aggiungerli al gruppo docker, con:

$> sudo usermod -aG docker giuseppe

now we logout and relogin, in order make the changes take effect.

To look at the docker version we do:

$> docker version

### Images vs Containers


Images:

 * read only template used to create containers
 * built by me or other docker users
 * stored in the docker hub or my local registry
Containers:
 * isolated application platform
 * contains everything needed to run my app
 * based on one or more images

we can browse images on dockerhub, and the one called "library/java" or 
"library/nginx", so in general starting with "library/" are the official ones.

Now it is important to define the difference between an image and 
a live container.

 * An image: is a snapshot which does not change unless we do a 
   commit, in order to instantiate an image we can do a "docker run"
 * A container is a living image, what we do here is persistent, 
   and we can have more containers which were originated from the 
   same image

### Display local images


To display local images we can do:

$> docker images #displays docker images

notice that each docker image has "tags" associated to it, these 
tags represent the actual version, for example for java we could 
have tags like "6-jre, 6-jdk, latest, 6b32, etc..." so tags refer 
to various versions, the default tag is "latest". So images are 
specified by "image:tag".

Containers can be specified using their ID or name, there are:

```sh
 # long ID
```
```sh
 # short ID
```
short ID and name of running containers can be obtained using

$> docker ps

while long ID are obtained by inspecting the container.

To display all the available containers we can do:

$> docker ps -a

### Creating a Container


To create a container we do:

```sh
 sudo docker run [options] [image] [command] [args]
```
so docker will create the image if it doesn't exist and execute 
the eventual command (if specified), so for example we could do:

```sh
 docker run ubuntu:14.04 echo "Hello World"
```
or

```sh
 docker run ubuntu:14.04 ps -aux
```
notice that docker run = docker create + docker start, hence any 
time we do a docker run, a new container is created, in order to 
have persistance among states, we should launch docker with start 
and attack

### Connecting to the Container


To instantiate an image and connect directly to the container 
with a shell we do:

$> docker run -i -t ubuntu:latest /bin/bash 

in this case the "-i" flag tells docker to connect to the STDIN 
on the container, and the flag "-t" flag specifies to get a 
pseudo-terminal. We MUST remember that the docker container is 
alive only when the process is alive and changes are not written 
by default, so we can do whatever we want and changes will not be 
done to our image. Anyway changes will continue to live in the 
container. We can view it with:

$> docker ps -a

and we can connect back to it with:$> docker start <id>

$> docker attach <id>

or more simply:

$> docker start -ia <id>

### Docker Detached Mode


Now we do:

docker run -d centos:7 ping 127.0.0.1 -c 100 in this case the 
docker process detaches itself from the current shell, so the 
operation specified is acting in the background, and only an ID 
is given to us, we can view it with "docker ps", viewing even the 
short ID, now with the short ID we can see what's printing on the 
standard output our process, with:

docker logs 62ba075bee18 in this case we see the ping output, 
since it was the command we gave. 

We can even attach ourself to the log file with:

docker logs -f 62ba075bee18

in this case we see the stdout in real time.

### Practical Example: A Web Application Container


Now we want to run a web application inside a container, we'll 
use the "-P" flag to map contiainer ports to host ports, so we 
do:

docker run -d -P tomcat:latest

now we can do:

docker ps here we see the short ID of our applications and the 
portsgiven to our system, indeed we'll see a string under "PORTS" 
with for example: "0.0.0.0:49153->8080/tcp" this means that the 
port 8080 of the container has been mapped to the port 49153 on 
the host system.

To kill an existing docker process, once we have seen its ID, we 
can do:

docker kill 62ba075bee18

now if we do something, like creating a new file or modifying an 
existing file, we can exit, and once we exit in order to get back 
we should launch docker with start and attach

### Practical Example: Giving Graphics (Xorg) and Sound to an 

  application

Before giving sound and video to a container, we have to enable 
the host machine to accept connections to the Xorg display from 
the containers, we can do this by typing (before starting the 
docker container):

```sh
 xhost local:localuser 
```
```sh
 xhost + 
 # it is an alternative to the previous command, 
 # allowing everyone to connect to the Xorg server
```

it's useful to put this in an initialization configuration file, 
such as "/etc/profile" or things of this kind. 

Once we have done this, the following command let us run 
graphical and/or audio application from inside the container:

```sh
docker run \
-v /tmp/.X11-unix:/tmp/.X11-unix \ # mount the X11 socket 
-e DISPLAY=unix$DISPLAY \ # pass the display called unix0
--device /dev/snd \ # sound
```

let's see an analogous example in which I ran a kali linux distro 
docker image:

```sh
docker run -it --device /dev/snd -v /tmp/.X11-unix:/tmp/.X11-unix 
-e DISPLAY=unix$DISPLAY kalilinux/kali-linux-docker /bin/bash
```

so we just have to remember that each time we have to run a 
graphical application we must provide:

 * necessary directories (-v)
 * necessary devices (--device)
 * necessary environment variables (-e)

notice that on SELinux (e.g., Fedora and friends), systems there 
are additional policies that have to be respected, in order to 
bypass them temporarily we just have to run:

```sh
 su -c "setenforce 0"
```
this is unsafe, to ensure safeness, look at SELinux, and how to 
add/modify policies, probably a command like:

```sh
 chcon -Rt svirt_sandbox_file_t /path/to/volume 
 # check command
```

## Image Layers

Images are comprised of multiple layers, a layer is also just 
another image, but everyimage contains a base layer, layers are 
read only. When we launch a container, docker creates a top 
writable layer for containers, parent images are read only, so 
all changes are made at the writeable layer. To save changes in a 
container as a new image we do:

```sh
docker commit [options] [container ID] [repository:tag]
```

repository name should be based on username/application, we can 
reference the container with the container name instead of ID, 
let's see an example, we can do:

```sh
docker commit 984d25f537c5 johnnyty/myapplycation:1.0
```

if we don't specify the tag, docker uses the default one "latest".
Let's see a scenario, so we do:

```sh
 docker run -it ubuntu:15.05 bash
```
then we do:

```sh
 sudo apt install curl
```

now we can exit the docker image, and see its ID with:

```sh
 docker ps -a 
 # view docker ID of the exited docker image we 
 # want to save the state
```

from here we copy its short ID and do:

```sh
 docker commit 984d25f537c5 johnnyty/myapp:1.0 
 # here the code is the short ID
```

now we can run 

```sh
 docker images 
 # this will let us view the last created image
```
and now we can run our image as we did before but with the new 
name, so with:

```sh
 docker run -it johnnyty/myapp:1.0
```

## Dockerfile

A dockerfile is a configuration file that contains instructions 
for buildinf a Docker image. Basically we have:

 * FROM instructions: who specify the base image to use, such as:
  -- FROM ubuntu:14.04

 * RUN instructions: who specify commands to execute, such as:
  -- RUN apt-get install vim
  -- RUN apt-get install curl

so an example dockerfile can be built as the following:

```docker
#Example of a comment

FROM ubuntu:14.04
RUN apt-get install vim
RUN apt-get install curl
```

the fact is that if we have 10 RUN instructions we do 10 commits, 
to avoid this we can use the "&&" shell operator to aggregate RUN 
instructions together, so we can build dockerfile with:

```docker
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
	curl \
	vim \
	openjdk-7-jdk
```

now to build an image following this dockerfile we do:

```sh
 docker build -t myrepo/myapp:1.0 path/to/folder/containing/theDockerfile
```

let's see another example:

```sh
 docker build -t myrepo2/mywebapp:latest . 
 # the build context 
 # here is the current directory
```

Notice that the dockerfile should be named "Dockerfile", we can 
even choose another name, but in this case we should mention the 
filename with the flag "-f"

In the Dockerfiles we can even specify commands that should be 
executed once the container is executed, this is done through the 
"CMD" directive, let's see an example:

```docker
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
	curl \
	vim
CMD ping -c 10 127.0.0.1
```

these commands anyway can be overridden by specifying a command 
in docker run, as we did in the first examples.

We can even specify the "ENTRYPOINT" directive which executes the 
image as an executable for example:

```docker
ENTRYPOINT ["ping"]
```

once we have put this at the end of our Dockerfile, when we run 
the docker image the commands do not override the ping command 
specified, instead they are taken as argument, this docker image 
acts exactly like an executable.

### Start and Stop Containers or exec other processes


We can list all containers with:

```sh
 docker ps -a
```
now we can start a container in background as nginx with:

```sh
 docker run -d nginx
```
we can stop this container with:

```sh
 docker stop <containerIDviewedWithPs-a>
```
we can resume the container with:

```sh
 docker start <containerIDviewedWithPs-a>
```
To start other processes within the same container (assuming the 
container has already started) we can do:

```sh
 docker exec -i -t <containerID> /bin/bash 
 # this opens another shell on the container
```
In order to create a container we can do:

```sh
 docker create 
 # creates a writeable container from the image 
 # and prepares it for running.
```
```sh
 docker run 
 # creates the container (same as docker create) and 
 # runs it. 
```

### Practical example: Tomcat

We can do:

```sh
 docker run -d tomcat:7 
 # this starts (and downloads if it 
 # doesn't exist) an image of tomcat version 7 in background
```

then

```sh
 docker ps 
 # here we see the short ID of the tomcat image
```
now we can attach to the container with another process by doing:

```sh
 docker exec -it <containerID> /bin/bash 
 # this starts a shell 
 # on the container
```
now here for example we can run:

```sh
 ps -ef 
 # shows the active processes inside the container, here 
 # we'll see all the processes attached to the container
```
from here if we execute:

```sh
 exit
```

we won't close the container since the bash was not the process 
with PID 1, which it was actually the tomcat background initial 
execution.

Now in order to resume the image we can do:

```sh
 docker images
```
```sh
 docker start f357e2faab77 
 # restart it in the background
```
```sh
 docker attach f357e2faab77 
 # reattach the terminal & stdin
```
and we are back to our machine, we can also be faster and 
execute:

```sh
 docker start `docker ps -q -l`
```
```sh
 docker attach `docker ps -q -l`
```

### Delete Containers

To remove a container we first have to stop it and then run:

```sh
 docker rm <nameOfTheContainer>
```
or we can execute:

```sh
 docker rmi myrepo/myapp:1.0
```
```sh
 docker rmi -f myrepo/myapp:1.0 
 # in this case we delete the image in a forced way
```
we can verify the deletion of an image with:

```sh
 docker images
```

### Tagging Images

We can tag images or rename a local image repo with:

```sh
 docker tag imageID repo:tag
```
or:

```sh
 docker tag localRepo:tag anotherRepo:tag
```

for example:

```sh
 docker tag edfc1234je32 trainingteam/testexample:1.0
```

or

```sh
 docker tag johnny/testimage:1.5 trainingteam/testexample
```

### Copying data into a Container


In order to copy data in a container we can do:

```sh
 docker cp /path/to/myfile.txt name_of_container:/dest/path
```
```sh
 docker cp name_of_container:/dest/path  /path/to/myfile.txt
```
### Pushing Image to Remote Repo


We can push an image with:

```sh
 docker push johnnytu/testimage:1.0 
 # this will begin the push once we specify the credentials
```

note that this psh will give us errors if on our account there 
isn't yet any repo called "johnnytu/testimage" so we first have 
to create it if it doesn't exist.

## Volumes


A volume is a designated directory in a container, which is 
designed to persist data, independent of the container's life 
cycle. These volumes:

 * can be shared between containers
 * can be mapped to a host directory
 * persist when a container is deleted
 * volume changes are excluded when updating an image

Volumes are mounted when creating or executing a container, 
volume path must be absolute, let's see some example:

```sh
 docker run -d -P -v /myvolume nginx:1.7 
 # in this case we mount 
 # the dir /myvolume into the filesystem of the specified image
```
or another interesting example is:

```sh
 docker run -it -v /data/src:/test/src nginx:1.7 
 # in this case 
 # we map the /data/src directory from the host into the /test/src 
 # directory in the container
```
We can specify volumes even into Dockerfiles, let's see some 
example:

```docker
VOLUME /myvol
```

or even multiple volumes like:

```docker
VOLUME /www/website1 /www/website2 /myvol
```

or with a JSON notation like:

```docker
VOLUME ["myvol", "myvol2"]
```

Mounting folders from the host is food for testing purposes but 
generally not recommended for production use, indeed it is not 
possible to do the mappings inside the Dockerfiles.

### Mapping of ports and services


We don't always need mapping of ports, once we have runned our 
system it will have its IP address, let's say we have a shell, we 
can inspect the IP with the common command "ifconfig", once we 
have the IP address of the dockered machine, if we launch a 
service (e.g. apache) on the dockered machine we can find it at 
ip address: port, just like a machine which is in LAN. Notice 
that by default on most systems docker will configure the 
dockered machine inside a NATted network inside the host machine.

Ok, now let's see a scenario where we want to mirror/map a 
service on the host machine, like for example running apache on 
the guest dockered machine and running it like it was running on 
the host machine. Let's see an example of manual mapping of 
ports:

```sh
 docker run -d -p 8080:80 nginx:1.7 
 # maps port 80 on the 
 # container to port 8080 on the host
```
instead if we would like to do the automatic mapping we do:

```sh
 docker run -d -P nginx:1.7 
 # this will do the automatic port 
 # mapping, but this only works for ports defined in the "EXPOSE" 
 # instructions inside the Dockerfile
```
indeed to let our image to support automatic port mappings we 
should add the ports to enable for automatic mappings in the 
Dockerfile:

EXPOSE 80 443 in this case we are enabling automatic mapping for 
port 80 and 443 (HTTP and HTTPS).

### Linking Containers


To create a link, we first have to create the source containeir 
and the create the recipient container and then use the "--link", 
let's see an example: (BEST PRACTICE: give the containers 
meaningful names)

We first create the source container using the postgres:

```sh
 docker run -d --name database postgres
```
then we create the recipient container and link it:

```sh
 docker run -d -P --name website --link database:db nginx:1.7 
 # here, after --link we put the name of the source folowed by its alias
```

let's see another example:

```sh
 docker run -d --name dbms postgres:latest
```
```sh
 docker run -it --name website --link dbms:db ubuntu:14.04 bash 
```
now inside our ubuntu container if we cat /etc/hosts, we can see 
clearly an entry for our alias name called "db" with its own IP 
address. 

We could check this ip address even from outside the container 
with:

```sh
 docker inspect dbms | grep IPAddress
```
as we can see the two IPs will match.

### We can automate Build Repos!!


On dockerhub there is the possibility to automate the building of 
the repo, let's assume we have a java source called 
JavaHelloWorld.java, we then create a Dockerfile like this:

```docker
FROM java:7
COPY JavaHelloWorld.java .
RUN javac JavaHelloWorld.java

CMD["java", "JavaHelloWorld"]
```

now if we have configured our Dockerhub repo correctly when we 
commit and push from git to github or the configured git repo on 
Dockerhub Dockerhub will notice the commit and create a new 
Docker image. We can even download the image in a second moment 
with:

```sh
 docker pull myrepo/myjavapp
```
and then execute with:

```sh
 docker run myrepo/myjavapp
```
this will run my java application; if we change the code and 
commit+push with git a new image will automatically be created. 
This process is called "CI" (Continous Integration).

  Container Logging

We can show whatever PID 1 writes to stdout

```sh
 docker logs <containerName>
```
or we can view and follow the output with:

```sh
 docker logs -f <containerName>
```
for example we can start a container with tomcat with:

```sh
 docker run -d tomcat
```
then with

```sh
 docker ps 
```

we view the currently running containers with their name, now we 
can do:

```sh
 docker logs containerName 
 # it opens the log for the process ID 1 of that container
```
we can follow the output with:

```sh
 docker logs -f containerName
```

we can even map a a directory on our host to the application log 
directory, so that we can see and operate locally on the 
container logs, for example with:

```sh
 docker run -d -P -v /nginxlogs:/var/log/nginx nginx 
 # now we can see logs in our directory /nginxlogs
```

## Inspecting a Container


We can inspect a container by executing the command:

```sh
 docker inspect containerName 
 # this will display all the details of the container in a JSON array
```

we can use grep to filter for a specific detail, for example:

```sh
 docker inspect containerName | grep IPAddress 
 # this will show the IP address of the specified container
```

## Configurazione del demone docker

Il file di configurazione è localizzato in "/etc/default/docker", 
possiamo usare DOCKER_OPTS per controllare le opzioni di startup 
del demone quando runna come servizio, dobbiamo riavviare il 
servizio per fare in modo che le modifiche abbiano effetto con:

```sh
 sudo service docker restart 
 # riavvia il demone, è possibile 
 # anche con sudo systemctl restart docker
```

possiamo invece ad esempio debuggare runtime avviando docker col 
comando:

```sh
 sudo docker -d --log-level=debug
```

## Security

Docker helps make applications safer as it provides a reduced set 
of default privileges and capabilities, namespaces provide an 
isolated view of the system, each container has its own set of:

 * IPC
 * network stack
 * rootfs
 * etc...

Processes running in one container cannot see and effect 
processes in another container, the technology at the base of all 
this, is "Control Groups" or CGroups which isolate resource usage 
per containers, anyway we must ensure that a compromised 
container won't bring down the entire host by exhausting 
resources. Quick security considerations are:

 * docker daemon service must be run only as root
 * watch who we add to the docker group
 * if binding the daemon to a TCP socket, secure it with TLS
 * use linux hardening solution, such as:
  * apparmor
  * SELinux
  * GRSEC


## Private Registry (alternative to DockerHub)

We can run a new container using the registry image with:

```sh
 docker run -d -p 5000:5000 registry:2.0 
 # we must run an image 
 # from dockerhub to make our own registry
```
once we have downloaded it, we can verify that it is running with 
"docker ps"

now to push and pull from our private registry we can do:

```sh
 docker tag <imageID> myserver.net:5000/my-app:1.0
```
then

```sh
 docker push myserver.net:5000/my-app:1.0 
 # where instead of 
 # myserver.net we can even put an IP address
```
while to pull an image from our registry we do:

```sh
 docker pull myserver.net:5000/my-app:1.0
```
let's see another example (done on localhost):

we first rename an image with:

```sh
 docker tag 91jnu21e9122 localhost:5000/myhello-world:1.0
```
then we do:

```sh
 docker push localhost:5000/myhello-world:1.0
```
notice that when we pull from an IP address we'll get an error if 
we are not using SSL/TLS, so we must modify docker options, but 
before we stop the docker daemon, with "sudo systemctl stop 
docker" and we do:

```sh
 sudo vim /etc/default/docker
```
and we modify the line:

```text
DOCKER_OPTS="--insecure-registry 104.131.142.17:5000"
```

and then we restart the docker daemon.


## Docker Compose

Docker compose is a tool for creating and managing multi 
container applications, containers are all defined in a single 
file colled "docker-compose.yml", and each container runs a 
particulart component/service of our application, for example:

 * web front end
 * user authentication
 * payments
 * database

container links should be defined and compose will spin up all 
our containers in a single command. We know that we can start 
different containers and link them together, but the number of 
components grows, this is very impractical, so compose helps us 
in this.

## Useful Things

## Removing all the intermediate layers by exporting a container into a new image

Let's say we have an image which have committed several times, 
this image will contain all the history of the commissions which 
have been done, we first have to run the container in which we 
are interested in and then export this snapshot with:

```sh
 docker export my-running-image-id > ~/my_image.tar 
 # the 
 # provided id must be the one we see in "docker ps", N.B.: The 
 # container must be running, export works with running containers
```
then now we can delete all the containers we have in the docker 
directory "/var/lib/docker/containers/", or first delete the easy 
ones with:

```sh
 docker rmi -f <container ID>
```
once we have removed even the images contained in the docker 
directory we have to import our tarball archive, we can do this 
with:

```sh
 docker import ~/my_image.tar
```
our image if imported in this way will have name and repo name 
set to "<none>", now we can rename the image with:

```sh
 docker tag imageID repoName:imageName
```
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
## Vagrant

Vagrant is a tool for building and managing virtual machine 
environments in a single workflow. With an easy-to-use workflow 
and focus on automation, Vagrant lowers development environment 
setup time, increases production parity, and makes the "works on 
my machine" excuse a relic of the past.

