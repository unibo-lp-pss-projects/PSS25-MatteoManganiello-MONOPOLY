# Analisi dell'uso di LLM nel progetto PSS25-MONOPOLY

- **Repository:** fork di `MatteoManganiello/PSS25-MONOPOLY` (branch `main`, HEAD `bfb3494`)
- **Autore del codice:** Matteo Manganiello (`matteo.manganiello@studio.unibo.it`)
- **Data dell'analisi:** 2026-10-21

Tutti i comandi vanno eseguiti dalla radice del repository. Gli hash dei commit
sono quelli della storia attuale del repository.

## Sintesi

Nel repository non ci sono tracce esplicite di strumenti di IA (trailer
`Co-Authored-By`, file di configurazione, citazioni di Claude, Copilot,
ChatGPT ecc.). La storia git contiene però diversi indizi concordanti di uno
sviluppo guidato da un LLM:

1. Nei primi commit compare un **piano di lavoro "Giorno 1…4"**, citato al
   futuro nel codice, che è stato poi **rimosso completamente**.
2. Un commit tocca **soltanto i commenti** e li riscrive in un registro più
   colloquiale, senza aggiungere contenuto.
3. Le ultime tracce del piano vengono eliminate da due commit consecutivi
   intitolati semplicemente `refactor`.
4. Il codice arriva in **blocchi molto grandi e già rifiniti**: circa 13.000
   righe in 9 giorni, con commit fino a ~2.500 righe.
5. I commenti hanno uno stile didattico ed esaustivo, e i messaggi di commit
   seguono il formato tipico degli LLM.
6. La relazione **non dichiara l'uso di strumenti di IA** e afferma che il
   lavoro è stato svolto "in autonomia".

Le prove sono indiziarie e nessuna, da sola, è conclusiva. Le evidenze 1–3,
considerate insieme, sono però difficilmente spiegabili in altro modo. Si
suggerisce una verifica orale sul codice, ad esempio chiedendo allo studente di
spiegare o modificare `TurnManager` o `SaveFileFormat`.

---

## Evidenza 0 — Assenza di tracce esplicite

Non c'è alcun riferimento esplicito a strumenti di IA, né nei messaggi di
commit né in alcuna versione dei file. Lo si riporta per completezza: l'assenza
di tracce esplicite non esclude l'uso di un LLM.

```bash
# Messaggi di commit che citano strumenti di IA (atteso: 0)
git log --all --format=%B | grep -ciE 'co-authored|claude|copilot|chatgpt|openai|gemini|codex|generated'

# File, in qualunque versione della storia, che citano strumenti di IA (atteso: 0)
git grep -liE 'claude|copilot|chatgpt|openai|gemini|codex|\bllm\b' $(git rev-list --all) | wc -l
```

---

## Evidenza 1 — Il piano "Giorno 1…4"

Nei primi commit il README e i commenti del codice fanno riferimento a un piano
di sviluppo numerato per giorni ("Giorno 1", "Giorno 2", …). È la struttura
tipica di un percorso guidato passo passo generato da un LLM.

### 1a. Checklist nel README del primo commit

```bash
git show c9df65d:README.md | grep -A6 'Stato di avanzamento'
```

Output:

```
## Stato di avanzamento

- [x] Giorno 1: setup del progetto e modello del dominio (Player, Token, Dice, Bank, Property, Tile, Board, GamePhase)
- [ ] Gestione dei turni (controller + GamePhase)
- [ ] Caselle speciali (imprevisti, probabilita', tasse, prigione)
- [ ] Interfaccia grafica
- [ ] Salvataggio della partita
```

### 1b. Il codice parla al futuro dei "giorni" successivi

I commenti annunciano il lavoro dei giorni successivi ("arriverà al Giorno 3",
"la GUI arriverà al Giorno 4") come se seguissero un piano già scritto. Uno
studente che prende appunti per sé difficilmente scrive commenti di questo tipo.

```bash
git log -p --reverse --format='commit %h' -- src \
  | grep -E '^commit|^\+.*(arriver|Giorno [0-9]: |futura casella|Dal GIORNO)' \
  | grep -B1 '^+'
```

Estratti dall'output:

```
commit c9df65d
+        System.out.println("=== Monopoly - Giorno 1: modello del dominio ===");
commit 96ac90e
+        // Tile sa da se' cosa fare. Le caselle vere arriveranno al Giorno 3.
+ * {@link monopoly.view.ConsoleGameObserver}; la GUI arrivera' al Giorno 4.
+        // Simula la futura casella "Vai in prigione" del Giorno 3.
commit 2635815
+     * chiedere. Quando al Giorno 4 arrivera' la GUI bastera' sostituire la chiamata
+ * E' il lato "model" del pattern Observer gia' usato al Giorno 2: le caselle, la
commit 702be83
+ * partita (controller) e le view che lo osservano. Dal GIORNO 4 la view principale e'
```

