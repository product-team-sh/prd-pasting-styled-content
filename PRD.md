# PRD – Pasting styled content: one rule, Remove Styling

**Design of record:** [prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/) · **Status:** Ready for dev review

---

## 1. Problem

**Pasting brings in styling the writer did not choose.** Writers paste into the step editor from Word, Google Docs and old emails. Word content arrives with its fonts, colours, sizes and highlights, plus hidden markup the writer cannot see.

- With Remove Styling on, that styling is stripped at send, so the editor shows something the prospect never receives.
- With Remove Styling off, it goes out as pasted.

**Tokens in a paste arrive broken.** Spintax and merge-tag syntax in pasted text lands as literal braces instead of chips. That text comes from external docs or AI-written copy. A customer escalated after literal spintax text reached prospects ([CS thread](https://app.basecamp.com/4378325/buckets/16408283/todos/10213661915)).

**Keeping styling in one email switches off styling removal for the whole sequence.** One click on Keep Styling in the Styling Detected bar silently turns Remove Styling off for every email in the sequence. The bar then stops asking for the rest of the session. The writer meant "keep it here" and changed the sequence's deliverability setting instead.

---

## 2. Why this approach

| Approach | Rejected on |
|---|---|
| Clean styling automatically as it lands | The writer neither sees nor chooses what was removed, and toolbar styling would be handled differently from pasted styling |
| A separate paste setting | It duplicates Remove Styling, which is on for 81.9% of new sequences (measured), and the two could disagree |
| **Reuse the Styling Detected bar for every source of styling** | Chosen. It is already in the product and already understood, and one surface covers paste, toolbar and existing content |

**Evidence**

| Figure | Tag |
|---|---|
| New sequences default to Remove Styling on ([edge `sequence-setting.ts#L57`](https://github.com/saleshandy/saleshandy-edge/blob/d908da065793036a3663b4f83b62c7f2a65078a6/src/sequence/enums/sequence-setting.ts#L57)) | measured, code |
| Of 64,928 sequences created since 1 Jul 2026: 81.9% Remove Styling, 16.6% off, 1.5% text only | measured, Metabase. Covers created sequences, not active ones |
| A Word paste and toolbar colour both land styled and show the bar. Keep Styling saves `text-only-email = 0`. Remove Styling saves `1`, sent twice | measured, live app, 7 Oct |
| A non-Word HTML paste already loses its inline styles | measured, one synthetic paste, 7 Oct |

**We do not claim** any inbox-placement improvement. We claim only that the writer decides what styling stays, and that the sequence's rule never changes by accident.

---

## 3. What we're building

1. **Remove Styling Automatically is the one rule for styling, per sequence, at paste and at send.** There is no other setting.
2. **The Styling Detected bar flags styled content from every source, every time, with both buttons.** This applies in the email body and in spintax variants. Remove Styling and Keep Styling act on that content only and never change the setting.
3. **Literal merge-tag and spintax tokens in a paste become chips, in every state.**

These hold in every state:

- **The sanitisation floor:** scripts, frames, event handlers and stylesheet blocks are never inserted.
- **`⌘⇧V` always pastes plain.**

---

## 4. User stories

| ID | Story |
|---|---|
| US-1 | As an SDR pasting from Word, I want to know my paste carried styling and choose what happens to it, so my email doesn't carry styling I didn't pick |
| US-2 | As a writer, I want to keep styling in one email without switching off styling removal for the whole sequence |
| US-3 | As a writer in the spintax editor, I want styled content handled exactly as in the email body |
| US-4 | As a sequence owner, I want to set styling once per sequence and see that it covers spintax variants |
| US-5 | As any user, I want merge tags and spintax recognised through a paste: chips stay chips, and literal tokens become chips |
| US-6 | As any user, I want `⌘⇧V` to always paste plain and `⌘Z` never to lose content |

---

## 5. Decisions

**D1 – One rule: Remove Styling Automatically, per sequence.** It is already the default and already where the sequence owner sets deliverability. **Consequence to accept explicitly:** there is no workspace-wide paste rule. Workspace control is limited to the default for new sequences.

**D2 – Styled content lands as is, and the bar asks. Nothing is removed automatically.** The writer sees the styling and chooses. **Consequence to accept explicitly:** a writer who ignores the bar sends styled HTML when Remove Styling is off. When it is on, the editor shows styling that send will strip, until they act.

**D3 – Keep Styling acts on that occurrence only and never writes the setting. The bar asks again every time styling appears.** **Consequence to accept explicitly:** a writer who keeps styling often will see the bar often. §10 measures this.

**D4 – Spintax variants follow the sequence's Remove Styling, with the same bar and both buttons.** **Consequence to accept explicitly:** styling can be kept inside a variant, so spin parsing must survive it (V-5, Q2).

**D5 – Text-only steps paste plain, with no bar.** This covers "Send emails as text only" for all emails, and step 1 when it is set to first email only. Text only removes all formatting on save and at send, so there is nothing to ask. **Consequence to accept explicitly:** a text-only step has no way to keep styling.

---

## 6. Surfaces

The visual record is the [prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/), tabs 1 to 3. Copy below is literal.

### 6.1 Sequence → Settings → Safety Settings

**What it's for.** The single place the rule is set, per sequence.

**What it must do**

- Append one sentence to the Remove Styling Automatically description: *"Styling you paste or add in the editor is flagged there, with an option to remove it."*
- Under Remove Styling, show *"Spintax variants: same as above"* with a tag giving the resolved value, **On** or **Off**. It is a statement, not a control.
- Everything else on the card stays as it is, including Remove Styling showing on but disabled when text only is set to all emails.

| ID | Case | Expected | P |
|---|---|---|---|
| S-1 | Open Safety Settings, Remove Styling on | Description carries the added sentence; spintax line reads "same as above" with tag **On** | P1 |
| S-2 | Turn Remove Styling off | Spintax tag reads **Off** immediately, before Save | P1 |
| S-3 | Text only set to all emails | Remove Styling on and disabled; spintax tag reads **On** | P2 |
| S-4 | Inspect the spintax line | Not focusable or clickable: no checkbox, no toggle | P1 |

### 6.2 The step editor: email body

**What it's for.** This is where styling enters. The writer sees it, is asked, and decides for this email.

| Remove Styling | Styled paste or toolbar styling | Bar | Keep Styling | Remove Styling | × |
|---|---|---|---|---|---|
| **On** | Lands as is | `Styling Detected (Remove to avoid spam filters):` · **Remove Styling** · **Keep Styling** · `×` | Keeps it in this email; setting untouched; bar asks again next time | Removes styling from this email; bold, italic, underline, links and lists stay; setting untouched | Removes styling |
| **Off** | Lands as is | Same bar, same copy, both buttons | Same as On | Removes styling from this email; **effect on the setting is Q1** | Closes the bar; styling stays |
| **Text only** | Lands as plain text | None | – | – | – |

**What it must do**

- Show the bar every time styled content appears: on paste, on toolbar styling, and on opening a step that contains styling. Keep Styling never silences it for the rest of the session.
- Show **both** buttons whenever the bar shows.
- Keep the "Styling Removes Automatically" / "Text only email" badge visible while the bar is up.
- Decide whether a step is text only from the sequence's settings for that step.

**Explicitly out of scope:** the template editor (Q3), and send-time stripping, which stays as it is.

| ID | Case | Expected | P |
|---|---|---|---|
| B-1 | Remove Styling **on**: paste Word content with font, colour and highlight | Content lands with its styling; bar shows with both buttons; nothing removed | P0 |
| B-2 | Remove Styling **on**: apply a text colour from the toolbar | Colour stays; bar shows with both buttons | P0 |
| B-3 | Remove Styling **off**: repeat B-1 and B-2 | Same bar, same copy, both buttons | P0 |
| B-4 | Any state where the bar renders | Remove Styling **and** Keep Styling both visible | P0 |
| B-5 | Click **Keep Styling**, Remove Styling on | Bar closes, styling stays. **No request is sent to `PATCH /sequences/{id}/settings`.** Read back with `GET /sequences/{id}/config`: `text-only-email` stays `1`, and the badge still shows | **P0, sign off by name. Owner: TBD** |
| B-6 | After Keep Styling, paste styled content or apply toolbar styling again, same session | Bar shows again | P0 |
| B-7 | After Keep Styling, save, close and reopen the step | Bar shows for the kept styling | P1 |
| B-8 | Click **Remove Styling**, Remove Styling on | Styling removed from this email; bold, italic, underline, links and lists stay; no settings request | P0 |
| B-9 | Click **×**, Remove Styling on | Styling removed | P1 |
| B-10 | Click **×**, Remove Styling off | Bar closes; styling stays; no settings request | P1 |
| B-11 | Paste plain text, or copy and paste within Saleshandy | No bar | P0 |
| B-12 | Text only (all emails), or step 1 with first email only: paste Word content | Lands as plain text; no bar; "Text only email" badge shown | P0 |
| B-13 | Bar is up | Corner badge stays visible, not covered by the bar | P2 |
| B-14 | `⌘⇧V` in any state | Plain; no bar | P0 |

### 6.3 The spintax editor (Add Spintax)

**What it's for.** Variants are part of the email, so they follow the same rule.

**What it must do**

- Apply §6.2 inside each variant field: same bar, same copy, both buttons and same behaviour, shown in that variant's field.
- Scope each action to its own variant. Keep or Remove in variant 1 does not touch variant 2 or the email body.

| ID | Case | Expected | P |
|---|---|---|---|
| V-1 | Remove Styling on or off: paste styled content, or apply toolbar styling, in variant 1 | Bar shows in variant 1 with both buttons; styling stays until a button is clicked | P0 |
| V-2 | Keep Styling in variant 1 | Styling stays in variant 1 only; variant 2 and the body untouched; no settings request | P0 |
| V-3 | Text only: paste Word content into a variant | Plain; no bar | P0 |
| V-4 | Literal `{{First Name}}` pasted into a variant | Becomes a merge-tag chip | P0 |
| V-5 | Keep styling inside a variant, then Save → reopen the modal → save the step → send a test | The spin still parses and spins. The chosen variant arrives styled when Remove Styling is off, and stripped when it is on. No literal `{spin}` text in any output. **Depends on Q2** | **P0, release blocker** |

---

## 7. Reconciliation with existing behaviour

**Scope map**

| Where styling comes from | What catches it | Covered by |
|---|---|---|
| Paste from Word | Bar, in the editor | §6.2, §6.3 |
| Paste from a web page or Google Docs | Arrives without inline styles in Chrome already (measured, one synthetic paste). Bar shows if any remain | §6.2 |
| Toolbar font, size or colour | Bar | §6.2 |
| Opening a step that has styled content | Bar | §6.2, B-7 |
| AI-generated content, or a template applied to a step | Bar, if styled | §6.2. The template editor itself is Q3 |
| What is sent | Remove Styling strips at send; text only converts on save and at send | Unchanged |

Nothing is removed from Safety Settings. The one line of copy that makes the relationship legible is S-1.

---

## 8. Cross-cutting

**Chip integrity – release blocker**

| ID | Case | Expected | P |
|---|---|---|---|
| C-1 | Copy a body containing all five chip types (error, date, merge, variable, spin) and paste it into the same editor | All five reproduced as chips; none literal, none duplicated | P0 |
| C-2 | Paste over a selection that contains chips | Selected chips removed with the selection; no leftover chip markup | P0 |
| C-3 | Chips elsewhere in the body during a paste | Untouched | P0 |
| C-4 | Save, then reload | Chips are still chips | P0 |
| C-5 | Paste literal `{spin}a\|b{endspin}` | Becomes a spin chip | P0 |
| C-6 | Paste malformed spintax (no `{endspin}`) | Stays literal text; no crash, no partial chip | P0 |
| C-7 | Paste literal merge-tag syntax | Becomes a merge-tag chip | P0 |
| C-8 | Paste a merge tag naming a field that does not exist | Becomes the error chip, the same as typing it | P0 |
| C-9 | Paste literal tokens into the subject line | They become chips there too | P0 |
| C-10 | Paste a token cut off by the paste boundary, or paste with the caret inside an existing token | Stays literal; no partial chip; the surrounding chip is intact | P0 |
| C-11 | Paste styled content containing literal tokens, then click Keep or Remove Styling | Chips survive either action | P0 |

Order of operations: sanitise first, then promote tokens. Promotion runs whether styling is kept or removed.

**General paste**

| ID | Case | Expected | P |
|---|---|---|---|
| G-1 | Clipboard holds only an image | Existing image behaviour; no bar | P1 |
| G-2 | Rich content on the clipboard with no plain-text alternative | Content lands; the insert is never empty | P0 |
| G-3 | Right-to-left or CJK text, or emoji joined into sequences | Correct direction; never split mid-sequence | P1 |
| G-4 | Paste over a multi-paragraph selection, mid-word, or into an empty body | Selection replaced, inserted at the caret, no blank first paragraph | P1 |
| G-5 | Paste rich text into the subject line | One plain line; line breaks become spaces; no bar | P0 |
| G-6 | Drag and drop text into the editor | Unchanged from today | P1 |
| G-7 | Paste while an IME composition is active | No corruption | P1 |
| G-8 | Paragraphs and lists, through editor → save → send | Preserved, not run together into one block | P0 |

**Keyboard**

| Key | Behaviour |
|---|---|
| `⌘⇧V` / `Ctrl+Shift+V` | Always plain, no bar, on every surface |
| `⌘Z` | The first press after a paste removes the paste. After Remove Styling, the first press restores the styling |

**Sanitisation floor.** Scripts, embedded frames, event handlers and stylesheet blocks are never inserted, in any state.

---

## 9. Implementation notes

- **Build on the existing Styling Detected bar** ([`styling-detected-action-bar.tsx`](https://github.com/saleshandy/saleshandy-webui/blob/a6e04a32782a2814da502d455366b160405ea310/src/components/sequence/shared/modals/email-modal/components/styling-detected-action-bar/styling-detected-action-bar.tsx#L27)). Change it so that:
    - Keep Styling stops writing `text-only-email`.
    - Nothing silences the bar after Keep.
    - Remove Styling sends no settings write.
- **Add the same bar to each variant field** in the spintax modal.
- **Reuse the paste sanitiser, token promotion, internal-copy detection and `⌘⇧V` handling** already built in [webui PR #8129](https://github.com/saleshandy/saleshandy-webui/pull/8129).
- **Resolve before estimation:** whether the spin parser and the send path tolerate markup inside `{spin}…{endspin}` (Q2). An open bug says they do not ([to-do](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458)). If that is still unfixed, the fix is part of D4. An estimate without it is wrong, not approximate.

---

## 10. Measurement

**Never log pasted content.** Counts and enum labels only.

| Event | Fires when | Properties |
|---|---|---|
| `paste_performed` | Any paste into an editor surface | `surface` (body/subject/variant) · `had_styling` · `remove_styling_state` (on/off/text_only) · `internal_copy` · `used_plain_shortcut` · `tokens_promoted` |
| `styling_detected_shown` | The bar renders | `surface` · `source` (paste/toolbar/open) · `remove_styling_state` · `shown_index_in_session` |
| `styling_detected_action` | A bar control is used | `action` (remove/keep/close) · `surface` · `remove_styling_state` |

Instrumentation ships with the feature; the gates below cannot be read without it.

| Leading indicator, day 1 to 14 | Healthy | Investigate |
|---|---|---|
| **Settings writes from the editor** | **0 (target)** | **Any – D3 is broken. The single most important early signal** |
| Keep rate (keep ÷ shown), Remove Styling on | No base rate; discovery | High and sustained – writers want styling that the sequence strips at send; review the default |
| Bar shows per session, 90th percentile | ≤ 5 (assumed; confirm at day 14) | Higher – the bar reads as nagging |

| Lagging, day 30 | Target |
|---|---|
| Share of new sequences with Remove Styling on | ≥ 81.9% (target, the measured baseline) |
| "Formatting" support tickets per 1,000 active users | No increase vs. the 4 weeks before release |

| Guardrail | Threshold | Action |
|---|---|---|
| Chip corruption | Any | Roll back |
| Spin fails to parse after styling is kept in a variant | Any | Roll back |
| Editor JS error rate on paste | Rise vs. baseline | Halt |

**Success at day 30, 100% exposure:** zero settings writes from the editor, zero chip or spin corruption, and no rise in tickets. All three are non-negotiable.

---

## 11. Rollout

The behaviour is the same for everyone, so staged exposure is blast-radius control, not an A/B cohort. No stage starts until B-5 is signed off and V-5 passes.

| Stage | Who | Gate |
|---|---|---|
| 1 | Internal | All P0 criteria pass on the build; B-5 signed off by name; V-5 checked with a real test send |
| 2 | 5% of workspaces | One week, no guardrail breached, zero settings writes from the editor |
| 3 | 100% | Stage 2 clean |

---

## 12. Open questions

| # | Question | Why it matters | Recommendation |
|---|---|---|---|
| Q1 | With Remove Styling **off**, should **Remove Styling** in the bar also switch the sequence's setting on? | It would be a sequence-wide change from one click in one email, the thing D3 rules out for Keep | **This email only.** Blocking for build. Hitesh |
| Q2 | Does the spin parser tolerate styling inside a variant through save, reopen and send? | D4 allows styling in variants, and an open bug says it breaks | Rajat confirms before estimation; if it breaks, the fix ships with this. **Blocking for estimation** |
| Q3 | The template editor has no sequence, so no rule | A template can carry styling into many sequences | Leave it as is; the bar applies once the template is used in a step. Revisit if tickets appear |
| Q4 | Text-only paste behaviour has not been checked in the live app | D5 may differ from what happens there now | Verify at technical analysis; build to D5 |

---

## 13. Priority & timeline

| Item | Status | Owner |
|---|---|---|
| Priority | Not set | Hemanshu |
| Target date | Not set until Q2 is answered | Rajat |
| Release gate | Chip integrity (§8), B-5 named sign-off, V-5 | Rajat |
| Q1 | Blocking for build | Hitesh |
| Build | Not started | Sunny |

---

## Appendix – reference index

- **[Prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/)** – design of record. Tab 1 is §6.1, tab 2 is §6.2, tab 3 is §6.3.
- **Evidence:**
    - Live-app check on 7 Oct, sequence 964458: the bar, what Keep and Remove write, and the duplicate request.
    - Metabase query on the Leo db: `sequence` joined to `sequence_setting` on `code = 'text-only-email'`, created on or after 1 Jul 2026.
- **Related:**
    - [Card](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873)
    - [Spintax styling bug](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458)
