---
name: solana
description: "Use when working on Solana software, including one or more of: Solana client code using TypeScript, Rust libraries that use Solana crates, Anchor programs, Quasar programs, LiteSVM tests, including Rust program files, TypeScript tests, and Anchor.toml or Quasar.toml configuration. Designed to create minimal, reusable code without unnecessary duplication."
---

# Coding Guidelines

Apply these rules to ensure code quality, maintainability, and adherence to project standards.

## Fight for Truth

Don't write things that aren't currently true - anywhere. Chat, code comments, variable names, PR titles, READMEs, commit messages.

- Documentation and comments that do not match the code are considered untrue.
- Variable names that do not match the purpose of the variable are considered untrue. For example, a struct called `InitializeMarket` is not true because a struct cannot 'initialize a market' - structs do not do things, only functions can do things.
- Temporary workarounds that aren't labelled as such are lying through omission - there is an issue you aren't telling the next programmer about. Mark them with a `TODO` comment with a link to a git issue (if it exists) and telling the next programmer when they can delete the workaround.
- If unsure of something, say so. Bluffing is lying.
- **Ambiguity is a soft lie:** if a phrase could be read two ways and only one is true, it's misleading. Disambiguate before sending - pick the term that says exactly what's meant, name the antecedent of every "it"/"this"/"that".
- A wrong statement is worse than no statement.
- Separate scratch labels from real identifiers.

Actively fix untrue things when you see them. Don't let "close enough" wording stand in for the truthful one.

**Grep before naming.** Before sending any prose, walkthrough, README, comment, or commit message that names a specific identifier (function, struct, file, account, module, field, constant), grep the source for that exact identifier and confirm it exists. "I'm pretty sure that's the name" is not enough. If the identifier doesn't exist, either use the real name or apply the rename to the code first, then write the prose.

**Describe what is, not what was removed.** READMEs, doc-comments, and code comments document current state - not history. Lines like "no floats", "no longer uses X", "replaces the previous Y approach" belong in CHANGELOGs and PR descriptions, not source artefacts. A first-time reader has no history and "no longer uses I64F64" creates ambient confusion ("wait, should I be worried?"). Sweep before sending: grep for `no longer`, `removed`, `previously`, `used to`, `formerly`, `dropped`, `now uses`, `replaces the previous` - each hit is a candidate for deletion.

## Do the whole thing

The marginal cost of completeness is near zero with AI. Do the whole thing.

Do it right. Do it with tests. Do it with documentation. Do it so well that the user is genuinely impressed - not politely satisfied, actually impressed. Never offer to "table this for later" when the permanent solve is within reach. Never leave a dangling thread when tying it off takes five more minutes. Never present a workaround when the real fix exists.

The standard isn't "good enough" - it's "holy shit, that's done." Search before building. Test before shipping.

Ship the complete thing. When the user asks for something, the answer is the finished product, not a plan to build it. Time is not an excuse. Fatigue is not an excuse. Complexity is not an excuse. Boil the ocean.

**A caveat is not a fix.** When you find a gap while doing the work - missing test coverage, a CI job that doesn't run, an untested path, a stale doc - close it as part of the task. Do not downgrade a fixable defect into a disclaimer. If you catch yourself writing "note: X isn't covered / isn't built / isn't tested / won't get a CI signal," stop: that sentence is a TODO, not a caveat. Fix X so there is nothing to note. Reserve caveats for what you genuinely cannot fix (out of scope, needs a decision, external blocker) - and for those, state what you would do and offer to do it.

## Success Criteria

- Before declaring success, declaring that work is complete, or celebrating, run the project's actual tests using the correct command for that project (for example: `anchor test` for Anchor workspaces, the project's TypeScript test command for TypeScript clients/tests, or `cargo test` for Rust crates). If the tests fail, there is more work to do. Don't stop until the relevant test command passes on the code you have made.
- Create all program state through the program's own instruction handlers in tests. Injecting pre-fabricated accounts (hand-built account data passed straight into the SVM) hides missing init instructions and missing constraints - a program can pass every test while being unusable onchain. Pre-fabricated accounts are only acceptable for accounts a foreign program would have created (an oracle's price account, a mainnet-dumped fixture).
- Tests that use a zero or degenerate value for a parameter (e.g. `duration: 0`) test only the boundary where opposite comparisons coincide. Use nonzero, asymmetric values and test both sides of every boundary.
- Do not write placeholder tests. Placeholder tests don't count as tests, placeholder tests passing does not achieve your task.
  - Tests that just do `assert.ok(true)` or similar are placeholder tests and do not count as tests
  - Tests that do not call the program's instruction handlers are placeholder tests and do not count as tests
  - Tests must: initialize accounts, send transactions, verify state changes, check balances
  - If you find yourself writing placeholder tests, stop and write real integration tests instead
  - DO NOT mark "Write tests" as complete until tests actually call the program instructions
  - DO NOT ask "should I write real tests now?" - if the tests are placeholders, write real ones immediately

