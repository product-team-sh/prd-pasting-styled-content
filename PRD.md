# PRD – Pasting styled content: one rule, Remove Styling

**Design of record:** [prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/) · **Status:** Ready for dev review

These prototype elements are review aids, not product UI:

- the Simulate buttons
- the Remove Styling switcher above the editor and in the spintax header
- the "Same as the editor" / "Differs from editor" label
- the "How this is handled" notes

---

## 1. Problem

- **Styling from Word goes unnoticed.** Pasted Word content brings fonts, colours, sizes and highlights the writer didn't choose. With Remove Styling on, send strips them, so the editor shows something the prospect never gets.
- **Tokens arrive broken.** Literal `{{First Name}}` or `{spin}…{endspin}` in pasted text stays as braces instead of becoming a chip ([escalation](https://app.basecamp.com/4378325/buckets/16408283/todos/10213661915)).
- **Keep Styling switches the setting off without saying so.** Today, Keep Styling turns Remove Styling off for every email in the sequence, and nothing tells the writer.

## 2. What we're building

1. **Remove Styling Automatically (per sequence) is the only rule for styling**, at paste and at send. There is no new setting.
2. **With Remove Styling on, styling is removed as you paste, and the bar tells you.** Keep Styling brings it back and turns Remove Styling off for the sequence. The bar says so, with Undo.
3. **Literal merge-tag and spintax tokens in a paste become chips.**

---

## 3. Safety Settings (prototype tab 1)

- Add one sentence to the Remove Styling Automatically description: *"Styling is also removed from content you paste, including in spintax variants."*
- Everything else stays as it is.

## 4. Email editor (prototype tab 2)

| Remove Styling | Styled paste | Toolbar styling |
|---|---|---|
| **On** | Lands **without styling**; bar: removed on paste | Stays; bar: Styling Detected |
| **Off** | Lands as is; bar: Styling Detected | Stays; bar: Styling Detected |
| **Text only** | Lands as plain text; no bar | Controls disabled |

**The bar, by state**

| Bar | Copy | Actions |
|---|---|---|
| Removed on paste | *"✓ Styling removed from your paste (Remove Styling is on for this sequence):"* | **Keep Styling** · `×` |
| Styling Detected | *"Styling Detected (Remove to avoid spam filters):"* | **Remove Styling** · **Keep Styling** · `×` |
| After Keep, Remove Styling was on | *"ⓘ Styling kept. Remove Styling is now off for this sequence."* | **Undo** · `×` |

**Actions**

- **Keep Styling, Remove Styling on:**
    - Restores the pasted styling.
    - Turns Remove Styling **off** for the sequence.
    - Shows the "Styling kept" bar. **Undo** removes the styling again and turns the setting back on.
- **Keep Styling, Remove Styling off:** keeps it in this email. The setting is unchanged.
- **Remove Styling:** removes styling from this email. Bold, italic, underline, links and lists stay; fonts, sizes, colours and highlights go. The setting is unchanged.
- **`×`:** closes the bar. On a Styling Detected bar with Remove Styling on, it also removes the styling.

**Also**

- **What counts as text only:** "Send emails as text only" for all emails, or for step 1 when it is set to first email only.
- **The corner badge** ("Styling Removes Automatically" / "Text only email") stays visible above the bar.
- **`⌘⇧V` / `Ctrl+Shift+V`** always pastes plain, with no bar.

## 5. Spintax editor (prototype tab 3)

- **Same as §4 inside each variant field:** same bars, same actions, following the sequence's Remove Styling.
- **Keep Styling in a variant, with Remove Styling on,** turns the setting off for the whole sequence, body included. The "Styling kept" bar and Undo appear in that variant.
- **Literal tokens pasted into a variant become chips.**

---

## 6. Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Remove Styling **on**: paste Word content with font, colour and highlight | Lands without styling (bold and links stay); bar reads "Styling removed from your paste (Remove Styling is on for this sequence)" with **Keep Styling** | P0 |
| A2 | A1, then **Keep Styling** | Styling restored. **One** `PATCH /sequences/{id}/settings` sets `text-only-email` to `0`; `GET /sequences/{id}/config` confirms it. "Styling kept" bar shows with Undo; badge disappears | **P0, sign off by name. Owner: TBD** |
| A3 | A2, then **Undo** | Styling removed again; `text-only-email` back to `1`; badge returns | P0 |
| A4 | Remove Styling **off**: paste Word content | Lands styled; Styling Detected bar with both buttons | P0 |
| A5 | Remove Styling **off**: Remove Styling or Keep Styling | Acts on this email only; no settings request | P0 |
| A6 | Remove Styling on or off: apply a text colour from the toolbar | Colour stays; Styling Detected bar with both buttons | P0 |
| A7 | Text only: paste Word content | Plain; no bar; "Text only email" badge | P0 |
| A8 | Paste plain text, or copy and paste within Saleshandy | No bar; chips stay chips | P0 |
| A9 | Paste literal `{{First Name}}` or `{spin}a\|b{endspin}` | Becomes a chip, in the body and in variants | P0 |
| A10 | `⌘⇧V` in any state | Plain; no bar | P0 |
| A11 | Safety Settings | Remove Styling description carries the new sentence; no separate spintax control | P1 |
| A12 | Spintax, Remove Styling on: paste into variant 1, then Keep | Variant 1 restored; Remove Styling off for the sequence; "Styling kept" bar in variant 1 | P0 |
| A13 | Spintax: keep styling in a variant, save, reopen, send a test | The spin still parses and spins; no literal `{spin}` in any output | **P0, release blocker** |

**Open before estimation:** A13 depends on the spin parser tolerating styling inside a variant. An [open bug](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458) says it does not today. Owner: Rajat.

---

**Related:** [card](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873) · [webui PR #8129](https://github.com/saleshandy/saleshandy-webui/pull/8129) (reuse its paste sanitiser and token-to-chip handling)
