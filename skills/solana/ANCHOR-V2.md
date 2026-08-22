# Porting an Anchor Program to Anchor v2

Rules for moving a program from Anchor 1.x to 2.0.0-rc.1, and for writing v2 from scratch. Read these alongside [ANCHOR.md](ANCHOR.md), which covers the rules both versions share, and [RUST.md](RUST.md).

Anchor 1.x is still the stable line. Write v1 unless the project has chosen v2.

## v2 Is a Rewrite, Not a Version Bump

Four foundations changed, and every rule below follows from one of them:

- The crate is `no_std` and built on **pinocchio**, so `AccountInfo` becomes `AccountView` and the `'info` lifetime disappears.
- `#[account]` is **zero-copy** and requires a `Pod` layout.
- Instruction data and account state are encoded with **wincode** rather than borsh.
- CPI account structs take **typed handles** (`CpiHandle`, `CpiHandleMut`) instead of cloned `AccountInfo`s, and the runtime enforces account borrows separately from the compiler.

The last one causes most of the runtime failures. A program can compile cleanly and fail every test with `AccountBorrowFailed`.

## Manifest Changes Every Crate Needs

- `anchor-lang = "2.0.0-rc.1"` and `anchor-spl = "2.0.0-rc.1"`.
- `wincode = { version = "0.5", features = ["derive"] }` as a **direct** dependency of every program crate. The `#[program]` macro expands to `wincode` paths for instruction-data serialization, so the crate has to name it.
- Drop `features = ["init-if-needed"]`. The constraint is always available.
- Drop `"anchor-spl/idl-build"` from the `idl-build` feature. anchor-spl has no such feature.
- anchor-spl's feature list is only `guardrails` (on by default) and `metadata`. `token_2022`, `token_2022_extensions` and `spl-token-interface` are gone, and those modules are unconditional now.

A program that stores an `Address` in serialized state also needs a version pin:

```toml
# anchor-lang 2.0.0-rc.1 is built against wincode 0.5, but solana-address 2.7
# moved to wincode 0.6. With both in the graph, `Address`'s wincode impls belong
# to the version the account derives are not using, and every `SchemaRead` /
# `SchemaWrite` bound fails. 2.6.1 is the last release on the 0.5 line.
solana-address = ">=2.6, <2.7"
```

Write the range with a lower bound. A bare `<2.7` lets cargo satisfy the requirement by reusing an older 1.x already in the graph, which fixes nothing.

## Handler Signatures and Renamed Accessors

- `Context<T>` becomes `&mut Context<T>`, and the user `'info` lifetime is gone: `Context<'info, T<'info>>` is `Context<T>`.
- `Signer<'info>` and `Program<'info, T>` are `Signer` and `Program<T>`. `Interface<'info, TokenInterface>` is the one wrapper that keeps a lifetime, as `Interface<'static, TokenInterface>`.
- `Pubkey` is `Address`. `AccountInfo<'info>` is `AccountView`, or `UncheckedAccount` inside an `Accounts` struct.
- `.key()` and `.key` become `.address()`, which returns `&Address`. Dereference where a value is wanted.
- `.to_account_info()` becomes `.cpi_handle()` or `.cpi_handle_mut()`.
- `ctx.remaining_accounts` is now the method `ctx.remaining_accounts()?`. It is fallible, takes `&mut self`, and returns an owned `Vec<AccountView>`, so call it before anything borrows `ctx.accounts`.
- `account.owner` is `account.owner()`. `Rent::minimum_balance` is `Rent::try_minimum_balance`.
- `account.data.borrow()` is `account.account().try_borrow()?`.
- `set_inner(X { .. })` is `*ctx.accounts.foo = X { .. }`.
- `account.reload()?` is `account.revalidate_after_cpi()?`. Zero-copy reads are already live, so nothing is reloaded; the call re-runs the schema checks a CPI could have invalidated.
- `error!(MyError::X)` is just `MyError::X`. The macro exists only under the `compat` feature. `?` converts, and a tail position needs `.into()`.
- `ProgramError::X.into()` is now a no-op that clippy rejects. Return `ProgramError::X`.
- `#[account(zero)]` is `#[account(zeroed)]`, and `#[account(zero_copy(unsafe))]` is plain `#[account]`.
- `AccountLoader<'info, T>` with `load()` / `load_mut()` / `load_init()` is `Account<T>`, which derefs straight to `T`. Delete the load calls rather than converting the state to borsh.

Dropping `'info` from handler signatures is mechanical, but leave `<'a>` on free functions that genuinely use it. A helper returning `[&'a [u8]; 4]` still needs its parameter.

