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

