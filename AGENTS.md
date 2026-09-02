# pdf-inspector

Fast PDF text extraction to structured Markdown. CLI binary: `pdf2md`. Detection binary: `detect-pdf`.

## Build & Test

```bash
cargo fmt                                    # format
cargo clippy -- -D warnings                  # lint (enforced, zero warnings)
cargo test                                   # unit + integration tests (267+ unit, 73+ integration)
cargo build --release                        # release binary for benchmarks
```

All three must pass before committing.

## Binaries

- `pdf2md` — extract PDF → Markdown. Supports `--json` for structured output.
- `detect-pdf` — classify PDF type (TextBased/Scanned/Mixed/ImageBased). Supports `--analyze --json`.

## Architecture

```
src/
  lib.rs                        – public API, process_pdf_with_options, encoding issue detection
  detector.rs                   – PDF type classification, tiled-scan detection, page sampling
  types.rs                      – TextItem, TextLine, PdfRect, PdfLine
  tounicode.rs                  – CMap/ToUnicode parsing, CID decoding
  text_utils.rs                 – CJK/RTL handling, Otsu threshold, ligature expansion, NFKC
  extractor/
    mod.rs                      – top-level extraction orchestrator
    content_stream.rs           – PDF operator state machine (Tj/TJ/Td/Tm/q/Q)
    fonts.rs                    – font width/encoding, CMapDecisionCache, TrueType cmap fallback
    layout.rs                   – column detection (histogram), newspaper/tabular classification,
                                  spanning-line pre-masking, sidebar detection
  tables/
    detect_rects.rs             – rect-based table detection (union-find clustering)
    detect_heuristic.rs         – heuristic table detection (gap-histogram, body-font tables)
    detect_lines.rs             – line-based table detection (H/V line grids)
    grid.rs                     – column/row boundaries, cell assignment
    format.rs                   – table→Markdown formatting, continuation row merging
  markdown/
    convert.rs                  – core line→Markdown loop, struct-tree role support
    analysis.rs                 – font stats, heading tiers, paragraph thresholds
    classify.rs                 – line classification (header, list, code, caption)
    preprocess.rs               – drop cap merging, heading line merging
    postprocess.rs              – dot leaders, hyphenation, page numbers, URL formatting
```

## Key design decisions

- **Primary audience is AI agents.** Output optimized for token efficiency and semantic quality, not visual formatting. No cosmetic padding.
- **Three table detection strategies** run in priority order: rect-based → line-based → heuristic. First valid result wins.
- **Column detection** uses horizontal projection histograms with valley detection. Multi-item spanning lines (titles, headers) are pre-masked using column-aware thresholds before column assignment.
- **Newspaper vs tabular** classification determines reading order: newspaper reads columns sequentially, tabular Y-interleaves them.
- **Tiled-scan detection** catches scanned PDFs with JBIG2/strip images where no single tile exceeds the template threshold but aggregate area does (≥2M pixels).
- **Garbage text upgrade** reclassifies Mixed PDFs as Scanned when extracted text is <50% alphanumeric.
- **Tagged PDF support** uses structure tree roles (H1-H6, P, L, Code, BlockQuote) when available, falling back to font-size heuristics.

## Testing

- **Unit tests**: inline `#[cfg(test)] mod tests` in each module with synthetic data.
- **Integration tests**: `tests/integration_tests.rs` with fixture PDFs in `tests/fixtures/`.
- **Regression suite**: sibling repo `pdf-evals` with ~200 snapshot PDFs. Run `cargo build --release` then `bench.py test` in that repo before committing. While iterating, prefer a subset run (`bench.py test -q` for the quick set, or `-s <name>` for a named test set) and save the full `bench.py test` for the final pre-commit check.
- **Semantic quality**: run `bench.py score` in `pdf-evals` for the semantic verdict (TEDS + MHS + reading order + char/word + list preservation, composited). Character-level diff alone misclassifies structural improvements (e.g., column-detection rewrites) as regressions — `score` is the tie-breaker. See `pdf-evals/CLAUDE.md` "Semantic scoring".

## Debugging

```bash
RUST_LOG=pdf_inspector::extractor::layout=debug cargo run --bin pdf2md -- file.pdf
RUST_LOG=pdf_inspector::tables=debug cargo run --bin pdf2md -- file.pdf
RUST_LOG=pdf_inspector::detector=debug cargo run --release --bin detect-pdf -- file.pdf
```

## Conventions

- Clippy: use `is_some_and(...)` not `map_or(false, ...)`
- lopdf quirk: `ParseError` is private — match by string for `InvalidFileHeader`
- Column limit for tables: 25 (wide statistical tables)
- `propagate_merged_cells` skipped for >10 columns (spanning rects = background fills)

<!-- graft:start -->
## Graft — repo context graph

This repo is indexed in `graft/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `graft map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `graft ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `graft grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `graft skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `graft callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `graft/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `graft ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `graft build` (deterministic,
no API key, $0).
<!-- graft:end -->
## Course edition change log

- Added visible Celaya Solutions Research site branding, pointed course links to the CSR repository, and documented the frozen, no-upstream-sync policy.
- Imported the upstream source as a Challenge project with preserved license and attribution. Student selection requires instructor approval.
