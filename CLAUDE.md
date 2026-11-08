# real-estate-supply-pulse

Automated weekly supply brief for a real estate listings marketplace.
Portfolio project. ALL DATA IS SYNTHETIC. Never add real volumes, ratios,
seasonality, internal table names or metric definitions from any employer.

## Rules
- Every number in the brief comes from SQL. Claude writes narrative only:
  never calculate, estimate, round differently or invent figures.
- Ratios are never stored. Store numerator and denominator; compute the
  ratio only after roll-up.
- Source of truth for definitions: `docs/data_contract.md` and
  `definitions/metric_definitions.yaml`. If they disagree, stop and flag it.
- Complete periods only in any comparison. A partial week or month never
  feeds a comparison.
- Churn lags real contract end by ~30 days (grace period); the brief must say so.
- Do not create files outside the agreed phase scope.
