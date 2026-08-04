# Quasar Guidelines (Quasar Programs)

Quasar is a zero-copy, zero-allocation Solana framework with Anchor-like syntax (`#[program]`, `#[derive(Accounts)]`, `#[account]`) but much better runtime performance: zero-copy account access, `no_std`, dense instruction discriminators, and compiled output roughly an order of magnitude smaller than equivalent Anchor binaries. Programs use the `quasar-lang` and `quasar-spl` crates, plus `quasar-test` for testing. (`quasar-metadata` no longer exists upstream - it was removed before the 0.1.0 release with no replacement. Metaplex-metadata programs on the 0.1.0 line vendor the last shipped copy of the crate, adapted to the 0.1.0 API, as a path dependency; `quicknode/solana-program-examples` carries such a copy at `tokens/quasar-metadata`.)

Read this alongside the general rules in [SKILL.md](SKILL.md) and the shared Rust + onchain-math rules in [RUST.md](RUST.md) (checked arithmetic, financial math, PDA bumps, project structure - those apply to Quasar too). The surface is familiar to Anchor developers, but several conveniences work differently or are absent.

## Versions and installation (0.1.0)

Quasar's public launch line is **0.1.0**, developed on the `0.1.0-release` branch of `https://github.com/blueshift-gg/quasar`. Its versioning contract (`VERSIONING.md` upstream): `0.1.z` is one compatible release line - the documented `quasar-lang`/`quasar-spl`/`quasar-test` Rust APIs, macro behavior, IDL wire format, generated-client signatures, and stable CLI commands stay compatible within it, and deliberate breaks go to `0.2.0`. (`quasar-cli` as a *library* has no stability promise - only its CLI commands do.)

At the time this skill was updated (July 2026), crates.io still hosts `0.0.0` placeholders for `quasar-lang` and `quasar-cli` - only `quasar-svm 0.1.0` is actually published - so the 0.1.0 line installs from git, pinned by rev to the `0.1.0-release` branch head:

```toml
[dependencies]
quasar-lang = { git = "https://github.com/blueshift-gg/quasar", rev = "be60fca" }

[dev-dependencies]
quasar-test = { git = "https://github.com/blueshift-gg/quasar", rev = "be60fca" }
```

and the CLI with:

```bash
cargo install --git https://github.com/blueshift-gg/quasar --rev be60fca quasar-cli --locked
```

Keep the dependency rev and the installed CLI rev in lockstep. Once the 0.1.0 crates are published, switch to `quasar-lang = "=0.1.0"` / `cargo install quasar-cli --version 0.1.0 --locked`.

## Quasar.toml (0.1.0 schema)

The canonical project file:

```toml
[project]
name = "my-program"

[testing]
command = { program = "cargo", args = ["test"] }

[clients]
path = "target/client"
targets = ["rust"]
```

`clients.targets` accepts `rust`, `kit`, `web3`, `python`, `go`, `c` (Python/Go/C are preview). The pre-0.1.0 keys are **hard parse errors**, not deprecations - migrate them:

| Old (pre-0.1.0) | 0.1.0 |
|---|---|
| `[toolchain] type = "solana"` | delete the section |
| `[testing] language = "rust"` | delete |
| `[testing.rust] framework = "quasar-svm"` + `[testing.rust.test]` | `[testing] command = { program = "cargo", args = ["test"] }` |
| `[testing.typescript]` | removed - testing is Rust `quasar-test` |
| `[clients] languages = [...]` | `targets = [...]` |

## Cargo.toml scaffold (0.1.0)

What `quasar init` generates, and what existing projects must match:

```toml
[lints.rust.unexpected_cfgs]
level = "warn"
check-cfg = ['cfg(target_os, values("solana"))']

[lib]
# "lib" is required alongside "cdylib": the IDL build compiles the crate
# as a host library.
crate-type = ["cdylib", "lib"]

[features]
alloc = []
client = []
debug = []
idl-build = ["quasar-lang/idl-build"]

[dependencies]
quasar-lang = { git = "https://github.com/blueshift-gg/quasar", rev = "be60fca" }
solana-instruction = { version = "3.2.0" }

[dev-dependencies]
quasar-test = { git = "https://github.com/blueshift-gg/quasar", rev = "be60fca" }
```

