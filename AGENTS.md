# AGENTS.md - npm Guardian Development Guide

> npm 供应链安全哨兵 | Rust + Tauri | macOS

---

## 1. Build & Test Commands

### Workspace Commands
```bash
# Check all crates compile
cargo check

# Build entire workspace
cargo build

# Build specific crate
cargo build -p npm-guardian-scanner
cargo build -p npm-guardian-app

# Run all tests
cargo test

# Run single test (by name pattern)
cargo test test_name_pattern

# Run tests in specific crate
cargo test -p npm-guardian-scanner

# Run tests in specific module
cargo test -p npm-guardian-scanner --lib models

# Run single test with exact path
cargo test -p npm-guardian-scanner --lib models::test_exact_name

# Run tests with output
cargo test -- --nocapture

# Run with release optimizations
cargo test --release
```

### Threat Intelligence
```bash
# Generate threat feed (when scripts exist)
python threat-intel/scripts/generate.py

# Run threat intel tests
cargo test -p npm-guardian-scanner threat_intel
```

### Formatting & Linting
```bash
# Format all code
cargo fmt

# Check formatting
cargo fmt --check

# Run clippy linter
cargo clippy

# Clippy with fixes
cargo clippy --fix

# Clippy for specific crate
cargo clippy -p npm-guardian-scanner
```

### Tauri App
```bash
# Run Tauri dev mode (when configured)
cargo tauri dev

# Build Tauri app
cargo tauri build
```

---

## 2. Code Style Guidelines

### Imports
- Group imports: std → external crates → local modules
- Use `use` statements at module level, not inline
- Avoid wildcard imports (`use foo::*`)
- Sort imports alphabetically within groups
- Re-export public API in `lib.rs`

### Formatting
- 4-space indentation (Rust default)
- Max line length: 100 characters
- Trailing commas in multi-line structures
- Use `cargo fmt` before commits
- No trailing whitespace

### Types & Naming
- **Types**: `PascalCase` (structs, enums, traits)
- **Functions/Variables**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE`
- **Modules**: `snake_case`
- Use descriptive names over abbreviations
- Prefix test modules with `mod tests`
- Enum variants: `PascalCase`

### Error Handling
- Use `thiserror` for error types (when added)
- Return `Result<T, Error>` for fallible operations
- Never use `.unwrap()` in production code
- Use `.expect("clear message")` only in tests
- Provide context in error messages (include paths, package names)
- Define crate-level `Error` enum with variants for each failure mode

### Testing
- Place tests in `#[cfg(test)] mod tests` at bottom of files
- Use `#[test]` attribute for unit tests
- Use fixtures from `tests/fixtures/` for integration tests
- Test both success and failure paths
- Name tests descriptively: `test_<scenario>_<expected_behavior>`
- Use `assert_eq!`, `assert!`, `matches!` macros

### Documentation
- Add `///` doc comments to public APIs
- Include examples for complex functions
- Document error conditions in `/// # Errors` section
- Update AGENTS.md when conventions change

### Architecture Conventions
- Follow PRD module structure: `models`, `scanner`, `matcher`, `threat_intel`, `lockfile`
- Keep crates independent where possible
- Use `Dependency` as unified model across parsers
- Never silently swallow errors - propagate or log
- `Safe` status requires: no risks, no partial failures, fresh data

### Security Rules
- Never upload user data or project paths
- Only request `storage.googleapis.com` and `raw.githubusercontent.com`
- MVP: no auto-delete, no auto-upgrade
- Validate all external data against schema
- Use ETag caching for threat feed

### Git Conventions
- Commit messages: imperative mood ("Add feature" not "Added feature")
- One logical change per commit
- Include test updates with feature changes
- Never commit secrets or API keys

---

## 3. Project Structure

```
npm-guardian/
├── Cargo.toml                    # Workspace root
├── crates/
│   ├── npm-guardian-scanner/     # Core scanning library
│   │   └── src/
│   │       ├── lib.rs            # Module declarations
│   │       ├── models.rs         # Domain types
│   │       ├── scanner.rs        # Orchestration
│   │       ├── matcher.rs        # Version matching
│   │       ├── threat_intel.rs   # Feed loading/cache
│   │       └── lockfile/         # Lockfile parsers
│   └── npm-guardian-app/         # Tauri application
│       └── src/
│           ├── main.rs           # Entry point
│           ├── tray.rs           # System tray
│           ├── commands.rs       # Tauri commands
│           └── state.rs          # Shared state
├── threat-intel/                 # Threat data
├── doc/                          # PRD, tasks
└── target/                       # Build artifacts (gitignored)
```

---

## 4. Key Development Notes

- **Edition**: Rust 2021
- **Resolver**: v2 (workspace)
- **Platform**: macOS (MVP)
- **Status Model**: `Safe` | `Scanning` | `RiskFound` | `AttentionNeeded`
- **Lockfile Support**: package-lock.json, yarn.lock v1, pnpm-lock v6/v9
- **Never mark unsupported formats as Safe** - use AttentionNeeded or skip
