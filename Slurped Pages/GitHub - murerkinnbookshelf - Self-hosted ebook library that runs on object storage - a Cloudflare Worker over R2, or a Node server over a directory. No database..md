---
link: https://github.com/murerkinn/bookshelf
site: GitHub
excerpt: Self-hosted ebook library that runs on object storage - a Cloudflare
  Worker over R2, or a Node server over a directory. No database. -
  murerkinn/bookshelf
twitter: https://twitter.com/@github
slurped: 2026-08-25T10:09
title: "GitHub - murerkinn/bookshelf: Self-hosted ebook library that runs on
  object storage - a Cloudflare Worker over R2, or a Node server over a
  directory. No database."
---

[![CI](https://github.com/murerkinn/bookshelf/actions/workflows/ci.yml/badge.svg)](https://github.com/murerkinn/bookshelf/actions/workflows/ci.yml)

A self-hosted library for the ebooks you already own. One server-rendered page lists them, filters them with a search box, serves downloads, and reads them in the browser — EPUB and PDF, each with its own reader — as a Cloudflare Worker over R2, or as a Node server over a directory on disk.

[![The shelf: a searchable list of books with covers, a profile switcher, and a Continue button on the book being read](https://github.com/murerkinn/bookshelf/raw/main/docs/shelf.webp)](https://github.com/murerkinn/bookshelf/blob/main/docs/shelf.webp)

Everything above is `npm run demo` — a generated shelf of public-domain titles, so the screenshots can be reproduced without finding books first.

## Getting started

[](https://github.com/murerkinn/bookshelf#getting-started)

Node 24 or newer, and a Unix-like system: the sync tool finds its image tools with `which`, so Windows is not supported. Covers come out better with `cwebp` and `pdftoppm` installed — see [publishing](https://github.com/murerkinn/bookshelf/blob/main/docs/publishing.md#covers), or use Docker, which ships both.

git clone https://github.com/murerkinn/bookshelf.git
cd bookshelf
npm install

Put some books — EPUB or PDF — in `books/`, then choose where the library should live.

### With Docker

[](https://github.com/murerkinn/bookshelf#with-docker)

The shortest path, and the image brings its own `cwebp` and `pdftoppm` so covers come out right without installing anything on the host.

mkdir books && cp ~/Downloads/*.epub books/
docker compose run --rm sync --create
docker compose up -d

The shelf is then on [http://localhost:3000](http://localhost:3000/). Flags pass through, so `docker compose run --rm sync --force` and `--dry-run` behave as they do locally.

One named volume, `library`, holds the published books _and_ everything the app writes to them — profiles and reading positions — so it is the only thing to back up.

The image is built for the filesystem provider: it is a Node server, and Cloudflare needs no container. It runs as a non-root user, and creates `/data` owned by that user so a named volume inherits an ownership the app can write to. If you would rather bind-mount a host directory, chown it first:

chown -R 1000:1000 /srv/bookshelf

### On a machine you own, without Docker

[](https://github.com/murerkinn/bookshelf#on-a-machine-you-own-without-docker)

No account anywhere. Point `bookshelf.config.json` at a directory, publish into it, and run the app:

npm run sync -- --create   # builds library/, then publishes it to shelf-data/
npm run build
npm start -w @bookshelf/app

`library/` is the tree the sync tool builds; `directory` is where it publishes to, and it holds the books being served. Keep it out of the repository — `shelf-data/` already is.

### On Cloudflare

[](https://github.com/murerkinn/bookshelf#on-cloudflare)

Needs a Cloudflare account and `npx wrangler login`. Two checked-in files carry this project's own bucket and Worker name and are meant to be edited — see [the R2 provider](https://github.com/murerkinn/bookshelf/blob/main/docs/providers/r2.md#two-files-carry-a-deployment).

npm run sync -- --create   # creates the bucket, then uploads to it
npm run deploy

The two must agree about which bucket holds the library, or the app will serve an empty shelf. They are checked against each other before anything uploads, and a mismatch is reported rather than published through.

### A demo shelf

[](https://github.com/murerkinn/bookshelf#a-demo-shelf)

No books to hand, or want something worth screenshotting? Nine generated public-domain titles — eight EPUBs and a PDF, so both readers are one click from the shelf — downloading nothing:

npm run demo                 # writes them into books/

Then publish and serve by whichever route above. The titles and authors are real works long out of copyright so a shelf of them looks like a shelf; the prose inside is placeholder. Note that the config in this repository points at R2, so `npm run sync` goes there unless you change it.

### A public instance

[](https://github.com/murerkinn/bookshelf#a-public-instance)

Anything reachable by strangers should refuse to be changed:

Storage keeps serving and stops accepting. Profiles cannot be added, renamed or deleted, and reading positions go back to living in the browser — the same degradation as a provider that cannot write, because to the app it is the same situation. Switching between existing profiles still works; that is a cookie, not a change.

It is enforced where the writing happens, not by hiding the forms, so posting the actions directly gets the same refusal.

**There is no authentication.** Anyone who can reach the app can read and download the whole library, so put it on a network you trust or behind something that asks who is calling. See [Not done yet](https://github.com/murerkinn/bookshelf#not-done-yet).

## Commands

[](https://github.com/murerkinn/bookshelf#commands)

All of these run from the repository root; Turborepo builds whatever the task depends on first.

npm run dev          # local dev server, against the local R2 bucket
npm run sync         # build the library and upload it to the bucket
npm run build        # build every workspace
npm run check-types  # typecheck every workspace
npm run preview      # build + run the Worker locally
npm run deploy       # build + deploy to Cloudflare Workers
npm test             # the test suite
npm run lint         # biome, across the repo

`npm run cf-typegen -w @bookshelf/app` regenerates `cloudflare-env.d.ts` after editing `wrangler.jsonc`.

## Documentation

[](https://github.com/murerkinn/bookshelf#documentation)

|||
|---|---|
|[Publishing a library](https://github.com/murerkinn/bookshelf/blob/main/docs/publishing.md)|the sync tool, its flags, and covers|
|[The library format](https://github.com/murerkinn/bookshelf/blob/main/docs/library-format.md)|what ends up in the bucket, and why it is regenerable|
|[Storage providers](https://github.com/murerkinn/bookshelf/blob/main/docs/providers/README.md)|the contract, and what the two shipped ones can each do|
|[Cloudflare R2](https://github.com/murerkinn/bookshelf/blob/main/docs/providers/r2.md)|configuration, deploying, publishing locally|
|[Filesystem](https://github.com/murerkinn/bookshelf/blob/main/docs/providers/fs.md)|running it on your own machine or a VPS|
|[Profiles](https://github.com/murerkinn/bookshelf/blob/main/docs/profiles.md)|who is reading, and where they got to|
|[Reading in the browser](https://github.com/murerkinn/bookshelf/blob/main/docs/reader.md)|how a chapter reaches the page|
|[Architecture](https://github.com/murerkinn/bookshelf/blob/main/docs/architecture.md)|ports, adapters, and the composition root|
|[The demo library](https://github.com/murerkinn/bookshelf/blob/main/docs/demo.md)|how the public shelf is built, and how to rebuild it|

## Not done yet

[](https://github.com/murerkinn/bookshelf#not-done-yet)

- **There is no authentication.** Anyone with the URL can read and download the whole library — and pick any profile while doing it. Profiles are a way to keep housemates' bookmarks apart, not a way to keep anyone out.
- Two devices reading as one profile at the same time is last-write-wins.

## Contributing

[](https://github.com/murerkinn/bookshelf#contributing)

See [CONTRIBUTING.md](https://github.com/murerkinn/bookshelf/blob/main/CONTRIBUTING.md). The short version: Node 24, `npm install`, and `npm run lint`, `npm run check-types` and `npm test` before you push. The tests reach the packages and the app's service layer but not its pages, so say what you ran as well.

Storage providers are the extension point and do not have to live here: a package published by anyone can be installed and named in the config.

## License

[](https://github.com/murerkinn/bookshelf#license)

MIT — see [LICENSE](https://github.com/murerkinn/bookshelf/blob/main/LICENSE).

That covers the code. It says nothing about the books you put in a library built with it, whose copyright is between you and their publishers.