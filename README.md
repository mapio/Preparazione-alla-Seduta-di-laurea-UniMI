# Script per la sessione di laurea

Due script per automatizzare il flusso: recupero della mail con le istruzioni
inviata dalla segreteria → generazione del foglio di calcolo con i dati degli
studenti da usare in seduta.

## Installazione

Serve [uv](https://docs.astral.sh/uv/) installato: `salva` e `converti` sono
script standalone (shebang `uv run --script`) che dichiarano da soli la
versione di Python richiesta (>= 3.11) e le proprie dipendenze
(`pdfplumber`, `openpyxl`); uv scarica/installa tutto automaticamente al
primo avvio, non serve creare un virtualenv a mano.

## Configurazione

Prima di usare `salva`, crea in questa cartella un file `.env` (permessi
`600`, dato che contiene una password in chiaro) con:

```
IMAP_HOST=imap.di.unimi.it
IMAP_USER=tuonomeutente
IMAP_PASSWORD=tuapassword
```

## `salva`

Cerca nella casella IMAP i messaggi di un determinato mittente, mostra un
elenco numerato compatto e chiede quale messaggio salvare (allegati inclusi).

```
./salva --from "Luigi Pepe" --out ./istruzioni_seduta_laurea
```

Esempio di output:

```
[ 1] 2026-06-16  Luigi Pepe             0 att  R: Sessione di luglio
[ 2] 2026-07-08  Luigi Pepe             8 att  Istruzioni seduta di laurea...

Save attachments of which message? [1-2, Enter to cancel]: 2
Saved: ./istruzioni_seduta_laurea/F1X presidente RIGHINI.zip
...
```

Premere Invio senza scrivere nulla annulla senza salvare.

## `converti`

Genera un foglio Excel a partire dagli allegati scaricati con `salva`:

- il PDF **Registro** (elenco studenti, matricole, titoli tesi, relatori);
- lo **ZIP del presidente** (un PDF "libretto" per studente, con media
  ponderata e lodi; talvolta contiene anche un certificato di incremento
  voto per mobilità Erasmus).

Il Registro viene individuato per **contenuto**, non per nome del file: la
segreteria lo chiama "Registro" per alcuni corsi e "Verbale" per altri, ma
il documento comincia sempre con l'intestazione "REGISTRO N. ..."; `converti`
analizza tutti i PDF nella cartella (esclusi quelli nello ZIP) e sceglie
quello che la contiene. Allo stesso modo, ogni PDF dentro lo ZIP viene
classificato leggendone il testo (libretto voti vs. certificato Erasmus),
non dal nome. Lo ZIP stesso è individuato per nome (l'unico `*.zip` nella
cartella). Se l'auto-discovery del Registro o dello ZIP è ambigua (più
candidati trovati), vanno passati esplicitamente con `--registro` / `--zip`.

```
./converti ./istruzioni_seduta_laurea
```

Il file `.xlsx` generato (default `studenti_<codice>_<data>.xlsx` nella
stessa cartella) contiene, una riga per studente:

| Colonna | Contenuto |
|---|---|
| Cognome, Nome | dal Registro |
| Titolo Tesi | dal Registro, in Title Case |
| Relatore/i | dal Registro |
| Media (/30), Media (/110) | dal libretto dello studente |
| N. Lodi | conteggiate dal libretto |
| Punti Erasmus | dal certificato di incremento voto, se presente (altrimenti 0) |
| Voto Finale | **da compilare a mano** durante la seduta |
| Punti Commissione | formula = Voto Finale − Media (/110), vuota finché Voto Finale non è compilato |

`Voto Finale` è l'unico dato che la commissione decide in seduta: è il voto
di laurea che viene assegnato allo studente, e va scritto a mano nella
cella evidenziata in giallo. `Punti Commissione` è calcolato "a ritroso" a
partire da quello, non il contrario: mostra quanti punti la commissione ha
aggiunto rispetto alla media ponderata (`Voto Finale − Media (/110)`), utile
per verificare a colpo d'occhio quanto ci si è discostati dalla media prima
di eventuali lodi/Erasmus. Resta vuoto finché `Voto Finale` non è compilato.

## Flusso tipico

```
./salva --from "Nome Cognome segreteria" --out ./istruzioni_seduta_laurea
./converti ./istruzioni_seduta_laurea
```
