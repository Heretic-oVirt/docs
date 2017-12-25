# Heretic oVirt Project - Overview

## Introduction

###  What is this about 

This project aims at the automatic (non interactive) and from-scratch (using new/recycled machines) setup of a complete enterprise infrastructure based on [oVirt][10]&nbsp;with [Self Hosted Engine][60] (ie with the oVirt Engine, the machine controlling the whole infrastructure, hosted as a virtual machine inside the infrastructure itself), hyperconverged (ie with [Gluster][16] storage provided by the same machines that provide virtualization services), single-fault tolerant and with advanced integrated network functions (by means of [OVN][18]).

By "_complete enterprise infrastructure_" we mean a solution based on standard hardware (64 bit Intel/AMD compatible machines, absolutely off-the-shelf) which _using free software provides all functionalities_ needed (so, by realizing everything from virtualization to storage and networking by means of software, we can say that we realize a "Software Defined Data Center": SDDC) by a full range of enterprises, from the smallest up to the medium/large ones (obviously by scaling the hardware specifications accordingly).
 

###  The HVP project

The project web site can be found at the URL:  
  
https://dangerous.ovirt.life/
  
while all the development happens on Github:  
  
https://github.com/Heretic-oVirt
  
The reason for the name "**Heretic**" (and for the tongue-in-cheek name of the web site) lays in our choice of applying some of the aforementioned technologies in a way that is not the one currently deemed "orthodox", ie "formally supported" on the "Enterprise" offerings (ie subject to payment, because bound to a support contract) of those same technologies (the "heretic" choices will be _underlined_ below).  
  

##  The hardware side

  
Our laboratory environment contains three different setups, two physical and one virtual, as an example of what can be used to realize (or simply try out) our project.


####  Physical setup 1

* 3 x HPE Microserver G7 each one with:
    * 1 x 500 GiB SATA disk dedicated to the operating system
    * 3 x 4 TiB SATA disks dedicated to data (only 1 x 2 TiB SATA disk for data on the last server)
    * 16 GiB RAM (only 12 GiB RAM on the last server)
    * 1 x 4-port additional Gigabit network adapter
* 1 generic PC made of:
    * an HPE Pavillion about 5 years old with 6 GiB RAM and a Core2 Duo processor
    * 3 x network adapters (1 embedded e 2 on expansion Gigabit cards)
* 2 x Gigabit network switches (one Jumbo-Frame capable)
* 21 network cables (CAT-5e UTP) of proper lengths
  
#### Physical setup 2

* 3 x HPE Microserver G8 each one with:
    * 1 x 500 GiB SATA disk dedicated to the operating system
    * 3 x 4 TiB SATA disks dedicated to data (only 1 x 2 TiB SATA disk for data on the last server)
    * 16 GiB RAM (only 12 GiB RAM on the last server)
    * 1 x 4-port additional Gigabit network adapter
* 1 generic PC made of:
    * a Dell about 5 years old with 2 GiB RAM and a Core2 Duo processor
    * 5 x network adapters (1 embedded, 1 on an expansion Gigabit card and 3 on Ethernet-USB adapters)
* 1 x Gigabit network switch (Jumbo-Frame, VLAN and LACP capable)
* 26 network cables (CAT-5e UTP) of proper lengths
  
#### Virtual setup

* 3 x virtual servers each one with:
    * 1 x 64 GiB virtual disk dedicated to the operating system
    * 2 x 200 GiB virtual disks dedicated to data (only 1 x 200 GiB virtual disk for data on the last server)
    * 4 GiB RAM and 2 CPU cores (with support for nested virtualization)
    * 4 x virtual network adapters
* 1 desktop virtual machine made of:
    * 2 GiB RAM and 1 CPU core
    * 5 x virtual network adapters
* 1 virtuale LAN in NAT mode and 4 isolated virtual LAN segments
  
The generic PC/VD is meant only as a reference during setup but it is assumed to be decommissioned after installation has been completed.
We also validated a mixed setup with a virtual generic PC/VD, hosted on a laptop with 5 network adapters (1 embedded and 4 on Ethernet-USB adapters).


