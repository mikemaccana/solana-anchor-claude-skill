# Account Diagram Guidelines

**These build on [SKILL.md](SKILL.md)** and apply whenever you draw a diagram of Solana accounts - for a book chapter, a docs site, a presentation, or any other illustrated document. The medium is hand-authored SVG. SKILL.md's "no ASCII art, no Mermaid diagrams" rule still governs READMEs and chat output; when a project calls for real figures, these rules say how to draw them. For the text-only equivalent (state-transition ledgers in a README or walkthrough), see [SUMMARIZING-PROGRAMS.md](SUMMARIZING-PROGRAMS.md).

## Rounded corners mean accounts, and nothing else

- **Every Solana account is a rounded-corner rectangle.** Wallets, token accounts, mints, vaults, custom PDAs, config accounts, and programs - if it lives on chain as an account, it draws as a rect with `rx="6"`. One radius everywhere; do not drift to 4, 5, or 8.
- **Nothing else gets rounded corners.** Annotation panels, offchain parties (bots, cranks, browsers), state-machine states, instruction rows, transaction envelopes, gates and checks, price-level rows, field chips drawn inside an account, and failed or hypothetical calls all draw with square corners. The reader must be able to tell at a glance what exists on chain as an account and what does not.
- **A program is an account.** Draw it as a rounded rect, and list its instruction handlers inside, left-aligned, one per line. When a figure walks through one instruction, mark that handler's line active (bold + accent).
- **A custom PDA's rectangle shows its struct as `key: value`, one field per line**, using the walkthrough's story values ("maker: Alice", "amount: 300"), not placeholder types. **Every key must be a field that actually appears in the struct definition, named verbatim** (the code's snake_case, e.g. `total_pool`, not a paraphrase) - never invent keys and never dress prose up as a field. Explanatory notes that aren't fields are italic annotation lines, visually distinct from the field list.
- **Every account box has the same anatomy: icon, title, rule, fields.** The icon and the title are centred on each other (for a title baseline `b`, the 14-unit glyph wrapper sits at `translate(x, b − 11)`), and a hairline runs beneath them across the box's inner width (`stroke="#888" stroke-width="0.8"`, ~5px under the title baseline), separating the heading from the fields so accounts are easy to tell apart at a glance. A box reduced to icon + title has nothing to separate and takes no rule.
- **A box reduced to its heading centres that heading vertically.** With no fields beneath it, a heading left at the box's top hangs off the ceiling with a band of dead space under it. Centre the icon/title pair on the box's middle instead: for a box at `y` of height `h`, the glyph wrapper sits at `translate(x, y + h/2 − 7)` and the title baseline follows it at `y + h/2 + 4`. The box keeps its own top position in the column - it is the *contents* that move, not the box.
- **Bold a value the step changes.** In a step figure, any value that this step changes is drawn bold - a PDA's `key: value` line and a token account's balance alike - so a reader sees what moved without re-reading the prose. Bold is ink weight only; the accent color stays reserved for the step's action.
- **Text stays inside its box, with padding.** Nothing inside an account rect may touch or cross the border: keep at least 6px of clear space between any text and every edge of the rect. If a line won't fit at the standard size, **reduce the font for the `key: value` (or handler) lines in that box** - 9.5 → 8.5 → 8, monospace 8.5 → 8 → 7.5 - rather than letting text overflow or clip. Verify by rendering.
- **People are wallet rectangles.** A person draws as a bordered rectangle with a head-and-shoulders glyph, a bold left-aligned name title (a role label like "Alice (maker)" where the figure has roles, plain "Alice's Account" otherwise), and their SOL balance beneath. Arrows carrying lamports or tokens to or from a person start and end at this rectangle's edge, never at a bare address dot and never at a stick figure.
- **No arrow or leader line passes through a label.** After re-anchoring, a label sitting on its own arrow's path is the most common defect there is, and a white halo cannot hide a stroke crossing a word. Labels sit beside their arrows, never on them.
- **Function names always carry parens**: `place_bet()`, never `place_bet` - in program boxes, arrow labels, annotations, and the surrounding prose alike. The parens are what let a reader tell a handler from an account or struct field at a glance. Struct fields, account names, and test names stay bare.

## The curve on top, columns beneath

- **The cluster curve is the part of the ball-shaped Ed25519 curve near X=0: flat at the top-left, gently sloping down and to the right.** Canonical path: `M 0,36 Q 400,36 640,116` on a width-640 canvas. It must be monotone - **never a curve that droops down in the middle, and never one that raises up in the middle** (no domes). The gentle slope leaves room underneath the right side of the curve for the rightmost column, and open empty space in the top-right corner **where all titles go**: the cluster label is right-aligned there (e.g. `x="628" y="24" text-anchor="end"`, white-haloed).
- **Every on-curve address dot - each person, each mint, the program - sits ON the curve.** For the canonical path, a dot at `cx` sits at `t = 2.5 − √(640000 − 640·cx)/320`, `cy = 36 + 80·t²` (reference points: x 0→36, 104→37, 210→42, 300→49, 404→62, 500→78, 548→90, 600→103, 640→116). The dot's `PUBLIC KEY` label sits above the dot in the open sky, or in the dot-to-box gap where the sky is taken, white-haloed.
- **Everything else hangs below the curve; a box's top clears the curve at the box's RIGHT edge by ≥6px** (the curve descends rightward, so the right edge is the binding point). Wallet boxes hang close beneath their dots on the left and progressively deeper toward the right; the rightmost (program) column's box top clears the curve's end height. **No line, arrow, or box ever crosses the curve.** A mint's small box sits in the open top-right region beneath the title (or in the thin strip above the curve's flat left), ≥6px clear of the curve, with its dot on the curve beneath it.
- **The curve is the heaviest line in the figure**: `stroke="#111" stroke-width="3.5"`. Token-movement arrows stay at `1.8`, box borders at `1.5` or below - nothing else may approach the curve's weight, so the cluster reads as the backbone at a glance.
- **A public-key account hugs its dot.** Put the dot at the point along the box's span where the curve comes *closest to the box* - for a box below the curve that is its **right** edge, for a box above the curve (a mint in the top strip) its **left** edge - then the box sits ~6px from the dot instead of floating below it. Placing the dot mid-box is the classic mistake: the curve descends across the box's width, so the required clearance gets measured at the far edge and the box drifts tens of pixels away from the dot it belongs to. Aim for a 6–16px gap; never exceed 20px. **If a box still cannot reach its dot, reduce the font size of every piece of text in that figure by one step** (10.5→9.5, 9.5→8.5, 8.5→8, 7.5→7, Menlo 8.5→8) and re-fit - a narrower box sits closer.
- **A person's accounts stack vertically beneath their dot**, in reading order: wallet box first, then their token account(s), then their per-program user/receipt account, then their bets/orders/positions. People read top to bottom; a reader finds Maria, then everything of Maria's, without scanning sideways.
- **Each column packs under its own wallet; columns do not line up across people.** The curve puts every person's dot at a different height, so their columns start at different heights and that is correct. A box hangs a normal margin below the one above it - just enough for the seed caption to sit in the gap with ~6px clearance top and bottom. Never align a row across columns by padding the short ones: that is what leaves a person's token account stranded 70px below their wallet.
- **Program-wide accounts stack beneath the program's dot** (rightmost column): the program box first, then config, then the market/event/pool PDA, then outcome/reserve PDAs, then program-owned vaults. Dashed authority links stay short and inside the column.
- **Step figures reuse the chapter account map's column layout.** Box top positions stay fixed across every figure in a chapter, so the reader keeps one mental map. Cross-column arrows route through the open space beneath the shorter columns.

