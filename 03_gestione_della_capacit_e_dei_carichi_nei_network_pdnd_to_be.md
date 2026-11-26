# 03 - Gestione della Capacità e dei Carichi nei Network PDND (TO-BE)

## 1. Analisi del Problema: L'Asimmetria Infrastrutturale e il Vincolo di Atomicità nel Modello Many-to-Many

Nel modello bilaterale tradizionale della **Piattaforma Digitale Nazionale Dati (PDND)** (*1 Erogatore ↔ 1 Fruitore*), la sostenibilità infrastrutturale è governata tramite negoziazione puntuale: l'**Erogatore** dichiara nel **Descrittore** dell'**e-Service** una capacità massima giornaliera ($C_{max}$) e per ciascun aderente. Per le singole **Finalità (Purpose)**, se le capacità indicate sono rispettate non è richiesta alcuna azione all'**Erogatore**; viceversa, l'**Erogatore** verifica la possibilità di accettare la richiesta lasciando in sospeso la **Finalità** (approccio **cautelativo per l'Erogatore**).

In un ecosistema **Network** (*Many-to-Many*), caratterizzato da adesioni automatizzate, decine o centinaia di **Aderenti** interconnessi e dalla necessità di garantire l'**attivazione atomica (*All-or-Nothing*)** delle **Finalità** su tutte le istanze di e-Service del Network, tale modello presenta tre criticità strutturali:

1. **Asimmetria dimensionale degli Aderenti:** Soggetti che ricoprono il medesimo **Ruolo di Network** (es. tutti i Comuni italiani) dispongono di capacità infrastrutturali e bacini d'utenza profondamente eterogenei (es. Roma Capitale rispetto a un Comune di 500 abitanti).
2. **Saturazione silenziosa (*Overbooking*) e barriere all'ingresso:** Imporre una capacità fissa uniforme lato **Erogatore** escluderebbe i piccoli enti privi di infrastrutture ad alta capacità, mentre lasciare ai singoli **Erogatori** la discrezionalità di dichiarare capacità arbitrarie al ribasso causerebbe saturazioni locali, compromettendo l'attivazione simultanea (*All-or-Nothing*) su tutto il Network.
3. **Inadeguatezza delle soglie statiche uniformi lato Fruitore:** Il **Governatore del Network** non può fissare un unico valore assoluto di consumo nei template per tutti i **Fruitori**, poiché il volume di traffico generato dipende sia dalla dimensione del richiedente sia dal bacino dati dell'**Erogatore** interrogato.

## 2. Modello Risolutivo a Tre Livelli (Doppio Tiering Speculare)

Per garantire l'attivazione istantanea e atomica (*All-or-Nothing*) delle **Finalità** su tutte le istanze di **e-Service** collegate, eliminando la discrezionalità dei singoli **Erogatori** senza creare barriere all'ingresso per gli enti di minori dimensioni, l'architettura introduce un motore di **Capacity Management Deterministico a Doppio Tiering** articolato su tre livelli:

### 2.1 Tiering Capacitivo sui Template di e-Service ($C_{net, j}$)

La capacità delle istanze di **e-Service** non viene impostata manualmente dall'**Erogatore** né fissata come valore unico uguale per tutti, bensì determinata automaticamente tramite una **tabella di parametrizzazione (*Tiering*)** definita dal **Governatore del Network** e ancorata agli **Attributi Certificati** dell'Erogatore $j$ (es. fascia demografica IPA/ISTAT, numero di dipendenti, posti letto ASL):

| Classe Dimensionale Erogatore ($Tier_E$) | Criterio Attributo Certificato (es. Popolazione) | Capacità Vincolante Ereditata ($C_{net, j}$) |
| :--- | :--- | :--- |
| **Tier E-S (Small)** | $< 5.000$ abitanti | $2.000$ chiamate/giorno |
| **Tier E-M (Medium)** | $5.000 - 50.000$ abitanti | $15.000$ chiamate/giorno |
| **Tier E-L (Large)** | $50.001 - 250.000$ abitanti | $75.000$ chiamate/giorno |
| **Tier E-XL (Enterprise/Metro)** | $> 250.000$ abitanti | $300.000$ chiamate/giorno |

In fase di istanziazione dell'**e-Service**, PDND risolve il $Tier_E$ dell'ente e popola **in sola lettura** il parametro $C_{net, j}$ nel **Descrittore**, senza possibilità di modifica da parte dell'Erogatore.

> OPEN POINT: La **tabella di parametrizzazione (*Tiering*)** è definite indipendentemente dal template e-service (**one-for-network**) o in relazione ai template e-service (**one-for-template-eservice**). Vintra: personalmente sono per **one-for-template-eservice** in quanto aumenta i gradi di liberta dei **Network**. 

### 2.2 Stima Parametrica dei Carichi sui Template di Finalità ($S_{i,j}$)

In modo speculare, il **Template di Finalità** non contiene un valore numerico statico, ma associa al **Fruitore** $i$ una stima di traffico proporzionata ai propri **Attributi Certificati** ($Tier_F$) e modulata rispetto alla classe dimensionale dell'**Erogatore** ($Tier_E$).

Il modello prevede **due modalità alternative** configurabili dal Governatore del Network per determinare il carico allocato $S_{i,j}$ sulla singola relazione bilaterale ($Fruitore_i \rightarrow Erogatore_j$):

#### Opzione A: Matrice Bidimensionale di Carico ($Tier_F \times Tier_E$)

Il **Template di Finalità** contiene una matrice di incrocio che assegna direttamente la stima esatta $S_{i,j}$ per ogni combinazione tra classe del **Fruitore** e classe dell'**Erogatore**, consentendo una calibrazione granulare cella per cella:

| Classe Fruitore ($Tier_F$) \ Classe Erogatore ($Tier_E$) | **Tier E-S** ($< 5k$) | **Tier E-M** ($5k - 50k$) | **Tier E-L** ($50k - 250k$) | **Tier E-XL** ($> 250k$) |
| :--- | :---: | :---: | :---: | :---: |
| **Tier F-S (Small)** ($< 5.000$ ab.) | $5$ ch./gg | $20$ ch./gg | $50$ ch./gg | $100$ ch./gg |
| **Tier F-M (Medium)** ($5.000 - 50.000$ ab.) | $15$ ch./gg | $150$ ch./gg | $500$ ch./gg | $1.000$ ch./gg |
| **Tier F-L (Large)** ($50.001 - 250.000$ ab.) | $30$ ch./gg | $500$ ch./gg | $2.500$ ch./gg | $5.000$ ch./gg |
| **Tier F-XL (Enterprise/Metro)** ($> 250.000$ ab.) | $50$ ch./gg | $2.000$ ch./gg | $10.000$ ch./gg | $20.000$ ch./gg |

#### Opzione B: Tabella Monodimensionale con Coefficiente di Peso ($\gamma_j$)

Il **Template di Finalità** mantiene una tabella monodimensionale semplice con la **Stima Base ($S_i$)** del **Fruitore**, mentre ciascun *Tier* dell'Erogatore introduce un **Coefficiente di Attrattività/Peso ($\gamma_j \in (0, 1]$)** che ridimensiona proporzionalmente il traffico atteso verso enti più piccoli ($S_{i,j} = S_i \times \gamma_j$):

* **Stima Base Fruitore ($S_i$):** **Tier F-S** = $100$ | **Tier F-M** = $1.000$ | **Tier F-L** = $5.000$ | **Tier F-XL** = $20.000$ chiamate/giorno.
* **Coefficiente Erogatore ($\gamma_j$):** **Tier E-S** ($\gamma = 0,0025$) | **Tier E-M** ($\gamma = 0,10$) | **Tier E-L** ($\gamma = 0,50$) | **Tier E-XL** ($\gamma = 1,00$).
* *Esempio di calcolo:* Un Fruitore **Tier F-XL** ($S_i = 20.000$) che si collega a un Erogatore **Tier E-S** ($\gamma_j = 0,0025$) alloca una stima $S_{i,j} = 20.000 \times 0,0025 = 50$ chiamate/giorno; verso un **Tier E-XL** ($\gamma_j = 1,00$) alloca $20.000$ chiamate/giorno.

> OPEN POINT: Per i **Coefficiente di Attrattività/Peso** vale lo stesso dubbio espresso per la **tabella di parametrizzazione (*Tiering*)**, fatto salvo la dipendenza o meno ai template finalità.

### 2.3 Attivazione Atomica (*All-or-Nothing*) e Governance Dinamica dei Volumi

Poiché in un ecosistema aperto il **Governatore del Network** non può conoscere a priori il numero esatto e la distribuzione futura dei **Fruitori** (perimetro dinamico), il dimensionamento iniziale delle tabelle di *Tiering* (livelli 2.1 e 2.2) rappresenta una **stima di esercizio ordinaria**. PDND Core governa a runtime la crescita effettiva dei carichi ($C_{allocata, j} = \sum_i S_{i,j}$) attraverso tre fasce operative:

* **Regime Ordinario ($C_{allocata, j} \le \alpha \times C_{net, j}$ per ogni istanza $j$):** Al momento della istanziazione del **Template di Finalità** da parte di un nuovo **Fruitore**, PDND risolve i *Tier* e verifica la capienza complessiva. Poiché tutte le istanze di **e-Service** dispongono di margine sufficiente, le **Finalità** vengono attivate **simultaneamente e automaticamente** su tutto il Network (*All-or-Nothing*).
* **Soglia di Guardia ($\alpha \times C_{net, j} < C_{allocata, j} \le C_{net, j}$, es. $\alpha = 80\%$):** L'attivazione resta istantanea e atomica per non interrompere l'attivazione delle fruizioni, ma PDND Core invia un **Alert Preventivo** al **Governatore del Network** affinché possa intervenire proattivamente prima di raggiungere la saturazione.
* **Saturazione di Perimetro ($C_{allocata, j} > C_{net, j}$ anche su una sola istanza $j$):** Qualora l'espansione imprevista del Network porti la domanda aggregata a superare la capacità vincolante di anche una sola classe di Erogatori, il sistema applica il **blocco atomico di salvaguardia**: l'attivazione della **Finalità** viene sospesa e accodata per il nuovo **Fruitore** sull'intero Network (evitando attivazioni parziali tra sottoinsiemi di Erogatori) e viene inviato un **Alert Critico** al **Governatore**.

#### Protocollo di Ricalibrazione e Adeguamento Infrastrutturale

Al superamento della Soglia di Guardia o in caso di blocco atomico per saturazione, il **Governatore del Network** dispone di due leve correttive:

1. **Ricalibrazione delle Stime ($S_{i,j}$) senza impatto sugli Erogatori:** Qualora la telemetria dei consumi reali dimostri che le stime teoriche sono sovradimensionate rispetto al traffico effettivo, il Governatore riduce i parametri $S_{i,j}$ (o i coefficienti $\gamma_j$) sul **Template di Finalità**. L'operazione libera immediatamente capacità virtuale ($C_{allocata, j}$) e sblocca le eventuali richieste in coda senza richiedere alcun intervento tecnico agli Erogatori.
2. **Innalzamento di $C_{net, j}$ con Finestra di Adeguamento (*Grace Period*):** Qualora sia necessario incrementare effettivamente la capacità erogata, il Governatore aggiorna la tabella $C_{net, j}$ sul **Template di e-Service** fissando una **data di entrata in vigore differita** (finestra di adeguamento). Durante tale periodo:
   * Gli **Erogatori** impattati ricevono notifica formale e adeguano la propria infrastruttura, aggiornando la propria istanza al nuovo SLA entro la scadenza (oppure esercitando la facoltà di recesso dal ruolo di Erogatore nel Network qualora non possano sostenere il nuovo carico).
   * Allo scadere della finestra di adeguamento, una volta allineate tutte le istanze attive nel Network, PDND Core sblocca automaticamente e in modo atomico le richieste dei **Fruitori** rimaste in coda.

## 3. Matrice delle Responsabilità sui Volumi (RACI)

*Legenda: **R** = Responsible (Esegue), **A** = Accountable (Approva/Responsabile finale), **C** = Consulted (Consultato), **I** = Informed (Informato).*

| Attività / Parametro | Governatore del Network | Erogatore | Fruitore | PDND Core |
| :--- | :---: | :---: | :---: | :---: |
| Definizione Tabelle di *Tiering* Capacitivo ($C_{net, j}$) sui Template di e-Service | **R / A** | I | I | C |
| Definizione Regole di Carico ($S_{i,j}$ via Matrice o Coefficienti $\gamma_j$) sui Template di Finalità | **R / A** | I | I | C |
| Definizione Soglia di Guardia ($\alpha$) per Alert di Governance | **R / A** | - | - | C |
| Risoluzione automatica $Tier_E$ ed ereditarietà in sola lettura di $C_{net, j}$ | - | I | - | **R / A** |
| Risoluzione automatica $Tier_F$ e calcolo allocazione stime ($\sum_i S_{i,j}$) | - | I | I | **R / A** |
| Attivazione atomica (*All-or-Nothing*) delle Finalità in Regime Ordinario e Soglia di Guardia | - | I | I | **R / A** |
| Applicazione del blocco atomico e accodamento richieste in caso di Saturazione di Perimetro | I | - | I | **R / A** |
| Invio Alert Preventivi (Soglia di Guardia) e Critici (Saturazione) | I | - | - | **R / A** |
| Ricalibrazione centralizzata dei Template ($S_{i,j}$ o $C_{net, j}$ con *Grace Period*) | **R / A** | I | I | C |
| Adeguamento infrastrutturale e aggiornamento istanza e-Service entro la *Grace Period* | I | **R / A** | - | C |
| Sblocco automatico delle richieste in coda al termine della finestra di adeguamento | I | I | I | **R / A** |