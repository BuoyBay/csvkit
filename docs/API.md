# CSVKit 0.1.0

Bounded CSV document parsing and writing in pure Buoy, inspired by
[Norman Palardy's CSVParser](https://github.com/npalardy/CSVParser).
`Import csvkit` exposes `CSVRecord`, `ParseCSV`, `WriteCSV` and `CSVStreamParser`.
No native dependencies or filesystem side effects. See [provenance](../PROVENANCE.md).

## API

- `CSVRecord.Fields As Array Of String`: a mutable ordered, resizable field array.
- `ParseCSV(text, delimiter = ",", maxCharacters = 1000000, maxRecords = 100000, maxFields = 10000) As Array Of CSVRecord`.
- `WriteCSV(records, delimiter = ",", maxCharacters = 1000000) As String`.

All limits must be positive. The delimiter must be exactly one Unicode codepoint
other than double quote, CR or LF; matching is case sensitive. A quote must begin
a field. Doubled quotes inside quoted fields decode into one quote. Embedded CR,
LF and CRLF inside quotes are preserved exactly. Outside quotes each CR, LF or
CRLF ends a record. Whitespace is data: whitespace after a closing quote is an
error, and fields are never trimmed. Ragged records are accepted; header names
and value types are application concerns. Empty input produces zero records;
a blank line produces one record containing one empty string. A final record
terminator produces no extra record. A final delimiter produces an empty field.
UTF-8 BOM removal and legacy byte decoding are caller responsibilities.

Malformed syntax, invalid delimiters, exceeded limits, Nil records and records
with zero fields throw `InvalidArgumentException`. Parsing returns no partial
result on failure and does not modify input. Syntax errors inside fields include
a one-based character offset; unterminated quotes and bounds failures have a
specific message without record/column metadata. No recovery callbacks exist.

The writer uses minimal quoting and CRLF record terminators, including after
the last record. It quotes a singleton empty field (`""`) so other CSV readers
preserve it as one field. Empty records cannot be written because they have no
unambiguous representation. An empty record array writes empty text. The writer
preserves strings exactly and adds no spreadsheet formula escaping or type
conversion. Supply spreadsheet-safe values when that is the intended consumer.

Character limits count Unicode codepoints, not UTF-8 bytes or grapheme clusters.
The parser checks input length before making its character array and collects
field fragments before joining, avoiding repeated rescans of Unicode prefixes.
Time and storage grow with the bounded input plus records/fields. This is a
whole-document parser; a million characters can use substantially more than a
million bytes because character fragments and row objects have overhead. The
writer bounds the resulting text while accumulating fields; it may temporarily
allocate an escaped field of up to roughly twice its original character count.

## Incremental parsing

`New CSVStreamParser(delimiter = ",", maxRecordCharacters = 1000000,
maxFields = 10000, maxReadyRecords = 128, maxBufferedCharacters = 1000000,
maxChunkCharacters = 16384)` accepts complete decoded String chunks through
`Feed(chunk)`. A chunk can end anywhere between Unicode codepoints, including
between CR/LF, doubled quotes or a quoted field's opening and closing quotes.
Embedded line endings and strict syntax follow `ParseCSV` exactly.

Drain completed records while `ReadyCount > 0` using `Take()`. A returned
`CSVRecord` belongs to the caller and is no longer retained by the parser.
`Finish()` marks EOF, validates pending quotes and emits a final unterminated
record if present. It is idempotent after success. Queued records remain
available after successful Finish. Feeding after Finish throws without changing
the finished state. Taking from an empty queue throws without poisoning.

Malformed input or a Feed/Finish budget failure discards all queued/current
records, sets `Failed = True` and requires `Reset()` before reuse. Records already
taken by the caller cannot be withdrawn. Failed instances reject Feed, Finish
and Take. Reset clears content, pending CR/quote state and lifecycle flags while
retaining configuration. `IsFinished` reports successful EOF processing.

The parser retains only the current record and a bounded ready queue; it does
not concatenate the document or retain drained rows. `BufferedCharacters` counts
wire codepoints belonging to retained records, including quoting/delimiters and
one record terminator. The LF following an outside-quote CR is skipped and not
counted. Both record and aggregate character bounds include those overhead
characters, so an input record with `N` data characters can require a larger
budget. `ReadyCount` and `maxFields` separately bound record/field metadata.

All limits are positive. Supported ceilings are 16 Mi codepoints per record,
100000 fields, 10000 queued records, 64 Mi buffered codepoints and 1 Mi codepoints
per Feed. A chunk's length is checked before its temporary character array is
created. Actual allocations exceed the character count because of strings,
arrays and objects; these are API retention bounds, not byte-memory quotas.
Caller-owned chunks and already returned records are outside those bounds.

Queue/buffer pressure is an error, not a resumable partial Feed. Drain between
small feeds, and size the queue for the greatest number of records one chunk
may complete. A feed containing more queued records than capacity poisons the
parser even if they would have been drained after that feed returned. There are
no callbacks or implicit disk reads. Bytes/encoding boundaries, BOM handling,
file ownership and asynchronous backpressure remain caller responsibilities.


This suite checks module/test/example on six targets, executes 100000 separately
drained rows under an eight-character buffer budget, and compares 40 Python CSV
tables at every two-chunk split and nine deterministic chunk schedules. It also
runs the original document parser/writer suite. The
streaming example (not included in this repository)
demonstrates draining around chunks that split quotes and CRLF.

## Relationship to existing Buoy code

Built-in String operations do not frame CSV records. StringUtils's
`SplitQuoted`/`JoinQuoted` serve one quoted record and remain useful there;
CSVKit adds whole-document CR/LF framing, multiline records, record and field
limits, strict document errors and canonical writing. CSVKit is standalone so
CSV consumers need not import unrelated StringUtils operations.

## Use and verification

This directory has compiler and BPM manifests. A local BPM dependency can use
`{"path":"../libraries/csvkit"}`. A sealed package is supplied at
`packages/csvkit-0.1.0.buipkg`; no remote publication was performed.

```sh
/usr/bin/buoy pkg seal . -o packages/csvkit-0.1.0.buipkg
```

The absolute package command is a workaround for this development host's
compiler wrapper; elsewhere use `buoy pkg seal` normally. `BUOY` and
`BUOY_PACKAGER` override the verification commands. The runner builds isolated
consumers using a freshly sealed package, checks module/test/example on six
desktop targets, runs strict syntax/bounds tests and compares 100 generated
records (500 fields) plus the singleton-empty-field edge case with Python's
standard-library CSV reader/writer. Cross-target checks do not establish native
runtime behavior on those platforms. Execution was verified on Linux.

## Incremental UTF-8 export

`CSVByteSink` has one synchronous method, `Sub WriteCSVBytes(data As Bytes)`.
`New CSVStreamWriter(sink, delimiter = ",", maxBytes = 16000000,
maxRecords = 100000, maxRecordBytes = 1000000)` accepts one `CSVRecord` at a
time through `Write(record)`. Each successful call sends exactly one UTF-8
chunk with canonical quoting and a final CRLF. Supply headers as ordinary
records; they count toward every limit. Existing `WriteCSV` and table adapters
continue to return complete strings.

All limits are positive; `maxRecordBytes` must be at most 16000000. The byte
limits count encoded UTF-8 including delimiters, quoting and CRLF, not Unicode
characters. A row that exceeds a byte or record limit is rejected before its
sink call: no prefix of that rejected row is emitted. `BytesWritten` and
`RecordsWritten` count only calls whose sink returned successfully. A sink can
write a prefix and then throw; those external bytes are not reflected in the
counters, cannot be rolled back and must not be retried through this writer.

Any `Write` failure makes `Failed` true permanently. Exceptions propagate;
subsequent writes fail without calling the sink. Recursive calls to the same
writer are rejected and poison it even if the sink catches the recursive
exception; the outer call then fails too. Concurrent use is unsupported. There
is no implicit flush, close, reset or transaction. The caller owns the sink and
its resource lifecycle, and decides whether to discard partial output.

The writer holds its sink and counters, not previous records or output chunks.
It reads each supplied record synchronously without modifying it. Each call
creates a fresh mutable `Bytes`; the sink may retain or mutate that chunk, and
the writer snapshots its size before the callback. Mutating the input record
after `Write` returns cannot change a delivered chunk. Memory retained by a
caller or sink is outside this guarantee.

Formatting is bounded per record by `maxRecordBytes` scalar characters before
UTF-8 encoding, then the actual byte length is checked. The temporary UTF-8
buffer can therefore reach four times that scalar limit before rejection;
quoting, arrays and strings introduce additional per-record intermediates.
This is bounded per-record work space, not an exact process-memory ceiling.
The caller's input field strings already exist; it must bound their creation.

For generated or query rows, create and submit one record inside the producer's
loop rather than building an array or JSONRows table. This file consumer keeps
resource ownership explicit (the path is supplied by the caller):

```buoy
Import csvkit

Class ExportFileSink Implements CSVByteSink
    Public Property Stream As BinaryStream
    Public Sub WriteCSVBytes(data As Bytes)
        Stream.Write(data)
    End Sub
End Class

Sub Main(args As Array Of String)
    Dim created As Result Of BinaryStream, IOError = BinaryStream.Create(New FileSystemItem(args(1)), True)
    If created.IsErr Then Throw New Exception("Cannot create CSV output")
    Dim stream As BinaryStream = created.Value
    Dim sink As New ExportFileSink()
    sink.Stream = stream
    Dim writer As New CSVStreamWriter(sink, ",", 1000000, 10001, 4096)
    Try
        Dim header As New CSVRecord()
        Array.Append(header.Fields, "id")
        writer.Write(header)
        For i As Integer = 1 To 10000
            Dim record As New CSVRecord()
            Array.Append(record.Fields, Str(i))
            writer.Write(record)
        Next
    Finally
        stream.Close()
    End Try
End Sub
```

`tests/run_csv_stream_writer_tests.py` runs a source-backed file consumer and
compares 100001 records containing quotes, newlines and Unicode against Python's
CSV writer, alongside exact-byte, record-limit, failure and ownership controls.

Test suites and fixtures are maintainer-local and are not part of this public repository.
