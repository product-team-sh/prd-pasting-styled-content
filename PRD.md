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
- **Keep Styling changes the whole sequence.** Today, one click on Keep Styling turns Remove Styling off for every email in the sequence, and the bar stops asking.

## 2. What we're building

1. **Remove Styling Automatically (per sequence) is the only rule for styling.** No new setting.
2. **The existing Styling Detected bar flags styled content from any source, every time, with both buttons**, in the email body and in spintax variants. Neither button ever changes the setting.
3. **Literal merge-tag and spintax tokens in a paste become chips.**

---

## 3. Safety Settings (prototype tab 1)

- Add one sentence to the Remove Styling Automatically description: *"Styling you paste or add in the editor is flagged there, with an option to remove it."*
- Under Remove Styling, add *"Spintax variants: same as above"* with an **On** / **Off** tag that mirrors the toggle. It is text, not a control.
- Everything else stays as it is.

## 4. Email editor (prototype tab 2)

| Remove Styling | Styled paste, or toolbar styling | Bar | × |
|---|---|---|---|
| **On** | Lands as is | Shows | Removes styling |
| **Off** | Lands as is | Shows | Closes the bar; styling stays |
| **Text only** | Lands as plain text | None | – |

- **Remove Styling** removes styling from this email. **Keep Styling** keeps it in this email. Both behave the same whether the setting is on or off.
- **Bar copy:** `Styling Detected (Remove to avoid spam filters):` · **Remove Styling** · **Keep Styling** · `×`. Both buttons always show.
- **The bar asks every time** styling appears: each paste, each toolbar style, and on opening a step that has styling. Keep Styling never silences it.
- **What Remove Styling keeps:** bold, italic, underline, links and lists. Fonts, sizes, colours and highlights go.
- **No control in the bar or editor writes the sequence setting.**
- **Text only** means "Send emails as text only" for all emails, or for step 1 when it is set to first email only. In text only, the font, size, colour and B/I/U controls are disabled.
- **The corner badge** ("Styling Removes Automatically" / "Text only email") stays visible above the bar.
- **Shortcut:** `⌘⇧V` / `Ctrl+Shift+V` always pastes plain, with no bar.

## 5. Spintax editor (prototype tab 3)

- **Same as §4 inside each variant field:** same bar, same buttons, same behaviour, following the sequence's Remove Styling.
- **Actions stay in their own variant.** Keep or Remove in one variant does not touch other variants or the email body.
- **Literal tokens pasted into a variant become chips.**

---

## 6. Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Remove Styling on or off: paste Word content with font, colour and highlight | Lands styled; bar shows with both buttons | P0 |
| A2 | Remove Styling on or off: apply a text colour from the toolbar | Colour stays; bar shows with both buttons | P0 |
| A3 | Click **Keep Styling** | Bar closes; styling stays. **No request to `PATCH /sequences/{id}/settings`.** Read back with `GET /sequences/{id}/config`: `text-only-email` unchanged | **P0, sign off by name. Owner: TBD** |
| A4 | After Keep Styling, paste or style again in the same session | Bar shows again | P0 |
| A5 | Click **Remove Styling** | Styling removed; bold, italic, underline, links and lists stay; no settings request | P0 |
| A6 | Click **×** | On: styling removed. Off: bar closes, styling stays. No settings request either way | P1 |
| A7 | Text only: paste Word content | Plain; no bar; "Text only email" badge | P0 |
| A8 | Paste plain text, or copy and paste within Saleshandy | No bar; chips stay chips | P0 |
| A9 | Paste literal `{{First Name}}` or `{spin}a\|b{endspin}` | Becomes a chip, in the body and in variants | P0 |
| A10 | `⌘⇧V` in any state | Plain; no bar | P0 |
| A11 | Safety Settings: toggle Remove Styling | Spintax tag switches between On and Off | P1 |
| A12 | Spintax: styled paste in variant 1, then Keep | Bar in variant 1 only; styling stays in variant 1; other variants and the body untouched | P0 |
| A13 | Spintax: keep styling in a variant, save, reopen, send a test | The spin still parses and spins; no literal `{spin}` in any output | **P0, release blocker** |

**Open before estimation:** A13 depends on the spin parser tolerating styling inside a variant. An [open bug](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458) says it does not today. Owner: Rajat.

---

**Related:** [card](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873) · [webui PR #8129](https://github.com/saleshandy/saleshandy-webui/pull/8129) (reuse its paste sanitiser and token-to-chip handling)
