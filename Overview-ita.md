# Heretic oVirt Project - Panoramica

## Introduzione

###  Di cosa si tratta 

Questo progetto mira all'approntamento automatizzato (non interattivo) e da zero (macchine nuove/riciclate) di una infrastruttura aziendale completa basata su [oVirt][10] con [Self Hosted Engine][60] (ovvero con l'oVirt Engine, la macchina di controllo dell'intera infrastruttura, ospitata come virtual machine all'interno dell'infrastruttura stessa), iperconvergente (ovvero con storage [Gluster][16] fornito dalle stesse macchine fisiche che fanno virtualizzazione), resistente ai singoli guasti e con evolute funzionalità integrate di rete (tramite [OVN][18]) e di file sharing (tramite [Samba][20]/[NFS-Ganesha][21]/[Gluster-Block][65]).  
  
Quando diciamo "_infrastruttura aziendale completa_" intendiamo una soluzione che, basandosi su hardware assolutamente standard (macchine a 64 bit Intel/AMD compatibili, del tutto generiche), _realizzi tramite software libero tutte le funzionalità_ che possono servire a realtà aziendali che vanno da quelle piccole/piccolissime fino a quelle medie/medio-grandi (ovviamente facendo crescere l'investimento hardware di conseguenza).  

Siccome creiamo tutto, dalla virtualizzazione allo storage e al networking, in software si può dire alla fine di avere un "Software Defined Data center": SDDC.
  
###  Il progetto HVP

Il sito del progetto si trova all'URL:  
  
https://dangerous.ovirt.life/
  
e lo sviluppo software avviene su Github:  
  
https://github.com/Heretic-oVirt
  
Il motivo della denominazione "**Heretic**" (e del nome scherzoso usato per il sito) sta nella scelta di usare alcune delle tecnologie sopra citate in una maniera che non è quella ritenuta correntemente "ortodossa", o meglio "formalmente supportata" da chi offre le versioni "Enterprise" (ovvero a pagamento, in quanto corredate di contratto di assistenza) di quelle tecnologie (i punti "eretici" verranno _evidenziati_ nel seguito).  
  

##  La parte hardware

  
Il nostro laboratorio comprende tre diversi setup, due fisici ed uno virtuale, che rappresentano un esempio di cosa si può usare per realizzare (o anche semplicemente sperimentare) il nostro progetto.


#### Setup fisico 1

* 3 HPE Microserver G7 ognuno opportunamente configurato con:
    * 1 disco SATA per sistema operativo da 500 GiB
    * 3 dischi SATA per dati da 4 TiB (solo 1 disco SATA per dati da 2 TiB sull'ultimo server)
    * 16 GiB di RAM (solo 12 GiB di RAM sull'ultimo server)
    * una scheda con 4 porte di rete Gigabit aggiuntive
* 1 PC generico composto da:
    * un HPE Pavillion di circa 5 anni con 6 GiB di RAM e processore Core2 Duo
    * 3 porte di rete (1 integrata e 2 su schede Gigabit aggiuntive)
* 2 switch di rete Gigabit (uno dei quali compatibile con Jumbo-Frame)
* 21 cavi di rete (CAT-5e UTP) di opportune lunghezze
  
#### Setup fisico 2

* 3 HPE Microserver G8 ognuno opportunamente configurato con:
    * 1 disco SATA per sistema operativo da 500 GiB
    * 3 dischi SATA per dati da 4 TiB (solo 1 disco SATA per dati da 2 TiB sull'ultimo server)
    * 16 GiB di RAM
    * una scheda con 4 porte di rete Gigabit aggiuntive
* 1 PC generico composto da:
    * un Dell di circa 5 anni con 2 GiB di RAM e processore Core2 Duo
    * 5 porte di rete (1 integrata, 1 su scheda Gigabit aggiuntiva e 3 con adattatori Ethernet-USB)
* 1 switch di rete Gigabit (compatibile con Jumbo-Frame, VLAN e LACP)
* 26 cavi di rete (CAT-5e UTP) di opportune lunghezze
  
#### Setup virtuale

* 3 server virtuali ognuno opportunamente configurato con:
    * 1 disco virtuale per sistema operativo da 64 GiB
    * 2 dischi virtuali per dati da 200 GiB (solo 1 disco per dati da 200 GiB sull'ultima macchina)
    * 4 GiB di RAM e 2 core
    * 4 porte di rete
* 1 macchina virtuale desktop composta da:
    * 2 GiB di RAM e 1 core
    * 5 porte di rete
* 1 LAN virtuale in NAT e 4 segmenti LAN virtuali isolati
  
Lo scopo del PC/VD generico è quello di fare da supporto solo durante l'installazione della soluzione e si suppone che, terminata l'installazione, possa essere dismesso.
È stato sperimentato anche un setup misto nel quale solo il PC/VD generico è virtuale, ospitato su di un portatile dotato di 5 porte di rete (1 integrata e 4 con adattori Ethernet-USB).
  

##  Le tecnologie di automazione

  
La nostra soluzione si basa:  

* per la parte iniziale sull'assodata tecnologia di automazione dell'installazione presente in CentOS/Fedora/RHEL: **[Kickstart][61]**
* per la parte successiva su nuove tecnologie di automazione della configurazione: **[gDeploy][62]** e **[Ansible][63]**


####  Kickstart

  
Si tratta di un file di testo tramite il quale si istruisce il programma di installazione (detto Anaconda) di CentOS/Fedora/RHEL a configurare da zero la macchina in maniera non interattiva (sostanzialmente si forniscono tutte le risposte alle domande che la modalità interattiva porrebbe, più altre scelte che anche in maniera interattiva non sarebbe semplice effettuare e richiederebbero invece un notevole lavoro o preventivo sull'hardware o successivo sulla macchina una volta installata).  
  
La grande flessibilità della tecnologia Kickstart risiede nella possibilità di fare precedere e seguire la fase che si sostituisce propriamente alle scelte interattive da altre fasi costituite dall'esecuzione di script, script definibili a piacere da chi compone il file di Kickstart.  
  

####  Ansible

  
Si tratta di un sistema assolutamente generico di gestione della configurazione che, basandosi su di una descrizione testuale (svariati file, detti playbook, in formato YAML) dello stato desiderato per le macchine in questione (nel senso più generale di "stato": pacchetti installati, utenti e gruppi, servizi attivi ecc.), si collega (tramite SSH: non servono agenti installati) ad esse, ne ricava lo stato attuale (ed altri parametri, detti "facts") e le porta nella condizione desiderata.  
  

####  gDeploy

  
Si tratta di un sistema di automazione della creazione di infrastrutture Gluster/RHGS che, basandosi su di una descrizione testuale (un file in formato INI) dello stato desiderato (nomi/IP delle macchine che devono comporre il trusted pool, configurazione dei dischi tramite LVM su ogni macchina, formattazione dei logical volume per ottenere i brick, creazione dei volumi Gluster a partire dai brick), genera automaticamente i playbook Ansible (vedi sopra) per ottenere quanto descritto.  
  
  
#### Come combiniamo le tecnologie di automazione tra loro


Per venire al nostro caso specifico, abbiamo inserito nei Kickstart (nelle fasi identificate dalle direttive %pre e %post) degli script in Bash che, avvalendosi dei normali comandi Linux presenti nell'ambiente minimale di installazione e opzionalmente riconoscendo alcuni parametri custom (per modificare i valori di default da noi scelti) passati tramite la commandline del kernel all'avvio (o tramite frammenti di configurazione reperiti assieme al file di Kickstart), effettuano un autoriconoscimento e una configurazione dell'intero ambiente hardware: connessioni di rete (decidere quali porte di rete appartengono a quale rete), destinazione delle parti storage (come usare i vari dischi presenti), ruolo della macchina specifica (assegnare l'identità del nodo nel gruppo di macchine oVirt), parametri specifici del software (conformandosi, per quanto possibile, alle cosiddette "best practices" pubblicate) e così via.  
  
Tali configurazioni automatiche non si fermano a quanto necessario in fase di installazione, ma creano e fanno trovare pronti anche i file di configurazione necessari ad attivare ex-novo funzionalità ulteriori non usualmente presenti nelle installazioni interattive, in particolare un sistema autosufficiente di risoluzione dei nomi di rete (DNS) sia locali (quelli dei sistemi che andiamo ad installare/creare) sia Internet.  
  
Il nostro caso d'uso tipico è infatti una realtà non necessariamente strutturata, ovverosia non supponiamo di trovarci in un'azienda che inserisca la nostra soluzione in un'architettura già completa quantomeno dei servizi base, quindi abbiamo aggiunto nella nostra soluzione funzionalità che permettono di partire "da zero" anche per quanto riguarda l'ambiente di rete aziendale: quello che è elencato sopra nei setup di esempio è quello che serve, nient'altro è "nascosto/ipotizzato" (a parte un collegamento Internet).  

Al termine delle installazioni automatizzate tramite Kickstart di PC/VD di supporto e dei server, gDeploy ed Ansible vengono usati per orchestrare automaticamente le configurazioni successive che creeranno le funzionalità di storage, virtualizzazione, networking e le virtual machine specifiche che forniranno alla rete aziendale i rimanenti servizi.


##  Il procedimento concreto
  

Passiamo ad illustrare il procedimento concreto.  
  

####  Il PC/VD di supporto
  
Si parte dal collegare il PC/VD di supporto:  
  
* su di una porta di rete (tipicamente quella "principale", incorporata nella macchina) il PC/VD deve essere collegato ad Internet (ad esempio passando da un router / access point)
* sulle altre porte di rete il PC/VD deve essere collegato agli switch presenti

##### Le reti
  
In particolare sugli switch dovranno essere realizzate **da una a quattro reti separate** (ovverosia isolate tra di loro e da tutto il resto, Internet incluso, o perché realizzate su più switch semplici distinti e non collegati o perché realizzate tramite VLAN su switch più sofisticati):  

1. Se il PC/VD avrà una sola porta di rete aggiuntiva (oltre a quella connessa ad Internet), questa dovrà essere connessa ad una rete separata: quella di **gestione** (ospiterà le comunicazioni tra i server, nel loro ruolo di nodi oVirt, e l'Engine oVirt).
2. Se il PC/VD avrà anche un'altra porta di rete aggiuntiva (come avviene nei nostri setup di esempio elencati sopra), questa dovrà essere connessa ad una ulteriore rete separata: quella di **Gluster** (ospiterà le comunicazioni di sincronizzazione tra i server nel loro ruolo di nodi Gluster).
3. Se il PC/VD avrà anche un'ulteriore porta di rete aggiuntiva, questa dovrà essere connessa ad una ulteriore rete separata: quella di **produzione** (per intenderci: quella cui sono connesse le postazioni client degli utenti ed eventuali altri server/apparati presenti, quindi quella su cui si "affacceranno" le virtual machine realizzate ed i servizi di file sharing offerti tramite Samba/Gluster-NFS dai server).
4. Se infine il PC/VD avrà anche un'ultima porta di rete aggiuntiva, questa dovrà essere connessa ad una ultima rete separata: quella di **isolamento** (quella tramite la quale la rete di produzione può essere isolata da Internet e/o da altre reti).
  
Nella successiva fase di installazione, il Kickstart riconoscerà le porte di rete effettivamente collegate e le assegnerà alle varie reti logiche nell'ordine sopra indicato (con appositi parametri, dall'indirizzamento IP alla MTU, controllabili tramite opzioni da commandline del kernel o inserite in appositi file di configurazione).  
  
Ovviamente la presenza di meno di 4 reti separate disponibili farà sì che il traffico relativo alle reti mancanti venga automaticamente spostato su quelle presenti (dobbiamo sottolineare il fatto che è in particolare assai "delicata" la compresenza di gestione oVirt e sincronizzazione Gluster sulla medesima rete, per questo nei nostri setup abbiamo sempre almeno due reti isolate separate: in quel caso automaticamente la seconda rete rimane ad esclusivo uso di Gluster e la prima rete assorbe tutte le altre funzioni).  

##### La resistenza ai guasti

Poche parole relative alla resistenza ai guasti ("fault tolerance"): perché una soluzione sia resistente ai guasti (e si intende sempre al più un singolo guasto qualsiasi alla volta) le componenti base devono essere ridondate; abbiamo almeno 3 server (3 e non 2 per avere sempre una maggioranza qualificata, ovvero un "quorum") per l'infrastruttura, ma ovviamente la presenza di switch singoli (ad esempio un solo switch per ogni rete isolata o un solo switch dotato di VLAN per ospitare tutte le reti isolate) presenterebbe un singolo punto debole (SPOF: "single point of failure") sufficiente ad inficiare la resistenza ai guasti; nei nostri setup per semplicità non lo abbiamo elencato, ma gli switch dovrebbero sempre essere doppi in cascata (2 switch in cascata per ogni rete isolata o 2 switch in cascata dotati di VLAN per ospitare tutte le reti isolate) e le porte di rete connesse da ogni server verso ogni rete separata dovrebbero sempre essere almeno doppie.  

##### L'installazione del PC/VD

Una volta avviata (ad esempio da un normale DVD CentOS7) l'installazione del PC/VD di supporto (anche questa gestita da un apposito Kickstart dedicato), dopo aver opzionalmente specificato sulla riga di comando del kernel (oltre alla collocazione del suddetto Kickstart, ad esempio con inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heresiarch.ks ) eventuali parametri custom (tutti con prefisso hvp_ ; l'elenco completo con spiegazione e relativi valori di default è fornito nei commenti in cima ad ogni Kickstart), si attende il riavvio automatico per ritrovarsi davanti al classico login grafico GNOME3 di CentOS7 (anche username e password per accedere sono personalizzabili ed i default sono documentati nei commenti interni ai Kickstart).  
  
Al termine dell'installazione il PC/VD di supporto sarà immediatamente pronto a fornire alle succitate reti isolate tutti i servizi necessari:  

* DNS (per risolvere i nomi sulle reti isolate e su Internet)
* NTP (per fornire riferimento orario)
* DHCP e PXE sulla rete di gestione (per effettuare il boot da rete nelle installazioni successive)
* Mirror locale (parziale) del sito HTTP del progetto (per fornire i file necessari ad avviare le installazioni successive)
* Gateway verso Internet e proxy HTTP trasparente (per accelerare le successive installazioni di pacchetti da Internet)
* Repository di script Bash e playbook Ansible per automatizzare la fase finale della configurazione (dopo l'installazione di tutti i server)
  
  
####  I server

  
Il passo successivo (da ripetere per ognuna delle macchine server che costituiranno l'infrastruttura permanente) è quello di collegare i server alle reti isolate.  
A seconda della disponibilità di porte di rete sui server (ma ne è supportata anche solo una, ovviamente perdendo in quel caso la ridondanza), è possibile collegare (sempre arbitrariamente) più di una porta di rete di ogni server alla medesima rete (ovvero switch dedicato o VLAN) e, nella successiva fase di installazione, il Kickstart (unico per tutti i server) riconoscerà ed interpreterà la situazione come un'intenzione di accorpamento dei link per ridondanza e bilanciamento del traffico (il cosiddetto "bonding", con modalità di bonding definite per default e controllabili tramite opzioni da commandline del kernel o da frammento di configurazione).  
  
L'elenco delle reti supportate è il medesimo indicato sopra nel caso del PC/VD di supporto (ma per i server non conta più l'ordine delle reti, in quanto la presenza appunto del PC/VD di supporto permette al Kickstart di identificare con esattezza a quale rete ogni porta sia connessa) e, nella successiva fase di installazione, il Kickstart riconoscerà le porte di rete arbitrariamente collegate e le assegnerà alle varie reti (con appositi parametri, dall'indirizzamento IP alla MTU, controllabili tramite opzioni da commandline del kernel o da frammento di configurazione).  
  
Eventuali schede hardware di gestione remota dei server (le iLO, nei casi dei server HPE come i nostri, o le iDRAC per i server Dell; altri produttori hanno soluzioni analoghe, a volte opzionali, sotto il nome di BMC o schede IPMI) vanno collegate alla rete isolata di gestione ed il PC/VD di supporto può essere utilizzato per accedere alla console remota offerta da tali schede.  
  
L'installazione concreta delle macchine server si prevede che avvenga tramite boot da rete (PXE), il che significa che sulle macchine server l'opzione di boot da rete deve essere attivata da firmware (BIOS o UEFI) e ci deve essere almeno una delle porte di rete predisposte al boot (quali siano dipende di nuovo dal firmware, ma la prima di quelle incorporate di solito è una scelta sicura) connessa alla rete di gestione sopra citata (ovvero: connessa al relativo switch dedicato o VLAN).  
  
La macchina server avviata da rete presenterà un menu di boot con voci precompilate (le ha create l'installazione del PC/VD di supporto, propagando anche eventuali modifiche ai default operate tramite opzioni da commandline del kernel, modifiche che non dovranno quindi essere ripetute) e l'unica scelta interattiva da compiere sarà selezionare il tipo (un server completo basato su CentOS, indicato come "Host", oppure un server minimale basato su oVirt-NextGenerationNode, indicato come "NGN") e l'identità (identificata come "Node 0", "Node 1" o "Node 2") della macchina che si sta installando, premere Invio ed attendere il prompt ad installazione terminata.  
  
Dietro le quinte il PC/VD di supporto istruirà i server ad installarsi tramite Kickstart con una riga di comando del kernel contenente i succitati parametri custom (oltre alla collocazione del suddetto Kickstart, ad esempio con inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heretic-ngn.ks che permetterà alla logica del Kickstart di individuare i frammenti di configurazione creati automaticamente).  
  
Sottolineiamo il fatto che il Kickstart dei server contiene una logica (sempre controllabile tramite opzioni custom da commandline del kernel o da frammento di configurazione) per scegliere il disco sul quale installare il sistema operativo; per assunto esso sarà il primo tra quelli con la minor dimensione disponibile: sarà poi compito di chi installa fare in modo che tale disco sia effettivamente avviabile, agendo sul firmware (BIOS o UEFI) della macchina server.  
  

####  La configurazione finale

  
Terminata l'installazione delle tre macchine server, si potrà procedere con la configurazione automatizzata delle funzionalità vere e proprie.  
In qualunque "punto fermo" raggiunto dall'automazione è sempre possibile ispezionare lo stato ed eventualmente addirittura procedere da quel punto in poi in maniera manuale/interattiva (ad esempio a questo punto in caso si sia scelto il tipo "NGN" per i server, si potrebbe accedere via web dal PC/VD di supporto all'interfaccia [Cockpit][24] del "Node 0" e proseguire interattivamente come da relativa documentazione di oVirt): il nostro scopo è di automatizzare sì, ma mantenendo la piena compatibilità e tracciabilità dei passi, ovvero senza realizzare un sistema "chiuso".  
  
La configurazione automatizzata successiva (in parte in fase di sviluppo) è basata su playbook Ansible (fatti trovare già pronti dall'installazione sul PC/VD di supporto sotto /usr/local/etc/hvp-ansible oltre che in /etc/ansible/hosts e sotto /etc/ansible/group_vars ) che (oltre a passi di servizio quali la propagazione delle chiavi SSH ed il reperimento di dati sui nodi, quali numero e dimensione dei dischi disponibili, per pilotare le logiche successive) compiono i seguenti passi nell'ordine:  

1. generano il file di configurazione gDeploy ed effettuano la creazione dello strato storage basato su Gluster (analogo alla fase corrispondente disponibile tramite Node Cockpit) con logica automatica di scelta dei dischi da dedicare a ciascuno dei volumi Gluster previsti (su ogni server sono gestiti fino a tre dischi per i dati dei volumi Gluster dedicati a: oVirt Engine, altre vm oVirt, immagini ISO, locking/clustering CTDB/NFS-Ganesha, file sharing CIFS/Windows e file sharing NFS/Unix; un ulteriore volume dedicato a Gluster-Block per i servizi iSCSI/FCoE verrà aggiunto in seguito)
2. generano il file di configurazione ed effettuano l'installazione del Self Hosted Engine oVirt sul "Node 0" (analogo alla fase corrispondente disponibile tramite Node Cockpit)
3. configurano gli storage domain Gluster in oVirt (importando lo storage domain principale del Datacenter, azione che causa l'automatico riconoscimento dello storage domain del Self Hosted Engine e della vm Engine in esso contenuta)
4. aggiungono al cluster oVirt i nodi rimanenti oltre il "Node 0"
5. configurano OVN ed altre reti (oltre a quella di management) sull'Engine e sui nodi (a breve creando anche un paio di reti interne isolate OVN potenzialmente utili per motivi di sicurezza e/o di test)
6. configurano ed avviano i servizi CTDB, Samba e Gluster-NFS (a breve anche Gluster-block, mentre Gluster-NFS verrà in futuro sostituito da NFS-Ganesha) basati su volumi Gluster (qui sta una prima _eresia_: usare lo storage iperconvergente anche per file sharing e non solo per la virtualizzazione)
7. configurano gli storage domain NFS in oVirt (attualmente solo quello delle immagini ISO)
8. creano virtual machine aggiuntive sull'infrastruttura oVirt (tutte da installare ex-novo tramite Kickstart dedicati analoghi ai precedenti; questa automazione è ancora in fase di realizzazione):
    1. un domain controller Active Directory con creazione automatizzata di un dominio ex-novo (seconda _eresia_: CentOS7 con pacchetto Samba preso da Fedora modificato e ricompilato per attivare le funzioni di DC usando le librerie Kerberos interne Heimdal ecc.)
    2. un printer server membro del dominio Active Directory di cui sopra
    3. un database server membro del dominio Active Directory di cui sopra (CentOS7 con a scelta: PostgreSQL, MySQL, Firebird o, vera _eresia_ ;-) , SQLServer!)
    4. un desktop virtuale CentOS7, membro del dominio Active Directory di cui sopra
    5. altre vm opzionali (un server gestionale che si appoggi al DB server di cui sopra, un server di messaggistica/comunicazione, un server firewall/proxy/VPN ecc. tutti membri del dominio Active Directory di cui sopra)
9. riconfigurano il Samba file server sui nodi Gluster come membro del dominio Active Directory di cui sopra

L'avvio (come utente root dal PC/VD di supporto) dei passi qui sopra elencati dovrà essere interattivo (ad esempio con ansible-playbook /usr/local/etc/hvp-ansible/hvp.yaml ), ma solo per dare modo a chi installa di scegliere il momento opportuno ed eventualmente apportare modifiche manuali preventive a parametri e passi: dopo l'avvio, tutto quanto avviene in maniera automatica.  
  
Tutto il software utilizzato è presente (se modificato) in appositi [repository yum][25] con relativi pacchetti sorgenti (tutti i pacchetti sono prodotti usando mock e firmati con la [chiave GPG del progetto][26]).  
  
Sottolineiamo il fatto che tutti i test e gli sviluppi compiuti finora hanno riguardato esclusivamente insiemi di **esattamente 3** **macchine server**, nel qual caso una delle 3, da installare come "Node 2" (ovvero quello indicato com "_l'ultimo server_" nelle specifiche hardware sopra citate), diverrà a livello Gluster per tutti i volumi un puro "**arbiter**" (ovvero non contiene/replica i dati dei file, ma solo i metadati), permettendo così di avere una macchina meno "carrozzata" delle altre due, quanto a dischi/CPU, senza influire sulle prestazioni globali; è però in previsione il supporto alla creazione iniziale di infrastrutture basate su **4 e più macchine server** e pure il supporto al più spinoso problema di espandere a 4 e più macchine server una infrastruttura inizialmente creata con 3 macchine server.  
  
Sottolineiamo anche che, come ennesima forma di provocazione/_eresia_, il nostro progetto utilizza (sempre prodotti e pubblicati come indicato sopra):  

* i **pacchetti Gluster/Samba/Ganesha** non community, bensì **ricompilati dai sorgenti di RHGS** (il Gluster con supporto offerto a pagamento da Red Hat) per motivi di estensione del ciclo di vita utile
* i **pacchetti Openvswitch ed OVN ricompilati da versioni Fedora** più recenti
  
  

##  Conclusione

  
A conclusione ricordiamo che abbiamo in progetto di porre mano a breve ad alcuni aspetti, in ordine di importanza:  

1. **documentare quanto fatto**, non solo in forma di commenti interni agli script/Kickstart e della presente panoramica generale, ma aggiungendo anche un guida rapida all'utilizzo ed una guida tecnica più approfondita;
2. **migliorare la parte Ansible/gDeploy** (ospitata nell'[apposito repository Github][64]) in modo da renderla generalmente usabile e più conforme agli standard per moduli/playbook;
3. investigare la fattibilità e l'interesse generale per un progetto di **ricompilazione dei pacchetti oVirt** non community, bensì presi **dai sorgenti di RHV** (l'oVirt con supporto offerto a pagamento da Red Hat) per motivi di estensione del ciclo di vita utile.
  
Tra gli ulteriori sviluppi futuri c'è anche l'aggiunta di soluzioni integrate di:  

* **backup** (usando il software libero [Bareos][27], di nuovo da ricompilare in una versione mantenuta aggiornata);
* **monitoraggio** (accentramento dei log e/o integrazione con le soluzioni di "Metrics Store" [in corso di sviluppo in oVirt][28]);
* **spegnimento totale non presidiato** a seguito di eventi esterni (si pensi al caso di piccole realtà con UPS singoli, o anche centralizzati, ma senza la garanzia di alimentazione ininterrotta tramite generatori ecc.).
  
  

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
[24]: http://cockpit-project.org/
[25]: https://dangerous.ovirt.life/hvp-repos/el7/
[26]: https://dangerous.ovirt.life/hvp-repos/RPM-GPG-KEY-hvp
[27]: https://www.bareos.org/
[28]: http://www.ovirt.org/develop/release-management/features/metrics/metrics-store/
[60]: https://www.ovirt.org/documentation/self-hosted/Self-Hosted_Engine_Guide/
[61]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations
[62]: https://github.com/gluster/gdeploy
[63]: https://www.ansible.com/
[64]: https://github.com/Heretic-oVirt/ansible
[65]: https://github.com/gluster/gluster-block
