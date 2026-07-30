# Account Diagram Guidelines

**These build on [SKILL.md](SKILL.md)** and apply whenever you draw a diagram of Solana accounts — for a book chapter, a docs site, a presentation, or any other illustrated document. The medium is hand-authored SVG. SKILL.md's "no ASCII art, no Mermaid diagrams" rule still governs READMEs and chat output; when a project calls for real figures, these rules say how to draw them. For the text-only equivalent (state-transition ledgers in a README or walkthrough), see [SUMMARIZING-PROGRAMS.md](SUMMARIZING-PROGRAMS.md).

## Rounded corners mean accounts, and nothing else

- **Every Solana account is a rounded-corner rectangle.** Wallets, token accounts, mints, vaults, custom PDAs, config accounts, and programs — if it lives on chain as an account, it draws as a rect with `rx="6"`. One radius everywhere; do not drift to 4, 5, or 8.
- **Nothing else gets rounded corners.** Annotation panels, offchain parties (bots, cranks, browsers), state-machine states, instruction rows, transaction envelopes, gates and checks, price-level rows, field chips drawn inside an account, and failed or hypothetical calls all draw with square corners. The reader must be able to tell at a glance what exists on chain as an account and what does not.
- **A program is an account.** Draw it as a rounded rect, and list its instruction handlers inside, left-aligned, one per line. When a figure walks through one instruction, mark that handler's line active (bold + accent).
- **A custom PDA's rectangle shows its struct as `key: value`, one field per line**, using the walkthrough's story values ("maker: Alice", "amount: 300"), not placeholder types. **Every key must be a field that actually appears in the struct definition, named verbatim** (the code's snake_case, e.g. `total_pool`, not a paraphrase) — never invent keys and never dress prose up as a field. Explanatory notes that aren't fields are italic annotation lines, visually distinct from the field list.
- **Every account box has the same anatomy: icon, title, rule, fields.** The icon and the title are centred on each other (for a title baseline `b`, the 14-unit glyph wrapper sits at `translate(x, b − 11)`), and a hairline runs beneath them across the box's inner width (`stroke="#888" stroke-width="0.8"`, ~5px under the title baseline), separating the heading from the fields so accounts are easy to tell apart at a glance. A box reduced to icon + title has nothing to separate and takes no rule.
- **Bold a value the step changes.** In a step figure, any value that this step changes is drawn bold — a PDA's `key: value` line and a token account's balance alike — so a reader sees what moved without re-reading the prose. Bold is ink weight only; the accent color stays reserved for the step's action.
- **Text stays inside its box, with padding.** Nothing inside an account rect may touch or cross the border: keep at least 6px of clear space between any text and every edge of the rect. If a line won't fit at the standard size, **reduce the font for the `key: value` (or handler) lines in that box** — 9.5 → 8.5 → 8, monospace 8.5 → 8 → 7.5 — rather than letting text overflow or clip. Verify by rendering.
- **Function names always carry parens**: `place_bet()`, never `place_bet` — in program boxes, arrow labels, annotations, and the surrounding prose alike. The parens are what let a reader tell a handler from an account or struct field at a glance. Struct fields, account names, and test names stay bare.

## The curve on top, columns beneath

- **The cluster curve is the part of the ball-shaped Ed25519 curve near X=0: flat at the top-left, gently sloping down and to the right.** Canonical path: `M 0,36 Q 400,36 640,116` on a width-640 canvas. It must be monotone — **never a curve that droops down in the middle, and never one that raises up in the middle** (no domes). The gentle slope leaves room underneath the right side of the curve for the rightmost column, and open empty space in the top-right corner **where all titles go**: the cluster label is right-aligned there (e.g. `x="628" y="24" text-anchor="end"`, white-haloed).
- **Every on-curve address dot — each person, each mint, the program — sits ON the curve.** For the canonical path, a dot at `cx` sits at `t = 2.5 − √(640000 − 640·cx)/320`, `cy = 36 + 80·t²` (reference points: x 0→36, 104→37, 210→42, 300→49, 404→62, 500→78, 548→90, 600→103, 640→116). The dot's `PUBLIC KEY` label sits above the dot in the open sky, or in the dot-to-box gap where the sky is taken, white-haloed.
- **Everything else hangs below the curve; a box's top clears the curve at the box's RIGHT edge by ≥6px** (the curve descends rightward, so the right edge is the binding point). Wallet boxes hang close beneath their dots on the left and progressively deeper toward the right; the rightmost (program) column's box top clears the curve's end height. **No line, arrow, or box ever crosses the curve.** A mint's small box sits in the open top-right region beneath the title (or in the thin strip above the curve's flat left), ≥6px clear of the curve, with its dot on the curve beneath it.
- **The curve is the heaviest line in the figure**: `stroke="#111" stroke-width="3.5"`. Token-movement arrows stay at `1.8`, box borders at `1.5` or below — nothing else may approach the curve's weight, so the cluster reads as the backbone at a glance.
- **A public-key account hugs its dot.** Put the dot at the point along the box's span where the curve comes *closest to the box* — for a box below the curve that is its **right** edge, for a box above the curve (a mint in the top strip) its **left** edge — then the box sits ~6px from the dot instead of floating below it. Placing the dot mid-box is the classic mistake: the curve descends across the box's width, so the required clearance gets measured at the far edge and the box drifts tens of pixels away from the dot it belongs to. Aim for a 6–16px gap; never exceed 20px. **If a box still cannot reach its dot, reduce the font size of every piece of text in that figure by one step** (10.5→9.5, 9.5→8.5, 8.5→8, 7.5→7, Menlo 8.5→8) and re-fit — a narrower box sits closer.
- **A person's accounts stack vertically beneath their dot**, in reading order: wallet box first, then their token account(s), then their per-program user/receipt account, then their bets/orders/positions. People read top to bottom; a reader finds Maria, then everything of Maria's, without scanning sideways.
- **Each column packs under its own wallet; columns do not line up across people.** The curve puts every person's dot at a different height, so their columns start at different heights and that is correct. A box hangs a normal margin below the one above it — just enough for the seed caption to sit in the gap with ~6px clearance top and bottom. Never align a row across columns by padding the short ones: that is what leaves a person's token account stranded 70px below their wallet.
- **Program-wide accounts stack beneath the program's dot** (rightmost column): the program box first, then config, then the market/event/pool PDA, then outcome/reserve PDAs, then program-owned vaults. Dashed authority links stay short and inside the column.
- **Step figures reuse the chapter account map's column layout.** Box top positions stay fixed across every figure in a chapter, so the reader keeps one mental map. Cross-column arrows route through the open space beneath the shorter columns.