##  The automation technologies

  
Our solution is based on:  

* for the initial part on the time-proven installation automation technology of CentOS/Fedora/RHEL: **[Kickstart][61]**
* for the following part on new configuration management technologies: **[gDeploy][62]** e **[Ansible][63]**


####  Kickstart
  
It consists of a text file which is used to instruct the CentOS/Fedora/RHEL installation program (called Anaconda) to configure from scratch the machine in a non-interactive mode (you basically pre-fill all the answers for the questions that the interactive mode would instead ask, augmented by further choices that would not be available interactively and would instead require a huge amount of work on the hardware beforehand or on the machine itself afterwards).
The great flexibility offered by the Kickstart technology consists in the availability of other optional steps which preceed and follow the step properly replacing user interaction: those optional steps are made of scripts which the Kickstart file author can elaborate at will.
  

####  Ansible


It consists of a generic configuration management solution which, starting from a textual description (several files, called playbooks, in YAML format) of the desired state for the machines (using the word "state" in the most general sense: installed packages, users and groups, active services and so on), connects to those (by means of SSH: no installed agents needed), inspects the current state (together with other parameters, called "facts") and brings the machines into the desired state.
  

####  gDeploy


It consists of a specialized Gluster/RHGS infrastructure setup automation system which, based on a textual description (in INI file format) of the desired state (IP/names of the machines member of the trusted pool, LVM disk configurations for each machine, logical volumes formatting to create the bricks, Gluster volumes creation from the bricks), automatically generates the Ansible playbooks (see above) needed to reach it.
  
  
#### How we combine the automation technologies


In our specific implementation, we added into the Kickstarts (using those steps identified by the %pre and %post directives) some Bash scripts which, by means of standard Linux commands available inside the minimal installation environment and optionally recognizing some custom parameters (to change our default settings) passed by the boot kernel commandline (or by configuration fragments retrieved together with the Kickstart file), perform an automated discovery and configuration of the whole hardware setup: network connections (find which port is connected to which network), storage resources assignments (how to use each one of the available disks), single machine role (define the identity of the node inside the oVirt machines cluster), software parameters settings (conforming, whenever possible, to known "best practices") and so on.

Those configurations do not only affect the installation part, but also create from scratch full configuration files for further functionalities which are usually not made available by interactive installations, eg a self-sufficient network resolution system (DNS) both for local system names (those of the machines we are going to configure/create) and for Internet names.
  
In fact, we aim at use cases not only inside full-blown settings; in other terms, we do not assume to be deployed inside a fully formed enterpise framework with already present basic facilities, consequently we added to our solution all the functionalities which allow a "from scratch" approach for the enterprise network environment: what we listed above in our sample setups is all that is needed, nothing more is "hidden/assumed" (except for an Internet connection).

At the end of the Kickstart-automated installations of the support PC/VD and of the servers, we use gDeploy and Ansible to automatically orchestrate all the further configurations which will set up the storage, virtualization, networking and all the specific virtual machines offering other services to the enterprise network.


##  The actual procedure
  

We now detail the actual procedure.  
  

####  The support PC/VD
  
Start from connecting the suppport PC/VD:  
  
* on one network port (usually the "main", embedded one) the PC/VD must be connected to the Internet (eg by means of a router / access point)
* on the other network ports the PC/VD must be connected to the available network switches

##### The networks
  
The network switches must create **from one to four distinct networks** (ie isolated from each other and from the rest of the network environment, Internet included, either by being realized on simple unconnected distinct switches or by being realized as different VLANs on more capable switches):

