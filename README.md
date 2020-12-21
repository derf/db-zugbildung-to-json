# db-wagenreihung-to-json - Convert wagon order PDFs to JSON

db-wagenreihung-to-json converts a train composition (“Zugbildungsplan”) PDF
obtained from
[data.deutschebahn.com](https://data.deutschebahn.com/dataset/zugbildungsplanzugbildungsplan-zpar)
to JSON. At the moment, conversion is limited to a map of train numbers to
estimated IC/ICE types (ICE 1, ICE 3, ICE 3 Redesign, ...), partial train
composition / wagon order data, and cycles ("Umlaufpläne").

The PDF-to-JSON conversion is somewhat fragile, so errors are expected. If you
find a bug or inconsistency in the JSON file, please first compare it with the
corresponding PDF on
[data.deutschebahn.com](https://data.deutschebahn.com/dataset/zugbildungsplanzugbildungsplan-zpar).
If it is indeed a bug in db-wagenreihung-to-json, please [open an issue on
GitHub](https://github.com/derf/db-wagenreihung-to-json/issues/new).

The latest JSON produced by db-wagenreihung-to-json is available online at
[dbdb/zugbildungsplan\_v0.json](https://lib.finalrewind.org/dbdb/zugbildungsplan_v0.json).

## Data Format

This README documents **version 0** of the format.

```js
{
	"deprecated": false,
	"source": "2021_ZpAR_Wi_Endstück.pdf",
	"train": {
		"4": { /* train details */ },
		"5": { /* train details */ }
		// ...
	},
	"valid": {
		"from": "YYYY-MM-DD",
		"through": "YYYY-MM-DD"
	}
}
```
