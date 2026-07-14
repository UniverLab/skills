# CI Workflow Commands & Common Failures

## Cargo binary projects (cadspec, ghscaff, gitkit, texforge, demostage)

```bash
# Generate/fix Cargo.lock
cargo generate-lockfile
cargo check --locked

# Format check
cargo fmt -- --check

# Lint
cargo clippy --locked --all-targets --all-features -- -D warnings

# Tests
cargo test --locked --all-features

# Coverage
cargo llvm-cov --locked --all-features --fail-under-lines 0 --summary-only

# Publish dry-run
cargo publish --locked --dry-run
```

## Node.js projects (quorum)

```bash
# Clean install (must sync lock file with package.json)
npm ci

# Test + coverage
npm run coverage
```

## Version bump for main PRs

- **Rust**: bump `version` in `Cargo.toml`
- **JS**: bump `version` in `package.json`
- CI check: `Main PR Version Bump` fails if branch version == main version

## Common CI failures

1. **Missing Cargo.lock** → `cargo generate-lockfile` + commit
2. **Cargo.lock out of sync** → delete + regenerate + commit
3. **Version not bumped** → edit Cargo.toml/package.json + commit
4. **.gitignore missing entries** → add .mp4, .mimocode/
5. **npm ci fails** → regenerate package-lock.json with `npm install`

## .gitignore must-haves for Rust binary repos

- `.mimocode/` (AI agent workspace)
- `*.mp4` (large video files in demo/dist/)
- **Do NOT ignore `Cargo.lock`** (binary projects need it tracked)
