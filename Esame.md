Links:
https://tor.stackexchange.com/questions/423/what-are-good-explanations-for-relay-flags

# Tor: introduzione
La rete TOR è formata da circa 3000 relay. Un client Tor seleziona (almeno) 3 relay per formare un *circuito*, un insieme di stream TCP per comunicare con l'host destinatario.

TOR misura la larghezza di banda che ogni relay fornisce per assegnargli un peso. I pesi sono usati nella selezione per distribuire il carico verso relay con più banda disponibile.

La *directory authority* assegna una status flag ai relay. La flag ==GUARD== è assegnata ai relay il cui uptime è pari o superiore alla mediana dei relay familiari (A node is 'familiar’ if 1/8 of all active nodes have appeared more recently than it, OR it has been around for a few weeks) e la loro banda è pari o superiore a min(250KiB/s, median_relay_bandwidth).

I client selezionano 3 GUARD attive e le usano come relay di ingresso per tutti i loro circuiti. Queste vengono ruotata randomicamente ogni 30-60 giorni.

Un relay ottiene la flag ==STABLE== se il suo mean time between failure pesato è pari o superiore alla mediana dei relay attivi.

La flag ==EXIT== è assegnata a quei relay che possono comunicare con host esterni. Questi relay specificano una exit policy: range di indirizzi e porte con cui sono disponibili a connettersi.

I client usano queste policy per determinare il relay da scegliere alla fine di un circuito.

Tra le altre regole che un client *enforces*: non scegliere due relay con stesso prefisso di rete a 16 bit, o due relay appartenenti alla stessa famiglia (set of relays that mutually indicate that they belong to a group together)

# Adversary model and metrics
### Adversary model
Un avversario realistico è in grado di osservare, ritardare, alterare (aggiungere o rimuovere parti di) una comunicazione. 

Tuttavia in questa analisi ci si limita ad un avversario passivo, end-to-end correlating. Tale avversario apprende la sorgente o destinazione di una comunicazione quando è in posizione di osservare uno o entrambi di questi. Inoltre è in grado di collegare le osservazioni di questo flusso di comunicazione ovunque esse siano nel loro percorso.

Tale avversario può aggiungere risorse di rete o corrompere quelle esistenti; non può però alterare in alcun modo il traffico di rete.


Le risorse che l'avversario possiede sono i relay o il server di destinazione => in senso più astratto l'avversario controlla una certa larghezza di banda già esistente o che può aggiungere alla rete TOR.

### Adversary endowment
[IXP, AS ???]

### Adversary goal
L'obbiettivo principale (e più generale) dell'avversario è quello di deanonimizzare (collegare sorgente e destinazione) quanti più circuiti possibile. 

È possibile che i goal siano più specifici come identificare le destinazioni di specifiche classi di utenti, o compromettere circuiti in modo da individuare chi si connette a specifiche destinazioni.


### Security metrics
In generale tali metriche sono definite rispetto ad uno specifico tipo di avversario, sono usate per determinare la sicurezza (su tempi umani) e devono permettere la stima di probabilità per eventi rilevanti.

Per questa analisi sono usate le seguenti metriche:
- distribuzione di probabilità sul numero di path compromessi per un dato utente
- distribuzione di probabilità sul tempo fino al primo path compromesso.

# Metodologia
Per valutare la sicurezza di TOR rispetto ad averrsario e metriche proposte, si usa il metodo Monte Carlo per campionare il flusso di traffico sulla rete durante varie tipi di attività degli utenti.

Per ogni campione si usa un modello della rete, si simula il comportamento degli utenti e si simula il risultato.

TorPS è un simulatore di selezione dei path, costruito a partire dai dati storici della rete per ricreare le condizioni sotto le quali i client hanno operato in passato. TorPs include un modello dei client, relay e dei loro stati.

### Tor Network Model
TorPS usa i dati di Tor Metrics per modellare gli stati passati della rete TOR. Tali dati permettono di determinare lo stato dei relay nel tempo (flags, exit policies). I relay senza descrittori o che non appaiono in un *consensus* (documento che la directory authority invia ai client contenente informazioni su tutti i relay), risultano come inattivi


### User Model
I ricercatori hanno sviluppato 5 modelli utente, corrispondenti a tipi di uso diversi del client TOR. Ogni modello consiste di una sequenza di stream Tor e l'orario. Uno stream include chiamate DNS e connessioni TCP.

Tre di questi modelli sono costruiti tracciando l'uso di alcune applicazioni. Ogni traccia corrisponde a 20 minuti di utilizzo. 

- ==Typical==: rappresenta un uso medio di Tor. Usa 4 tracce: Gmail+Google Chat alle 9.00, Google Calendar + Docs alle 12.00, Facebook alle 15.00, and navigazione web a parite dalle 18.00. 
- ==IRC==: rappresenta l'uso di Tor allo scopo esclusivo di accedere a chat IRC. Usa la traccia di una singola sessione IRC, ripetuta dalle 8.00 alle 17.00 da lunedì a venerdì.
- ==BitTorrent==: rappresenta l'uso di BitTorrent su Tor. Consiste nel download di un singolo file. Il modello ripete la traccia dalle 12.00 alle 18:00, sabato e domenica.

- WorstPort. modifica il modello Typical rimpiazzando la porta con la 6523. Tale porta è la penultima in capacità in uscita. [TODO?]
- ==BestPort==: modifica il modello Typical rimpiazzando la porta con la 443 (HTTPS).


| Model      | Streams/wee | # IPs | # Ports |
| ---------- | ----------- | ----- | ------- |
| Typical    | 2632        | 205   | 2       |
| IRC        | 135         | 1     | 1       |
| BitTorrent | 6768        | 171   | 118     |
| WorstPort  | 2632        | 205   | 1       |
| BestPort   | 2635        | 205   | 1       |

### Tor Client Model
TorPS simula il client nella creazione di circuiti. Prende in considerazione caratteristiche quali peso dei relay, selezione e rotazione delle guardie, exit policy, famiglia e prefisso di rete.

Tuttavia essendo una simulazione, ogni costruzione di circuito ha successo. Non viene valutato il caso in cui questa fallisca o il circuito cada durante l'uso.

# Internet Map 
?????


# SECURITY ANALYSIS
Si considerano due tipi di avversari: uno in grado di eseguire relay su TOR, un altro è un ISP in grado osservare porzioni di rete TOR.


# Relay Adversary
Avversari in grado di eseguire/controllare relay è la minaccia più plausibile per gli utenti TOR, dato che chiunque può fornire un relay alla rete senza alcuna restrizione.

I client scelgono un relay in proporzione alla sua larghezza di banda, pertanto il traffico che un avversario può deanonimizzare è limitata solo dalla sua banda. 

Pertanto l'avversario in considerazione ha un banda significante (ma ragionevole) di 100MiB/s. 

Interessante notare che la banda non deve essere fornita da un singolo relay, per cui un avversario può fornirla tramite una botnet.

La banda ha un costo, per cui arrivare a controllare una grossa parte di TOR è estremamente difficile. Tuttavia la velocità continua ad aumentare, per cui il costo che un avversario affronta per arrivare a minacciare la rete continuerà a diminuire. 

### Adversary Resource Allocation
L'avversario deve determinare come allocare la sua banda per massimizzare la probabilità di compromettere circuiti.

Dato che un realy non può essere scelto due volte nelllo stesso circuiti, deve controllare almeno 2 relay per eseguire una correlazione.

Si suppone che l'avversario possieda un guard ed un exit relay con uscita verso ogni indirizzo e porta. Entrambi hanno abbastanza banda da ottenere la flag ==fast== 


# Network Attack

# Sniper Attack