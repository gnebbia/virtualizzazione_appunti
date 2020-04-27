# Virtualbox

Virtualbox e' un famoso hypervisor utilizzato per la virtualizzazione
e costituisce un'ottima opzione quando c'e' la necessita' di virtualizzare
sistemi operativi diversi, ambienti misti con diversi sistemi operativi o 
quando in genere non ci si vuole appoggiare sul proprio kernel come avviene
per le tecnologie basate sui container.

Virtualbox fornisce un'ottima interfaccia grafica, ma diventa ancora piu' potente
quando viene utilizzato da riga di comando.

Vediamo di seguito alcuni comandi per utilizzarlo.

## Macchine Virtuali Esistenti

Per elencare le macchine virtuali disponibili eseguire:
```sh
VBoxManage list vms
# mostra le macchine virtuali disponibili
```

```sh
VBoxManage list runningvms
# mostra le macchine virtuali attualmente in running
```
Per avere un output ancora piu' dettagliato sulle runningvms possiamo eseguire:
```sh
VBoxManage list -l runningvms
```

## Creazione Macchine Virtuali

Quando creiamo una nuova macchina virtuale, l'unico argomento obbligatorio e' il nome della VM.
Tutti gli altri parametri possono essere cambiati anche post creazione.

Per elencare i sistemi operativi supportati per la creazione della nuova VM possiamo eseguire:
```sh
VBoxManage list ostypes
```

Per creare una nuova macchina virtuale di tipo Oracle Linux possiamo eseguire:
```sh
VBoxManage createvm --name testVM --ostype Oracle_64 --register
```
Nota che la creazione di una VM senza registrazione `--register` crea
solo un file XML con la definizione della macchina ma non la registra
su virtualbox per essere utilizzata.

A questo punto abbiamo creato una VM vuota, dobbiamo ancora scegliere
la configurazione e installare un sistema operativo.

A questo punto possiamo visualizzare le caratteristiche della VM con:
```sh
VBoxManage showvminfo testVM
```
Possiamo modificare la configurazione con:
```sh
VBoxManage modifyvm OracleLinux6Test --cpus 2 --memory 2048 --vram 12
```

Possiamo aggiungere una scheda di rete in configurazione bridged con:
```sh
VBoxManage modifyvm OracleLinux6Test --nic1 bridged --bridgeadapter1 eth0
```


## Avviamento Macchine Virtuali

Possiamo avviare una macchina virtuale creata con o senza GUI, questo e' 
particolarmente utile soprattutto quando si vuole fare accesso con ssh
oppure con RDP o altri protocolli di gestione remota.


Per avviare una macchina virtuale con GUI eseguire:
```sh
```

Per avviare una macchina virtuale senza GUI eseguire:
```sh
vboxheadless --startvm "NomeVM"
# il NomeVM possiamo capirlo anche da un `vboxmanage list vms`
```


Ricorda quindi che tutta la gestione di VM non solo e' scriptabile/automatizzabile
ma possiamo accedere da remoto a VM anche senza utilizzare extension pack proprietari.

Ad esempio se configuriamo una scheda di rete in NAT, possiamo abilitare il port
forwarding con:
```config
Proto = TCP
Host IP = 127.0.0.1 ;; per esporla solo su macchina locale e poi renderla accessibile con tunnel
Host Port = 2501 ;; la porta su cui sara' disponibile la nostra VM, e' arbitraria la scelta
Guest IP = 0.0.0.0 ;; esponiamo dalla VM di virtualbox il servizio su tutte le interfacce
Guest Port = 3389 ;; la porta relativa all'RDP, ma il procedimento e' analogo per altri servizi
```

A questo punto da una macchina remota possiamo accedere alla VM se e' stata accesa con
`vboxheadless --startvm "NomeVM"` creando un tunnel SSH, in questo modo:
```sh
ssh -L 127.0.0.1:40000:127.0.0.1:2501 user@remoteprovider.example.com -p <sshport>
# dove sshport e' la porte dove esiste il server ssh (di default 22)
# possiamo anche aggiungere un -N per evitare di ottenere una shell e un -f per 
# mandare tutto in background
```


## Creazione Server DHCP

Puo' essere utile creare server DHCP che possono essere utilizzati
all'interno di reti interne, cioe' configurazioni "internal network".

Per creare un server per una rete interna chiamata "intnet" possiamo eseguire:
```sh
vboxmanage dhcpserver add --netname intnet --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.10 --upperip 10.10.10.20 --enable
# Gli IP assegnabili sono nel range 10.10 al 10.20
```

Per visualizzare la lista di server DHCP disponibili possiamo eseguire:
```sh
vboxmanage list dhcpservers
```
