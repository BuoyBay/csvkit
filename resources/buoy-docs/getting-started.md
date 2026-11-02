---
title: "CSVKit getting started"
summary: "Configure and use CSVKit with explicit input and ownership rules."
category: "Guides"
order: 10
keywords: ["csv", "delimiter", "records", "quoting"]
related: ["index.md", "api.md", "typed-api.md"]
---

# CSVKit getting started

Install the csvkit archive in your packages directory or configure a local BPM path dependency to its source directory. The example parses a header plus one quoted record, changes a field, and writes canonical CSV text without reading or writing files.

```basic
Import csvkit

Sub Main()
    Dim records As Array Of CSVRecord = ParseCSV("name,note\r\nAda,\"hello, world\"\r\n")
    If Array.Count(records) > 1 Then
        records(1).Fields(0) = "Grace"
    End If
    Print WriteCSV(records)
End Sub
```

## Choose application rules

ParseCSV defaults to comma, one million characters, 100000 records, and 10000 fields per record. Set positive limits appropriate to your input before parsing untrusted documents. Treat header names and missing/extra columns explicitly rather than assuming every row has the same width.

Empty input produces zero records. A blank line is one empty field. A final terminator adds no extra record, while a final delimiter adds an empty field. WriteCSV uses minimal quoting and always adds CRLF after the final record. It quotes a singleton empty field to keep that record unambiguous.

## Failure handling

Catch InvalidArgumentException for syntax, bounds, invalid delimiter, Nil writer records, or zero-field writer records. Text after a closing quote, including whitespace, rejects. The writer does not trim values, convert types, or escape spreadsheet formulas.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
