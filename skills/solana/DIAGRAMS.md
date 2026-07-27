# Account Diagram Guidelines

**These build on [SKILL.md](SKILL.md)** and apply whenever you draw a diagram of Solana accounts — for a book chapter, a docs site, a presentation, or any other illustrated document. The medium is hand-authored SVG. SKILL.md's "no ASCII art, no Mermaid diagrams" rule still governs READMEs and chat output; when a project calls for real figures, these rules say how to draw them. For the text-only equivalent (state-transition ledgers in a README or walkthrough), see [SUMMARIZING-PROGRAMS.md](SUMMARIZING-PROGRAMS.md).

## Rounded corners mean accounts, and nothing else

- **Every Solana account is a rounded-corner rectangle.** Wallets, token accounts, mints, vaults, custom PDAs, config accounts, and programs — if it lives on chain as an account, it draws as a rect with `rx="6"`. One radius everywhere; do not drift to 4, 5, or 8.
- **Nothing else gets rounded corners.** Annotation panels, offchain parties (bots, cranks, browsers), state-machine states, instruction rows, transaction envelopes, gates and checks, price-level rows, field chips drawn inside an account, and failed or hypothetical calls all draw with square corners. The reader must be able to tell at a glance what exists on chain as an account and what does not.
- **A program is an account.** Draw it as a rounded rect, and list its instruction handlers inside, left-aligned, one per line. When a figure walks through one instruction, bold that handler's line.
- **A custom PDA's rectangle shows its struct as `key: value`, one field per line**, using the walkthrough's story values ("maker: Alice", "amount: 300"), not placeholder types.

## Include every account; fade the ones the step doesn't touch

- **Per-step figures show the program's whole account picture.** A chapter's account map fixes the layout: every participant, the program, the config and state PDAs, the vaults. Each walkthrough step reuses that layout with the same positions, so the reader keeps one mental map instead of re-orienting on every figure.
- **Accounts the step doesn't touch stay in the figure, faded.** Wrap each untouched account (its rect, its address dot, its labels) in `<g opacity="0.3">`. Accounts the step reads or writes draw at full ink. Fading is the only de-emphasis for accounts; do not delete an account from a step figure just because the step ignores it.
- **Emphasis is fading and line weight, never color.** Figures must survive greyscale print. Ink `#111` on paper `#fff`, panel fill `#f4f4f2`, greys `#444`/`#555`/`#888` for secondary text and dashed relationship lines.

## Token movement arrows carry coins

- **When tokens move, the arrow says so twice: a bold amount label, and coins at its midpoint.** Solid ink arrow (`stroke="#111"`, arrowhead marker), the amount as a bold label ("1 TSLAx", "447 USDC"), and a small coin glyph — two overlapping white-filled, ink-stroked circles with a currency mark — sitting on the middle of the arrow:

  ```svg
  <g transform="translate(MIDX,MIDY)">
    <circle cx="4" cy="2" r="6" fill="#fff" stroke="#111" stroke-width="1.3"/>
    <circle cx="-3" cy="0" r="6" fill="#fff" stroke="#111" stroke-width="1.3"/>
    <text x="-3" y="3" font-size="8" font-weight="bold" text-anchor="middle">$</text>
  </g>
  ```

- **Coins mean token value moved.** Lamport and rent flows, "owns", and "is authority for" relationships stay dashed grey (`stroke="#777" stroke-dasharray="5 3"`) with no coins. A reader scanning for value transfer follows the coins.

## Addresses are white-centered dots, labeled

- **Every address dot has a white center**, on-curve and off-curve alike: `<circle r="6" fill="#fff" stroke="#111" stroke-width="2"/>`. Do not fill address dots with ink.
- **Beside every dot, say what the address is.** An on-curve address gets the words `PUBLIC KEY`. A PDA or ATA gets the word `SEEDS` followed by its actual seed list: `SEEDS: "offer" + MAKER'S ADDRESS + ID`, or for an ATA, `SEEDS: OWNER'S ADDRESS + TOKEN PROGRAM + MINT`. The label carries the on-curve/off-curve distinction; the dots themselves are identical.
- **Seeds are never drawn inside the account's rectangle.** Seeds are inputs to the address, not fields of the struct. They live beside the address dot, which sits above or beside the account box, linked to it when layout demands by a short dashed grey line.

## Do not use

- Color to encode meaning — the palette is monochrome; fading and weight carry emphasis.
- `opacity` values other than the fade value `0.3` — one fade level, applied to whole account groups.
- Mermaid, Graphviz, or generated diagrams for account figures — the layout decisions (stable positions across steps, fade grouping) are the content, and generators cannot make them.
- Filled address dots, seeds inside account rects, or coins on non-token arrows.
