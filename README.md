# db-zugbildung-to-json - Convert wagon order PDFs to JSON

db-zugbildung-to-json converts a train composition (“Zugbildungsplan”) PDF
obtained from
[data.deutschebahn.com](https://data.deutschebahn.com/dataset/zugbildungsplanzugbildungsplan-zpar)
to JSON. At the moment, conversion is limited to a map of train numbers to
estimated IC/ICE types (ICE 1, ICE 3, ICE 3 Redesign, ...), partial train
composition / wagon order data, and cycles ("Umlaufpläne").

The PDF-to-JSON conversion is somewhat fragile, so errors are expected. If you
find a bug or inconsistency in the JSON file, please first compare it with the
corresponding PDF on
[data.deutschebahn.com](https://data.deutschebahn.com/dataset/zugbildungsplanzugbildungsplan-zpar).
If it is indeed a bug in db-zugbildung-to-json, please [open an issue on
GitHub](https://github.com/derf/db-zugbildung-to-json/issues/new). The format
specification (see below) details how trustworthy the JSON content is.

## Online Services

The latest JSON produced by db-zugbildung-to-json is available online at
[dbdb/db\_zugbildung\_v0.json](https://lib.finalrewind.org/dbdb/db_zugbildung_v0.json).

Graphical cycle maps are available on
[dbdb/db\_umlauf](https://lib.finalrewind.org/dbdb/db_umlauf/).

Data is © 2020 DB Fernverkehr AG, licensed under CC-BY-4.0.

## Data Format

This README documents **version 0** of the format. It is not stable yet; data
layout or semantic changes may not be reflected in the version number.
Starting with v1, schema and semantics will be stable. See also the
[OpenAPI spec](https://lib.finalrewind.org/dbdb/db_zugbildung_v0.yml).

```js
{
	"deprecated": false,
	"source": "2021_ZpAR_Wi_Endstück.pdf",
	"valid": "2020-12-13/2021-06-12",
	"train": {
		"10": { /* train details */ },
		"11": { /* train details */ }
		// ...
	},
}
```

### Deprecation Notice

```js
{
	"deprecated": false,
}
```

**true** iff this file uses a deprecated version of the db-zugbildung-to-json
schema. It may not be updated when DB releases a new train composition.

### Source

```js
{
	"source": "2021_ZpAR_Wi_Endstück.pdf"
}
```

The PDF file used to generate the data set. Useful when reporting an issue and
to check whether content is up-to-date.

### Validity

```js
{
	"valid": "YYYY-MM-DD/YYYY-MM-DD"
}
```

An ISO 8601 interval describing the valid range of the train composition data
as noted in the PDF file.

### Train details

```js
{
	"10": {
		"rawType": "ICE-W",
		"type": "ICE 3",
		"shortType": "3",
		"name": "ICE International",
		"route": { /* scheduled route */ },
		"commonAttr": { /* train/powercar attributes */ },
		"attrVariants": [ { /* train/powercar attributes */ } /* ... */ ],
		"schedules": [ /* scheduled service days and route deviations */ ],
		"cycle": { /* cycle ("Umlauf") data */ },
		"hasWagon": { /* wagon type map */ },
		"wagons": [ /* wagon list */ ],
	}
}
```

Each train is identified by its number. It is unique in context of DB
long-distance trains, but may be used by other european operators as well. For
instance, the IC services Amsterdam – Berlin and Koebenhavns – Aarhus often use
identical three-letter numbers.

rawType, type, and route are always present. The remaining properties are
optional.

#### rawType

The train type as specified in the PDF file, e.g. **ICE-A**, **IC**, or
**LNF**.

#### type

The scheduled train type as estimated from rawType and wagon data. This
information is **mostly reliable**. If a train type is unknown, either due to
an unhandled composition or due to different train types for different time
ranges (which are not supported by this schema yet), `rawType` is used.

Values:

* ICE 1/2/4
* ICE 1
* ICE 2
* ICE 3
* ICE 3 Redesign
* ICE 3 Velaro
* ICE 4
* ICE T
* Metropolitan
* IC
* IC2
* IC2 KISS
* *anything present in rawType*

#### shortType

A short identifier which can be used to differentiate between ICE 3 and ICE 3
Redesign or between IC1 and IC2. If the type cannot be estimated, this
property is not present. Just like `type`, it is **mostly reliable**.

Values:

* **1** (ICE 1)
* **2** (ICE 2 / IC2 / IC2 KISS)
* **3** (ICE 3)
* **3R** (ICE 3 Redesign)
* **3V** (ICE 3 Velaro)
* **4** (ICE 4)
* **M** (Metropolitan)
* **T** (ICE T)

#### name

Optional train name / line name / description provided by the PDF file. Not
present if the train has no such entry.

Examples include "ICE International", "Kieler Bucht", "Blauer Enzian", and
"DB-ÖBB EuroCity". Several trains may share a common name / description.

#### empty

true if this is an empty train without passenger service ("Leerfahrt").
Otherwise, the property is not present. It is **reliable**.

#### route

```js
{
	"route": {
		"end": "München (10:02)",
		"middle": [
			"Berlin",
			"Berlin Südkreuz",
			"Halle (Saale)",
			"Erfurt (07:40/07:45)",
			"Nürnberg"
		],
		"preStart": "Berlin-Rummelsburg (Triebzuganlage)",
		"start": "Berlin-Gesundbrunnen (05:53)"
	}
}
```

Scheduled route. It contains up to five properties:

* **preStart**: Station (without passenger service) where the train is prepared / provisioned.
* **start**: First station(s) with passenger service. Often contains the scheduled departure time.
* **middle**: List of noteworthy stations along the route. May be empty.
  This is not the complete route. Individual stations may contain timestamps.
* **end**: Terminal station(s) with passenger service. Often contains the scheduled arrival time.
* **postEnd**: Station (without passenger service) where the train is parked.

Station entries are taken as-is from the PDF file. They may differ from station
names used in iris-tts or HAFAS. A missing preStart / postEnd entry does not
imply that the train is prepared / parked at the first / last station with
passenger service. Station names may be surrounded by brackets.

#### train/powercar attributes

```js
{
	"brakingPercentage": 177,
	"length": 402,
	"series": "406",
	"station": "Basel SBB",
	"vmax": 300
}
```

Taken as-is from the PDF file. commonAttr contains only properties which are
identical in each variant. *station* is translated from DS100 to a station name
and likely the first station on the train's route where the corresponding
attributes are valid. Presumably, they valid until the station mentioned in the
next entry is reached (or the train reaches its terminus). *station* may be
prefixed with a plus sign.

#### schedules

*work in progress*

#### cycle

```js
{
	"cycle": {
		"from": [
			"78546",
			"1051",
			"78526"
		],
		"to": [
			"1051"
		]
	}
}
```

Cycle ("Umlauf") data, i.e., which trains may make up this train and which
trains it may end up in. **Reliability unknown**.

**from** is a list of train numbers which may make up this train.

**to** is a list of train numbers which this train may end up in.

PDF annotations covering date ranges, wings, and segments (e.g. a two-segment
train which becomes a one-segment train along the way) are not taken into
account. Missing cycle data may be caused by known parser issues. Referenced
train numbers may not be present in this JSON file; this is often the case for
inter-european connections. Referenced trains may be added / removed along the
way, not just at the start / end station.

#### hasWagon

```js
{
	"hasWagon": {
		"147.5": true,
		"DApza": true,
		"DBpbzfa": true,
		"DBpza": true
	}
}
```

Contains a true property for each wagon and locomotive type scheduled for the
train. Does not take time ranges into account. For example, if a train is
scheduled as IC1 on weekdays and IC2 on weekends, `hasWagon` will contain both
IC1 and IC2 wagons.  **Mostly reliable**. The wagon parser utilizes regular
expressions and may miss rarely used wagon types.

#### wagons

```js
{
	"wagons": [
		{
			"type": "DBpbzfa",
			"number": 1
		},
		{
			"type": "DBpza",
			"number": 2
		},
		{
			"type": "DBpza",
			"number": 3
		},
		{
			"type": "DBpza",
			"number": 4
		},
		{
			"type": "DApza",
			"number": 5
		},
		{
			type: "147.5",
		}
	]
}
```

Ordered list of wagons and (for ICE 1, ICE 2, and IC2) matched locomotives.
Unset if the wagon order cannot be reliably determined or if it depends on the
date. Does not contain IC1 locomotives.

**Mostly reliable**. Just like `hasWagon`, rarely used wagon types may be
missing.