## Fade what the step doesn't touch, down to its heading

- **Per-step figures show the program's whole account picture**, but accounts the step doesn't touch are wrapped in `<g opacity="0.3">` AND reduced to icon + title only — no struct fields, no balances. A shorter box, same top position, same address dot and seeds caption (faded). The detail lives in the figures where the account is actually doing something.
- **Accounts that do not exist yet at that point in the story are omitted**, not faded.
- **The fade level is `0.3`, and it is the only opacity value allowed.**

## One accent color, on the active story only

- **The palette is ink on paper plus one accent green**: ink `#111`, paper `#fff`, panel `#f4f4f2`, greys `#444`/`#555`/`#888` for secondary text and dashed lines, and accent `#1e7a3c` reserved for the step's action: the invoked handler line in the program box, token-movement arrows with their coins and amount labels, and NEW badges.
- **No arrow, leader, or dashed line may pass through a text label.** A label sitting on its own arrow's path is the most common defect after re-anchoring: the white halo is far too thin to hide a stroke crossing a word. Place each label beside its arrow, never on it, and verify by rendering at 4–5x — a stroke touching any glyph is a failure.
- **The accent must never carry meaning alone.** Accented elements are also the boldest marks below the curve (which stays the single heaviest line), so the figure reads identically in greyscale print. Never introduce a second hue, and never use the accent for anything the step isn't doing.

## Icons name the account kind

Each account box carries a small monochrome glyph beside its title, so the kind is scannable without reading:

