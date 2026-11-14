# Previsioni Calcio — feed del turno

Un file al giorno, `turno_corrente.json`, con le partite del giorno e le
probabilità 1X2 di un modello statistico (Poisson/Dixon-Coles su gol e
xG, con Elo e calibrazione). Le versioni dei giorni passati stanno in
`storico/`.

Il file viene riscritto ogni mattina da un processo automatico, **anche
quando qualcosa va storto**: in quel caso `stato` vale `errore` e il
motivo è in `motivo_errore`. Prima di usarlo, controllare sempre
`giorno_operativo`, `stato` e il blocco `freschezza`.

## Cosa contiene e cosa no

- **Contiene** le probabilità del modello, la quota d'ingresso che le
  renderebbe interessanti (`quota_ingresso_richiesta`), la qualità dei
  dati di ogni partita e i flag.
- **Non contiene quote di bookmaker** né il vantaggio calcolato su di esse.

## Stato del modello

Il modello **non batte il mercato**: sul test storico è dietro alle quote
di chiusura, e un backtest di settembre 2026 non ha trovato nessuna
nicchia con CLV positivo. Il feed è quindi in **modalità shadow**
(`regole.modalita`): serve a misurare il modello su prezzi reali, non a
scommettere. Nulla di quanto è qui è un consiglio di gioco.

## I campi principali (schema 1.1, dal 28 ottobre 2026; `regole.decide`, `ruolo_modello` e `senza_modello` dal 15 novembre 2026)

| Campo | Significato |
|---|---|
| `giorno_operativo` | il giorno a cui si riferisce il turno: se non è oggi, il file è vecchio |
| `generato_il`, `avviato_il` | quando il giro è finito e quando è partito (UTC) |
| `durata_secondi` | quanto è durato il giro |
| `innesco` | chi l'ha lanciato: `workflow_dispatch` = giro puntuale; `schedule` = riserva di GitHub, che arriva in ritardo |
| `stato` | `ok`; `degradato` (una fonte non va: i `candidati` restano vuoti); `errore` (il turno non vale) |
| `motivi_degrado` | una riga leggibile per ogni fonte che non va |
| `freschezza` | una voce per fonte, con `ok` e, quando serve, `motivo` e `uso` |
| `regole` | le regole in vigore: `versione_regole`, `sconto_modello`, `soglia_pp`, `stake_fisso`, `stake_max_pick`, `esposizione_max_giorno` (in decimi di unità), `combo_ammesse`, `mercati_misurabili`, `revisione` |
| `stato_registro` | a che punto è il registro privato delle pick, **solo conteggi e date**: pick giocate, contatore della revisione (`pick_regime_v21_misurate` su `soglia_revisione`), `ultimo_giro_clv` e `giro_clv_ok` (falso se il calcolo settimanale non gira da più di 8 giorni), `candidati_ultimi_7_giorni` e `candidati_con_decisione`. Non contiene partite, quote, stake né risultati, e non entra in `stato` |
| `regole.decide` | chi stabilisce le probabilità delle pick ufficiali: `analyst` |
| `regole.ruolo_modello` | cosa fa il modello in questa fase |
| `regole.senza_modello` | le regole per i campionati senza modello (Brasile, Argentina): sconto, soglia, quota massima, massimo pick al giorno |
| `regole.formula_quota_ingresso` | come si calcola la quota d'ingresso: una formula per le pick ufficiali (stima dell'Analyst) e una per lo shadow del modello |
| `partite[].quota_ingresso_richiesta` | la quota d'ingresso **del modello in shadow**, non quella della pick ufficiale: `1 / (p_modello × sconto_modello − soglia_pp)`, arrotondata per eccesso al centesimo |
| `partite[].assenze` | gli assenti noti (`fuori` o `in dubbio`, con il motivo), per il controllo a mano |
| `partite[].qualita.assenze_alimentate` | sempre `false`: il modello non usa le assenze |
| `partite[].qualita.assenze_elencate` | `true` se l'elenco degli assenti è arrivato |
| `partite[].qualita.xg_stagione_corrente` | sempre `false`: gli xG entrano dalle stagioni concluse, la stagione in corso entra coi gol |
| `partite[].misurabile_clv` | sui candidati: l'esito ha una chiusura pubblica per misurare il CLV |
| `partite[].sospeso_per_degrado` | l'esito avrebbe passato i filtri, ma oggi una fonte non va |
| `partite_senza_modello` | partite del giorno che il modello non copre: Brasile e Argentina (`misurabile: "solo 1X2"`); coppe europee e altri campionati (`misurabile: "nessuno"`, flag `COMPETIZIONE NON COPERTA`) |

### Il blocco `freschezza`

- `stagione`: la stagione del modello contro quella del calendario.
- `football_data`: risultati; non va se una partita già giocata da più
  di 4 giorni non ha il risultato (`per_lega` dice dove). Una sosta non
  è un guasto.
- `understat_xg`: gli xG delle stagioni concluse. `ultima_stagione` è la
  precedente per scelta (`uso`).
- `api_football_odds`: le quote del giorno (restano private).
- `api_football_injuries`: gli assenti; `chiamate_ok`, `chiamate_fallite`,
  `righe_salvate`, `fixture_con_assenze`.
- `transfermarkt`: i valori di rosa della stagione precedente, usati
  come prior per scelta; `foto_giorni` dice quanto è vecchia la foto.