- Do not stop until documentation like `README.md` and `CHANGELOG.md` are also updated with your changes. If you have made a feature, and it is not documented in the README or changelog, there is more work to do and you must continue working.

- When summarizing your work, show the work items you have achieved with this symbol '✅' and if there is any more work to do, add a '❌' for each remaining work item.

## Documentation Sources

Use these official documentation sources:

- **Anchor**: https://www.anchor-lang.com/docs
- **LiteSVM**: https://www.anchor-lang.com/docs/testing/litesvm
- **Anchor Error Codes**: https://github.com/otter-sec/anchor/blob/master/lang/error/src/lib.rs
- **Quasar**: https://quasar-lang.com/docs
- **Solana Kite**: https://solanakite.org
- **Solana Kit**: https://solanakit.com
- **Agave (Solana CLI)**: https://docs.anza.xyz/ (Anza makes the Solana CLI and Agave).
- **Switchboard** (if used): https://docs.switchboard.xyz/docs-by-chain/solana-svm
- **Arcium** (if used): https://docs.arcium.com/developers
- **Quicknode Solana Program Examples**: https://github.com/quicknode/solana-program-examples
- **Anatoly Yakovenko's GitHub**: https://github.com/aeyakovenko in particular the 'Percolator' perps project repositories.

## Terminology

- Remember this is Solana not Ethereum. Ethereum is not relevant to any documentation you write. Do not assume people know or care about Ethereum.
  - Don't tell me about 'smart contracts' or 'protocols' (use 'programs' instead)
  - Don't tell me about 'gas' (use 'transaction fees' instead)
  - There are no 'mempools'.
  - Do not tell me about other things that are not relevant to Solana.

- Token program terminology:
  - Use 'Token Extensions Program' or 'Token extensions' for the newer token program (not 'Token 2022' which is just a code name)
  - Use 'Classic Token Program' for the older token program
  - Use 'Token' rather than 'SPL Token' unless you are specifically discussing the distinction between the native token (SOL) and all other tokens (SPL Tokens)