- **Person (wallet)** — generic head and shoulders:

  ```svg
  <g transform="translate(X,Y)">
    <circle cx="7" cy="4" r="3.1" fill="none" stroke="#111" stroke-width="1.3"/>
    <path d="M 1.5,13 Q 1.5,8.4 7,8.4 Q 12.5,8.4 12.5,13" fill="none" stroke="#111" stroke-width="1.3"/>
  </g>
  ```

- **Token account / vault** — piggy bank. The curly tail trails behind the body on the left; the snout is opaque (`fill="#fff"`, drawn after the body so the body's outline never shows through it) and carries two nostrils. Keep this element order — it is what makes the snout cover the body edge:

  ```svg
  <g transform="translate(X,Y)">
    <path d="M 2.6,7.8 Q 0.6,7.4 0.9,5.6 Q 1.1,4.4 2.3,4.9" fill="none" stroke="#111" stroke-width="1"/>
    <ellipse cx="8" cy="7" rx="5.4" ry="4.4" fill="none" stroke="#111" stroke-width="1.2"/>
    <ellipse cx="12.9" cy="6.4" rx="1.8" ry="1.5" fill="#fff" stroke="#111" stroke-width="1"/>
    <circle cx="12.35" cy="6.4" r="0.35" fill="#111"/>
    <circle cx="13.45" cy="6.4" r="0.35" fill="#111"/>
    <line x1="5.2" y1="11.4" x2="5.2" y2="13" stroke="#111" stroke-width="1.2"/>
    <line x1="10.6" y1="11.4" x2="10.6" y2="13" stroke="#111" stroke-width="1.2"/>
    <line x1="6.8" y1="4.4" x2="9.2" y2="4.4" stroke="#111" stroke-width="1"/>
  </g>
  ```

- **Program** — gear:

  ```svg
  <g transform="translate(X,Y)">
    <circle cx="7" cy="7" r="3.4" fill="none" stroke="#111" stroke-width="1.3"/>
    <circle cx="7" cy="7" r="1.2" fill="none" stroke="#111" stroke-width="1"/>
    <g stroke="#111" stroke-width="1.3">
      <line x1="7" y1="1" x2="7" y2="3"/><line x1="7" y1="11" x2="7" y2="13"/>
      <line x1="1" y1="7" x2="3" y2="7"/><line x1="11" y1="7" x2="13" y2="7"/>
      <line x1="2.8" y1="2.8" x2="4.2" y2="4.2"/><line x1="9.8" y1="9.8" x2="11.2" y2="11.2"/>
      <line x1="11.2" y1="2.8" x2="9.8" y2="4.2"/><line x1="2.8" y1="11.2" x2="4.2" y2="9.8"/>
    </g>
  </g>
  ```

- **Data-struct PDA** — table:

  ```svg
  <g transform="translate(X,Y)" stroke="#111" stroke-width="1.2" fill="none">
    <rect x="1" y="2" width="12" height="10"/>
    <line x1="1" y1="5.4" x2="13" y2="5.4"/>
    <line x1="5.5" y1="5.4" x2="5.5" y2="12"/>
  </g>
  ```

- **Mint** — a mint building, pediment over columns:

  ```svg
  <g transform="translate(X,Y)" stroke="#111" stroke-width="1.2" fill="none">
    <path d="M 1,5 L 7,1 L 13,5 Z"/>
    <line x1="3" y1="6.5" x2="3" y2="11"/>
    <line x1="7" y1="6.5" x2="7" y2="11"/>
    <line x1="11" y1="6.5" x2="11" y2="11"/>
    <line x1="1.5" y1="12.5" x2="12.5" y2="12.5"/>
  </g>
  ```

Draw the glyphs exactly as specified — one vocabulary book-wide, no per-figure variants.

## Token movement arrows carry coins

- **When tokens move, say the amount once and mark it with coins.** Solid accent arrow (`stroke="#1e7a3c"`, matching arrowhead marker), the amount as a bold accent label ("1 TSLAx", "447 USDC"), and the coin mark immediately beside that label — about 5px clear of it, vertically centred, on the label's left unless something already occupies that space:

  ```svg
  <g class="coin" transform="translate(X,Y)" fill="#fff">
    <g stroke="#fff" stroke-width="4.5"><circle cx="5" cy="1.5" r="4.5"/><circle cx="0" cy="0" r="4.5"/><circle cx="-5" cy="1.5" r="4.5"/></g>
    <g stroke="#1e7a3c" stroke-width="1.3"><circle cx="5" cy="1.5" r="4.5"/><circle cx="0" cy="0" r="4.5"/><circle cx="-5" cy="1.5" r="4.5"/></g>
  </g>
  ```

- **Three rings, and no currency symbol inside them.** Two rings read as a pair of address dots at print size, where the accent colour is gone and a 2mm mark is all the reader has; three is unmistakable. The `$` that used to sit in the front ring asserted dollars on arrows carrying ACME, NVDAx, SPCXx, LP tokens and SOL — the ticker in the adjacent label is what names the asset, and the mark only has to say *tokens move here*. The doubled group is a white halo, the same trick the labels use, so the mark occludes any arrow it sits on.
- **One coin mark per amount, and none without an amount.** The coin belongs to the *statement of the quantity*, not to the arrow, so a figure has exactly as many coin marks as it has amount labels. Where a numbered legend carries the amounts, the coins sit in the legend beside them; where one legend line covers both legs of a handler call ("5 NVDAx in, 5 NVDAx shares out"), that is one amount and gets one coin. An arrow that moves value and states no amount is the bug — add the amount rather than leave a coin floating on a bare arrow.

- **Tokens move directly between accounts — nothing sits in between.** An arrow runs from one account box to another, unbroken. Never terminate it on a panel (a fee gate, a check, a calculation) and resume from the far side: on chain there is no intermediate holder, and drawing one invents a place for value to rest. Fees, gates and math belong in an italic annotation *beside* the arrow, or as the resulting field value inside the account that records it.
- **The arrow's label is the amount actually transferred.** If the handler moves 100 and prices the trade off 99.70, the arrow says 100 and the 99.70 lives in an annotation. Labelling the arrow with a derived intermediate quantity misstates what the instruction does.
- **A callout never restates account state.** If a step changes `admin_fees_owed_b`, show it as that field's new value inside its PDA box — bold, per the rule above — not as a separate box saying `admin_fees_owed_b += $0.05`. The account is the single source of truth for its own state; a floating callout duplicates it and will drift.
- **Coins mean token value moved, in this step.** Lamport and rent flows, "owns", and "is authority for" relationships stay dashed grey (`stroke="#777" stroke-dasharray="5 3"`) with no coins and no accent. A coin never appears on a faded element or on a movement the figure is not illustrating: the mark is accent green, and accent green means the step's action. A reader scanning for value transfer follows the green.

## Addresses are white-centered dots

- **Every address dot has a white center**, on-curve and off-curve alike: `<circle r="6" fill="#fff" stroke="#111" stroke-width="2"/>`. Do not fill address dots with ink.
- **A dot on the curve carries no label.** Its position on the curve already says the address is a public key with a private key behind it. Do not write `PUBLIC KEY` beside it — teach the convention once, in prose, where the reader first meets a person and their token account.
- **An off-curve dot carries its seed list, and nothing else.** Write the seeds themselves — `"offer" + MAKER'S ADDRESS + ID`, or for an ATA `ATA PROGRAM + OWNER'S ADDRESS + MINT`. No `SEEDS:` prefix: a list of seeds is self-evidently a list of seeds, and the absence of a curve dot is what marks it as derived.
- **Seeds are never drawn inside the account's rectangle.** Seeds are inputs to the address, not fields of the struct. They live beside the address dot, above or beside the account box.

## Do not use

- A second hue, or the accent on anything except the step's action.
- `opacity` values other than the fade value `0.3`.
- Mermaid, Graphviz, or generated diagrams for account figures — the layout decisions (stable columns, fade grouping) are the content, and generators cannot make them.
- Filled address dots, seeds inside account rects, coins on non-token arrows, or bare handler names without `()`.
