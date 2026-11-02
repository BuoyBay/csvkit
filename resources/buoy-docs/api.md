---
title: "CSVKit API"
summary: "Public types, functions, validation, and ownership rules for CSVKit."
category: "Reference"
order: 20
keywords: ["csv", "delimiter", "records", "quoting"]
symbols: ["CSVRecord", "CSVRecord.Fields", "ParseCSV", "WriteCSV", "CSVStreamParser", "CSVStreamParser.Feed", "CSVStreamParser.Take", "CSVStreamParser.Finish", "CSVStreamParser.Reset"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# CSVKit API

| API | Contract |
| --- | --- |
| CSVRecord | Mutable Fields As Array Of String, preserving field order. |
| ParseCSV(text, delimiter, maxCharacters, maxRecords, maxFields) | Returns Array Of CSVRecord; defaults comma, 1000000, 100000, 10000. |
| WriteCSV(records, delimiter, maxCharacters) | Returns minimally quoted String with CRLF record terminators; defaults comma and 1000000. |
| CSVStreamParser | Incremental decoded-String parser with bounded current record and ready queue. |
| Feed(chunk), Take() | Feed a bounded chunk; drain completed records while ReadyCount is positive. |
| Finish(), Reset() | Validate EOF or clear content/failure state while retaining configuration. |

## Incremental lifecycle

The stream constructor defaults are comma delimiter, 1000000 record characters,
10000 fields, 128 queued records, 1000000 buffered characters and 16384 characters
per chunk. Chunks may split quoted fields, doubled quotes or CRLF, but must
already be complete decoded Strings. The parser does not concatenate the whole
document or retain drained rows. `ReadyCount`, `BufferedCharacters`, `Failed` and
`IsFinished` expose its state.

Successful Finish is idempotent and leaves queued records available. Syntax or
Feed/Finish budget failures poison the parser and discard retained records;
Reset permits reuse. Already drained rows cannot be withdrawn. Feeding after
successful Finish and taking from an empty queue reject without poisoning.
Queue pressure is an error, not resumable backpressure: drain between feeds and
size the queue for the number of rows a chunk can complete.

## Ownership and errors

ParseCSV does not mutate its input and returns no partial result on failure. CSVRecord fields are mutable/resizable and retain literal strings. WriteCSV reads the supplied records and returns text; an empty record array writes empty text.

All limits must be positive. InvalidArgumentException reports malformed syntax or exceeded bounds. Some syntax failures contain a one-based character offset; unterminated quotes and bound errors have specific messages without structured record/column information.

## Deliberate exclusions

No file reader, stream callbacks, byte decoder, header schema, typed fields,
custom enclosure or recovery callbacks are supplied. Stream budgets count wire
codepoints and metadata, not exact process memory. The writer may temporarily
allocate an escaped field up to roughly twice its original character count.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)

[Full typed API: properties, parameters, defaults, and returns](typed-api.md)

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

```basic
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