1. If the PC/VD has only one further network port (beyond the one connected to the Internet), this must be connected to a separate network: the **management** one (this network will allow the communication between the servers, as oVirt nodes, and the oVirt Engine).
2. If the PC/VD has one additional further network port (like on our aforementioned example setups), this must be connected to a further separate network: the **Gluster** network (this network will allow the synchronization communications between the servers, as Gluster nodes).
3. If the PC/VD has another one additional further network port, this must be connected to another further separate network: the **production** network (ie the network to which also users' client PCs and any other pre-existing server/appliance are connected and consequently also the network to which our future virtual machines and Samba/Gluster-NFS file sharing services will "accede").
4. Finally, if the PC/VD has one additional further network port, this must be connected to a further separate network: the **isolation** network (this network allows to separate the production network from the Internet and/or from other networks).
  
During the subsequent installation phase, the Kickstart will recognize network ports actually connected and will assign to the logical networks following the order listed above (further control on those networks, from IP addressing to MTU, can be exercised by means of kernel commandline parameters or by custom configuration files).
  
Obviously, if less than 4 separate networks are available, then the communications related to the missing separate networks will conflate on the actually present ones (we must stress the fact that it is mostly "difficult" the coexistence of oVirt management and Gluster synchronization traffic on the same network, and this is why we always have at least two separate networks in our setups: in this case the second network gets automatically relegated to the exclusive use of Gluster synchronization while the other network takes hold of all other functions).

##### Fault tolerance

Poche parole relative alla resistenza ai guasti ("fault tolerance"): perché una soluzione sia resistente ai guasti (e si intende sempre al più un singolo guasto qualsiasi alla volta) le componenti base devono essere ridondate; abbiamo almeno 3 server (3 e non 2 per avere sempre una maggioranza qualificata, ovvero un "quorum") per l'infrastruttura, ma ovviamente la presenza di switch singoli (ad esempio un solo switch per ogni rete isolata o un solo switch dotato di VLAN per ospitare tutte le reti isolate) presenterebbe un singolo punto debole (SPOF:&nbsp;"single point of failure") sufficiente ad inficiare la resistenza ai guasti; nei nostri setup dimostrativi per semplicità non lo abbiamo sempre dimostrato, ma gli switch dovrebbero sempre essere doppi in cascata (2 switch in cascata per ogni rete isolata o 2 switch in cascata dotati di VLAN per ospitare tutte le reti isolate) e le porte di rete connesse da ogni server verso ogni rete separata sempre almeno doppie.  

##### The installation

Una volta avviata (ad esempio da un normale DVD CentOS7) l'installazione del PC/VD di supporto (anche questa gestita da un apposito Kickstart dedicato), dopo aver opzionalmente specificato sulla riga di comando del kernel (oltre alla collocazione del suddetto Kickstart, ad esempio con inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heresiarch.ks ) eventuali parametri custom (tutti con prefisso hvp_ ; l'elenco completo con spiegazione e relativi valori di default è fornito nei commenti in cima ad ogni Kickstart), si attende il riavvio automatico per ritrovarsi davanti al classico login grafico GNOME3 di CentOS7 (anche username e password per accedere sono personalizzabili ed i default sono documentati nei commenti interni ai Kickstart).  
  
Al termine dell'installazione il PC/VD di supporto sarà immediatamente pronto a fornire alle succitate reti isolate tutti i servizi necessari:  