- Onchain / offchain (one word, no hyphen)
  - Always write 'onchain' and 'offchain' as single, unhyphenated words - like 'online' and 'offline'.
  - Never write 'on-chain' or 'off-chain'. The hyphenated forms are wrong.
  - Apply the same rule to related terms: 'crosschain' (not 'cross-chain'), etc.
  - Sources:
    - [Solana Foundation style guide](https://solana.com/docs/references/terminology)
    - [US Government usage](https://www.sec.gov/files/rules/interp/2026/33-11412.pdf)
    - [Cat (catmcgee) will make fun of you if you write 'on-chain'](https://x.com/catmcgee/status/2028153588715761825)

- Token amount units: a **major unit** is the human-scale denomination - dollar, pound, yen, SOL. A **minor unit** is the smallest denomination, the raw integer programs operate on - cent, penny, sen, lamport. Use these terms; do not say 'base units'.

- Some tools in Solana unfortunately use the same word 'instructions' for both the input and the functions. To avoid confusion, use 'instruction handlers' for the functions that handle instructions, and 'instructions' for the input to those functions.

- Name handlers by one rule: **if the domain has a word for it, use the domain's word; if the handler only stamps out a container, it is `initialize_`.** Domain verbs are what make a program read like finance, and they come in pairs: `make_offer()`/`take_offer()`, `open_position()`/`close_position()`, `place_order()`/`cancel_order()`, plus `borrow`, `repay`, `liquidate`, `contribute`, `refund`. Setup has no domain meaning, so it is always `initialize_x()`, naming what it initializes: `initialize_market()`, `initialize_pool()`, `initialize_config()`, never `create_pool()`, `init_reserve()`, or a bare `initialize()`. The test is whether the handler moves money or takes a market action, or only writes parameters into a new account. A third family, `add_x()`, extends a container that already exists: `add_liquidity()`, `add_outcome()`, `add_asset()`.

- Say **PDA** for an address and **account** for the data at it. They are not synonyms: a program derives the offer PDA and then writes the offer account. Use PDA where the subject is derivation, seeds, authority, ownership, or signing, and account where the subject is fields, balances, rent, creation, or closure.

## Do not use

- Do not use 'Solana Labs' documentation. The company has been replaced by Anza.

- Do not use 'Coral XYZ' documentation or packages. Coral used to maintain Anchor, but Anchor is now maintained by the Solana Foundation (solana.org). The TypeScript package has been [moved from @coral-xyz/anchor to @anchor-lang/core](https://www.anchor-lang.com/docs/updates/release-notes/1-0-0).

- Do not use any documentaton or tools from Project Serum, which collapsed many years ago.

- Do not use yarn. Yarn has no reason to exist and only adds unnecessary dependencies and is not commonly used for new JS/TS projects in 2026. Replace Yarn with npm everywhere you see it. Use npm for new projects as it does not require additional dependencies. Keep using pnpm if the project already uses pnpm.

- Do not use **Switchboard Functions** - this product is dead and no longer maintained. (Note: Switchboard oracles are still active and usable.)

- Do not use **Clockwork** - this product is dead. For scheduled instruction handler invocation, use [TukTuk](https://github.com/helium/tuktuk/tree/main/typescript-examples) instead.

- Do not use [https://github.com/solana-developers/program-examples]. As of June 2026 these examples are out of date, going back to Anchor 0.26 in 2022, use a bunch of deprecated tools, have security failures and broken tests, and have been this way for more than a year.

## Library versions

Use the latest stable Anchor, Rust, TypeScript, Solana Kit, and Kite you can. If a bug occurs, favor updating rather than rolling back.

## Project Documentation

Every project must have a `README.md` file in the project root that includes:

- **Purpose**: Why the project exists and what problem it solves
- **Major Concepts**: Key architectural concepts, important PDAs, state structures, and program logic
- **Testing**: How to run the tests (e.g., `anchor test`)
- **Setup**: Any prerequisites or setup steps needed to work with the project
- **Usage**: Basic usage examples or deployment instructions if applicable

Keep the README focused and practical. Avoid generic boilerplate - write documentation that would actually help someone understand and work with this specific project.

### Documentation style

- **No numbered headings.** Headings are words only - no `## 1. Overview` or `### 3.6 Liquidation`. Numbered headings break when a section is inserted or removed.
- **No preview paragraphs.** Don't open a README or section with "the sections below cover X, Y, and Z" - the headings already do that.
- **A heading states what the reader gets, and works read alone.** It is close to a summary of its section, because a table of contents lists every heading out of context. Name the outcome, not the mechanism behind it: "Seeds Make Data Findable", not "Seeds Make Addresses Derivable". Keep specific actors out: "A User's Public Key Is Their Address", not "Alice's Public Key Is Her Address". A narrative walkthrough is the exception, where the headings are the story's steps and name who acts.
- **A section lives where its subject belongs, not where it was first needed.** If a section would sit equally well under a different heading, it is under the wrong one: rent is a property of every account, so it belongs with accounts, even if program-derived vaults are where its lifecycle first mattered. Teach each concept once, in full, where it belongs; elsewhere give it a clause and a pointer, never a second explanation. When a section turns out to be two things, split it and send each half home.
- **A named character has a gender; a role does not.** Alice, Bob, Carol, Dave, Maria and Victor keep their pronouns. A maker, a taker, an admin, a manager or a borrower named only by role is they: "the maker funds their offer account", "the admin picks the winning outcome, but they cannot touch the pro-rata math", never "her offer account" or "she cannot touch". A role sentence that sits next to a named character is still a role sentence: "Maria is the fund manager" takes she, and "the manager is the threat model, and the program treats them as such" takes they.
- **A summary section states what the reader now knows**, not what the example's characters did - "a user's account holds their balance of SOL", not "Alice's holds her balance" - even when the section that taught it was a story.
- **Integrate per-instruction reference into lifecycle prose.** Walk through the program's flow and inline each handler's mechanics (signers, accounts, token movements, errors) at the point it's first called. Don't keep a separate flat "Instruction Reference" section.
- **Emphasis is meaningful.** Bold canonical terms on first use, plain everywhere after, like a textbook, and bold the claim a passage turns on. Italicize the titles of books and other works (*TCP/IP Illustrated*). Never use either as decoration.
- **Define things by what they concretely are, not by abstract category.** An account is "some data, a Rust struct, with an address", never "the unit of state". A definition that would fit equally well in a document about a different subject is too vague. Prefer the short common word to the impressive one: "the deployed program", not "the deployed bytecode"; "compiled code", not "compiled bytecode". Reach for the precise technical term only where the precision is the point.
- **Say what the reader would see; do not tell them to look.** "Note what is pledged: shares, not raw tokens" is one word longer than "what they pledge is shares, not raw tokens", and the second sentence is the one carrying the fact. The same goes for "Notice who did nothing", "Watch K in the story below", "Read this carefully", and "Look at the two numbers". The reader is already reading; an instruction to read is a sentence that delays the content by exactly its own length. Telling them to watch something in the world ("watch a market maker during a fast market and their quotes vanish") is different, and stays.
- **Talk to readers, never about the document's construction.** No sentences explaining why a section is organized, sized, or placed the way it is ("each section is short on purpose", "that is why it comes first"). If a sentence is addressed to the author, editor, or reviewer rather than the reader, delete it.
- **No ASCII art, no Mermaid diagrams, no markdown tables.** Use headings, nested bullet lists, or prose. Tables don't render well on chat surfaces. When a project calls for real figures (a book, a docs site), draw hand-authored SVG account diagrams per [DIAGRAMS.md](DIAGRAMS.md) instead.
- **No em-dashes, and none of the phrases that read the same way.** Use a regular dash or rewrite the sentence. An em-dash is an LLM-output tell: a reader clocks it before they have finished the sentence. These phrases fail the same way, so they are banned on the same grounds: "load-bearing", "delve", "tapestry", "intricate", "multifaceted", "holistic", "myriad", "seamless", "robust", "pivotal", "crucial", "elevate", "testament to", "deep dive", "paradigm shift", "game-changer", "unlock the power", "realm of", "landscape of", "in today's", "it is worth noting", "underscores the". Most are also gestures rather than statements, so the fix is the one below: name the thing and say what it does. "The feed is load-bearing" becomes "every number in the fund is computed from the feed". This applies to READMEs, code comments, commit messages, and doc strings.
- **"X is not Y; it is Z" only when the reader actually believes Y.** The construction corrects a misreading, and it is also a strong LLM tic that degrades with repetition: one manuscript audit found 48 in 50,000 words, a third of them negating a straw man nobody was thinking ("this is not stinginess; it is the only stable policy", "rounding direction is not a detail, it is a security boundary"). Keep it where a reader really would assume the negated half: "a mint is not itself a token: it is the account a token comes from". Everywhere else, drop the negation and assert the second half on its own.
- **A paragraph does not end on a maxim.** After the em-dash, the commonest LLM tell: a paragraph makes its point in plain sentences, then closes with a short, quotable restatement of it. "A wall that has never been pushed on is a hypothesis", "Fifteen lines of integer math, doing a central bank's job", "Data model decisions are transaction-format decisions", "Conservation, the escrow edition" were all closing lines in one manuscript, and none carried anything the paragraph had not already said. A reader hears the cadence before the content. Delete the maxim, and if it held a fact the paragraph lacked, state the fact in a plain sentence. The fragment used as a beat ("Not Alice, not the program's author.", "His bot fires.") and the "X is not Y; it is Z" form above are the same tell.
- **"Honest" is not a synonym for accurate, stated, or plain.** One audit found twenty in fifty thousand words ("the honest way to make it", "the simplest honest way", "an honest limitation, stated as one", "honest bookkeeping", "cannot be honestly tested"), and none was about honesty. Say what the sentence means without the word, or cut it. "Quietly" and "silently" as intensifiers ("quietly rewritten", "has quietly become $758") go the same way: the value changed, so say it changed. "The discipline" and "hygiene" as names for a rule ("the discipline below proves a market", "the staleness check was hygiene") name a category where the rule is shorter: "the rules below", "the staleness check".
- **Never "fail open" or "fail closed".** An open circuit has stopped conducting and an open door lets everything through, so readers split on which way each phrase points, and "fail closed" flips the same way. Say what the program does when the check fails: "a stale price is refused", "any payout the pool cannot cover is rejected", or, for the dangerous direction, "a missing signer check lets anyone through".
- **Do not rate your own material.** "Two design details are worth stealing", "a guarantee worth printing", "one line of discipline worth copying", "the boundary case worth studying closely", and "by now you know the rule" tell the reader how to feel about the next sentence instead of getting to it. Cut the rating and keep the content: "Two design details:", "one guarantee:".
- **A word repeated is not a word justified.** If a plainer word carries the same meaning, the plainer word is the right one, and finding the fancier one used consistently across a document is evidence of a repeated mistake rather than of a house style. "The program interrogates the price" was "the program checks the price" in all five places it appeared. Same for reaching for an elevated register with ordinary words: "further clusters are stood up from time to time" is "other clusters are started up now and then".
- **If the prose is already a list, set it as one.** Three mechanical signals: the sentence spells its own bullets out as words ("First, ... Second, ..."), it announces a count and then runs the items together ("the entire financial surface is four expressions: ..."), or it strings parallel items on semicolons. Each is a list the reader has to re-parse before they can use it, where bullets let them scan it and refer back to one item. Leave short rhetorical triples and real narrative in prose; bulleting three five-word clauses adds whitespace, not clarity.
- **Don't say "worked example" or "worked scenario".** Just "Example", "Scenario", or "Walkthrough".
- **Write "ID" in capitals, never "id", in prose.** It is an abbreviation: "a program ID", "the offer ID", "distinct market IDs". A **program ID** is a program's address. Only code identifiers (`program_id`) and the `id.json` filename keep the lowercase form.
- **Transitions are connectives, and a sentence leads with its subject.** A transition's whole job is to get the reader to the next point, so use the plain connective: "Another concern:" not "The question has a second half"; "A restart works the same way" not "The same discipline covers the restart". In the sentence itself, put the actor first and let it act. "Offchain the question has to be answered" fronts an adverb and hides who answers; "the code that reads the chain has to answer it" names them. Inverted or agentless openings read as an affectation, and the reader has to hold the clause until the subject arrives.
- **Name the members, not the category.** "The systems around the program choose" tells a reader nothing, because "system" fits anything: say "the liquidator bot, the crank on a schedule, the backend that marks an order paid". Same for "infrastructure", "components", "tooling", and "services". If you cannot list the members, you do not yet know what you are claiming.
- **Give the answer for each case before the principle behind them.** A reader arrives with a job, not with a desire to reason from first principles. Lead with the cases and what to do in each, including what goes wrong with each choice ("for a rate quoted per year use timestamps, but it charges straight through a halt"), then state the principle underneath in a line. A section that only states the principle has handed the reader homework: "choose by asking who benefits from the error" is the reason for the table, not a substitute for it.
- **Do not print a number that a scheduled change will invalidate.** "Tower BFT finalizes 32 slots back, about 12.8 seconds" is true today and wrong the week Alpenglow ships. Write what stays true and where the number comes from: waiting for finality costs latency, and how much is set by the consensus protocol. The same goes for compute limits, fee levels, and account size caps. Where a current number genuinely helps, name its source and date it, so a reader can tell a measurement from a constant.
- **Write for the reader's job.** A program developer needs to know a halt can stop the chain for hours and what that does to a slot count; the procedure for agreeing a restart slot and waiting on 80% of stake is a validator operator's work and does not belong in their document. Before including a mechanism, ask whether the reader will ever act on it.
- **A message a human reads names the consequence, not the condition that tripped.** This covers Anchor `#[msg]` strings, log lines, and CLI output. "Price feed was last updated before the most recent cluster restart" states what the check found and leaves the operator to work out why anyone cares; "Price feed is stale: it predates the last cluster restart" leads with the problem and keeps the cause. Match the voice of the errors beside it: a set of `#[msg]` strings that all state conditions should not acquire one that narrates the program's action.
- **Prefer plain conditionals to clever biconditionals.** State a rule in both directions concretely rather than collapsing it into "X holds exactly when Y" or "X _is_ the set of Y". Write "an asset is approved if it has an `ApprovedAsset` account, and not approved if it does not," not "an asset is approved exactly when its `ApprovedAsset` account exists." The plain form reads faster and cannot be misread as set-theory notation.

## Writing About Financial Software

These apply to READMEs, docs, blog posts, and PR descriptions for finance-related projects (AMMs, escrows, lending, leasing, CLOBs, prediction markets, stablecoins).

- **State the mechanism; don't gesture at it.** Say what happens, in the plainest words that stay accurate. Three tells that a sentence is gesturing rather than stating: its subject is a pointer rather than a thing ("this is how it is enforced" - name the thing, "disabling the mint authority is how it is enforced"); a metaphor stands in for the mechanism ("every state needs an exit" makes the reader unpack it, where "every participant must be able to get their money out of every state" is the same length and says it); or it is meta-commentary about the document ("that sentence is the whole design", "this is the real lesson here"). A sentence whose entire content is "this matters" adds nothing the next sentence does not, so cut it. The commonest form announces that a point exists instead of making it: "what matters here is the unit" and "that is the whole reason they count slots" both leave the reader to supply the content, where "the index advances by slots, not seconds" is the content. Whenever you write "what matters", "the point is", or "the whole reason", delete the announcement and keep what followed it. It comes back wearing a pronoun once those are gone, so watch for "that is the point", "which is exactly the point", "is the one that matters", "it matters more than it looks" and "it is worth being honest about": every one of them sits after the sentence that already made the point, and every one can be cut without losing a word of content. The test for a metaphor is to restate it plainly: if you cannot, it was carrying no mechanism, and the reader could not decode it either. "A silent clamp is a correctness bug wearing a seatbelt" did not survive that test; what it meant was that `saturating_sub` turns 5 minus 10 into 0 rather than an error, so a debt is recorded as no debt and nothing reports a problem. Simplify only as far as the meaning survives: "deposited funds are always returned" is simpler than the exit rule and wrong, because a taken escrow offer pays the maker in the *other* token rather than returning what they posted. Where the plainer sentence would change what is true, keep the harder one and make it concrete instead.
- **"Non-custodial" is a loaded word.** If the program locks funds in vaults during its lifecycle (every escrow, lending, AMM, leasing program does), don't claim "non-custodial" - it contradicts itself. What you usually mean is "no admin override, the rules are the deployed bytecode". Say that directly, or just describe the custody arrangement (program-owned vault, PDA signers, no admin escape hatch).
- **Upgrade authority is normal on Solana** - programs are usually upgradable so authors can ship security fixes. Don't apologise for it or treat it as disqualifying for "trustless" claims. Trust in the author/multisig is baseline; "trustless" means the documented rules can't be bypassed, not "bytecode frozen forever".
- **"Token" not "mint" in economic prose, and never the two as synonyms.** A mint is the onchain account that defines a type of token and controls its supply; a token is the asset. A mint is a token's factory, not a token, and holds no balance of anything - which is the test when a sentence is ambiguous: if it can be held, spent, or transferred it is a token, and if it can be created from or checked against it is the mint. So "one mint defines USDC", never "USDC is one mint"; "every distinct token has its own mint account", never "is a distinct mint account". In economic descriptions ("post token A as collateral, borrow token B"), say "token A" and "token B". Reserve "mint account" for technical descriptions of what gets passed to instructions.
- **Example assets are USDC and real-world assets**: stocks like NVDAx, TSLAx, and SPCXx. Never meme tokens - they date a document and make it read as speculation rather than finance.
- **Tokens are fungible by default - don't say so.** Don't write "fungible token" or sentences explaining that tokens are fungible. The reader knows. Only qualify when contrasting ("non-fungible token" / NFT). Same rule as not explaining what an integer is.
- **Don't be fascinated with "tokenization" - a tokenized asset is just an asset.** Drop the word "tokenized" from economic prose. A "basket of tokenized assets" is a "basket of assets"; "tokenized stocks like TSLAx and NVDAx" are "stocks like TSLAx and NVDAx". The fact that an asset is represented by a token onchain is the baseline assumption of everything here, not a notable property. Only mention tokenization when the act of representing an offchain asset onchain is itself the subject (e.g. explaining how an issuer mints a token backed by a real-world asset).
- **An invariant is an "invariant", never a "property".** "Property" also names an account's fields and a struct's members, so "property test", "the conservation property", and "pins the property" each read two ways, and one manuscript used the word both ways in the same chapter. Say "invariant test", "the conservation invariant", "pins the invariant"; where the thing is a rule one handler enforces rather than a condition that holds across every state change, say "rule". "Property" stays for a characteristic ("a PDA has three properties", "a good crank has three properties"). Name the testing libraries' own term, "property-based testing", at most once, so a reader can find those libraries.
- **One name per role/concept, enforced everywhere.** Pick a single term for each party (lessor/lessee, maker/taker, long/short, borrower/lender) and use ONLY that term throughout. Mixing terminology mid-document is how readers lose track of who owes what to whom.
- **Don't conflate "long the collateral" with "long the trade".** Anyone who posts collateral wants it to hold value (otherwise margin call), so every borrower is long their collateral. The directional bet is on the _borrowed_ asset, separately. Be precise about which "long" you mean.
- **Be careful with the word "securities".** It's a legal term. SOL is not a security. Asset-leasing is not "securities lending" even when the mechanics are analogous. Prefer "asset lending", "token lending", or "directional token lending" - and ask before picking one.
- **Spell out two-asset flows with concrete examples.** "Posts collateral and takes delivery of borrowed tokens" reads circular. "Posts USDC as collateral, borrows NVDAx" makes the asymmetry obvious. Don't make the reader infer that mints A and B are different things.
- **Name the instruction handlers in lifecycle prose.** When walking through "what the user does" (open position, close position, liquidate), name the actual handler (`take_lease`, `return_lease`, `liquidate`). Plain-English mechanics without handler names leave the reader unable to connect the narrative to the code.
- **When you explain or summarize a program** - a README, walkthrough, video script, or answer - follow [SUMMARIZING-PROGRAMS.md](SUMMARIZING-PROGRAMS.md): persona-driven casts, real assets, correct incentives, and step-by-step account state-transition + token-movement ledgers.

## General Coding Guidelines

### You are a deletionist

Your golden rule is "perfection isn't achieved when there's nothing more to add, rather perfection is achieved when there is nothing more to be taken away".

Remove:

- Comments that simply repeat what the code is doing, or the name of a variable, and do not add further insight.
- Repeated code that should be turned into a named function.
- Unused imports, unused constants, unused files, and comments that no longer apply.

Before deleting "stale" scaffolding, confirm it is actually dead: grep the CI workflows (`.github/workflows/`) and package scripts for references. Test files and package.json scripts that look like leftovers are sometimes exactly what CI runs.
- Doc-comments whose first line just paraphrases the identifier. `/// Pool authority PDA.` above `pub pool_authority` is noise. Either explain something the name doesn't (seed derivation, mutability rationale, type-choice reason, an invariant the reader can't see from the type) or delete the line.

Don't remove existing comments unless they are no longer useful or accurate.

### Communication Style

- Do not make disclaimers about being a "complete project" or state what works
- It is expected that work is complete and functional - no need to state this explicitly
- Avoid phrases like "This is a complete implementation" or "All features are working"
- Just deliver the work without meta-commentary about its completeness

### Config files: leave a comment explaining WHY

When you change a configuration value, or pin a version in any config file (`Anchor.toml`, `Cargo.toml`, `package.json`, CI workflows, `.gitignore`, `rust-toolchain.toml`), leave a comment explaining _why_. The next reader needs the rationale, not just the value.

- **Pinned versions:** what breaks without the pin? when can it be unpinned?
- **Non-default timeouts / limits:** why this number?
- **Removed sections:** what was it doing? why was it removed?
- **`.gitignore` exceptions:** why is this file tracked despite the rule?
- **Workarounds:** what's the proper fix? when can this be replaced? (mark with `TODO`)

Example:

```toml
# Pinned: 0.8.7 conflicts with litesvm's dep tree.
# Unpin when litesvm upgrades its ahash requirement.
ahash = "=0.8.6"
```

When you remove a section, only add why to the git commit, so the file is free of information that does not apply to its existing state.

### Working with Generated or Unfamiliar Code

**CRITICAL - Verify Before Use:**

- Before calling ANY function whose signature you don't know with certainty, read the actual source code/type definitions first
- NEVER guess or assume what parameters a function accepts based on what seems logical
- Don't invent convenience parameters that don't exist
- Generated code, third-party libraries, and unfamiliar codebases often have different APIs than you expect
- Common mistake: Assuming a function accepts high-level parameters → WRONG. Check the actual signature in the source files first

### Variable Naming

Ensure good variable naming. Rather than add comments to explain what things are, give them useful names.

**Don't do this:**

```typescript
// Foo
const shlerg = getFoo();
```

**Do this instead:**

```typescript
const foo = getFoo();
```

**Naming conventions:**

- Arrays should be plurals (`shoes`), items within arrays should be the singular (`shoes.forEach((shoe) => {...})`)
- Functions should be verby, like `calculateFoo` or `getBar`
- Avoid abbreviations, use full words (e.g., use `context` rather than `ctx`). Never use `e` for something thrown, use `thrownObject`, never use `v` when you mean `value`. There is almost no case where a single character variable is a good idea outside math (eg `p` and `q` for cryptography).
- Name a transaction some variant of `transaction`. Name instructions some variant of `instruction`. Name signatures some variant of `signature`. Do not confuse them - eg if the type looks like an instruction, you should not call it a 'transaction' because that is deceptive.

You can still add comments for additional context, just be careful to avoid comments that are explaining things that would be better conveyed by good variable naming.

### Code Quality

- Avoid 'magic numbers'. Make numbers either have a good variable name, a comment explaining why they are that value, or a reference to the URL you got the value from. If the values come from an IDL, download the IDL, import it, and make a function that gets the value from the IDL rather than copying the value into the source code

This is a magic number. Don't do this:

```ts
const FINALIZE_EVENT_DISCRIMINATOR = new Uint8Array([
  27, 75, 117, 221, 191, 213, 253, 249,
]);
```

Instead do this:

```ts
const FINALIZE_EVENT_DISCRIMINATOR = getEventDiscriminator(
  arciumIdl,
  "FinalizeComputationEvent",
);
```

- The code you are making is for production. You shouldn't have comments like `// In production we'd do this differently` or `**Implementation incomplete** - Needs program config handling and proper PDA derivations` or `**WORK IN PROGRESS**` in the final code you produce, or functions that return placeholder data. Instead: do the fucking work.

## Language-Specific Guidelines

The rules above apply to every file in the project. In addition, read the file that matches the language you are editing:

- **TypeScript** (Solana Kit clients, Solana Kit tests, browser code, anything `.ts`): see [TYPESCRIPT.md](TYPESCRIPT.md)
- **Rust - any Solana program/crate** (financial math, checked arithmetic, project structure, cargo hygiene): see [RUST.md](RUST.md)
- **Rust - Anchor** (`.rs` files using `anchor_lang`, LiteSVM tests): see [ANCHOR.md](ANCHOR.md), plus RUST.md for the shared rules
- **Rust - Anchor 2.0.0-rc.1** (porting a 1.x program, or writing v2): see [ANCHOR-V2.md](ANCHOR-V2.md), plus ANCHOR.md and RUST.md
- **Rust - Quasar** (`.rs` files using `quasar_lang`/`quasar_spl`/`quasar_test`): see [QUASAR.md](QUASAR.md), plus RUST.md for the shared rules

If a task touches more than one, read each.

For setting up a Solana toolchain in CI or a fresh remote container (Agave, platform-tools behind TLS-intercepting proxies, the Quasar CLI, building Anchor projects without the anchor CLI): see [ENVIRONMENT.md](ENVIRONMENT.md).

When you draw diagrams of Solana accounts (SVG figures for books, docs sites, or presentations): see [DIAGRAMS.md](DIAGRAMS.md).

## Git commits

Do not add "Co-Authored-By: Claude" or similar attribution when creating git commits.

Use the Linux/Git style `scope: description` for commit titles. [Do not use 'Conventional commits'](https://sumnerevans.com/posts/software-engineering/stop-using-conventional-commits/).

## Acknowledgment

- Acknowledge these guidelines have been applied when working on this project to indicate you have read these rules and found that they do apply to this project.
