# Environment Setup (CI and remote execution)

How to get a working Solana toolchain in a fresh Linux container (CI runners, Claude Code on the web, devcontainers). Verified June 2026 against `quicknode/solana-program-examples`.

## Agave (Solana CLI) and cargo-build-sbf

```bash
curl -sSfL https://release.anza.xyz/stable/install -o /tmp/solana-install.sh
sh /tmp/solana-install.sh
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"
```

`cargo build-sbf` downloads its platform-tools toolchain on first use. In environments with a TLS-intercepting proxy this download fails with `invalid peer certificate: UnknownIssuer`, because `cargo-build-sbf` uses its own certificate store rather than the system's. Work around it by downloading with `curl` (which trusts the system store) and placing the archive where `cargo-build-sbf` caches it:

```bash
# Match the version cargo-build-sbf asks for in its error message (vX.YY).
PT_VERSION=v1.53
curl -sSfL "https://github.com/anza-xyz/platform-tools/releases/download/${PT_VERSION}/platform-tools-linux-x86_64.tar.bz2" -o /tmp/platform-tools.tar.bz2
mkdir -p ~/.cache/solana/${PT_VERSION}/platform-tools
tar -xjf /tmp/platform-tools.tar.bz2 -C ~/.cache/solana/${PT_VERSION}/platform-tools
```

Different tools pin different platform-tools versions (the Quasar CLI pins its own); repeat the download for each version requested in error messages.

## Anchor CLI

Install from crates.io (a long compile, roughly ten minutes):

```bash
cargo install anchor-cli --locked
```

`anchor test` reads the wallet path from `Anchor.toml` (`~/.config/solana/id.json` by default), so create a throwaway keypair once per container:

```bash
solana-keygen new --no-bip39-passphrase -s -o ~/.config/solana/id.json
```

In an existing repository the original program keypairs are usually not committed, so `anchor build` fails its program-ID check against the freshly generated keypair. Do NOT run `anchor keys sync` (it would change the program IDs, which this skill forbids); build with the check skipped, and test against the in-process SVM with no validator and no deploy:

```bash
anchor build --ignore-keys
anchor test --skip-local-validator --skip-deploy
```

(Verified: this runs the project's `test = "cargo test"` LiteSVM suite. Without `--skip-local-validator` anchor tries to spawn a `surfpool` validator; without `--skip-deploy` it tries to deploy to `localhost:8899`.)

### Without the anchor CLI

`anchor build` and `anchor test` are wrappers. If the anchor CLI is not installed, the equivalent is:

```bash
cargo build-sbf   # produces target/deploy/<name>.so
cargo test        # runs the LiteSVM tests, which include_bytes! the .so
```

Always rebuild the `.so` after changing program code, before re-running tests: the tests embed the binary at compile time via `include_bytes!`, so a stale `.so` silently tests old code.

## Quasar CLI

The 0.1.0 line still installs from git (crates.io hosts `0.0.0` placeholders for `quasar-cli`). Pin the exact rev your project's dependencies pin - as of the 0.1.0 launch that is the `0.1.0-release` branch head:

```bash
cargo install --git https://github.com/blueshift-gg/quasar --rev be60fca quasar-cli --locked
```

Per project, run `cargo generate-lockfile` once in a fresh checkout (the 0.1.0 IDL step runs `cargo metadata --locked`), then `quasar build` (generates the IDL and the `target/client/rust/<name>-client` crate, then compiles the program) and `quasar test` or `cargo test`. The 0.1.0 CLI pins platform-tools **v1.52** (`cargo build-sbf --tools-version v1.52`) and refuses a `cargo-build-sbf` whose bundled platform-tools is older, so the platform-tools workaround above may be needed for v1.52 specifically.

## Disk hygiene

Each Anchor build target is 1-2 GiB. Run `cargo clean` (and `quasar clean`) per project after its tests pass, especially when building many examples in one session.

## Repository session hooks

For repositories used with Claude Code on the web, put the steps above in a SessionStart hook or setup script so every fresh container gets a working toolchain without manual intervention. See https://code.claude.com/docs/en/claude-code-on-the-web for environment configuration.