The declared `idl-build` feature and the `"lib"` crate-type are hard requirements of `quasar build` - it generates the IDL and clients first (host build), then compiles the program with `cargo build-sbf --tools-version v1.52`. The IDL step runs `cargo metadata --locked`, so in a checkout without a committed `Cargo.lock` run `cargo generate-lockfile` before the first `quasar build`.

## Quasar has a `Vec` - do not claim otherwise

Quasar's prelude re-exports bounded, zero-copy collection types from the `zeropod` crate:

```rust
// quasar_lang::prelude (via quasar_lang::pod → zeropod::pod)
pub use crate::pod::{PodString as String, PodVec as Vec};
```

So in a Quasar program, `String` and `Vec` already mean Quasar's bounded types:

- **`Vec<T, const N: usize, const PFX: usize = 2>`** (`PodVec`) - a fixed-capacity vector. `N` is the capacity (maximum element count). The element `T` must be a fixed-size zero-copy type (`ZcElem`) - e.g. `u8`, `[u8; K]`, or one of the alignment-1 `PodU*` scalar wrappers below. The third generic is the length-prefix width in bytes, defaulting to `2` (a `u16` prefix).
- **`String<const N: usize, const PFX: usize = 1>`** (`PodString`) - a fixed-capacity UTF-8 string. `N` is the capacity in bytes. The second generic is the length-prefix width, defaulting to `1` (a `u8` prefix, so stored lengths up to 255). For longer strings, set it explicitly, e.g. `String<1024, 2>` for a `u16` prefix.

The capacity is **mandatory**: bare `String` / `Vec` (without `<N>`) are not accepted. Always supply `<N>`.

This is Quasar's equivalent of Anchor's `#[max_len(N)]` - the capacity lives in the type, not in an attribute. Where Anchor writes:

```rust
#[max_len(50)]
pub name: String,
```

Quasar writes:

```rust
pub name: String<50>,
```

### Field layout constraints

- **Fixed-size fields must precede dynamic ones.** In an account struct, put every fixed-size field (`u8`, `u64`, `Address`, `[u8; K]`, …) first, then the `String<N>` / `Vec<T, N>` fields.
- **No *nested* dynamic types.** A `Vec` of a dynamic element (e.g. `Vec<String<50>, N>`) is not supported - the element must be a fixed-size POD. This is the real limitation: "Quasar has no Vec" is wrong; "Quasar has no Vec of a *dynamic* element" is correct. (It is why a port of an Anchor account with `hobbies: Vec<String>` has to drop or restructure that field.)
- **There is no `realloc` account constraint** and no fixed maximum to plan around manually: declare dynamic fields with `String<N>` / `Vec<T, N>`, and `set_inner` automatically resizes the account when a dynamic field grows.

## Reading and writing account fields

Account state structs are declared with friendly native types (`u64`, `i64`, `u16`, `u8`, `Address`, `String<N>`, …) under `#[account(discriminator = N, set_inner)]`, but the generated accessors are **Pod-wrapped** for zero-copy access. Assign with a `Pod*` value and read back with `.into()`:

```rust
// write
accounts.counter.count = PodU64::from(current.checked_add(1).unwrap());
// read
let current: u64 = accounts.counter.count.into();
```

The same applies to `PodU32`, `PodI64`, `PodBool`, etc. (and a field may be declared directly as `PodBool`/`PodU64` too). Forgetting the conversion is a compile error or produces a wrong value.

## Other prelude types

- **`Address`** - Quasar's public-key type. Use `Address` (not `Pubkey`) in account fields and in `#[seeds(...)]`, e.g. `#[seeds(b"favorites", user: Address)]`.
- **Alignment-1 POD scalars**, re-exported from `zeropod::pod`: `PodBool`, `PodU16`/`PodU32`/`PodU64`/`PodU128`, `PodI16`/`PodI32`/`PodI64`/`PodI128`, and `PodOption`. Use these for zero-copy field storage and as `Vec` elements.

## Macros and entrypoints (vs Anchor)

