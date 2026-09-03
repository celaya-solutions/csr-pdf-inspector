> **Celaya Solutions Research Course Edition. Challenge project.** Read [COURSE_EDITION.md](COURSE_EDITION.md) and [UPSTREAM.md](UPSTREAM.md) before you pick it. An instructor has to approve this one. Use fake data only.

# PDF Inspector

Project 07 on the [Zero to Agent project shelf](https://zerotoagent.org/course/landing.html#projects). First of the four Challenge projects.

It reads a PDF and tells you whether the text is really in there or whether the page is just a picture of text. When the text is there, it pulls it out with the layout intact and turns it into clean Markdown. It does that fast, on your own machine, without sending the file anywhere.

## Why this one is a Challenge

Two reasons, and you should hear both before you choose it.

The first is the language. This is Rust, and nothing else in the course is. The course does not teach Rust and will not start. You are on your own for the language, and the six Core projects are not easier work, they are the same work in a language the class shares.

The second is the bigger one. **There is no backend here, and no database. It is a library, not an app.** The capstone asks for a change you can see, a change to what gets stored, a live frontend on Vercel, and a backend that is still running tomorrow. Half of that does not exist in this repo yet. You would be designing and building it yourself: deciding what a user uploads, what gets kept, what the API looks like, and how it goes online.

That is a real project and it can be a great one. It is not a smaller one.

## What you owe before an instructor will approve it

Bring a written plan first. It does not have to be long, but it has to answer:

- What does a person do with this, in one sentence.
- What gets stored, and in what tables.
- Where the backend runs and how the browser reaches it.
- What you will show in the three minute demo.

Then the same five things every project on the shelf has to hit:

1. A change you can see on the screen.
2. A change to the server or to what gets stored.
3. The frontend live on Vercel.
4. The backend running on Railway, still running tomorrow.
5. A three minute demo: the problem, the before, the after.

## What is in here already

| Path | What it is |
| --- | --- |
| `src/` | The Rust library: detection, extraction, table finding, Markdown conversion |
| `src/bin/` | The two command line tools, `pdf2md` and `detect-pdf` |
| `wasm/` | The browser build, so the same parser runs in a web page |
| `napi/` | The Node build |
| `site/` | A small static page that uses the browser build |
| `tests/`, `examples/` | Tests and sample files |
| `docs/` | Reference for the Rust, Python, and Node interfaces, plus benchmarking and debugging notes |

There is no `src/server`, no schema, and no place data is kept. That is the work.

## Running it

The toolchain version is pinned in `rust-toolchain.toml`. Install Rust, then:

```bash
cargo build
cargo test
cargo run --bin pdf2md -- yourfile.pdf
cargo run --bin detect-pdf -- yourfile.pdf --json
```

Optional character recognition for scanned pages is a build flag, and it needs two more libraries installed separately. You do not need it to pass. If you want it, [docs/ocr-runtime.md](docs/ocr-runtime.md) has the setup.

## Source and license

Imported from an open source PDF library. The source project, the exact commit, and the course status are recorded in [UPSTREAM.md](UPSTREAM.md). The original MIT license and copyright notice are kept in [LICENSE](LICENSE) and stay with any copy you make.

The package names, the crate name, and the registry identifiers still carry the source project's names, and they stay that way. Changing them would break the build and would take credit that is not ours. This copy is frozen: it has no link back to the source project and does not take its updates.

This is a course edition, not a product. It is free and noncommercial, and the Celaya Solutions Research Course Edition notice stays on it.