## Account State Chooses Between Zero-Copy and Borsh

v2's `#[account]` is zero-copy and requires a `Pod` layout: no implicit padding, no `bool` (use `PodBool`), no `String`, no `Vec`.

- State holding a `String`, a `Vec` or an enum goes to `#[account(borsh)]` with `BorshAccount<T>`.
- Fixed-layout state can stay zero-copy, but every padding byte has to be named. A `u32` followed by a `u8` leaves three bytes that the struct must declare.
- `#[derive(InitSpace)]` and `#[max_len(N)]` still work on borsh accounts. Size Pod accounts with `core::mem::size_of`.

**Default to `#[account(borsh)]` when porting.** v1 accounts were borsh-encoded, and `BorshConfig` makes wincode's wire format byte-identical, so the on-chain layout and the clients and tests that decode it are unaffected. Keep zero-copy only where the program deliberately wants it, such as a large order-book slab, where converting to borsh would defeat the point of the account.

`Owner` is a const in v2 (`const OWNER: Address`), which makes a foreign-owned account straightforward to declare for a vendored Pyth or Bubblegum type.

## Reading and Writing an Account From remaining_accounts

v2 has no `Account::try_from(&AccountView)`. To load an account **for writing**, use `AnchorAccount::load_mut`. It is `unsafe` because the caller has to guarantee no other live `&mut` to the same data, and `exit()` writes it back:

```rust
let mut order = unsafe { BorshAccount::<Order>::load_mut(*view) }?;
order.filled_quantity += fill;
order.exit()?;
```

`exit()` takes no arguments in v2.

To read one without taking on the write path, check the discriminator and decode the payload:

```rust
let data = account.try_borrow()?;
let disc_len = <T as anchor_lang::Discriminator>::DISCRIMINATOR.len();
require!(
    data.len() > disc_len
        && &data[..disc_len] == <T as anchor_lang::Discriminator>::DISCRIMINATOR,
    MyError::BadAccount
);
let mut payload = &data[disc_len..];
<T as wincode::SchemaRead<anchor_lang::BorshConfig>>::get(&mut payload)
    .map_err(|_| MyError::BadAccount)?
```

## Borrows Across CPIs Are What Tests Catch and the Compiler Does Not

Every rule in this section compiles either way and shows up only when a test runs the program.

### Release a Data Account's Borrow While It Signs

A data account (`Account`, `BorshAccount`, `InterfaceAccount`) holds a live borrow on its buffer. When it signs a CPI, the runtime rejects the CPI's own borrow of the same account with `AccountBorrowFailed`. Hand the borrow back across the call:

```rust
// Copy the seed inputs out first: `release_borrow` needs `&mut`.
let bump = [ctx.accounts.offer.bump];
let maker = ctx.accounts.offer.maker;

ctx.accounts.offer.release_borrow()?;
// ... CPI signed by `offer` ...
ctx.accounts.offer.reacquire_borrow_mut()?;
```

`release_borrow` flushes the pending writes, and `reacquire_borrow_mut` reads them back and re-runs the load-time owner and discriminator checks, because a CPI in the release window could have changed either.

Three ways this goes wrong:

- **Dereferencing after release panics** with `account borrow released (closed)`. That includes the derive's own use of the account after the handler returns, so the reacquire has to happen before the handler ends.
- **Release and reacquire have to be on the same branch.** Releasing inside `if fee > 0` and reacquiring outside it re-borrows an account still held.
- **A read-only account cannot be reacquired.** There is no read-only reacquire. If the derive references the account after the handler, declare it `mut`.

### Box Does Not Forward cpi_handle_mut

`Box<T>`'s `AnchorAccount` impl supplies only `account()`, so `cpi_handle_mut()` on a Boxed account falls through to the default, which builds a handle **without** releasing the wrapper's data borrow. `Box`'s `ToCpiHandleMut` impl does forward to the inner type, where the release lives.

**On a Boxed account, call `to_cpi_handle_mut()` and `to_cpi_handle()`.** Both spellings compile. Getting this wrong makes every CPI in the program fail.

### Read-Only Slots Take the Wrapper's Own Handle

`cpi_handle()` borrows `&self`, so calling it twice on the same field fills two read-only slots with no workaround needed. On a data account it also relaxes the runtime borrow check, which a handle built by hand does not.

`CpiHandle::readonly(&copied_view)` over a live data account passes the compiler and fails at runtime. Reserve the copied-`AccountView` trick for wrappers that hold no data borrow (`Signer`, `UncheckedAccount`, `Program`) or for a data account whose borrow has already been released.

