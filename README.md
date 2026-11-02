# Strict CSV

Bounded CSV document parsing and writing in pure Buoy, inspired by
[Norman Palardy's CSVParser](https://github.com/npalardy/CSVParser).
`Import csvkit` exposes `CSVRecord`, `ParseCSV`, `WriteCSV` and `CSVStreamParser`.
No native dependencies or filesystem side effects. See [provenance](PROVENANCE.md).

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add csvkit --git https://github.com/BuoyBay/csvkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer csvkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
