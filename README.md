# db-wagenreihung-to-json - Convert wagon order PDFs to JSON

db-wagenreihung-to-json converts a wagon order (“Zugbildungsplan”) PDF obtained
from
[data.deutschebahn.com](https://data.deutschebahn.com/dataset/zugbildungsplanzugbildungsplan-zpar)
to JSON. At the moment, conversion is limited to a map of train numbers to
IC/ICE types (ICE 1, ICE 3, ICE 3 Redesign, ...). Train composition, exceptions
on certain dates and other details are lost.

The latest JSON produced by d-wagenreihung-to-json is available online at
[dbdb/ice\_type.json](https://lib.finalrewind.org/dbdb/ice_type.json).
