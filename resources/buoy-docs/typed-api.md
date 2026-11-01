---
title: "csvkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["CSVByteSink", "CSVRecord", "CSVStreamParser", "CSVStreamWriter"]
dependencies: []
symbols: ["CSVByteSink", "CSVRecord", "CSVStreamParser", "CSVStreamWriter"]
related: ["api.md", "getting-started.md", "index.md"]
---

# csvkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## CSVRecord

`Public Class CSVRecord`

One CSV record. Fields are mutable strings in an ordered resizable array.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Fields` | `Array Of String` | No | No | — | — |

## CSVKit functions

### ParseCSV

`Public Function ParseCSV(text As String, delimiter As String = ",", maxCharacters As Integer = 1000000, maxRecords As Integer = 100000, maxFields As Integer = 10000) As Array Of CSVRecord`

Parse complete CSV text. Quoted fields preserve embedded CR/LF exactly.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `text` | `String` | `ByVal` | Required |
| `delimiter` | `String` | `ByVal` | `","` |
| `maxCharacters` | `Integer` | `ByVal` | `1000000` |
| `maxRecords` | `Integer` | `ByVal` | `100000` |
| `maxFields` | `Integer` | `ByVal` | `10000` |

#### Returns

| Type |
| --- |
| `Array Of CSVRecord` |

### WriteCSV

`Public Function WriteCSV(records As Array Of CSVRecord, delimiter As String = ",", maxCharacters As Integer = 1000000) As String`

Encode records with CRLF endings, always including a final record terminator.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `records` | `Array Of CSVRecord` | `ByVal` | Required |
| `delimiter` | `String` | `ByVal` | `","` |
| `maxCharacters` | `Integer` | `ByVal` | `1000000` |

#### Returns

| Type |
| --- |
| `String` |

## CSVStreamParser

`Public Class CSVStreamParser`

Incremental character-chunk CSV parser with a bounded drainable record queue.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `ReadyCount` | `Integer` | Yes | No | — | — |
| `BufferedCharacters` | `Integer` | Yes | No | — | — |
| `Failed` | `Boolean` | Yes | No | — | — |
| `IsFinished` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(delimiter As String = ",", maxRecordCharacters As Integer = 1000000, maxFields As Integer = 10000, maxReadyRecords As Integer = 128, maxBufferedCharacters As Integer = 1000000, maxChunkCharacters As Integer = 16384)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `delimiter` | `String` | `ByVal` | `","` |
| `maxRecordCharacters` | `Integer` | `ByVal` | `1000000` |
| `maxFields` | `Integer` | `ByVal` | `10000` |
| `maxReadyRecords` | `Integer` | `ByVal` | `128` |
| `maxBufferedCharacters` | `Integer` | `ByVal` | `1000000` |
| `maxChunkCharacters` | `Integer` | `ByVal` | `16384` |

Returns: no value.

### Reset

`Public Sub Reset()`

No parameters.

Returns: no value.

### Feed

`Public Sub Feed(chunk As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `chunk` | `String` | `ByVal` | Required |

Returns: no value.

### Finish

`Public Sub Finish()`

No parameters.

Returns: no value.

### Take

`Public Function Take() As CSVRecord`

No parameters.

#### Returns

| Type |
| --- |
| `CSVRecord` |

## CSVByteSink

`Public Interface CSVByteSink`

Synchronous recipient of one complete UTF-8 CSV record per call.

### WriteCSVBytes

`Sub WriteCSVBytes(data As Bytes)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `data` | `Bytes` | `ByVal` | Required |

Returns: no value.

## CSVStreamWriter

`Public Class CSVStreamWriter`

Incremental CSV encoder. Failed writes are terminal; output is not atomic.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `BytesWritten` | `Integer` | Yes | No | — | — |
| `RecordsWritten` | `Integer` | Yes | No | — | — |
| `Failed` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(sink As CSVByteSink, delimiter As String = ",", maxBytes As Integer = 16000000, maxRecords As Integer = 100000, maxRecordBytes As Integer = 1000000)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `sink` | `CSVByteSink` | `ByVal` | Required |
| `delimiter` | `String` | `ByVal` | `","` |
| `maxBytes` | `Integer` | `ByVal` | `16000000` |
| `maxRecords` | `Integer` | `ByVal` | `100000` |
| `maxRecordBytes` | `Integer` | `ByVal` | `1000000` |

Returns: no value.

### Write

`Public Sub Write(record As CSVRecord)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `record` | `CSVRecord` | `ByVal` | Required |

Returns: no value.