* DNS (per risolvere i nomi sulle reti isolate e su Internet)
* NTP (per fornire riferimento orario)
* DHCP e PXE sulla rete di gestione (per effettuare il boot da rete nelle installazioni successive)
* Mirror locale (parziale) del sito HTTP del progetto (per fornire i file necessari ad avviare le installazioni successive)
* Gateway verso Internet e proxy HTTP trasparente (per accelerare le successive installazioni di pacchetti da Internet)
* Repository di script Bash e playbook Ansible per automatizzare la fase finale della configurazione (dopo l'installazione di tutti i server)
  
  
####  The servers

  
Il passo successivo (da ripetere per ognuna delle macchine server che costituiranno l'infrastruttura permanente) è quello di collegare i server alle reti isolate.  
A seconda della disponibilità di porte di rete sui server (ma ne è supportata anche solo una, ovviamente perdendo in quel caso la ridondanza), è possibile collegare (sempre arbitrariamente) più di una porta di rete di ogni server alla medesima rete (ovvero switch dedicato o VLAN) e, nella successiva fase di installazione, il Kickstart (unico per tutti i server) riconoscerà ed interpreterà la situazione come un'intenzione di accorpamento dei link per ridondanza e bilanciamento del traffico (il cosiddetto "bonding", con modalità di bonding definite per default e controllabili tramite opzioni da commandline del kernel o da frammento di configurazione).  
  
L'elenco delle reti supportate è il medesimo indicato sopra nel caso del PC/VD di supporto (ma per i server non conta più l'ordine delle reti, in quanto la presenza appunto del PC/VD di supporto permette al Kickstart di identificare con esattezza a quale rete ogni porta sia connessa) e, nella successiva fase di installazione, il Kickstart riconoscerà le porte di rete arbitrariamente collegate e le assegnerà alle varie reti (con appositi parametri, dall'indirizzamento IP alla MTU, controllabili tramite opzioni da commandline del kernel o da frammento di configurazione).  
  
Eventuali schede hardware di gestione remota dei server (le iLO, nei casi dei server HPE come i nostri, o le iDRAC per i server Dell; altri produttori hanno soluzioni analoghe, a volte opzionali, sotto il nome di BMC o schede IPMI) vanno collegate alla rete isolata di gestione ed il PC/VD di supporto può essere utilizzato per accedere alla console remota offerta da tali schede.  
  
L'installazione concreta delle macchine server si prevede che avvenga tramite boot da rete (PXE), il che significa che sulle macchine server l'opzione di boot da rete deve essere attivata da firmware (BIOS o UEFI) e ci deve essere almeno una delle porte di rete predisposte al boot (quali siano dipende di nuovo dal firmware, ma la prima di quelle incorporate di solito è una scelta sicura) connessa alla rete di gestione sopra citata (ovvero: connessa al relativo switch dedicato o VLAN).  
  
La macchina server avviata da rete presenterà un menu di boot con voci precompilate (le ha create l'installazione del PC/VD di supporto, propagando anche eventuali modifiche ai default operate tramite opzioni da commandline del kernel, modifiche che non dovranno quindi essere ripetute) e l'unica scelta interattiva da compiere sarà selezionare il tipo (un server completo basato su CentOS, indicato come "Host", oppure un server minimale basato su oVirt-NextGenerationNode, indicato come "NGN") e l'identità (identificata come "Node 0", "Node 1" o "Node 2") della macchina che si sta installando, premere Invio ed attendere il prompt ad installazione terminata.  
  
Dietro le quinte il PC/VD di supporto istruirà i server ad installarsi tramite Kickstart con una riga di comando del kernel contenente i succitati parametri custom (oltre alla collocazione del suddetto Kickstart, ad esempio con&nbsp;inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heretic-ngn.ks che permetterà alla logica del Kickstart di individuare i frammenti di configurazione creati automaticamente).  
  
Sottolineiamo il fatto che il Kickstart dei server contiene una logica (sempre controllabile tramite opzioni custom da commandline del kernel o da frammento di configurazione) per scegliere il disco sul quale installare il sistema operativo; per assunto esso sarà il primo tra quelli con la minor dimensione disponibile: sarà poi compito di chi installa fare in modo che tale disco sia effettivamente avviabile, agendo sul firmware (BIOS o UEFI) della macchina server.  
  

####  The final configuration

  
Terminata l'installazione delle tre macchine server, si potrà procedere con la configurazione automatizzata delle funzionalità vere e proprie.  
In qualunque "punto fermo" raggiunto dall'automazione è sempre possibile ispezionare lo stato ed eventualmente addirittura procedere da quel punto in poi in maniera manuale/interattiva (ad esempio a questo punto in caso si sia scelto il tipo "NGN" per i server, si potrebbe accedere via web dal PC/VD di supporto all'interfaccia [Cockpit][24] del "Node 0" e proseguire interattivamente come da relativa documentazione di oVirt): il nostro scopo è di automatizzare sì, ma mantenendo la piena compatibilità e tracciabilità dei passi, ovvero senza realizzare un sistema "chiuso".  
  
La configurazione automatizzata successiva (in parte in fase di sviluppo) è basata su playbook Ansible (fatti trovare già pronti dall'installazione sul PC/VD di supporto sotto /usr/local/etc/hvp-ansible oltre che in /etc/ansible/hosts e sotto /etc/ansible/group_vars ) che (oltre a passi di servizio quali la propagazione delle chiavi SSH ed il reperimento di dati sui nodi, quali numero e dimensione dei dischi disponibili, per pilotare le logiche successive) compiono i seguenti passi nell'ordine:  

1. generano il file di configurazione gDeploy per la creazione dello strato storage basato su Gluster (analogo alla fase corrispondente disponibile tramite Node Cockpit) con logica automatica di scelta dei dischi da dedicare a ciascuno dei volumi Gluster previsti (su ogni server sono gestiti fino a tre dischi per i dati dei volumi Gluster dedicati a: oVirt Engine, altre vm oVirt, immagini ISO, locking/clustering CTDB/NFS-Ganesha, file sharing CIFS/Windows e file sharing NFS/Unix)
2. effettuano l'installazione del Self Hosted Engine oVirt sul "Node 0" (analogo alla fase corrispondente disponibile tramite Node Cockpit)
3. configurano gli storage domain Gluster in oVirt (importando lo storage domain principale del Datacenter, azione che causa l'automatico riconoscimento dello storage domain del Self Hosted Engine e della vm Engine in esso contenuta)
4. aggiungono al cluster oVirt i nodi rimanenti oltre il "Node 0"
5. configurano ed avviano i servizi CTDB, Samba e Gluster-NFS (a breve anche Gluster-block, mentre Gluster-NFS verrà in futuro sostituito da NFS-Ganesha) basati su volumi Gluster (qui sta una prima _eresia_: usare lo storage iperconvergente anche per file sharing e non solo per la virtualizzazione)
6. configurano gli storage domain NFS in oVirt (essendo esportati dagli IP virtuali gestiti da CTDB, necessitano del passo precedente come prerequisito)
7. configurano OVN sull'Engine e sui nodi (a breve creando anche un paio di reti interne isolate potenzialmente utili per motivi di sicurezza e/o di test)
8. creano virtual machine aggiuntive sull'infrastruttura oVirt (tutte da installare ex-novo tramite Kickstart dedicati analoghi ai precedenti; questa automazione è ancora in fase di realizzazione):
    1. un domain controller Active Directory con creazione automatizzata di un dominio ex-novo (seconda _eresia_: CentOS7 con pacchetto Samba preso da Fedora&nbsp;e ricompilato per attivare le funzioni di DC usando le librerie interne Heimdal ecc.)
    2. un printer server membro del dominio Active Directory di cui sopra
    3. un database server membro del dominio Active Directory di cui sopra (CentOS7 con a scelta: PostgreSQL, MySQL, Firebird o, vera _eresia_&nbsp;;-) , SQLServer!)
    4. un desktop virtuale CentOS7, membro del dominio Active Directory di cui sopra
    5. altre vm opzionali (server gestionali che si appoggino al DB server di cui sopra, server di messaggistica/comunicazione, server firewall/proxy/VPN ecc. tutti membri del dominio Active Directory di cui sopra)
9. riconfigurano Samba come membro del dominio Active Directory di cui sopra

L'avvio (come utente root dal PC/VD di supporto) dei passi qui sopra elencati dovrà essere interattivo (ad esempio con&nbsp;ansible-playbook /usr/local/etc/hvp-ansible/hvp.yaml&nbsp;), ma solo per dare modo a chi installa di scegliere il momento opportuno ed eventualmente apportare modifiche manuali preventive a parametri e passi: dopo l'avvio, tutto quanto avviene in maniera automatica.  
  
Tutto il software utilizzato è presente (se modificato) in appositi [repository yum][25] con relativi pacchetti sorgenti (tutti i pacchetti sono prodotti usando mock e firmati con la [chiave GPG del progetto][26]).  
  
Sottolineiamo il fatto che i test compiuti finora hanno riguardato insiemi di **esattamente 3** **macchine server**, nel qual caso una delle 3, da installare come "Node 2" (ovvero quello indicato com "_l'ultimo server_" nelle specifiche hardware sopra citate), diverrà a livello Gluster per tutti i volumi un puro "**arbiter**" (ovvero non contiene/replica i dati dei file, ma solo i metadati), permettendo così di avere una macchina meno "carrozzata" delle altre due, quanto a dischi/CPU, senza influire sulle prestazioni globali; è però in previsione il supporto alla creazione iniziale (tramite modifica alle logiche di creazione del file gDeploy) di infrastrutture basate su **4 e più macchine server** e pure il supporto al più spinoso problema di espandere a 4 e più macchine server una infrastruttura inizialmente creata con 3 macchine server.  
  
Sottolineiamo anche che, come ennesima forma di provocazione/_eresia_, il nostro progetto utilizza (sempre prodotti e pubblicati come indicato sopra):  

* i **pacchetti Gluster** non community, bensì **ricompilati dai sorgenti di RHGS** (il Gluster con supporto offerto a pagamento da Red Hat) per motivi di estensione del ciclo di vita utile
* i **pacchetti Openvswitch ed OVN ricompilati da versioni Fedora** più recenti
  
  

##  Conclusions

  
A conclusione ricordo che abbiamo in progetto di porre mano a breve&nbsp;ad alcuni aspetti, in ordine di importanza:  

1. **documentare quanto fatto**, non solo in forma di commenti interni agli script/Kickstart (la sezione documentazione del sito è al momento colpevolmente vuota);
2. **estrarre la parte Ansible/gDeploy**&nbsp;dal file di Kickstart del PC/VD di supporto e darle vita propria nell'[apposito repository Github][64];
3. investigare la fattibilità e l'interesse generale per un progetto di **ricompilazione dei pacchetti oVirt** non community, bensì presi **dai sorgenti di RHV** (l'oVirt con supporto offerto a pagamento da Red Hat) per motivi di estensione del ciclo di vita utile.
  
Concludo indicando che tra gli ulteriori sviluppi futuri c'è l'aggiunta di soluzioni integrate di:  

* **backup** (usando il software libero [Bareos][27], di nuovo da ricompilare in una versione mantenuta aggiornata);
* **monitoraggio** (accentramento dei log e/o integrazione con le soluzioni di "Metrics Store"&nbsp;[in corso di sviluppo in oVirt][28]);
* **spegnimento totale non presidiato** a seguito di eventi esterni (si pensi al caso di piccole realtà con UPS singoli, o anche centralizzati, ma senza la garanzia di alimentazione ininterrotta tramite generatori ecc.).
  
  

[1]: http://1.bp.blogspot.com/-ExvH-X8_0pE/VjD27i2QALI/AAAAAAAAOdo/lQj-MO1ilhc/s1600-r/oVirt%2Bitalia.png
[2]: http://www.ovirt-italia.it/
[3]: http://www.ovirt-italia.it/2015/10/che-cose-ovirt.html
[4]: http://www.ovirt-italia.it/2016/11/vmware-e-ovirt-confronto.html
[5]: http://www.ovirt-italia.it/p/ovirt-live.html
[6]: https://resources.blogblog.com/img/icon18_wrench_allbkg.png
[7]: //www.blogger.com/rearrange?blogID=7835061380127359940&amp;widgetType=PageList&amp;widgetId=PageList1&amp;action=editWidget§ionId=crosscol-overflow "Modifica"
[8]: https://www.bglug.it/
[9]: https://www.fablabbergamo.it/
[10]: https://www.ovirt.org/
[11]: https://www.redhat.com/en/technologies/virtualization
[12]: https://www.centos.org/
[13]: https://www.redhat.com/it/technologies/linux-platforms/enterprise-linux
[14]: https://www.ovirt.org/node/
[15]: https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.1/html/installation_guide/red_hat_virtualization_hosts
[16]: https://www.gluster.org/
[17]: https://www.redhat.com/en/technologies/storage/gluster
[18]: http://openvswitch.org/support/dist-docs/ovn-architecture.7.html
[19]: https://ctdb.samba.org/
[20]: https://www.samba.org/
[21]: https://github.com/nfs-ganesha/nfs-ganesha/wiki
[22]: https://1.bp.blogspot.com/-zAM2Wlh6RYI/WaSPzHsUdVI/AAAAAAAAABE/l0hDafn2tRMx8OYeObVeGfY-m-rcFWFqgCLcBGAs/s320/HVP-demo.jpg
[23]: https://2.bp.blogspot.com/-20DaVbl-5AI/Waih2WfE-yI/AAAAAAAAABY/QPiPqD3-PMwmWban_oM3pn-GOqjv4rLnQCLcBGAs/s320/HVP-PXE-menu.JPG
[24]: http://cockpit-project.org/
[25]: https://dangerous.ovirt.life/hvp-repos/el7/
[26]: https://dangerous.ovirt.life/hvp-repos/RPM-GPG-KEY-hvp
[27]: https://www.bareos.org/
[28]: http://www.ovirt.org/develop/release-management/features/metrics/metrics-store/
[29]: https://plus.google.com/101771608258751283128 "author profile"
[30]: http://www.ovirt-italia.it/2017/09/heretic-ovirt-project-il-recap.html "permanent link"
[31]: https://resources.blogblog.com/img/icon18_edit_allbkg.gif
[32]: https://www.blogger.com/post-edit.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;from=pencil "Modifica post"
[33]: https://www.blogger.com/share-post.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;target=email "Invia tramite email"
[34]: https://www.blogger.com/share-post.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;target=blog "Postalo sul blog"
[35]: https://www.blogger.com/share-post.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;target=twitter "Condividi su Twitter"
[36]: https://www.blogger.com/share-post.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;target=facebook "Condividi su Facebook"
[37]: https://www.blogger.com/share-post.g?blogID=7835061380127359940&amp;postID=6085606257169617124&amp;target=pinterest "Condividi su Pinterest"
[38]: http://www.ovirt-italia.it/2017/02/il-recap-del-workshop-di-milano.html "Post più vecchio"
[39]: http://www.ovirt-italia.it/feeds/6085606257169617124/comments/default
[40]: http://linuxdaymilano.org/2017/
[41]: http://www.ovirt-italia.it/p/eventi.html
[42]: //www.blogger.com/rearrange?blogID=7835061380127359940&amp;widgetType=Text&amp;widgetId=Text1&amp;action=editWidget§ionId=sidebar-right-1 "Modifica"
[43]: //www.blogger.com/rearrange?blogID=7835061380127359940&amp;widgetType=Poll&amp;widgetId=Poll2&amp;action=editWidget§ionId=sidebar-right-1 "Modifica"
[44]: javascript:void(0)
[45]: http://www.ovirt-italia.it/2017/
[46]: http://www.ovirt-italia.it/2017/09/
[47]: http://www.ovirt-italia.it/2017/09/heretic-ovirt-project-il-recap.html
[48]: http://www.ovirt-italia.it/2017/02/
[49]: http://www.ovirt-italia.it/2017/01/
[50]: http://www.ovirt-italia.it/2016/
[51]: http://www.ovirt-italia.it/2016/11/
[52]: http://www.ovirt-italia.it/2016/06/
[53]: http://www.ovirt-italia.it/2015/
[54]: http://www.ovirt-italia.it/2015/11/
[55]: http://www.ovirt-italia.it/2015/10/
[56]: http://www.ovirt-italia.it/2015/09/
[57]: //www.blogger.com/rearrange?blogID=7835061380127359940&amp;widgetType=BlogArchive&amp;widgetId=BlogArchive1&amp;action=editWidget§ionId=sidebar-right-1 "Modifica"
[58]: https://www.blogger.com
[59]: //www.blogger.com/rearrange?blogID=7835061380127359940&amp;widgetType=Attribution&amp;widgetId=Attribution1&amp;action=editWidget§ionId=footer-3 "Modifica"
[60]: https://www.ovirt.org/documentation/self-hosted/Self-Hosted_Engine_Guide/
[61]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations
[62]: https://github.com/gluster/gdeploy
[63]: https://www.ansible.com/
[64]: https://github.com/Heretic-oVirt/ansible
