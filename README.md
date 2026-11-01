# Connect native fields to typed formatting and masks

Independent editing sessions and native commit-format fields. Import `FieldKit`.
Depends only on MaskKit 0.1.0; no FormatKit, TemporalKit or SettingsKit dependency.
Canonical values are serialized strings; numeric/temporal adapters belong to callers.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add fieldkit --git https://github.com/BuoyBay/fieldkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.
BPM also resolves `maskkit`.

## API at a glance

`FieldCodec` has `Analyze(text, profile) As FieldAnalysis`,
`FormatDisplay(canonical, profile) As String`, and
`FormatEdit(canonical, profile) As String`. Profile is an immutable serialized
String snapshot, not a mutable profile object. Codecs must be deterministic,
side-effect free, preserve full precision in edit format, and return a canonical
candidate only for Valid analysis. Display may round without changing storage.

`FieldAnalysis(state, candidate As Optional Of String, errorCode, errorPosition)`
is immutable. `FieldState` is Empty, Incomplete, Invalid, Valid. Positions are
codepoint offsets, -1 when unavailable. Valid requires Some(candidate); other
states require None. Empty is exactly zero-length draft, handled by the session;
whitespace is passed to the codec. Empty canonical string (Some("")) and None
are distinct. Codecs cannot return Empty for nonempty input.

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer fieldkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

The generated agent files and editor settings stay local and are ignored by Git.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