### 1c. I "giorni" non coincidono con il calendario

Il "Giorno 1" è il 10/09, i "Giorni 2–3" coincidono con l'11–12/09 e il
"Giorno 4" con il 14/09. I giorni del piano sono quindi tappe logiche del
percorso, non giorni effettivi di lavoro.

```bash
git log --reverse --format='%h %ad %s' --date=format:'%Y-%m-%d %H:%M' \
  -G 'Giorno|GIORNO' -- src README.md
```

### 1d. Nella versione consegnata non ne resta traccia

La storia contiene 394 righe con "Giorno" in 24 diverse revisioni, mentre la
versione finale ne contiene zero.

```bash
# Occorrenze nell'intera storia (atteso: 394)
git grep -n -E 'Giorno|GIORNO' $(git rev-list --all) -- . | wc -l

# Occorrenze nella versione consegnata (atteso: 0)
git grep -n -E 'Giorno|GIORNO' HEAD -- . | wc -l
```

---

## Evidenza 2 — Un commit che riscrive soltanto i commenti

Il commit `db3382d` ("refactor: improve code comments for clarity and
consistency", 17/09 alle 01:32) modifica 6 file con +163/−183 righe, ma **tutte
le righe modificate sono commenti**: il codice non cambia. Oltre a eliminare i
riferimenti ai "Giorni", la riscrittura non aggiunge contenuto: trasforma
commenti formali e curati in frasi volutamente colloquiali. Si può
ragionevolmente interpretare come un tentativo di far sembrare "scritti a mano"
commenti generati automaticamente.

```bash
# Statistiche del commit
git show db3382d --stat --format='%h %ad %s' --date=format:'%Y-%m-%d %H:%M'

# Righe modificate che NON sono commenti (atteso: 0)
git show db3382d --format= -U0 | grep -E '^[-+]' | grep -vE '^(\+\+\+|---)' \
  | grep -vE '^[-+]\s*(\*|/\*\*|//|\*/)' | wc -l

# Confronto parola per parola delle riformulazioni
git show db3382d --word-diff=color -- src/main/java/it/unibo/monopoly/MonopolyApp.java \
  src/main/java/it/unibo/monopoly/model/board/Board.java
```

Esempi di riformulazione:

| Prima | Dopo |
|---|---|
| `Punto di ingresso dell'applicazione.` | `Da qui parte tutto il programma.` |
| `Copia difensiva: la lista interna non puo' essere modificata da fuori.` | `Copiamo la lista, cosi' nessuno da fuori puo' cambiarla.` |
| `Aspetto di default: la partita si gioca lo stesso.` | `Pazienza, resta l'aspetto normale: si gioca lo stesso.` |
| `Classe di utilita': non deve essere istanziata.` | `Costruttore privato: questa classe ha solo metodi statici, non va creata.` |
| `Dal GIORNO 4 la view principale e' la finestra grafica` | `La view principale e' la finestra grafica` |

---

## Evidenza 3 — Rimozione sistematica delle ultime tracce

Dopo `db3382d`, i riferimenti residui ai "Giorni" spariscono in altri commit,
in particolare nei due commit `1bb56d0` e `1a2b58d`. Entrambi sono intitolati
soltanto `refactor` e risalgono al 18/09 alle 17:08, a un minuto da
`ff8aea0` (17:07). In `1a2b58d` vengono rimosse 10 righe con "Giorno", tra cui
`Nata al Giorno 1 per riempire il tabellone prima che esistessero le caselle vere`.

```bash
# Per ogni commit: righe con "Giorno" aggiunte (+) e rimosse (-)
for c in $(git log --format=%h -G 'Giorno|GIORNO' --reverse -- src README.md); do
  a=$(git show $c --format= | grep -cE '^\+.*(Giorno|GIORNO)')
  r=$(git show $c --format= | grep -cE '^-.*(Giorno|GIORNO)')
  echo "$c +$a -$r $(git log -1 --format='%ad %s' --date=format:'%m-%d %H:%M' $c)"
done

# Dettaglio delle righe rimosse nei due commit "refactor"
git show 1bb56d0 1a2b58d --format='=== %h %s' -U0 | grep -E '^===|^-.*(Giorno|GIORNO)'
```

Output del primo comando:

```
c9df65d +6 -0   feat: implement initial MVC structure and core Monopoly domain
883341c +0 -1   docs: simplify README instructions
96ac90e +11 -3  Implement core game state and turn mechanics
2635815 +9 -6   feat: add JailManager, event support and tests
702be83 +10 -3  feat(gui): add components for player info and game display
b35d585 +0 -3   feat(gui): update game state handling and UI refresh logic ...
87fb00c +1 -0   Add game state save and load functionality
db3382d +0 -4   refactor: improve code comments for clarity and consistency
0146b66 +0 -1   feat: implement property purchase decision flow
ff8aea0 +0 -5   feat: update README
1bb56d0 +0 -1   refactor
1a2b58d +0 -10  refactor
```

L'ultimo commit che aggiunge riferimenti ai "Giorni" è del 15/09. Da quel
momento in poi i commit li rimuovono soltanto.

---

## Evidenza 4 — Volume e granularità dei commit

Il progetto conta 13.026 righe di Java in 88 file, con 153 test. Quasi tutto il
codice è stato scritto tra il 10 e il 18 settembre (9 giorni), in commit molto
grandi e già completi di Javadoc e test. Ad esempio:

- `87fb00c`: 1.982 righe aggiunte, 37 minuti dopo il commit precedente;
- `4fa8abe`: 1.141 righe aggiunte, 40 minuti dopo il commit precedente;
- `2635815`: 2.445 righe aggiunte, tutte in un unico commit.

```bash
# Righe aggiunte in src/ per ogni commit, con data e ora
git log --no-merges --reverse --format='%h %ad%x09%s' --date=format:'%Y-%m-%d %H:%M' \
  --shortstat -- src \
  | awk '/^[0-9a-f]{7} /{h=$0; next}
         /changed/{ins=0; for(i=1;i<=NF;i++) if($(i+1) ~ /^insertion/) ins=$i;
                   printf "%6d  %s\n", ins, h}'

# Dimensione del progetto: righe Java, file Java, numero di test
git ls-files 'src/*.java' | xargs cat | wc -l
git ls-files 'src/*.java' | wc -l
grep -rc '@Test' src/test | awk -F: '{s+=$2} END{print s}'
```

---

## Evidenza 5 — Stile dei commenti e dei messaggi di commit

### 5a. Commenti didattici ed esaustivi

Il Javadoc è lungo e spiega in tono da manuale il motivo di ogni scelta di
design. Ad esempio, `TurnManager` dedica un paragrafo a spiegare perché la classe
"sta nel model, e non nel controller". Anche i file di configurazione sono
commentati riga per riga (`.gitignore`, `build.gradle.kts`) e il `main`
contiene una spiegazione numerata "1. Model / 2. View / 3. Controller".

```bash
sed -n '1,40p' src/main/java/it/unibo/monopoly/model/game/TurnManager.java
cat .gitignore
git show c9df65d:build.gradle.kts | head -40
```

### 5b. Messaggi di commit strutturati

I messaggi dei commit principali sono elenchi puntati e raggruppati per
categoria ("Architecture & Setup:", "Core Domain Models:", "Testing:"),
il formato tipico generato dagli LLM. Il messaggio di `cc73f8c` è racchiuso tra
backtick (`` `refactor: move all packages under it.unibo.monopoly` ``), segno
che è stato copiato da un testo in formato Markdown.

```bash
git show -s --format='--- %h%n%B' c9df65d 96ac90e 2635815 87fb00c
git log -1 --format=%s cc73f8c
```

---

## Evidenza 6 — Nessuna dichiarazione nella relazione

La relazione (`report.pdf`) non menziona l'uso di strumenti di IA. Nella
sezione 4.2 afferma: *"Nel complesso, affrontare in autonomia tutte le fasi del
progetto (analisi, design, sviluppo, test) è risultato impegnativo ma
formativo"*.

```bash
# Ricerca di riferimenti all'IA nel testo della relazione (atteso: nessun risultato pertinente)
pdftotext report.pdf - | grep -niE 'intelligenza artificiale|\bIA\b|\bAI\b|LLM|chatgpt|claude|copilot|gemini'

# Frase sullo svolgimento "in autonomia"
pdftotext report.pdf - | grep -n -A2 'in autonomia'
```

---

## Elementi a discarico

- Le prove sono indiziarie: lo stile dei commenti e la grandezza dei commit, da
  soli, non dimostrano nulla.
- Gli accenti sono scritti come apostrofi (`e'`, `cosi'`, `puo'`), un'abitudine
  comune a chi usa una tastiera non italiana. Un LLM normalmente usa le lettere
  accentate, a meno che non gli venga chiesto diversamente. Questo dettaglio
  potrebbe indicare un intervento manuale, oppure un'istruzione esplicita data
  al modello.
- Il codice potrebbe essere stato sviluppato altrove e caricato in blocchi, il
  che spiegherebbe in parte la grandezza dei commit. Non spiegherebbe però i
  riferimenti al futuro ("arriverà al Giorno 4") né la loro successiva
  rimozione.