## Fade what the step doesn't touch, down to its heading

- **Per-step figures show the program's whole account picture**, but accounts the step doesn't touch are wrapped in `<g opacity="0.3">` AND reduced to icon + title only - no struct fields, no balances. A shorter box, same top position, same address dot and seeds caption (faded), and the surviving heading centred in the box per the anatomy rule above. The detail lives in the figures where the account is actually doing something.
- **Accounts that do not exist yet at that point in the story are omitted**, not faded.
- **The fade level is `0.3`, and it is the only opacity value allowed.**

## One accent color, on the active story only

- **The palette is ink on paper plus one accent green**: ink `#111`, paper `#fff`, panel `#f4f4f2`, greys `#444`/`#555`/`#888` for secondary text and dashed lines, and accent `#1e7a3c` reserved for the step's action: the invoked handler line in the program box, token-movement arrows with their coins and amount labels, and NEW badges.
- **No arrow, leader, or dashed line may pass through a text label.** A label sitting on its own arrow's path is the most common defect after re-anchoring: the white halo is far too thin to hide a stroke crossing a word. Place each label beside its arrow, never on it, and verify by rendering at 4–5x - a stroke touching any glyph is a failure.
- **The accent must never carry meaning alone.** Accented elements are also the boldest marks below the curve (which stays the single heaviest line), so the figure reads identically in greyscale print. Never introduce a second hue, and never use the accent for anything the step isn't doing.

