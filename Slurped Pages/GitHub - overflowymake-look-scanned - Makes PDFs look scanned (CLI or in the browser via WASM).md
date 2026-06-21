---
link: https://github.com/overflowy/make-look-scanned
site: GitHub
excerpt: Makes PDFs look scanned (CLI or in the browser via WASM) -
  overflowy/make-look-scanned
twitter: https://twitter.com/@github
slurped: 2026-06-21T10:32
title: "GitHub - overflowy/make-look-scanned: Makes PDFs look scanned (CLI or in
  the browser via WASM)"
---

A CLI that takes a PDF and degrades it to look like a physical scan of a printout — skew, grayscale, warm paper tone, scanner grain, defocus, edge shadow, and JPEG compression artifacts. Also [runs client-side in the browser](https://overflowy.github.io/make-look-scanned/) via WASM.

[![example](https://private-user-images.githubusercontent.com/98480250/610801884-9653fd3b-3abe-4427-ad82-03a78d4429b5.jpg?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODIwMzEwMjAsIm5iZiI6MTc4MjAzMDcyMCwicGF0aCI6Ii85ODQ4MDI1MC82MTA4MDE4ODQtOTY1M2ZkM2ItM2FiZS00NDI3LWFkODItMDNhNzhkNDQyOWI1LmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA2MjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNjIxVDA4MzIwMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWVmODg5OWVlZjExOTU2YjllZWFlYTExYTE0NGI0YjVmMTI4NjIwMWQ5OTVkNzEwMWE5ZTc2MDQ0MTZlNTE1YzQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JnJlc3BvbnNlLWNvbnRlbnQtdHlwZT1pbWFnZSUyRmpwZWcifQ.Xfpv3aS7drIY0-b8LoGpavxIPLRdfOYVyrajxUTkZLY)](https://private-user-images.githubusercontent.com/98480250/610801884-9653fd3b-3abe-4427-ad82-03a78d4429b5.jpg?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODIwMzEwMjAsIm5iZiI6MTc4MjAzMDcyMCwicGF0aCI6Ii85ODQ4MDI1MC82MTA4MDE4ODQtOTY1M2ZkM2ItM2FiZS00NDI3LWFkODItMDNhNzhkNDQyOWI1LmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA2MjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNjIxVDA4MzIwMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWVmODg5OWVlZjExOTU2YjllZWFlYTExYTE0NGI0YjVmMTI4NjIwMWQ5OTVkNzEwMWE5ZTc2MDQ0MTZlNTE1YzQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JnJlc3BvbnNlLWNvbnRlbnQtdHlwZT1pbWFnZSUyRmpwZWcifQ.Xfpv3aS7drIY0-b8LoGpavxIPLRdfOYVyrajxUTkZLY)

Each page is rasterized to an image, run through the effect pipeline, and reassembled into a new **image-only** PDF (the original selectable text is gone — faithful to a basic scanner).

## Build

[](https://github.com/overflowy/make-look-scanned#build)

Requires Go and a C toolchain (go-fitz links MuPDF via cgo, so the binary is self-contained — nothing to install at runtime).

go build -o make-look-scanned .

## Usage

[](https://github.com/overflowy/make-look-scanned#usage)

make-look-scanned [flags] input.pdf

Flags may appear before or after the input filename.

make-look-scanned in.pdf                 # -> in.scanned.pdf
make-look-scanned in.pdf -o out.pdf
make-look-scanned in.pdf --noise 0.4 --skew 2.5 --jpeg-quality 30

### Flags

[](https://github.com/overflowy/make-look-scanned#flags)

|Flag|Default|Meaning|
|---|---|---|
|`-o`|`<input>.scanned.pdf`|output path|
|`--preset`|—|named preset from `config.toml`|
|`--seed`|content hash|random seed (override for a new look)|
|`--force`|false|overwrite an existing output file|
|`--dpi`|150|render resolution|
|`--skew`|0.6|max rotation degrees (0 disables)|
|`--grayscale`|true|desaturate (`--grayscale=false` keeps color)|
|`--paper-tone`|0.6|warm paper tint strength 0..1|
|`--noise`|0.08|scanner grain 0..1|
|`--blur`|0.4|defocus gaussian sigma|
|`--edge-shadow`|0.15|border vignette 0..1|
|`--jpeg-quality`|70|JPEG quality 1..100|

Each numeric knob disables its effect at `0`.

## Determinism

[](https://github.com/overflowy/make-look-scanned#determinism)

Output is **deterministic by default**: the seed is derived from the input PDF's content, so the same file always produces the same scan. Pass `--seed N` for a different (but reproducible) look. Same input + seed yields a byte-identical PDF.

## Presets

[](https://github.com/overflowy/make-look-scanned#presets)

Define reusable bundles in `$XDG_CONFIG_HOME/make-look-scanned/config.toml` (falls back to `~/.make-look-scanned/config.toml` when `XDG_CONFIG_HOME` is unset). Keys mirror the flag names with underscores:

[presets.medium]
skew = 1.5
paper_tone = 0.6
noise = 0.2
blur = 0.6
edge_shadow = 0.3
jpeg_quality = 45

make-look-scanned --preset medium in.pdf

Precedence: built-in defaults → selected preset → explicit CLI flags (flags always win).

## Browser (WebAssembly)

[](https://github.com/overflowy/make-look-scanned#browser-webassembly)

The effect pipeline also runs in the browser. go-fitz/MuPDF can't compile to wasm, so the browser uses **PDF.js** to rasterize pages and hands the pixels to the _same_ Go effects + assembly code compiled to wasm.

Dev (needs network for the PDF.js CDN):

./web/build.sh                       # builds web/main.wasm + wasm_exec.js
(cd web && python3 -m http.server 8080)   # then open http://localhost:8080

Single self-contained file (works offline, nothing to serve):

task build:web                       # writes dist/make-look-scanned.html (~8 MB)

`dist/make-look-scanned.html` inlines the wasm, Go's runtime glue, and PDF.js (library + worker) as base64 — open it directly in a browser. Output is visually equivalent to the CLI but not byte-identical, since PDF.js and MuPDF rasterize differently.

## License

[](https://github.com/overflowy/make-look-scanned#license)

[AGPL-3.0](https://github.com/overflowy/make-look-scanned/blob/main/LICENSE). The CLI statically links MuPDF (via go-fitz), which is AGPL-3.0, so the combined binary is AGPL-3.0 — distributing it requires offering the corresponding source. The browser build does not include MuPDF (it uses PDF.js, Apache-2.0)