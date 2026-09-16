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
