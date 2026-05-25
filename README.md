# Log analysis (refactoring project)

Rust library and CLI for parsing and filtering trading-system log lines. This crate is the **Module 6** coursework project: refactor legacy prototype code toward idiomatic Rust while keeping all tests and CLI output unchanged.

## Requirements

- [Rust](https://www.rust-lang.org/) (edition 2024; stable toolchain recommended)

## Quick start

From the project directory:

```bash
# run all tests (recommended: show println output)
cargo test -- --nocapture

# run the CLI on the sample log
cargo run example.log

# run a single integration test
cargo test test_all -- --nocapture
```

## Project layout

| Path | Role |
|------|------|
| `src/lib.rs` | `read_log`, `ReadMode`, integration test `test_all` |
| `src/parse.rs` | Combinator-style parser, domain types (`LogLine`, `LogKind`, …) |
| `src/main.rs` | CLI binary (`cli`) |
| `example.log` | Sample log file (same data as in tests) |

## Public API (library)

- **`ReadMode`** — `All`, `Errors`, or `Exchanges` (journal operations: buy/sell, users, assets, cash).
- **`read_log(reader, mode, request_ids)`** — read from any `std::io::Read`, skip empty lines, parse each line into `LogLine`, optionally filter by `request_id` list (empty slice = no filter).
- **`parse::parse_log_line(line)`** — parse one trimmed log line; returns `None` on failure or trailing garbage.
- **`parse::just_parse_anouncements(input)`** — parse an announcements list string (used by the CLI demo).

## Refactoring summary

The original prototype was refactored along the hints left in the code (Yandex Module 6 checklist):

| Topic | Change |
|-------|--------|
| Allocations | `Parser` takes `&str`; remainder is a substring, not owned `String` |
| Shared ownership | Removed `Rc<RefCell<Box<dyn MyReader>>>`; `read_log` accepts `R: Read` directly |
| `unsafe` | Removed `transmute` lifetime hack in the log iterator |
| Iterators | `read_log` built from iterator adapters instead of manual `for` + `Vec::push` |
| Control flow | `ReadMode` enum + `match` instead of `u8` constants and `if` chains; no `panic!` on unknown mode |
| Tight types | `NonZeroU32` / `NonZeroI32` for numeric parsers that reject zero |
| Large enum variants | `AuthData` stored in `Box<[u8; 1024]>`, `Connect(Box<AuthData>)` |
| Singleton | Removed `LOG_LINE_PARSER` / `OnceLock`; use `parse_log_line` or `LogLine::parser()` |
| Duplication | Dropped redundant `just_parse_*` wrappers; kept `just_parse_anouncements` |
| `Alt` combinators | Short-circuit via `.or_else` instead of cloning input |

Some **unused scaffolding** from the original repo (`AsIs`, `Either`, `all3`/`all4`, `Status`, etc.) is intentionally left in place per assignment notes (“unimplemented features — fixing not required”). They may trigger `dead_code` warnings; behavior and tests are unaffected.

## CLI output

`cargo run example.log` prints a short demo parse, then opens `example.log` and lists parsed `LogLine` values under `got logs:` — same semantics as `test_all` in `lib.rs`.

## License

See [LICENSE](LICENSE).
