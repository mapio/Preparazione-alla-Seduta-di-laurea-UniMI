# Script per la sessione di laurea

Due script per automatizzare il flusso: recupero della mail con le istruzioni
inviata dalla segreteria → generazione del foglio di calcolo con i dati degli
studenti da usare in seduta.

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
  ponderata e lodi).

I due file vengono individuati automaticamente nella cartella indicata; se
ce n'è più di uno vanno passati esplicitamente con `--zip` / `--registro`.

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
| Voto Finale | **da compilare a mano** durante la seduta |
| Punti Commissione | formula = Voto Finale − Media (/110), vuota finché Voto Finale non è compilato |

## Flusso tipico

```
./salva --from "Nome Cognome segreteria" --out ./istruzioni_seduta_laurea
./converti ./istruzioni_seduta_laurea
```
