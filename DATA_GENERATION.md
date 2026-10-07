# PaySim Data Generation

## Source
- Simulator: PaySim (Lopez-Rojas, Elmir & Axelsson, EMSS 2016), github.com/EdgarLopezPhD/PaySim, commit `d1feae8327ff2317771f3658f14d214cbf897dab`
- Dependency: MASON agent-based simulation library, github.com/eclab/mason, commit `08c95296de1b4fa73635e0c301ca9a272af8749e`, compiled from source (Maven repositories were unavailable)

## Modification
The 2019 drug-network typology extension was removed, because it requires Apache TinkerPop, which couldn't be installed. This affects only the optional drug-dealer/consumer actors. Core client, merchant, and fraudster behavior is unchanged, and this matches the original PaySim1 setup.

## Parameters
Defaults from `PaySim.properties`, except `seed=42` (default is `time`) for reproducibility:
- 720 hourly steps (~30 days)
- 20,000 clients, 1,000 fraudsters, 34,749 merchants, 5 banks
- fraudProbability = 0.001

## Reproducibility check
The simulator ran 5 repetitions with seed 42. All 5 transaction logs were byte-identical (308,987,268 bytes).

## Output
`paysim_transactions.csv.gz`: 3,410,988 transactions, 12 columns

| Column | Meaning |
|---|---|
| step | Hour of simulation (0–719) |
| action | CASH_IN, CASH_OUT, DEBIT, PAYMENT, TRANSFER |
| amount | Transaction amount |
| nameOrig / nameDest | Sender / recipient account IDs |
| oldBalanceOrig / newBalanceOrig | Sender balance before / after |
| oldBalanceDest / newBalanceDest | Recipient balance before / after |
| isFraud | Label |
| isFlaggedFraud | Simulator's built-in rule flag |
| isUnauthorizedOverdraft | Overdraft beyond limit |

## Known properties of this run (check before modeling)
- 1,368 fraud transactions (0.04%): 684 TRANSFER + 684 CASH_OUT, a takeover-then-cash-out pattern across 672 victim accounts
- Fraud occurs only in TRANSFER and CASH_OUT
- **Balance leak:** every fraudulent transaction empties the sender's account exactly (newBalanceOrig = 0 and oldBalanceOrig = amount); no legitimate transaction does
- **ID leak:** account IDs prefixed "CC" are fraudster-controlled mule accounts; every transaction originating from one is fraud
- **isFlaggedFraud never fires** (transferLimit is set very high), so the rules baseline must be written separately
- Fraud amounts are far larger than normal CASH_OUTs (fraud median ~3.6M vs. legitimate 99th percentile ~286K)