- `#[program]` module, `declare_id!(...)`, and `#[instruction(discriminator = N)]` on each handler.
- Account state structs use `#[account(discriminator = N, set_inner)]` and `#[seeds(b"prefix", field: Address)]`.
- The instruction context type is **`Ctx<T>`**, not Anchor's `Context<T>`. Handlers return `Result<(), ProgramError>`.
- **`#[derive(Accounts)]` structs are written without explicit `<'info>` lifetimes** (unlike Anchor - e.g. `pub struct TakeOffer { ... }`, no lifetime parameter).
- Programs are `no_std`: begin `lib.rs` with `#![cfg_attr(not(test), no_std)]`.
- **Use the same program ID as the Anchor build** when a program ships in both frameworks. Offchain tooling derives PDAs from the program ID; matching IDs means clients work against either build unchanged.

## Logging

`log()` takes a `&str` only - no `format!`, no interpolation (it is `no_std`). To log a value, issue multiple `log()` calls with separate static strings, or log the raw bytes.

## CPI

- **SPL token CPIs use the helper style on the token-program handle**, then `.invoke()` (or `.invoke_signed(&seeds)?` when a PDA signs):

  ```rust
  accounts
      .token_program
      .transfer(&accounts.from, &accounts.to, &accounts.authority, amount)
      .invoke()?;
  ```

- **Generic CPIs** use `CpiCall::new(...)` with `InstructionAccount`s for a fixed, known set of accounts, or `CpiDynamic::<MAX_ACCOUNTS, MAX_DATA>::new(...)` when the account list is built dynamically, then `.invoke()` / `.invoke_signed(...)`:

  ```rust
  let mut cpi = CpiDynamic::<1, 128>::new(accounts.lever_program.address());
  // ... push accounts / data ...
  cpi.invoke()?;
  ```

  Anchor's `CpiContext::new(program, accounts).invoke()` does not exist in Quasar.

- **Closing accounts uses the `close(dest = ...)` constraint**, mirroring Anchor's `close = ...`:

  ```rust
  #[account(mut, close(dest = user))]
  pub thing: Account<Thing>,
  ```

  The generated epilogue moves the account's lamports to `dest` and clears its data. Do not write "Quasar has no close constraint" - it does.

## Testing Quasar programs (quasar-test)

Quasar 0.1.0 programs test with the **`quasar-test` fixture harness** (which drives QuasarSVM internally - the SVM is an implementation detail tests never touch). The only test dev-dependency is `quasar-test` itself; it pulls the published `quasar-svm 0.1.0` from crates.io. The pre-0.1.0 pattern - a direct `quasar-svm` git dependency, `QuasarSvm::new().with_program(...)`, `include_bytes!`, `process_instruction`, `assert_success()` - is gone.

Run tests with `quasar test` (builds first), or plain `cargo test` when a compiled artifact already exists: `#[quasar_test]` discovers the program `.so` in `target/deploy/` at **runtime** - there is no `include_bytes!`, so always `quasar build` after changing program code before re-running `cargo test`.

The test shape (this is a complete real test):

```rust
use {
    crate::{
        cpi::{IncrementInstruction, InitializeCounterInstruction},
        state::Counter,
    },
    quasar_test::prelude::*,
};

// Deterministic addresses keep tests independent of discovery order.
const PAYER: Pubkey = Pubkey::new_from_array([1; 32]);

#[quasar_test]
fn initialize_counter_creates_the_pda(test: &mut Test) {
    test.add(Wallet::new().at(PAYER));
    let counter = test.derive_pda(Counter::seeds(&PAYER));

    // The counter PDA and system program are canonical derivations, so the
    // generated instruction only asks for the payer.
    test.send(InitializeCounterInstruction { payer: PAYER }).succeeds();

    let state = test.read::<Counter>(counter);
    assert_eq!(u64::from(state.count), 0);
}
```

The pieces:

- **`#[quasar_test] fn name(test: &mut Test)`** behaves like an ordinary `#[test]`: filters, `#[ignore]`, `#[should_panic]`, and `Result<(), E>` returns all work.
- **`crate::cpi` instruction builders**: `#[program]` generates an in-crate `cpi` module (host builds only) with one `{PascalFnName}Instruction` struct per handler. Fields are the **caller-controlled** accounts (as `Pubkey`) plus the instruction arguments; accounts pinned by `address = X::seeds(...)`, `Program<...>`, `Sysvar<...>`, and canonical ATAs are derived automatically and omitted. Prefer these over the generated client crate in tests - a `path = "target/client/rust/..."` dev-dependency breaks `cargo generate-lockfile`/`cargo metadata` before the first build. (The build-generated client crate additionally has `{Name}InstructionRaw`, `find_*_address` helpers, and account/event decoders for offchain consumers.)
- **Fixtures are values** added with `test.add(...)`: `Wallet::new().at(P).lamports(n)` (10 SOL default, `DEFAULT_WALLET_LAMPORTS`), `Mint::new(authority).at(M).supply(n).decimals(d)`, `TokenAccount::new(mint, owner).at(A).amount(n)`, `AssociatedTokenAccount::new(mint, owner).amount(n)`, `Program::new(id, elf)`; add `.token_program(TokenProgram::Token2022)` for Token-2022 variants. Custom protocol states implement `Fixture` once and are reused across tests. Do NOT pre-create empty accounts for init targets: missing writable accounts enter the transaction as empty system accounts, committed on success and left as no placeholder on failure.
- **World helpers**: `test.derive_pda(X::seeds(&key))` / `derive_pda_with_bump`, `test.derive_ata(owner, mint, TokenProgram::Token)`, typed state seeding with `test.write(addr, XInner { ... })`, raw bytes with `test.set_account(Account::new(addr, owner, lamports, data))`, `test.warp_to_timestamp(t)` (sets the clock's `unix_timestamp` only - there is no slot warp in `quasar-test`; a test that must control `Clock.slot` needs a direct `quasar-svm = "=0.1.0"` dev-dependency and the low-level API).
- **`test.send(ix)` returns an `Outcome`** (also `send_all([...])` for atomic sequences, `simulate` for no-commit): assert with `.succeeds()`, `.fails_with(MyError::Variant)` (`#[error_code]` enums implement `Into<u32>`), `.fails(ProgramError::MissingRequiredSignature)`, `.cu_at_most(n)`, `.has_lamports(addr, n)`, `.has_tokens(addr, n)`, `.has_supply(mint, n)`, `.is_closed(addr)`; inspect with `outcome.account_changes()`, `.events()`, `.return_value()`, `.logs()`, `.compute_units()`. Read typed post-state with `test.read::<X>(addr)` (Pod fields convert with `u64::from(...)`).
- **Negative tests** (wrong PDA, wrong signer): `crate::cpi` has no `Raw` variant - convert and tamper instead: `let mut ix: Instruction = FooInstruction { .. }.into(); ix.accounts[i].pubkey = WRONG; test.send(ix).fails_with(...)`. Account order matches the `#[derive(Accounts)]` struct field order.
- **CPI across programs**: every sibling `.so` with a matching `-keypair.json` in the same `target/deploy` is preloaded automatically; a program from another project's target dir is added explicitly with `test.add(Program::new(OTHER_ID, &std::fs::read("../other/target/deploy/other.so").unwrap()))`.
- Use deterministic addresses (`Pubkey::new_from_array([1; 32])`), never `Pubkey::new_unique()` - the world's generated addresses are deterministic per test and unique-counter addresses depend on test discovery order.

## Fight for Truth about Quasar

Per the truthfulness rules in SKILL.md: do not write "Quasar has no `Vec`" or "Quasar has no dynamic types". It has bounded `Vec<T, N>` and `String<N>`; the genuine limitations are (1) no nested dynamic types and (2) fixed-size fields must come before dynamic ones. Before writing any claim about Quasar's type system or APIs, grep the `quasar_lang` prelude or an existing Quasar example to confirm it - Quasar's API moves, and stale notes are everywhere: old names (`BufCpiCall`, mandatory `<'info>` lifetimes), the removed `quasar-metadata` crate, the pre-0.1.0 Quasar.toml keys (`[toolchain]`, `testing.rust`, `clients.languages`), and the pre-0.1.0 QuasarSVM test API (`with_program`, `process_instruction`, `assert_success`, `create_keyed_system_account`, a git `quasar-svm` dev-dependency) all describe versions that no longer exist on the 0.1.0 line.