When one account fills a writable slot **and** a read-only one, erase the writable handle rather than copying the view. `CpiHandleMut` is `Copy`, and `into_readonly()` carries the relaxed borrow flag across:

```rust
// A mint that is its own mint authority.
let mint_handle = ctx.accounts.mint_account.cpi_handle_mut();
let authority_handle = mint_handle.into_readonly();
MintTo { mint: mint_handle, to: ..., authority: authority_handle }
```

Take the writable handle **last**. It borrows the field mutably for the rest of the scope, so any `msg!` or `.address()` on the same account has to come first.

### A mut Data Account Cannot Be Borrowed Again

Loading a data account mutably sets pinocchio's exclusive sentinel (`borrow_state == 0`), so **any** `try_borrow()` on it fails. Three ways out, in order of preference:

1. For a Token-2022 extension, use anchor-spl's accessor. `TokenInterfaceAccountExtensions::get_extension::<T>()` on an `InterfaceAccount<Mint>` or `InterfaceAccount<TokenAccount>` parses the TLV through the borrow the wrapper already holds, and checks the account is owned by Token-2022 on the way. It is bounded on `Pod`, so it covers every fixed-size extension but not a variable-length one like `TokenMetadata`.
2. Drop the `mut` if the account is not written. A mint passed to `transfer_checked_with_fee` is read-only, because the withheld fee accrues on the destination token account.
3. Declare the field `UncheckedAccount` and load the typed wrapper by hand. `AnchorAccount::load` is safe, runs exactly the validation the derive would have run, and registers a **shared** borrow rather than an exclusive one, so an ordinary `try_borrow()` still has room:

```rust
/// CHECK: loaded and validated as an `InterfaceAccount<Mint>` below.
#[account(mut)]
pub mint_account: UncheckedAccount,
...
let mint = InterfaceAccount::<Mint>::load(*ctx.accounts.mint_account.account())?;
let buffer = mint.account().try_borrow()?;
```

Keep the `mut`. It is what marks the account writable in the IDL, which the client needs for the CPI that writes to it, and `cpi_handle_mut()` on an `UncheckedAccount` checks the runtime writable flag rather than the wrapper's own mutability. Drop the loaded wrapper before the CPI so its shared borrow is released. This is the route to a variable-length extension such as `TokenMetadata`, which `get_extension` cannot reach.

`unsafe { account.account().borrow_unchecked() }`, reading through the exclusive borrow already held, is what `get_extension` does internally and is sound on anything reached through `ctx.accounts`. It should still be the last resort: one of the three options above has covered every case so far.

A read-only account is different: a second shared borrow is fine, so `account().try_borrow()?` works there.

### Sysvars Beyond Clock and Rent

pinocchio ships typed accessors for Clock and Rent only, so anchor-lang v2 exposes no more than that. For anything else, declare the layout and read it with `pinocchio::sysvars::get_sysvar(&mut buf, &sysvar_id, 0)`. It is a safe wrapper over the `sol_get_sysvar` syscall, and off-chain it is a no-op that leaves the buffer zeroed, so IDL and client builds compile without a `cfg` split of their own. Reaching for `solana-define-syscall` and calling `sol_get_sysvar` directly is the same read with an `unsafe` block and a `MaybeUninit` around it.

## Constraints That Moved or Disappeared

### has_one Becomes address on the Sibling

`has_one` is deprecated. Under `cargo clippy -- -D warnings` that makes it a build failure. The check moves off the owning account and onto the sibling it names:

```rust
// v1: on `offer`
#[account(mut, close = maker, has_one = maker)]
pub offer: BorshAccount<Offer>,

// v2: the constraint lives on `maker`
#[account(mut, address = offer.maker)]
pub maker: SystemAccount,
```

Any `@ MyError` carries across. Forward references are fine, so the sibling may be declared before its owner.

### SPL init Constraints Need the Token Program Named

When the token program is an `Interface`, an `init` constraint has to spell out which program it means. Without it the init CPI is rejected with `InvalidArgument`:

```rust
#[account(
    init,
    payer = owner,
    token::mint = liquidity_mint,
    token::authority = reserve,
    token::token_program = token_program,
)]
pub liquidity_vault: InterfaceAccount<TokenAccount>,
```

The same applies to `mint::token_program` and `associated_token::token_program`.

### A Constraint Cannot Reference the Account Being Initialized

`mint::authority = mint_account` on `mint_account` itself, a PDA that is its own mint authority, is rejected when the macro expands. An SPL `init` constraint has to name a **sibling** field. The same goes for `token::authority = <self>`.

Where that pattern is the point of the program, build the account by hand with `create_account` plus `initialize_mint2` or `initialize_account3`, which take the authority as an address. Adding a second field for the same address instead would trip the duplicate-account check below.

