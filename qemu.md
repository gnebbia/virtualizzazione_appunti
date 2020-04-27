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
 yum install qemu-kvm
```
```sh
 yast -i kvm
```
mentre nel caso in cui avessimo una CPU senza estensioni di 
virtualizzazione allora installeremo

```sh
 apt-get install qemu qemu-kvm libvirt-bin
```
```sh
 yum install qemu
```
```sh
 yast -i qemu
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

 * il "copy on write"
 * encryption
 * compressione
 * snapshot di macchine virtuali

anche se Qemu supporta correntemente questi formati di immagine 
disco:

 * raw: questo è un formato binario semplice di un'immagine disco
   ed è molto portable
 * cloop: Compressed Loop format, usata principalmente per particolari 
   immagini live come Knoppix e altri cd live
 * cow: immagine copy on write, supportata per ragioni storiche
 * qcow: immagina nativa di qemu, versione precedente, supportata 
   per questioni di compatibilità
 * qcow2: immagine nativa di qemu
 * vmdk: immagine di VMware
 * vdi: immagine di Virtualbox

### Creazione di un'immagine disco (ovvero come fare il set up di un sistema guest)

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
 qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img -cdrom nomeIso.iso -boot d 
 # in questo caso stiamo inizializzando un 
 # sistema con 256MB di RAM, e utilizzando la iso menzionata come 
 # immagine montata al boot, ricorda che se qemu non viene 
 # trovato, dobbiamo usare kvm, solitamente "kvm" è 
 # un'abbreviazione di "qemu-system-[myArch] -enable-kvm", 
 # attenzione se l'architettura emulata ha la stessa architettura
```
```sh
 qemu-system-x86_64 -enable-kvm -m 256 -hda miaImg.img -cdrom /dev/cdrom -boot d 
 # in questo caso viene preso proprio il 
 # contenuto del lettore cd/dvd del sistema host come boot per 
 # l'immagine, al posto di "/dev/cdrom" potremmo avere "/dev/sr0" 
 # o "/dev/dvd" a differenza della configurazione HW/SW del 
 # sistema host
```
N.B.: Si può passare in modalità full-screen con la combinazione "Ctrl+Alt+f".

Vediamo altre opzioni di boot più complessa:

```sh
 qemu-system-x86_64 -enable-kvm -cpu host -smp 2 -hda miaImg.img \
 -cdrom W7ALLINONE.iso -boot d -m 1024 -k it -netdev  \
 user,id=user.0 -device e1000,netdev=user.0 -usb -usbdevice \
 tablet -vga qxl -display sdl -monitor stdio 
 # con smp specifichiamo il numero di processori, con -cpu host 
 # specifichiamo la cpu del sistema ospitante
```
```sh
 qemu-system-x86_64 -enable-kvm -cpu host -smp 2 -hda miaImg.img \
 -cdrom debian-8.2.0-amd64-netinst.iso -boot d -m 1024 -usb -vga qxl 
 # per vga esistono diverse opzioni, se dovessimo avere 
 # problemi col video possiamo provare le altre, possiamo 
 # analizzarle attraverso "man qemu-system-x86_64" e poi cercando "-vga"
```
```sh
 qemu-system-ppc -hda miaImg.img -cdrom \
 debian-8.2.0-powerpc-netinst.iso -boot d -m 1024 -usb 
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
 qemu-system-mips -cpu 4Km -hda miaImg.img -cdrom debian_8.0_mips.iso -boot d -m 1024
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