## Icons name the account kind

Each account box carries a small monochrome glyph beside its title, so the kind is scannable without reading. The five marks come from [Iconify](https://icon-sets.iconify.design/), wrapped so each fills the book's 14-unit glyph box at the shared line weight. **Paste them verbatim** - the inner `transform` is what fits and centres the mark, and retracing one by hand puts it back out of step with the rest.

| Account kind | Icon |
| --- | --- |
| Person (wallet) | `octicon:person-16` |
| Token account / vault | `hugeicons:piggy-bank` |
| Token mint | `boxicons:bank` |
| Data-struct PDA | `tabler:table` |
| Program | `streamline-flex:cog` |

- **Person (wallet)** - `octicon:person-16`:

  ```svg
  <g transform="translate(X,Y)"><g transform="translate(-1.607,-1.079) scale(1.076)" fill="#111"><path d="M10.561 8.073a6 6 0 0 1 3.432 5.142a.75.75 0 1 1-1.498.07a4.5 4.5 0 0 0-8.99 0a.75.75 0 0 1-1.498-.07a6 6 0 0 1 3.431-5.142a3.999 3.999 0 1 1 5.123 0M10.5 5a2.5 2.5 0 1 0-5 0a2.5 2.5 0 0 0 5 0"/></g></g>
  ```

- **Token account / vault** - `hugeicons:piggy-bank`:

  ```svg
  <g transform="translate(X,Y)"><g transform="translate(-0.62,-0.62) scale(0.635)" fill="none" stroke="#111" stroke-linecap="round" stroke-linejoin="round" stroke-width="2.047"><path d="M14.5 5.5h-4A6.5 6.5 0 0 0 7 17.478l.288 1.738c.07.423.105.635.202.798a1 1 0 0 0 .49.416c.178.07.392.07.822.07c.397 0 .596 0 .764-.062a1 1 0 0 0 .479-.374c.1-.148.15-.34.246-.727l.209-.837h3l.21.837c.096.386.144.58.245.727a1 1 0 0 0 .479.374c.168.062.367.062.764.062c.43 0 .644 0 .821-.07a1 1 0 0 0 .49-.416c.098-.163.133-.375.203-.798L17 17.478a6.5 6.5 0 0 0 2.502-2.978l.89-.178c.77-.154 1.155-.231 1.381-.508c.227-.276.227-.669.227-1.453v-.3c0-.75 0-1.124-.212-1.396c-.212-.27-.575-.362-1.303-.544L20 10c0-1.5-1.167-2.833-2-3.5v-3h-.264c-1.37 0-2.623.774-3.236 2"/><path d="M15.875 9.75h-.125m.25 0a.25.25 0 1 1-.5 0a.25.25 0 0 1 .5 0M2 8v2a2 2 0 0 0 2 2"/></g></g>
  ```

- **Token mint** - `boxicons:bank`:

  ```svg
  <g transform="translate(X,Y)"><g transform="translate(-1.4,-1.401) scale(0.7)" fill="#111"><path d="m21.49 7.13l-9-5a.99.99 0 0 0-.97 0l-9.01 5C2.19 7.31 2 7.64 2 8v3c0 .55.45 1 1 1h2v4H3c-.55 0-1 .45-1 1v4c0 .55.45 1 1 1h18c.55 0 1-.45 1-1v-4c0-.55-.45-1-1-1h-2v-4h2c.55 0 1-.45 1-1V8a1 1 0 0 0-.51-.87M7 12h2v4H7zm6 0v4h-2v-4zm7 6v2H4v-2zm-3-2h-2v-4h2zm3-6H4V8.59l8-4.44l8 4.44z"/><path d="M12 6a1.5 1.5 0 1 0 0 3a1.5 1.5 0 1 0 0-3"/></g></g>
  ```

- **Data-struct PDA** - `tabler:table`:

  ```svg
  <g transform="translate(X,Y)"><g transform="translate(-1.467,-1.467) scale(0.706)" fill="none" stroke="#111" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.843"><path d="M3 5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2zm0 5h18M10 3v18"/></g></g>
  ```

- **Program** - `streamline-flex:cog`:

  ```svg
  <g transform="translate(X,Y)"><g transform="translate(-0.113,-0.112) scale(1.016)" fill="none" stroke="#111" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.28"><path d="M11.808 7.727c0 .05.017.1.048.14l.667.84a.96.96 0 0 1 .082 1.08l-.392.676a.96.96 0 0 1-.975.47l-1.063-.16a.23.23 0 0 0-.145.027l-1.26.727a.23.23 0 0 0-.096.112l-.392 1a.96.96 0 0 1-.893.611h-.782a.96.96 0 0 1-.893-.61l-.392-1a.23.23 0 0 0-.096-.113L3.967 10.8a.23.23 0 0 0-.145-.028l-1.063.162a.96.96 0 0 1-.976-.47l-.39-.677a.96.96 0 0 1 .08-1.08l.67-.84a.22.22 0 0 0 .05-.14V6.273c0-.05-.018-.1-.05-.14l-.67-.84a.96.96 0 0 1-.08-1.08l.39-.676a.96.96 0 0 1 .976-.47l1.06.16a.23.23 0 0 0 .145-.027l1.262-.73a.23.23 0 0 0 .096-.113l.394-.996A.96.96 0 0 1 6.61.75h.784a.96.96 0 0 1 .893.61l.392.997q.028.075.096.116l1.26.727a.23.23 0 0 0 .145.028l1.062-.162a.96.96 0 0 1 .976.47l.391.677a.96.96 0 0 1-.081 1.08l-.67.84a.22.22 0 0 0-.049.14v1.454Z"/><path d="M7 8.996c1.277 0 1.996-.719 1.996-1.996S8.277 5.004 7 5.004S5.004 5.723 5.004 7S5.723 8.996 7 8.996"/></g></g>
  ```

Draw the glyphs exactly as specified - one vocabulary book-wide, no per-figure variants. Four of the sets are MIT; Streamline's Flex free icons are CC BY 4.0, so a document that ships these marks credits the sets it uses.

## Token movement arrows carry coins

- **When tokens move, say the amount once and mark it with coins.** Solid accent arrow (`stroke="#1e7a3c"`, matching arrowhead marker), the amount as a bold accent label ("1 TSLAx", "447 USDC"), and the coin mark immediately beside that label - about 5px clear of it, vertically centred, on the label's left unless something already occupies that space:

  ```svg
  <g class="coin" transform="translate(X,Y)" fill="#fff">
    <g stroke="#fff" stroke-width="4.5"><circle cx="5" cy="1.5" r="4.5"/><circle cx="0" cy="0" r="4.5"/><circle cx="-5" cy="1.5" r="4.5"/></g>
    <g stroke="#1e7a3c" stroke-width="1.3"><circle cx="5" cy="1.5" r="4.5"/><circle cx="0" cy="0" r="4.5"/><circle cx="-5" cy="1.5" r="4.5"/></g>
  </g>
  ```

- **Three rings, and no currency symbol inside them.** Two rings read as a pair of address dots at print size, where the accent colour is gone and a 2mm mark is all the reader has; three is unmistakable. The `$` that used to sit in the front ring asserted dollars on arrows carrying ACME, NVDAx, SPCXx, LP tokens and SOL - the ticker in the adjacent label is what names the asset, and the mark only has to say *tokens move here*. The doubled group is a white halo, the same trick the labels use, so the mark occludes any arrow it sits on.
- **One coin mark per amount, and none without an amount.** The coin belongs to the *statement of the quantity*, not to the arrow, so a figure has exactly as many coin marks as it has amount labels. Where a numbered legend carries the amounts, the coins sit in the legend beside them; where one legend line covers both legs of a handler call ("5 NVDAx in, 5 NVDAx shares out"), that is one amount and gets one coin. An arrow that moves value and states no amount is the bug - add the amount rather than leave a coin floating on a bare arrow.

- **An arrow is one smooth stroke.** Where two Bézier segments meet, the incoming handle, the shared anchor and the outgoing handle must be collinear, or the stroke shows a corner in the middle of a flow. Route an arrow with as few segments as it needs, and when a join is unavoidable, aim both handles along one direction and keep their lengths.
- **The arrowhead points along the stroke it terminates, and the line meets the middle of its base.** With `orient="auto"` the head is rotated to the tangent at the path's *very last point*, so a short terminal hook - a final `C` whose last control point sits sideways from the endpoint - swings the head away from the line the reader actually sees, and the stroke emerges through the head's flank. The head is about `9 × stroke-width` long, so the last ~16 units of the path must already be travelling in the head's direction. Rotating the head is safe: `refX="8"` of `markerWidth="9"` puts the anchor at the tip, so the tip stays on the box it points at and only the base swings.
- **Tokens move directly between accounts - nothing sits in between.** An arrow runs from one account box to another, unbroken. Never terminate it on a panel (a fee gate, a check, a calculation) and resume from the far side: on chain there is no intermediate holder, and drawing one invents a place for value to rest. Fees, gates and math belong in an italic annotation *beside* the arrow, or as the resulting field value inside the account that records it.
- **The arrow's label is the amount actually transferred.** If the handler moves 100 and prices the trade off 99.70, the arrow says 100 and the 99.70 lives in an annotation. Labelling the arrow with a derived intermediate quantity misstates what the instruction does.
- **An asset never changes type along one arrow.** A token account holds exactly one mint, and a transfer moves that mint's units to another account of the same mint, so an arrow from a USDC account to a TSLAx account draws an instruction that cannot exist. A swap is two arrows and a venue between them: USDC out to the swap program, the bought asset back from it, each leg labelled with the amount and ticker that leg actually carries ("360 USDC out", "1.44 TSLAx in"). The same applies to a deposit that is deployed at weights, a rebalance, and a liquidation that seizes one asset to repay another: whenever the ticker at the two ends of an arrow differs, a step is missing from the figure.
- **A callout never restates account state.** If a step changes `admin_fees_owed_b`, show it as that field's new value inside its PDA box - bold, per the rule above - not as a separate box saying `admin_fees_owed_b += $0.05`. The account is the single source of truth for its own state; a floating callout duplicates it and will drift.
- **Coins mean token value moved, in this step.** Lamport and rent flows, "owns", and "is authority for" relationships stay dashed grey (`stroke="#777" stroke-dasharray="5 3"`) with no coins and no accent. A coin never appears on a faded element or on a movement the figure is not illustrating: the mark is accent green, and accent green means the step's action. A reader scanning for value transfer follows the green.

## Addresses are white-centered dots

- **Every address dot has a white center**, on-curve and off-curve alike: `<circle r="6" fill="#fff" stroke="#111" stroke-width="2"/>`. Do not fill address dots with ink.
- **A faded dot still occludes the curve.** `<g opacity="0.3">` composites the whole group, so a faded dot's white centre goes translucent along with everything else and the curve - the heaviest line in the figure - shows straight through the dot, which no unfaded dot ever does. Back the dot with an opaque white disc drawn immediately *before* its faded group, so it sits above the curve and beneath the dot: `<circle cx="X" cy="Y" r="7" fill="#fff"/>` for an `r="6"` dot, `r="6.5"` for an `r="5.5"` one. The extra unit covers the dot's 2-wide ring, so the curve is hidden under the whole mark rather than showing through the ring.
- **A dot on the curve carries no label.** Its position on the curve already says the address is a public key with a private key behind it. Do not write `PUBLIC KEY` beside it - teach the convention once, in prose, where the reader first meets a person and their token account.
- **An off-curve dot carries its seed list, and nothing else.** Write the seeds themselves - `"offer" + MAKER'S ADDRESS + ID`, or for an ATA `ATA PROGRAM + OWNER'S ADDRESS + MINT`. No `SEEDS:` prefix: a list of seeds is self-evidently a list of seeds, and the absence of a curve dot is what marks it as derived.
- **A built-in address hangs off the curve and carries its address, not seeds.** The System Program, the token programs, and the sysvars are neither public keys somebody holds the key for nor PDAs derived from seeds: the byte string was chosen and the runtime treats it as the thing it names. Draw the dot off the curve, because placement encodes whether a signature can ever move what the address holds, and label it with the address itself, truncated (`SysvarC1ock111...`), in the monospace face at the seed list's size. No derivation arrow: nothing derives it, and the missing arrow is what distinguishes it from a PDA at a glance. Whether the bytes happen to decode to a point on the curve is an accident and never a reason to draw the dot on it.
- **Seeds are never drawn inside the account's rectangle.** Seeds are inputs to the address, not fields of the struct. They live beside the address dot, above or beside the account box.
- **Every dot has an account rectangle.** A dot drawn on its own says the address exists but holds no account, which is never what a figure means. Draw the account too, rounded, with its icon and title, so a figure never implies that public keys have accounts and PDAs do not. The one exception is a legend key, where a lone dot beside its explanatory line is the whole point.

## Some accounts may be named instead of drawn

- **An account may be named instead of drawn when the figure says nothing about it.** The test is a property of the account, not of how crowded the page is: every reference to it in that figure is a seed component or a field naming it, the figure states none of its own fields, and no arrow starts or ends on it. Mint accounts on an account map usually qualify - a box holding a title and no fields is the least informative object on the page, and the seed lists already name things they do not draw, `TOKEN PROGRAM` among them.
- **An account the figure has something to say about is never omittable.** If the figure states one of its fields, if an arrow starts or ends on it, or if the prose calls it out by role, it is drawn. A mint that is minted to or burned from is touched by an arrow and stays, which is why a share mint stays in every figure of the fund that issues it.
- **A figure that omits an account says so, and names it.** One italic annotation in the same register as the figure's other notes, listing each omitted account and why they were safe to omit - "not drawn: the TSLAx, NVDAx and USDC mints - every account that names them does so as a seed." A silent omission reads as an account model with a hole in it.
- **This is a permission, not an obligation.** A figure with room may draw every account, and existing figures are not wrong for doing so. Nothing here is mechanically checked: which references are "only a seed" is a judgement the linter cannot make.

## Do not use

- A second hue, or the accent on anything except the step's action.
- `opacity` values other than the fade value `0.3`.
- Mermaid, Graphviz, or generated diagrams for account figures - the layout decisions (stable columns, fade grouping) are the content, and generators cannot make them.
- Filled address dots, bare dots with no account rectangle, seeds inside account rects, coins on non-token arrows, or bare handler names without `()`.
- Hand-drawn substitutes for the five account glyphs, or a heading left at the top of a box that carries nothing else.
- A faded address dot with the cluster curve showing through its centre.
