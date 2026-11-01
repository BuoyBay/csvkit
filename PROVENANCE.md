# Source and scope

The [Great White Software open-source page](https://www.great-white-software.com/open-source/)
links [npalardy/CSVParser](https://github.com/npalardy/CSVParser). Inspected commit:
`8bf56076ec6d248156510f717e014dd07947d2db`.

Inspected the repository README, MIT LICENSE, CSVParser, CSVRecordSet, CSVField,
BufferedBinaryInputStream and CSVWriter source. Upstream attribution is retained
in [UPSTREAM-LICENSE.txt](UPSTREAM-LICENSE.txt). This is a newly written Buoy
implementation of the CSV document technique, not a syntax conversion of Xojo
source or an assertion of original API/behavior parity.

| Upstream facility | Buoy package treatment |
| --- | --- |
| CSVParser.Parse(String) and field/line events | ParseCSV returns materialized CSVRecord objects; CSVStreamParser incrementally frames decoded chunks into a bounded drainable queue, without callbacks |
| CSVParser.Parse(FolderItem) | Not implemented; callers explicitly read/decode text |
| Configurable fieldseparator/fieldenclosure | One exact Unicode delimiter; quote fixed to double quote |
| CSVRecordSet MoveNext/EOF/IdxField | Materialized array iteration; no database-like wrapper |
| CSVField type conversions | Strings remain strings; use Buoy conversion APIs explicitly |
| BufferedBinaryInputStream | No byte stream or encoding adapter; input is Buoy String |
| CSVWriter file/stream lifecycle and EOL styles | WriteCSV returns text with fixed CRLF; caller owns file writing |
| Embedded line endings and doubled quotes | Implemented and tested |
| Error callbacks and permissive recovery | Strict exceptions; no partial document returned |

The inspected upstream parser can continue collecting after a closing quote
followed by non-delimiter text. CSVKit deliberately rejects that input. Empty
input, blank lines, final terminators and final empty fields have explicit
contracts in README.md rather than claiming upstream edge-case equivalence.

The incremental character-chunk API now supplies explicit ownership, bounds,
queue-pressure failure and lifecycle contracts. Remaining useful work includes
bounded streaming writes, structured source locations, encoding-aware file
adapters and optional header/table integration. Those require additional APIs
and tests; the parsers need no compiler changes.
A shared quoted-field scanner could later consolidate StringUtils and
CSVKit, but this package does not change StringUtils's existing contract.
