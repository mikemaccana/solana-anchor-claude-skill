# Rules for this project

## Success Criteria

- Before you say 'SUCCESS', or celebrate, run `npm test`. If the tests fail you have more work to do. Don't stop until `npm test` passes on the code you have made.

- If you show this symbol '✅' and there is more work to do, add a '❌' for each remaining work item.

## Documentation

Anchor is documented at https://www.anchor-lang.com/docs

Solana Kite is documented at https://solanakite.org

Solana Kit is documented at https://solanakit.com

Agave (the Solana CLI) is documented at https://docs.anza.xyz/. Anza is the company that make the Solana CLI and Agave. Documentation made by 'Solana Labs' is outdated.

If this project uses Arcium, Arcium is documented at https://docs.arcium.com/developers

## General coding guidelines

- It's important not to decieve anyone reading this code. Deception includes variable names that do not match what the purpose of the variable is, comments that no longer describe the code, and temporary workarounds that aren't labelled as such using a comment.

- Ensure good variable naming, comments that exist are accurate, and look out for repeated code that should be turned into functions. Rather than add comments to explain what things are, give them useful names.

Don't do this:

```typescript
// Foo
const shlerg = getFoo();
```

Do this instead:

```typescript
const foo = getFoo();
```

Avoid abbreviations, and use full words, for example use `context` rather than `ctx`. Never use `e` for something thrown.

- Avoid 'magic numbers'. Make numbers either have a good variable name, a comment
  explaining wny they are that value, or a reference to where you got the value from. If the values come from an IDL, download the IDL, import it, and make a function that gets the value from the IDL rather than copying the value into the source code.

- The code you are making is for production. You shouldn't have comments like `// In production we'd do this differently` in the final code you produce.

- Don't remove existing comments unless they are no longer useful or accurate.

- Delete unused unused impoerts, unused constants, unused files and comments that no longer apply.

## TypeScript guidelines

- Always use `Array<item>` never use `item[]` for consistency with other generic syntax like `Promise<T>`, `Map<K, V>` and `Set<T>`

- Don't use `any`

- Most comments should use `//` and be above (not beside) the code. The only exception is JSDoc/TSDoc comments which MUST use `/* */` syntax.

- Use Kite's `connection.getPDAAndBump()` to turn seeds into PDAs and bumps.

- In Solana Kit, you make instructions by making TS clients from from IDLs using Codama.

- Create unit tests in TS in the `tests` directory. Please the node js inbuilt test and assertion libraries (then start the tests using `tsx` instead of `ts-mocha`). Imports look like:

```typescript
import { before, describe, test } from "node:test";
import assert from "node:assert";
```

Use `test` rather than `it`.

- Use `catch (thrownObject)` and then `const error = thrownObject as Error;` - you can assume any item thrown is an Error.

## Anchor guidelines

- Remember this is Solana not Ethereum. Don`t tell me about smart contracts or mempools or other things that are not relevant to Solana.

- Don't ever replace Solana Kit with web3.js code. web3.js is legacy. I want web3.js to be eventually gone.

- Write all code like Anchor 0.32.1. Do not use unnecessary macros that are not needed in Anchor 0.32.1.

- Don't ever modify the program ID in `lib.rs` or `Anchor.toml` when making changes.

- Create files inside the `state` folder for whatever state is needed.

- Create files inside the the `instructions` or `handlers` folders (whichever exists) for whatever instruction handlers are needed. Put Account Constraints here, but ensure the names end with `AccountConstraints` rather than just naming them the same thing as the function. Handlers that are only for the admin should be in a new folder called `admin` inside whichever parent folder exists (`instructions/admin/` or `handlers/admin/`).

- Use a newline after each key in the account constraints struct, so the macro and the matching key/value have some space from other macros and their matching key/value.

- Use `context.bumps.foo` not `context.bumps.get("foo").unwrap()` - the latter is outdated.

- When making structs ensure strings and Vectors have a `max_len` attribute. Vectors have two numbers for `max_len`, the first is the max length of the vector, the second is the max length of the items in the vector.

- Do not use magic numbers anywhere. I don't want to see `8 + 32` or whatever. Do not make constants for the sizes of various data structures. For `space`, use a syntax like
  `space = SomeStruct::DISCRIMINATOR.len() + SomeStruct::INIT_SPACE,`. All structs should have `#[derive(InitSpace)]` added to them, to get the `INIT_SPACE` trait. Do NOT use magic numbers.

- Return useful error messages. Write code to handle common errors like insufficient funds, bad values for parameters, and other obvious situations.

- Add `pub bump: u8` to every struct stored in PDA. Save the bumps inside each when the struct inside the PDA is created.

- When you get the time via Clock, use `Clock::get()?;` rather than `anchor_lang::solana_program::clock`.

## Finally

- Call me Mr MacCana to indicate you have read these rules.