### Duplicate Mutable Accounts Are Rejected

v2 rejects an account that appears in more than one declared slot while any of those slots is mutable, with `ConstraintDuplicateMutableAccount` (custom error 2040). Minting or sending to yourself hits this, because the authority and the recipient are the same account.

The rule exists because v1 deserialized each `Account<T>` into an owned copy and wrote it back at the end of the instruction. Two slots over one account meant two independent copies, and the second write-back silently clobbered the first. That is the classic self-transfer bug: debit one copy, credit the other, write both, and the balance goes up. v2 accounts are zero-copy views into the runtime's buffer, so two mutable wrappers over one account would be two `&mut` to the same bytes. The loader rejects rather than warns.

`#[account(unsafe(dup))]` opts a slot out. It implies `mut`, so it replaces the `mut` rather than joining it. The walker flags **both** indices of a duplicate, so marking only the second one leaves the first intersecting the mutable mask: every slot that can legitimately alias needs the constraint, including a `payer`.

The `unsafe` is real, so check two things before reaching for it. First, that the accounts can actually alias: if no caller ever passes the same address twice, the constraint weakens a live safety check for nothing, and the fix is to leave it off. Second, that the aliasing slots are wrappers holding no deserialized state, such as `Signer`, `SystemAccount` or `UncheckedAccount`. There is no lost-update hazard there because there is nothing to write back. Two aliasing `Account<T>` slots are the case the check is for, and the fix is to restructure the accounts so only one of them exists.

## Discriminators, and Programs That Implement an Interface

By default a handler still dispatches on `sha256("global:<name>")[..8]`, so existing clients and tests keep working. The override changed:

- `#[interface(...)]` and the `interface-instructions` feature are gone.
- `#[discrim = N]` on an executable `#[program]` takes a **single byte**, and it is all or nothing. If one handler has it, every handler needs one.
- `#[program(interface, program_id = ID)]` accepts arbitrary discriminator bytes, but it declares an interface for other programs to CPI into. It generates a CPI client and no dispatch, so the crate builds to a roughly 900-byte object with no `entrypoint` symbol and fails to load with `ProgramLoad("Entrypoint out of bounds")`.

That leaves no direct way to write a program answering to a foreign eight-byte discriminator, such as an SPL transfer hook's `Execute`. Take the entrypoint over instead:

- Set `default = ["no-entrypoint"]` in the crate's features. anchor then exports its dispatch as `__anchor_dispatch` rather than claiming `entrypoint`.
- Add a `src/entrypoint.rs` that claims `entrypoint`, swaps the interface discriminator for the matching handler's, and calls `crate::__anchor_dispatch(input, ix_data_ptr)`. The payload behind the discriminator is unchanged, so nothing else has to be replicated.
- With `no-entrypoint` set, the crate also has to invoke `pinocchio::default_allocator!()` and `pinocchio::default_panic_handler!()` itself. anchor only emits those on the path where it owns the entrypoint.

## Hand-Built invoke Calls Match Handles to Metas Positionally

A program that builds its own `Instruction`, because the callee's crate is on an incompatible Solana version, hands `invoke` a slice of `CpiHandle` rather than `AccountInfo`. Four rules, all enforced at runtime:

- **The handles are positional.** v2 walks the instruction's account metas and binds each to the next handle in the slice. A handle that does not match the meta at its position fails the whole call with `InvalidArgument`.
- **The program account is not a handle.** v1 code habitually pushed the callee program's `AccountInfo` into the infos vec. In v2 that extra leading entry is what breaks the match.
- **An account filling two meta slots supplies two handles.** Bubblegum's `Transfer` names the same PDA as both `leaf_owner` and `leaf_delegate`, so its handle appears twice.
- **Writability has to be at least as strong as the meta.** A writable meta needs a writable handle; a read-only meta accepts either. Going the other way, `cpi_handle_mut()` on an account not declared `mut` panics with *"cpi_handle_mut called on a read-only account"*, which surfaces as `ProgramFailedToComplete` and a `src/traits.rs` log line.

Mixing handle kinds in one vec means converting element-wise, as in `vec![a.cpi_handle_mut().into(), b.cpi_handle()]`. A trailing `.map(CpiHandle::from)` forces every element to be a `CpiHandleMut`.

Vendored types that derive `BorshSerialize` over `Address` fields need the `borsh` feature on `solana-address`. Their `serialize` returns `io::Error`, which no longer converts into `ProgramError`, so map it:

```rust
args.serialize(&mut data)
    .map_err(|_| ProgramError::InvalidInstructionData)?;
```

