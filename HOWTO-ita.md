
# oVirt Italia: Heretic oVirt Project - Il recap dell'incontro BgLUG

[ ![oVirt Italia][1] ][2]

È l'ora di oVirt! Nel datacenter non c'è più bisogno di VMware.

* [Blog home][2]
* [Che cos'è oVirt?][3]
* [VMware e oVirt a confronto][4]
* [Prova subito oVirt Live][5]

[ ![][6] ][7]

## venerdì 1 settembre 2017

###  Heretic oVirt Project - Il recap dell'incontro BgLUG 

Mercoledì 2 agosto, nella sede del [BgLUG][8] presso [FABLAB][9] a Bergamo, si è tenuta una presentazione/demo live sull'approntamento automatizzato (non interattivo) e da zero (macchine nuove/riciclate) di una soluzione oVirt&nbsp;con Self Hosted Engine (ovvero con l'oVirt Engine, la macchina di controllo dell'intera infrastruttura, ospitata come virtual machine all'interno dell'infrastruttura stessa) ed&nbsp;iperconvergente (ovvero con storage Gluster fornito dalle stesse macchine fisiche che fanno virtualizzazione) resistente ai singoli guasti.  
  
L'incontro ha registrato il pieno nella sala messa a disposizione e, nonostante il caldo e alcuni inconvenienti tecnici nella parte live, l'interesse della platea è parso decisamente elevato.  
  
  
  

###  Il progetto HVP

  
Andrea Flori ed io abbiamo presentato un progetto, chiamato provocatoriamente **Heretic oVirt Project**, volto a produrre una soluzione interamente automatizzata per realizzare, partendo appunto da zero, un'infrastruttura aziendale completa basata su [oVirt][10]/[RHV][11] e tecnologie collegate: [CentOS][12]/[RHEL][13], [oVirt Node][14]/[RHVH][15],&nbsp;[Gluster][16]/[RHGS][17], [OVN][18], [CTDB][19], [Samba][20], [NFS-Ganesha][21].  
  
Quando diciamo "_infrastruttura aziendale completa_" intendiamo una soluzione che, basandosi su hardware assolutamente standard (macchine a 64 bit Intel/AMD compatibili, del tutto generiche), _realizzi tramite software libero tutte le funzionalità_ che possono servire (in questo senso, essendo tutto, dalla virtualizzazione, allo storage, al networking realizzato in software si può dire alla fine di avere un "Software Defined Data center": SDDC) a realtà aziendali che vanno da quelle piccole/piccolissime fino a quelle medie/medio-grandi (ovviamente facendo crescere l'investimento hardware di conseguenza).  
  
Il sito del progetto si trova all'URL:  
  
<https: dangerous.ovirt.life="">  
  
e lo sviluppo software avviene su Github:  
  
<https: github.com="" heretic-ovirt="">  
  
Il motivo della denominazione "**Heretic**" (e del nome scherzoso usato per il sito) sta nella scelta di usare alcune delle tecnologie sopra citate in una maniera che non è quella ritenuta correntemente "ortodossa", o meglio "supportabile formalmente" (i punti "eretici" verranno _evidenziati_ nel seguito).  
  
Per dissipare i timori su questo punto, ci siamo brevemente soffermati sul fatto che chi deve fornire _garanzie contrattuali_ sul supporto e deve farlo "facendo tornare i conti" _economicamente su larga scala_ (ad esempio: Red Hat) ha necessariamente (e ben comprensibilmente) un approccio più "cauto" nel limitare le modalità di configurazione sulle quali formalmente fornisce supporto, ma la comunità o gli integratori (magari più "piccoli ed agguerriti" rispetto a Red Hat), proprio per le possibilità insite nella natura del _software libero_, possono dimostrare concretamente come il limite possa essere spostato (anche se speriamo vivamente che si tratti solo di un "_anticipare un poco i tempi_").  
  

###  La parte hardware

  
Sul banco della demo erano presenti:  

* 3&nbsp;HPE Microserver&nbsp;ognuno opportunamente configurato con:
    * 1 disco&nbsp;SATA per sistema operativo da 500 GiB
    * 3 dischi SATA per dati da 4 TiB (solo 1 disco SATA per dati da 2 TiB sull'ultimo server)
    * 16 GiB di RAM (solo 12 GiB di RAM sull'ultimo server)
    * una scheda con 4 porte di rete Gigabit aggiuntive
* 1 PC generico composto da:
    * un qualunque desktop anche non nuovissimo (noi avevamo un HPE Pavillion di circa 5 anni)
    * 3 porte di rete (ottenibili o con schede aggiuntive o con adattatori Ethernet-USB, quindi anche un portatile può essere usato per tale ruolo)
* 2 switch di rete Gigabit
* il necessario gomitolo di cavi
  
Lo scopo del PC generico è quello di fare da riferimento solo durante l'installazione della soluzione e si suppone che, terminata l'installazione, se ne possa tranquillamente "tornare a casa" assieme a chi installa :-D  
  

###  Le tecnologie di automazione

  
La presentazione è iniziata indicando in che senso e come avvenisse la completa automazione da zero (o, come si suol dire, da "bare metal" ovvero dall'hardware "vuoto" appena comprato/riciclato) per tutto quanto presente sul banco.  
  
Abbiamo premesso subito che non abbiamo "inventato" nulla: quanto da noi realizzato è già oggi ottenibile in maniera "manuale" (se vi sono già delle automazioni preconfezionate complete, noi non ne siamo a conoscenza), ma richiede un numero considerevole di nozioni, scelte ed interventi tra di loro altamente correlati e con elevata probabilità di errori, omissioni ed improprietà.  
Come si diceva un tempo, se abbiamo ottenuto qualcosa in più è solo perché "siamo nani sulle spalle di giganti" :-)  
  
La nostra soluzione si basa:  

* per la parte iniziale sull'assodata tecnologia di automazione dell'installazione presente in CentOS/Fedora/RHEL: **Kickstart**
* per la parte successiva su nuove tecnologie di automazione della configurazione: **gDeploy** e **Ansible**
Abbiamo quindi brevemente introdotto queste tecnologie per chi non ne avesse già sentito parlare.  
  

####  Kickstart

  
Si tratta di un file di testo tramite il quale si istruisce il programma di installazione (detto Anaconda) di CentOS/Fedora/RHEL&nbsp;a configurare da zero la macchina in maniera non interattiva (sostanzialmente si forniscono tutte le risposte alle domande che la modalità interattiva porrebbe, più altre scelte che anche in maniera interattiva non sarebbe semplice effettuare e richiederebbero invece un notevole lavoro o preventivo sull'hardware o successivo sulla macchina una volta installata).  
  
La grande flessibilità della tecnologia Kickstart risiede nella possibilità di fare precedere e seguire la fase che si sostituisce propriamente alle scelte interattive da altre fasi costituite dall'esecuzione di script, script definibili a piacere da chi compone il file di Kickstart.  
  

####  Ansible

  
Si tratta di un sistema assolutamente generico di gestione della configurazione che, basandosi su di una descrizione testuale (svariati file, detti playbook, in formato YAML) dello stato desiderato per le macchine in questione (nel senso più generale di "stato": pacchetti installati, utenti e gruppi, servizi attivi ecc.), si collega (tramite SSH: non servono agenti installati) ad esse, ne ricava lo stato attuale (ed altri parametri, detti "facts") e le porta nella condizione desiderata.  
  

####  gDeploy

  
Si tratta di un sistema di automazione della creazione di infrastrutture Gluster/RHGS che, basandosi su di una descrizione testuale (un file in formato INI) dello stato desiderato (nomi/IP delle macchine che devono comporre il trusted pool, configurazione dei dischi tramite LVM su ogni macchina, formattazione dei logical volume per ottenere i brick, creazione dei volumi Gluster a partire dai brick), genera automaticamente i playbook Ansible (vedi sopra) per ottenere quanto descritto.  
  
  
  
Per venire al nostro caso specifico, abbiamo inserito nei Kickstart (nelle fasi identificate dalle direttive&nbsp;%pre e %post) degli script in&nbsp;Bash&nbsp;che, avvalendosi dei normali comandi Linux presenti nell'ambiente minimale di installazione e opzionalmente riconoscendo alcuni parametri&nbsp;custom&nbsp;(per modificare i valori di default da noi scelti) passati tramite la&nbsp;commandline&nbsp;del&nbsp;kernel&nbsp;all'avvio, effettuano un&nbsp;autoriconoscimento&nbsp;e una configurazione dell'intero ambiente hardware: connessioni di rete (decidere quali porte di rete appartengono a quale rete), destinazione delle parti&nbsp;storage&nbsp;(come usare i vari dischi presenti), ruolo della macchina specifica (assegnare l'identità del nodo nel gruppo di macchine&nbsp;oVirt), parametri specifici del software (conformandosi, per quanto possibile, alle cosiddette "best&nbsp;practices" pubblicate) e così via.  
  
Tali configurazioni automatiche non si fermano a quanto necessario in fase di installazione, ma creano e fanno trovare pronti anche i file di configurazione necessari ad attivare ex-novo funzionalità ulteriori non usualmente presenti nelle installazioni interattive, in particolare un sistema autosufficiente di risoluzione dei nomi di rete (DNS) sia locali (quelli dei sistemi che andiamo ad installare/creare) sia Internet.  
  
Il nostro caso d'uso tipico è infatti una realtà non necessariamente strutturata, ovverosia non supponiamo di trovarci in un'azienda che inserisca la nostra soluzione in un'architettura già completa quantomeno dei servizi base, quindi abbiamo aggiunto nella nostra soluzione funzionalità che permettono di partire "da zero" anche per quanto riguarda l'ambiente di rete aziendale: quello che avevamo sul banco è quello che serve, nient'altro è "nascosto/ipotizzato" (a parte un collegamento Internet).  

  
Ci siamo a questo punto scusati per lo stato attuale della documentazione del progetto, al momento consistente solo ed esclusivamente nei commenti interni ai file di Kickstart; tali commenti sono scritti in inglese (per motivi di generalità, ma non temete: è un inglese "de' noaltri" :-D ) e sono marcati dalla dicitura&nbsp;TODO&nbsp;per le parti che indicano bug/mancanze e&nbsp;Note&nbsp;per le parti che indicano qualcosa cui porre attenzione.  
  

![][22]

  

###  Il procedimento concreto

  

Siamo quindi passati ad illustrare il procedimento concreto.  
  

####  Il PC di supporto

  
Si parte dal collegare il PC di supporto:  
  

* su di una porta di rete (tipicamente quella "principale", incorporata nella macchina, ma ne dovrà essere comunque individuato il nome Linux, ad esempio tramite un avvio preventivo da un DVD Live o dal DVD di installazione medesimo in modalità rescue; supporremo nel seguito che tale nome sia eno1) il PC deve essere collegato ad Internet (ad esempio passando da un router / access point)
* sulle altre porte di rete il PC deve essere collegato agli switch presenti
  
In particolare sugli switch dovranno essere realizzate **da una a quattro reti separate** (ovverosia isolate tra di loro e da tutto il resto, Internet incluso, o perché realizzate su più&nbsp;switch&nbsp;semplici distinti e non collegati o perché realizzate tramite VLAN su switch più sofisticati):  

1. Se il PC avrà una sola porta di rete aggiuntiva (oltre a quella connessa ad Internet), questa dovrà essere connessa ad una rete separata: quella di **gestione **(ospiterà le comunicazioni tra i server fisici, nel loro ruolo di nodi oVirt, e l'Engine oVirt).
2. Se il PC avrà anche un'altra porta di rete aggiuntiva (come era durante la nostra presentazione), questa dovrà essere connessa ad una ulteriore rete separata: quella di **Gluster** (ospiterà le comunicazioni di sincronizzazione tra i server fisici nel loro ruolo di nodi Gluster).
3. Se il PC avrà anche un'ulteriore porta di rete aggiuntiva, questa dovrà essere connessa ad una ulteriore rete separata: quella di **produzione **(per intenderci: quella cui sono connesse le postazioni client degli utenti ed eventuali altri server/apparati presenti, quindi quella su cui si "affacceranno" le virtual machine realizzate ed i servizi di file sharing offerti tramite Samba/NFS-Ganesha dai server fisici).
4. Se infine il PC avrà anche un'ultima porta di rete aggiuntiva, questa dovrà essere connessa ad una ultima rete separata: quella di **isolamento**&nbsp;(lo scopo di questa rete è in fase di sviluppo).
  
Nella successiva fase di installazione, il Kickstart riconoscerà le porte di rete arbitrariamente collegate e le assegnerà alle varie reti nell'ordine sopra indicato (con appositi parametri, dall'indirizzamento IP alla MTU, controllabili tramite opzioni da commandline del kernel).  
  
Ovviamente la presenza di meno di 4 reti separate disponibili farà sì che il traffico relativo alle reti mancanti venga automaticamente spostato su quelle presenti (abbiamo fatto notare che è in particolare assai "delicata" la compresenza di gestione oVirt e sincronizzazione Gluster sulla medesima rete, per questo nella nostra presentazione avevamo due reti isolate separate: in quel caso automaticamente la seconda rete rimane ad esclusivo uso di Gluster e la prima rete assorbe tutte le altre funzioni).  
  
Una parola a questo punto relativa alla resistenza ai guasti ("fault tolerance"): perché una soluzione sia resistente ai guasti (e si intende sempre al più un singolo guasto qualsiasi alla volta) le componenti base devono essere ridondate; abbiamo almeno 3 server (per avere sempre una maggioranza qualificata, ovvero un "quorum") per l'infrastruttura, ma ovviamente la presenza di switch singoli (ad esempio un solo switch per ogni rete isolata o un solo switch dotato di VLAN per ospitare tutte le reti isolate) presenterebbe un singolo punto debole (SPOF:&nbsp;"single point of failure") sufficiente ad inficiare la resistenza ai guasti; nel nostro esempio per semplicità non lo abbiamo dimostrato, ma gli switch dovrebbero sempre essere doppi in cascata (2 switch in cascata per ogni rete isolata o 2 switch in cascata dotati di VLAN per ospitare tutte le reti isolate) e le porte di rete connesse da ogni server verso ogni rete separata sempre almeno doppie.  
  
Una volta avviata (ad esempio da un normale DVD CentOS7) l'installazione del PC di supporto (anche questa gestita da un apposito Kickstart dedicato), dopo aver opzionalmente specificato sulla riga di comando del kernel (oltre al nome Linux della porta di rete collegata ad Internet, ad esempio con ip=eno1:dhcp , ed alla collocazione del suddetto Kickstart, ad esempio con inst.ks=<https: dangerous.ovirt.life="" hvp-repos="" el7="" ks="" heresiarch.ks=""> ) eventuali parametri custom (tutti con prefisso hvp_ ; l'elenco completo con spiegazione e relativi valori di default è fornito nei commenti in cima ad ogni Kickstart), si attende il riavvio automatico per ritrovarsi davanti al classico login grafico GNOME3 di CentOS7 (anche username e password per accedere sono personalizzabili ed i default sono documentati nei commenti interni ai Kickstart).  
Per motivi di tempo, durante la presentazione il PC di supporto era già stato preventivamente installato.  
  
  
Al termine dell'installazione il PC di supporto sarà immediatamente pronto a fornire alle succitate reti isolate tutti i servizi necessari:  

* DNS (per risolvere i nomi sulle reti isolate e su Internet)
* NTP (per fornire riferimento orario)
* DHCP e PXE sulla rete di gestione (per effettuare il boot da rete nelle installazioni successive)
* Mirror locale del sito HTTP del progetto (per fornire i file necessari ad avviare le installazioni successive)
* Gateway verso Internet e proxy HTTP trasparente (per accelerare le successive installazioni di pacchetti da Internet)
* Repository di script Bash e playbook Ansible per automatizzare la fase finale della configurazione (dopo l'installazione di tutte le macchine)
  
  
Dalla platea (Simone Tiraboschi) è venuto il suggerimento di rimpiazzare queste funzioni realizzate ad-hoc con un sistema _Red Hat Satellite_: l'idea è interessante ed il motivo per cui non è stato usato è solo che per abitudine il Satellite lo si è sempre visto come parte "permanente" di una infrastruttura aziendale, non qualcosa che arriva per installare e se ne va alla fine per essere a sua volta reinstallato da zero per la realizzazione successiva, ma abbiamo promesso che valuteremo con maggiore attenzione il suggerimento (come controbattuto da Simone, infatti, nulla vieta di usare il Satellite in questo modo "transitorio").  
  

####  I server

  
Il passo successivo (da ripetere per ognuna delle macchine server che costituiranno l'infrastruttura permanente) è quello di collegare i server alle reti isolate.  
A seconda della disponibilità di porte di rete sui server (ma ne è supportata anche solo una, ovviamente perdendo in quel caso la ridondanza), è possibile collegare (sempre arbitrariamente) più di una porta di rete di ogni server alla medesima rete (ovvero switch dedicato o VLAN) e, nella successiva fase di installazione, il Kickstart (unico per tutti i server) riconoscerà ed interpreterà la situazione come un'intenzione di accorpamento dei link per ridondanza e bilanciamento del traffico (il cosiddetto "bonding", con modalità di bonding definite per default e controllabili tramite opzioni da commandline del kernel).  
  
L'elenco delle reti supportate è il medesimo indicato nel caso del PC di supporto (ma per i server non conta più l'ordine delle reti, in quanto la presenza appunto del PC di supporto permette al Kickstart di identificare con esattezza a quale rete ogni porta sia connessa) e, nella successiva fase di installazione, il Kickstart riconoscerà le porte di rete arbitrariamente collegate e le assegnerà alle varie reti (con appositi parametri, dall'indirizzamento IP alla MTU, controllabili tramite opzioni da commandline del kernel).  
  
Eventuali schede hardware di gestione remota dei server (le iLO, nei casi dei server HPE come i nostri, o le iDRAC per i server Dell; altri produttori hanno soluzioni analoghe, a volte opzionali, sotto il nome di BMC o schede IPMI) vanno collegate alla rete isolata di gestione ed il PC di supporto può essere utilizzato per accedere alla console remota offerta da tali schede.  
  
L'installazione concreta delle macchine server si prevede che avvenga tramite boot da rete (PXE), il che significa che sulle macchine server l'opzione di boot da rete deve essere attivata da firmware (BIOS o UEFI) e ci deve essere almeno una delle porte di rete predisposte al boot (quali siano dipende di nuovo dal firmware, ma la prima di quelle incorporate di solito è una scelta sicura) connessa alla rete di gestione sopra citata (ovvero: connessa al relativo switch dedicato o VLAN) ed opportunamente identificata (ne dovrà essere individuato il nome Linux, ad esempio tramite un avvio preventivo dalla voce di menu PXE relativa alla modalità&nbsp;rescue&nbsp;di CentOS7; supporremo nel seguito che tale nome sia&nbsp;em1).  
  

![][23]

  
  
La macchina server avviata da rete presenterà un menu di boot con voci precompilate (le ha create l'installazione del PC di supporto, propagando anche eventuali modifiche ai default operate tramite opzioni da commandline del kernel, modifiche che non dovranno quindi essere ripetute; dovrà essere però cura di chi installa modificare tale menu in /var/lib/tftpboot/default&nbsp;per inserire il corretto nome Linux della porta di rete di boot per ogni macchina) e l'unica scelta interattiva da compiere sarà selezionare l'identità della macchina che si sta installando (identificata come "Node 0", "Node 1" o "Node 2"), premere invio ed attendere il prompt ad installazione terminata (per motivi pratici, il primo riavvio dopo l'installazione dei server è seguito da un successivo riavvio automatico entro un minuto e non siamo ancora riusciti ad inibire il messaggio di login nell'intermezzo: si consiglia quindi di attendere un paio di minuti prima di accedere).  
  
Dietro le quinte il PC di supporto istruirà i server ad installarsi tramite Kickstart con una riga di comando del kernel contenente i succitati parametri custom (oltre al nome Linux della porta di rete collegata alla rete di gestione, ad esempio con&nbsp;ip=em1:dhcp&nbsp;, ed alla collocazione del suddetto Kickstart, ad esempio con&nbsp;inst.ks=<https: dangerous.ovirt.life="" hvp-repos="" el7="" ks="" heretic-ngn.ks="">&nbsp;).  
  
Sottolineiamo il fatto che il Kickstart dei server contiene una logica (sempre controllabile tramite opzioni custom da commandline del kernel) per scegliere il disco sul quale installare il sistema operativo; per assunto esso sarà l'ultimo tra quelli con la minor dimensione disponibile: sarà poi compito di chi installa fare in modo che tale disco sia effettivamente avviabile, agendo sul firmware (BIOS o UEFI) della macchina server.  
  

####  La configurazione finale

  
Terminata l'installazione delle tre macchine server, si potrà procedere con la configurazione automatizzata delle funzionalità vere e proprie.  
Ci abbiamo tenuto a specificare che, in qualunque "punto fermo" raggiunto dall'automazione, è sempre possibile fermarsi ed ispezionare lo stato ed eventualmente addirittura procedere da quel punto in poi in maniera manuale/interattiva (ad esempio a questo punto si potrebbe accedere via web dal PC di supporto all'interfaccia [Cockpit][24] del "Node 0" e proseguire interattivamente come da documentazione di oVirt): il nostro scopo è di automatizzare sì, ma mantenendo la piena compatibilità e tracciabilità dei passi, ovvero senza realizzare un sistema "chiuso".  
  
Giunti a questo punto la presentazione concreta in sala si è esaurita per raggiunti limiti di tempo (complici anche le difficoltà tecniche cui si era accennato, che hanno pure reso ahinoi poco "visibile" anche quanto sopra), ma è stata comunque delineata a voce la fase successiva.  
  
La configurazione automatizzata successiva (in parte in fase di sviluppo) è basata su playbook Ansible (fatti trovare già pronti dall'installazione sul PC di supporto sotto /usr/local/etc/hvp-ansible oltre che in /etc/ansible/hosts e sotto /etc/ansible/group_vars ) che (oltre a passi di servizio quali la propagazione delle chiavi SSH ed il reperimento di dati sui nodi, quali numero e dimensione dei dischi disponibili, per pilotare le logiche successive) compiono i seguenti passi nell'ordine:  

1. generano il file di configurazione gDeploy per la creazione dello strato storage basato su Gluster (analogo alla fase corrispondente disponibile tramite Cockpit) con logica automatica di scelta dei dischi da dedicare a ciascuno dei volumi Gluster previsti (su ogni server sono gestiti fino a tre dischi per i dati dei volumi Gluster dedicati a: oVirt Engine, altre vm oVirt, immagini ISO, locking/clustering CTDB/NFS-Ganesha, file sharing CIFS/Windows e file sharing Unix/NFS)
2. configurano ed avviano i servizi CTDB e Samba (a breve anche NFS-Ganesha e Gluster-block) basati su volumi Gluster (qui sta una prima _eresia_: usare lo storage iperconvergente anche per file sharing e non solo per la virtualizzazione)
3. effettuano l'installazione del Self Hosted Engine oVirt sul "Node 0" (analogo alla fase corrispondente disponibile tramite Cockpit)
4. configurano gli storage domain e le reti aggiuntive in oVirt
5. aggiungono al cluster oVirt i nodi rimanenti oltre il "Node 0"
6. configurano OVN sull'Engine e sui nodi (a breve creando anche un paio di reti interne isolate potenzialmente utili per motivi di sicurezza e/o di test)
7. creano virtual machine aggiuntive sull'infrastruttura oVirt (tutte da installare ex-novo tramite Kickstart dedicati analoghi ai precedenti; questi Kickstart sono ancora in fase di realizzazione):
    1. un domain controller Active Directory con creazione automatizzata di un dominio ex-novo (seconda _eresia_: CentOS7 con pacchetto Samba preso da Fedora&nbsp;e ricompilato per attivare le funzioni di DC usando le librerie interne Heimdal ecc.)
    2. un printer server membro del dominio Active Directory di cui sopra
    3. un database server membro del dominio Active Directory di cui sopra (CentOS7 con a scelta: PostgreSQL, MySQL, Firebird o, vera _eresia_&nbsp;;-) , SQLServer!)
    4. altre vm opzionali (desktop virtuali CentOS7 o Windows10, server gestionali che si appoggino al DB server di cui sopra, tutte membri del dominio Active Directory di cui sopra)
L'avvio (come utente root dal PC di supporto) dei passi qui sopra elencati dovrà essere interattivo (ad esempio con&nbsp;ansible-playbook /usr/local/etc/hvp-ansible/hvp.yaml&nbsp;), ma solo per dare modo a chi installa di scegliere il momento opportuno ed eventualmente apportare modifiche manuali preventive a parametri e passi: dopo l'avvio, tutto quanto avviene in maniera automatica.  
  
Quanto qui sopra elencato sopra a livello di automazione della configurazione finale è reso possibile unicamente dal notevole lavoro di supporto a oVirt incluso in Ansible 2.3 ed ai chiarissimi ed articolati esempi di playbook realizzati da Simone Tiraboschi.  
  
Tutto il software utilizzato è presente (se modificato) in appositi [repository yum][25] con relativi pacchetti sorgenti (tutti i pacchetti sono prodotti usando mock e firmati con la [chiave GPG del progetto][26]).  
  
Sottolineiamo il fatto che la presentazione ed i test compiuti finora hanno riguardato insiemi di **esattamente 3** **macchine server**, nel qual caso una delle 3, da installare come "Node 2" (ovvero quello indicato com "_l'ultimo server_" nelle specifiche hardware sopra citate), diverrà a livello Gluster per tutti i volumi un puro "**arbiter**" (ovvero non contiene/replica i dati dei file, ma solo i metadati), permettendo così di avere una macchina meno "carrozzata" delle altre due, quanto a dischi/CPU, senza influire sulle prestazioni globali; è però in previsione il supporto alla creazione iniziale (tramite modifica alle logiche di creazione del file gDeploy) di infrastrutture basate su **4 e più macchine server** e pure il supporto al più spinoso problema di espandere a 4 e più macchine server una infrastruttura inizialmente creata con 3 macchine server.  
  
Sottolineiamo anche che, come ennesima forma di provocazione/_eresia_, il nostro progetto utilizza (sempre prodotti e pubblicati come indicato sopra):  

* i **pacchetti Gluster** non community, bensì **ricompilati dai sorgenti di RHGS** (il Gluster con supporto a pagamento offerto da Red Hat) per motivi di estensione del ciclo di vita utile
* i **pacchetti Openvswitch ed OVN ricompilati da versioni Fedora** più recenti
  
  

###  Conclusione

  
A conclusione di questo riassunto (che, me ne scuso, probabilmente omette altro che è stato detto, di passaggio o in forma di domande/risposte salutandosi all'uscita) ricordo che abbiamo anche promesso di porre mano a breve&nbsp;ad alcuni aspetti, in ordine di importanza:  

1. **documentare quanto fatto**, non solo in forma di commenti interni agli script/Kickstart (la sezione documentazione del sito è al momento colpevolmente vuota, ma la gentilissima offerta da parte di Stefano Stagnaro di questo spazio ci permette di onorare, almeno in parte, la promessa, per di più in italiano, mentre come indicato all'incontro tutta la documentazione ed i commenti sono e saranno, a meno di offerte di aiuto esterne, sempre in inglese per motivi di generalità);
2. **estrarre la parte Ansible/gDeploy**&nbsp;dal file di Kickstart del PC di supporto e darle vita propria in un apposito repository Github;
3. investigare la fattibilità e l'interesse generale per un progetto di **ricompilazione **dei pacchetti oVirt non community, bensì presi dai sorgenti **di RHV** (l'oVirt con supporto a pagamento offerto da Red Hat) per motivi di estensione del ciclo di vita utile.
  
Concludo indicando che tra gli ulteriori sviluppi futuri (oltre a tutto quanto già sopra indicato ed annunciato durante la presentazione) c'è l'aggiunta di soluzioni integrate di:  

* **backup **(usando il software libero [Bareos][27], di nuovo da ricompilare in una versione mantenuta aggiornata);
* **monitoraggio **(accentramento dei log e/o integrazione con le soluzioni di "Metrics Store"&nbsp;[in corso di sviluppo in oVirt][28]);
* **spegnimento totale non presidiato** a seguito di eventi esterni (si pensi al caso di piccole realtà con UPS singoli, o anche centralizzati, ma senza la garanzia di alimentazione ininterrotta tramite generatori ecc.).
  
  
Ringraziamo nuovamente tutti i partecipanti alla serata e ringraziamo in particolare anche Stefano Stagnaro per averci offerto qui la possibilità di ricapitolare (e magari chiarire) il contenuto della presentazione per chi quella sera fosse stato (comprensibilmente) ostacolato nel seguire dalle mie scarse doti oratorie, dal caldo e dagli inconvenienti tecnici e anche per chi non avesse avuto modo di partecipare.  
  

Pubblicato da  [ Giuseppe Ragusa ][29] a  [02:02][30] [ ![][31] ][32]

[Invia tramite email][33][Postalo sul blog][34][Condividi su Twitter][35][Condividi su Facebook][36][Condividi su Pinterest][37]

[Post più vecchio][38] [Home page][2]

Iscriviti a: [Commenti sul post (Atom)][39]

## Prossimi eventi

28 ottobre 2017 ore 11.00 → 12.00

### [ Hyperconverged Infrastructure con oVirt+GlusterFS+Ansible][40]

_Università degli Studi di Milano, dip. Informatica, Aula 4_  
  
Realizzare un'infrastruttura iperconvergente dove virtualizzazione, networking e storage vengono erogati da hadrware general purpose, utilizzando software open source di livello enterprise. Verranno descritte le tecnologie coinvolte, le tecniche di deployment e i casi d'uso in produzione con l'ausilio di una demo.  
  

## [TUTTI GLI EVENTI →][41]

[ ![][6] ][42]

## In quali di queste zone vorresti che si tenesse il prossimo evento di oVirt Italia?

[ ![][6] ][43]

## Archivio blog

* [ ▼&nbsp; ][44] [ 2017 ][45] (3)
    * [ ▼&nbsp; ][44] [ settembre ][46] (1)
        * [Heretic oVirt Project - Il recap dell'incontro BgL...][47]
    * [ ►&nbsp; ][44] [ febbraio ][48] (1)
    * [ ►&nbsp; ][44] [ gennaio ][49] (1)
* [ ►&nbsp; ][44] [ 2016 ][50] (2)
    * [ ►&nbsp; ][44] [ novembre ][51] (1)
    * [ ►&nbsp; ][44] [ giugno ][52] (1)
* [ ►&nbsp; ][44] [ 2015 ][53] (4)
    * [ ►&nbsp; ][44] [ novembre ][54] (2)
    * [ ►&nbsp; ][44] [ ottobre ][55] (1)
    * [ ►&nbsp; ][44] [ settembre ][56] (1)

[ ![][6] ][57]

| -----|  
|  

  |  

  |  

| ----- |
| 

 | 

 | 

Tema Fantastico S.p.A.. Powered by [Blogger][58]. 

[ ![][6] ][59]

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
