---
title: "CSVKit"
summary: "Parse whole documents or incremental CSV chunks and write CSV while preserving string values and quoted line endings."
category: "Overview"
order: 0
keywords: ["csv", "delimiter", "records", "quoting"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# CSVKit

CSVKit provides pure Buoy whole-document parsing/writing and incremental parsing
through CSVStreamParser. It adds record framing, multiline quoted fields, strict
validation and explicit bounds. There are no native dependencies or implicit
filesystem operations.

Fields are strings and whitespace is data. Header naming, type conversion, BOM removal, byte decoding, and spreadsheet formula handling belong to the caller. Ragged records are accepted. Parsing is strict and returns no partial document on failure.

## Format and bounds

The delimiter is one Unicode codepoint other than quote, CR, or LF. Double quote is the fixed field enclosure. Doubled quotes within quoted fields decode to one quote; embedded CR/LF sequences remain exact. Outside quotes, CR, LF, or CRLF terminates a record. Character bounds count Unicode codepoints rather than UTF-8 bytes or grapheme clusters.

ParseCSV materializes the whole document. CSVStreamParser retains only its
bounded current record and ready queue; callers drain records and provide decoded
chunks. Memory includes fragments and objects, so character bounds are not byte
memory quotas. This independent implementation does not claim the upstream
event API or permissive recovery behavior.

## Read next

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
