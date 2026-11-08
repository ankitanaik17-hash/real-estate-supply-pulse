# real-estate-supply-pulse

## What it is
An automated weekly supply brief for a real estate listings marketplace:
listings, customers and subscription revenue, with week-on-week and
year-on-year comparisons. **All data is synthetic.** No real volumes, ratios
or definitions from any company are used.

## Why
A portfolio project showing how to keep a recurring business brief trustworthy:
every number comes from SQL against a documented data contract, and the
narrative layer only explains the figures, never calculates them.

## Architecture
_Placeholder. To be added once the pipeline exists._

## Data contract
Tables, metric definitions, aggregation and comparison rules, and planted
test cases are documented in [docs/data_contract.md](docs/data_contract.md).
Machine-readable definitions: `definitions/metric_definitions.yaml`.

## Quick start
_Placeholder. To be added once there is something to run._

## Roadmap
- [x] Data contract and metric definitions
- [ ] Synthetic data generator
- [ ] SQL metrics layer
- [ ] Anomaly detection
- [ ] Weekly brief (narrative on top of SQL figures)

## License
MIT. See [LICENSE](LICENSE).