## Tests Assert on Error Codes and Use Host-Side Sysvar Types

- **v2 does not log Anchor error names.** A test asserting that a failure message contains `"StalePriceFeed"` can never pass. Assert on the numeric custom code: the `#[error_code]` discriminant plus the default 6000 offset, overridable with `#[error_code(offset = ...)]`. `#[error_code]` no longer generates `From<MyError> for u32`; it makes the enum `#[repr(u32)]`, so `my_error as u32 + 6000` is the value.
- `anchor_lang::solana_program` has no `system_program` submodule. The real module is at the crate root and exposes `ID`, not `id()`.
- `solana_program::pubkey::Pubkey` exists only under the `compat` feature. `anchor_lang::Address` is the same 32-byte type.
- `anchor_lang::prelude::Clock` is pinocchio's on-chain type. LiteSVM's `get_sysvar` and `set_sysvar` want the host-side `solana_clock::Clock`, and the same holds for other sysvars via `solana-sysvar`.
- Tests that decode account bytes with borsh keep working, because `BorshConfig` makes the wire format byte-identical. A Pod account that grew explicit padding needs that padding mirrored in the test's decode struct, since `try_from_slice` rejects trailing bytes.

## Building a Workspace With More Than One Program

`cargo-build-sbf` at an anchor workspace root builds every member in one invocation, and cargo unifies features across them. A program that depends on a sibling with `features = ["cpi"]`, which implies `no-entrypoint`, therefore makes that sibling's own `.so` build with `no-entrypoint` too. In v2 that exports its dispatch as `__anchor_dispatch` rather than `entrypoint`, and the result loads with `ProgramLoad("Entrypoint out of bounds")`.

`anchor build` builds each program separately and is unaffected. Anything else driving `cargo-build-sbf` has to do the same.

## Sysvars pinocchio Does Not Ship

pinocchio ships `Clock` and `Rent`. Anything else, `LastRestartSlot` for instance, has to be declared locally and read through the `sol_get_sysvar` syscall with `solana-define-syscall`. `solana-sysvar`'s own `get` is bound to that crate's `Sysvar` trait, not pinocchio's, so it is not usable from a v2 program.

## What 2.0.0-rc.1 Cannot Do Yet, and What to Undo When It Can

Three things below are worked around rather than fixed, because the fix is not in a published release. Undo them together when Anchor publishes anything after `2.0.0-rc.1`, and check each one rather than assuming the release covered it.

- **`#[derive(IdlType)]` on an enum produces an IDL that Anchor itself cannot read.** `anchor-derive-accounts` writes the JSON key `fields` for every type kind, while `anchor-lang-idl-spec` requires `variants` for `IdlTypeDefTy::Enum`, so `anchor build` fails on its own output with ``Error: missing field `variants` ``. Any program with an enum in its IDL hits it; nothing in the program is wrong. The fix is on `anchor-next` (PR 4947) and unreleased. Until then the only workaround is `anchor build --no-idl`, which also has to go on `anchor test`, because that regenerates the IDL too. A project that consumes its IDL, through `declare_program!` or a committed `idls/*.json`, cannot use this workaround at all.
- **The LiteSVM test harness is not on crates.io.** `anchor-v2-testing`, which `anchor test --profile`, `anchor debugger` and `anchor coverage` all require, exists only as a git dependency on `anchor-next`. Pin a revision rather than tracking the branch, or a build can change behaviour with no commit to explain it. Tests calling `LiteSVM::new()` directly still pass but record no traces, so those three tools silently produce nothing.
- **A type reaching the IDL without a v2 derive fails only during IDL generation.** `` `T` has no IDL type information `` means a plain struct needs `#[derive(IdlType)]`; account and event types get it from `#[account]` / `#[event]`. Leftover `impl anchor_lang::IdlBuild for T {}` from 1.x is worse, because `IdlBuild` no longer exists and the impl usually sits behind `#[cfg(feature = "idl-build")]`, where `cargo check`, `cargo clippy` and `cargo fmt` never compile it. Only `anchor build` does.

That last point generalises: **anything behind the `idl-build` feature is invisible to every check except `anchor build`.** A repository whose CI builds only changed projects can carry such a break for months.

## Two Ambiguities That Bite

- `seeds` takes a byte array directly and binds it itself. Write `id.to_le_bytes()`, not `id.to_le_bytes().as_ref()`, which produces a temporary that dies before the derive uses it.
- The prelude exports an `Event` trait, so a state struct named `Event` reaching the crate root through a glob re-export becomes ambiguous. Import it explicitly with `use crate::state::Event;`.
