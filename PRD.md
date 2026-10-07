# PRD – Pasting styled content: one rule, Remove Styling

**Design of record:** [prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/) · **Status:** Ready for dev review

---

## 0. What changed since v2.2

v2.2 ([still live, unchanged](https://product-team-sh.github.io/prd-paste-as-plain-text/)) was built as [webui PR #8129](https://github.com/saleshandy/saleshandy-webui/pull/8129) and [edge PR #11339](https://github.com/saleshandy/saleshandy-edge/pull/11339). Dev raised five points against it ([Sunny, 5 Oct](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873#__recording_10371718163)). This version resolves all five.

| # | v2.2 said | v3 says | Why |
|---|---|---|---|
| 1 | A workspace setting, Admin Settings → Pasting content, decides what a paste becomes | The sequence's existing **Remove Styling Automatically** is the only rule. No new setting anywhere | Remove Styling is already on for 81.9% of new sequences (measured). Two rules in two places disagreed (Sunny, points 2 and 5) |
| 2 | A rich paste is converted to plain text on the way in | A styled paste lands as pasted, and the existing **Styling Detected** bar asks: Remove Styling or Keep Styling | The editor only holds HTML, so "plain text" was never true (Sunny, point 1). One surface now covers every source of styling |
| 3 | A toast reports each paste; a standing "Rule:" chip and a first-run spotlight state the rule | The Styling Detected bar reports. The existing "Styling Removes Automatically" badge states the rule | One surface for every source of styling, already in production |
| 4 | The toast offers "Always keep formatting" / "Always paste as plain text", which write the workspace setting | Nothing in the editor writes any setting | A writer must not change a rule by accident while editing one email (Sunny, point 4) |
| 5 | Spintax variants have their own rule, inheriting the editor's unless unticked | Variants follow the sequence's Remove Styling, with the same bar and both buttons | One rule per sequence. Safety Settings states it for variants |
| 6 | Behaviour under "Send emails as text only" not specified | Text-only steps paste plain, with no bar | Nothing to choose: text only removes all formatting on save and at send (Sunny, point 3) |
| 7 | Production today: Keep Styling silently turns Remove Styling **off for the whole sequence**, then hides the bar for the rest of the session (verified 7 Oct) | Keep Styling keeps the styling **for that occurrence only**, never writes the setting, and the bar asks again every time | Same principle as change 4, at sequence level |

**Void from v2.2.** These IDs tested a structure that no longer exists: the workspace setting, plain-text conversion, the toast, the rule chip, the enrol, and the variant inheritance model.

| Area | Void IDs |
|---|---|
| Admin Settings card | §5.1 entirely: S1-1 to S1-26 |
| User stories | US-3, US-5, US-6, US-9, US-10 |
| Plain-text conversion | S2-1 to S2-4, S2-12a to S2-12d, S2-37 |
| Toast, flip and enrol | S2-11, S2-16 to S2-28, S2-39 to S2-44, S2-F0, S2-F2, S2-F4, S2-F5, S2-F5a, S2-F5b |
| Rule chip and spotlight | S2-33, S2-33a, S2-34, S2-34a, S2-34b, S2-35 |
| Replaced by §6.2 | S2-38 |
| Spintax | §5.3 entirely: S3-1 to S3-13 |
| Measurement and rollout | Every v2.2 §8 event except `paste_performed`; every §8b to §8e metric tied to the toast, chip, enrol, spotlight or variant inheritance; rollout stage 1b |
| Support | v2.2 §10, For Customer Support |
| Decisions | v2.2 D1 (workspace scope), D3 (variant own rule) and D7 (standing chip) |

**Carried forward unchanged:** US-1, US-7 and US-8. Chip integrity X-1 to X-12; X-8 and X-12 are reworded to "any Remove Styling state", since there is no plain/keep setting any more. The `⌘⇧V` keyboard invariant and the sanitisation floor (S2-8, S2-9). `⌘Z` is redefined in §8, because there is no conversion left to undo. Also S2-5, S2-6, S2-7, S2-13, S2-14a to S2-14c, S2-15, S2-29 to S2-32 and S2-36.

---

## 1. Problem

**Pasting brings styling the writer did not choose.** Writers paste into the step editor from Word, Google Docs and old emails. Word content arrives with its fonts, colours, sizes and highlights, plus hidden markup the writer cannot see. In a sequence with Remove Styling on, that styling is stripped at send, so the editor shows something the prospect never receives. In a sequence with Remove Styling off, it goes out as pasted.

**Tokens in a paste arrive broken.** Spintax and merge-tag syntax in pasted text, from an external doc or AI-written copy, lands as literal braces instead of chips. A customer escalated after literal spintax text reached prospects ([CS thread](https://app.basecamp.com/4378325/buckets/16408283/todos/10213661915)).

**Today's Styling Detected bar undoes the sequence's rule.** Production already shows the bar when styling appears. But one click on Keep Styling silently switches Remove Styling off for every email in the sequence. The bar then stops asking for the rest of the session (verified on my.saleshandy.com, 7 Oct). The writer meant "keep it here" and changed the sequence's deliverability setting.

**v2.2 added a second rule in a second place.** A workspace paste setting beside the sequence's own Remove Styling meant two answers to one question, and the toast could say "Formatting kept" while send stripped it.

---

## 2. Why this approach

| Approach | Rejected on |
|---|---|
| v2.2: workspace paste setting, toast, rule chip | Duplicates Remove Styling, which is on for 81.9% of new sequences (measured). The two can disagree, and they live in two places |
| Clean styling automatically as it lands | The writer does not see or choose what was removed (Hitesh, 7 Oct). It also leaves toolbar styling handled differently from pasted styling |
| **Reuse the Styling Detected bar for every source of styling** | Chosen. Already in production, already understood, and it covers paste, toolbar and existing content with one surface |

**Evidence**

| Figure | Tag |
|---|---|
| New sequences default to Remove Styling on ([edge `sequence-setting.ts#L57`](https://github.com/saleshandy/saleshandy-edge/blob/d908da065793036a3663b4f83b62c7f2a65078a6/src/sequence/enums/sequence-setting.ts#L57)) | measured, code |
| 81.9% Remove Styling, 16.6% off, 1.5% text only, of 64,928 sequences created since 1 Jul 2026 | measured, Metabase, created sequences rather than active ones |
| With Remove Styling on, a Word paste and toolbar colour both land styled and show the bar. Keep Styling saves `text-only-email = 0`. Remove Styling saves `1`, sent twice | measured, live app, 7 Oct |
| A non-Word HTML paste already loses its inline styles | measured, one synthetic paste, 7 Oct |

**We do not claim** any inbox-placement improvement. We claim only that the writer decides what styling stays, and that the sequence's rule is never changed by accident.

---

## 3. What we're building

1. **Remove Styling Automatically is the one rule for styling, per sequence, at paste and at send.** No new setting anywhere.
2. **The existing Styling Detected bar flags styled content from every source, every time, with both buttons**, in the email body and in spintax variants. Remove Styling and Keep Styling act on that content only and never write the setting.
3. **Literal merge-tag and spintax tokens in a paste become chips, in every state.**

Invariant in every state: the sanitisation floor (scripts, frames, event handlers and stylesheet blocks are never inserted), and `⌘⇧V` always pastes plain.

---

## 4. User stories

| ID | Story |
|---|---|
| US-1 | As an SDR pasting from Word, I want to know my paste carried styling and choose what happens to it, so my email doesn't carry styling I didn't pick. *(Reworded: told and asked, not converted silently)* |
| US-7 | As any user, I want merge tags and spintax recognised through a paste: chips stay chips, and literal tokens become chips. *(Carried)* |
| US-8 | As any user, I want `⌘⇧V` to always paste plain and `⌘Z` never to lose content. *(Carried)* |
| **US-11** | **As a writer, I want to keep styling in one email without switching off styling removal for the whole sequence.** *(New. The reason this version exists)* |
| **US-12** | **As a writer in the spintax editor, I want styled content handled exactly as in the email body.** *(New)* |
| **US-13** | **As a sequence owner, I want to set styling once per sequence and see that it covers spintax variants.** *(New)* |

---

## 5. Decisions

**D8 – One rule: Remove Styling Automatically, per sequence.** *Reverses v2.2 D1.* It is already the default and already where the sequence owner sets deliverability. **Consequence to accept explicitly:** there is no workspace-wide paste rule. Workspace control stays limited to the existing default for new sequences.

**D9 – Styled content lands as is, and the bar asks. Nothing is removed automatically.** *Reverses v2.2's plain-by-default conversion.* The writer sees and chooses. **Consequence to accept explicitly:** a writer who ignores the bar sends styled HTML in a sequence with Remove Styling off. With it on, send still strips styling, so the editor can differ from what is sent until they click.

**D10 – Keep Styling acts on that occurrence only, never writes the setting, and the bar asks again every time styling appears.** *Reverses production behaviour since 24 Sep (commit `551500a10b`): silent switch-off plus suppression for the session.* **Consequence to accept explicitly:** a writer who keeps styling often sees the bar often. §10 measures this.

**D11 – Spintax variants follow the sequence's Remove Styling, with the same bar and both buttons.** *Reverses v2.2 D3 and PR #8129's always-plain variants.* **Consequence to accept explicitly:** styling can be kept inside a variant, so spin parsing must survive it (V-5, Q2).

**D12 – Text-only steps paste plain, with no bar.** This covers "Send emails as text only" on all emails, or on step 1 when it is set to first email only. Text only removes all formatting on save and at send, so there is nothing to ask. **Consequence to accept explicitly:** no Keep Styling path exists in text-only steps.

---

## 6. Surfaces

Visual record: the [prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/), tabs 1 to 3. Copy below is literal.

### 6.1 Sequence → Settings → Safety Settings

**What it's for.** The single place the rule is set, per sequence. Unchanged except for two lines.

**What it must do**

- Append one sentence to the Remove Styling Automatically description: *"Styling you paste or add in the editor is flagged there, with an option to remove it."*
- Under Remove Styling, show *"Spintax variants: same as above"* with a tag giving the resolved value, **On** or **Off**. It is a statement, not a control.
- Everything else on the card behaves as today, including Remove Styling showing on but disabled when text only is set to all emails.

| ID | Case | Expected | P |
|---|---|---|---|
| S-1 | Open Safety Settings, Remove Styling on | Description carries the added sentence; spintax line reads "same as above" with tag **On** | P1 |
| S-2 | Turn Remove Styling off | Spintax tag reads **Off** immediately, before Save | P1 |
| S-3 | Text only set to all emails | Remove Styling shows on and disabled (as today); spintax tag reads **On** | P2 |
| S-4 | Inspect the spintax line | It is not focusable or clickable: no checkbox, no toggle | P1 |

### 6.2 The step editor: email body

**What it's for.** Where styling enters. The writer sees it, is asked, and decides for this email.

| Remove Styling | Styled paste or toolbar styling | Bar | Keep Styling | Remove Styling | × |
|---|---|---|---|---|---|
| **On** | Lands as is | `Styling Detected (Remove to avoid spam filters):` · **Remove Styling** · **Keep Styling** · `×` | Keeps it in this email. Setting untouched. Bar asks again next time | Removes styling from this email; bold, italic, underline, links and lists stay. No setting write | Removes styling, as today |
| **Off** | Lands as is | Same bar, same copy, both buttons | Same as On | Removes styling from this email. **Whether it also turns the setting on is Q1** | Dismisses the bar, as today |
| **Text only** | Lands as plain text | None | – | – | – |

**What it must do**

- Show the bar every time styled content appears: on paste, on toolbar styling, and on opening a step that contains styling. There is no suppression for the rest of the session after Keep.
- Show **both** buttons whenever the bar shows.
- Keep the "Styling Removes Automatically" / "Text only email" badge visible while the bar is up. Today the bar hides it.
- Decide whether a step is text only from the sequence's settings for that step. No workspace value is read.
- Remove PR #8129's toast, rule chip, spotlight and Admin Settings card. Re-enable the bar in step editors, since the PR switches it off.

**Explicitly out of scope:** the template editor (Q3), and changes to send-time stripping. Known send-time gaps, such as `<font>` and `<style>` surviving Remove Styling, belong on a separate card.

| ID | Case | Expected | P |
|---|---|---|---|
| B-1 | Remove Styling **on**: paste Word content with font, colour and highlight | Content lands with its styling; bar shows with both buttons; nothing removed | P0 |
| B-2 | Remove Styling **on**: apply a text colour from the toolbar | Colour stays; bar shows with both buttons | P0 |
| B-3 | Remove Styling **off**: repeat B-1 and B-2 | Same bar, same copy, both buttons | P0 |
| B-4 | Any state where the bar renders | Remove Styling **and** Keep Styling both visible | P0 |
| B-5 | Click **Keep Styling**, Remove Styling on | Bar closes, styling stays. **No request is sent to `PATCH /sequences/{id}/settings`.** Read back with `GET /sequences/{id}/config`: `text-only-email` unchanged at `1`; badge still shown | **P0, sign off by name. Owner: TBD** |
| B-6 | After Keep Styling, paste styled content or apply toolbar styling again, same session | Bar shows again | P0 |
| B-7 | After Keep Styling, save, close and reopen the step | Bar shows for the kept styling | P1 |
| B-8 | Click **Remove Styling**, Remove Styling on | Styling removed from this email; bold, italic, underline, links and lists stay; no settings request (today it is sent twice) | P0 |
| B-9 | Click **×**, Remove Styling on | Styling removed (as today) | P1 |
| B-10 | Click **×**, Remove Styling off | Bar closes; styling stays; no settings request | P1 |
| B-11 | Paste plain text, or copy and paste within Saleshandy | No bar | P0 |
| B-12 | Text only (all emails), or step 1 with first email only: paste Word content | Lands as plain text; no bar; "Text only email" badge shown | P0 |
| B-13 | Bar is up | Corner badge remains visible and unobscured | P2 |
| B-14 | `⌘⇧V` in any state | Plain; no bar | P0 |
| B-15 | Admin Settings page | No "Pasting content" card; no workspace paste value stored | P0 |

### 6.3 The spintax editor (Add Spintax)

**What it's for.** Variants are part of the email and follow the same rule.

**What it must do**

- Apply §6.2 inside each variant field: same bar, same copy, both buttons, same semantics, shown in that variant's field.
- Scope each action to its own variant. Keep or Remove in variant 1 does not touch variant 2 or the email body.

| ID | Case | Expected | P |
|---|---|---|---|
| V-1 | Remove Styling on or off: paste styled content, or apply toolbar styling, in variant 1 | Bar shows in variant 1 with both buttons; styling stays until a button is clicked | P0 |
| V-2 | Keep Styling in variant 1 | Styling stays in variant 1 only; variant 2 and the body untouched; no settings request | P0 |
| V-3 | Text only: paste Word content into a variant | Plain; no bar | P0 |
| V-4 | Literal `{{First Name}}` pasted into a variant | Becomes a merge-tag chip (X-7 applies) | P0 |
| V-5 | Keep styling inside a variant, then Save → reopen the modal → save the step → send a test | The spin still parses and spins. The chosen variant arrives styled under Remove Styling off, or stripped under on. No literal `{spin}` text in any output. **Depends on Q2** | **P0, release blocker** |

---

## 7. Reconciliation with existing behaviour

**Scope map**

| Where styling comes from | What catches it | Covered by |
|---|---|---|
| Paste from Word | Bar, in the editor | §6.2, §6.3 |
| Paste from web or Docs | Already arrives without inline styles in Chrome (measured, one synthetic paste); the bar shows if any remain | §6.2 |
| Toolbar font, size or colour | Bar | §6.2 |
| Opening a step with styled content | Bar | §6.2, B-7 |
| AI-generated content or templates applied to a step | Bar, if styled | §6.2; template editor itself is Q3 |
| What is sent | Remove Styling strips at send; text only converts on save and at send | Unchanged |

Nothing is deleted from Safety Settings. The one line of copy that makes the relationship legible is S-1.

---

## 8. Cross-cutting

**Chip integrity – release blocker.** X-1 to X-12 are carried from v2.2 unchanged in substance. X-8 and X-12 now read "in any Remove Styling state" in place of "setting on/off". The order of operations still holds: sanitise first, then promote tokens. Promotion applies whether styling is kept or removed.

**Keyboard invariants**

| Key | Behaviour |
|---|---|
| `⌘⇧V` / `Ctrl+Shift+V` | Always plain, no bar, every surface |
| `⌘Z` | First press after a paste removes the paste. After Remove Styling, the first press restores the styling |

**Sanitisation floor.** Scripts, embedded frames, event handlers and stylesheet blocks are never inserted, in every state (S2-8, S2-9 carried).

---

## 9. Implementation notes

- **From PR #8129, keep:**
  - The paste sanitiser
  - Token promotion and the internal-copy marker
  - `⌘⇧V` handling
  - Undo
  - Analytics plumbing
- **From PR #8129, remove:**
  - Plain-text conversion of the body
  - The toast, rule chip and spotlight
  - The Admin Settings card and the `/user/meta` paste rule
  - The switch-off of the Styling Detected bar in step editors
  - Always-plain variants
- **Close [edge #11339](https://github.com/saleshandy/saleshandy-edge/pull/11339)** unmerged. It cites Jira DE-4596, which our Jira login cannot open.
- **Change the existing bar** ([`styling-detected-action-bar.tsx`](https://github.com/saleshandy/saleshandy-webui/blob/a6e04a32782a2814da502d455366b160405ea310/src/components/sequence/shared/modals/email-modal/components/styling-detected-action-bar/styling-detected-action-bar.tsx#L27)) so that:
  - Keep Styling stops writing `text-only-email`.
  - No suppression persists after Keep.
  - Remove Styling stops the duplicate settings write.
- **Add the bar to the variant field** in the spintax modal. This is new UI, using the same component.
- **Resolve before estimation:** whether the spin parser and send path tolerate markup inside `{spin}…{endspin}` (Q2). A known bug says they do not ([to-do](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458)). If that is unfixed, D11 includes that fix, and an estimate without it is wrong rather than approximate.

---

## 10. Measurement

**Never log pasted content:** counts and enum labels only.

| Event | Fires when | Properties |
|---|---|---|
| `paste_performed` | Any paste into an editor surface | `surface` (body/subject/variant) · `had_styling` · `remove_styling_state` (on/off/text_only) · `internal_copy` · `used_plain_shortcut` · `tokens_promoted` |
| `styling_detected_shown` | The bar renders | `surface` · `source` (paste/toolbar/open) · `remove_styling_state` · `shown_index_in_session` |
| `styling_detected_action` | A bar control is used | `action` (remove/keep/close) · `surface` · `remove_styling_state` |

Instrumentation ships with the feature; the gates below cannot be read without it.

| Leading indicator, day 1 to 14 | Healthy | Investigate |
|---|---|---|
| **Settings writes from the editor** | **0 (target)** | **Any – D10 is broken. The single most important early signal** |
| Keep rate (keep ÷ shown), Remove Styling on | No base rate; discovery | High and sustained – writers want styling the sequence strips at send; review the default |
| Bar shows per session, 90th percentile | ≤ 5 (assumed; confirm at day 14) | Higher – the bar reads as nagging under D10 |

| Lagging, day 30 | Target |
|---|---|
| Share of new sequences with Remove Styling on | ≥ 81.9% (target, the measured baseline): Keep no longer switches it off, so it should not fall |
| "Formatting" support tickets per 1,000 active users | No increase vs. the 4 weeks before release |

| Guardrail | Threshold | Action |
|---|---|---|
| Chip corruption | Any | Roll back |
| Spin fails to parse after styling kept in a variant | Any | Roll back |
| Editor JS error rate on paste | Rise vs. baseline | Halt |

**Success at day 30, 100% exposure:** zero settings writes from the editor; zero chip or spin corruption; no ticket rise. All three are non-negotiable.

---

## 11. Rollout

The change is per sequence and shows the same UI to everyone, so staged exposure is blast-radius control, not an A/B cohort. No stage starts until B-5 is signed off and V-5 passes.

| Stage | Who | Gate |
|---|---|---|
| 1 | Internal | All P0 criteria pass on the build; B-5 signed off by name; V-5 checked with a real test send |
| 2 | 5% of workspaces | One week, no guardrail breached, settings writes from the editor = 0 |
| 3 | 100% | Stage 2 clean |

---

## 12. Open questions

| # | Question | Why it matters | Recommendation |
|---|---|---|---|
| Q1 | With Remove Styling **off**, should **Remove Styling** in the bar also switch the sequence's setting on? (It does today) | It is the same class of silent sequence-wide change D10 removes for Keep | **This email only**, consistent with D10. **Blocking for build.** Hitesh |
| Q2 | Does the spin parser tolerate styling inside a variant through save, reopen and send? | D11 allows it. A known bug says it breaks | Rajat confirms before estimation; if it breaks, the fix ships with this. **Blocking for estimation** |
| Q3 | Template editor: no sequence, so no rule | A template can carry styling into many sequences | Leave as today; the bar applies once the template is used in a step. Revisit if tickets appear |
| Q4 | Text-only paste behaviour was not checked in production | D12 may differ from what ships today | Verify at TA; build to D12 |

---

## 13. Priority & timeline

| Item | Status | Owner |
|---|---|---|
| Priority | Not set | Hemanshu |
| Target date | Not set until Q2 is answered | Rajat |
| Release gate | §8 chip integrity, B-5 named sign-off, V-5 | Rajat |
| Q1 | Blocking for build | Hitesh |
| PR #8129 rework, edge #11339 closure | Dev | Sunny |

---

## Appendix – reference index

**Design**
- [Prototype](https://product-team-sh.github.io/prd-pasting-styled-content/prototype/) – design of record: tab 1 is §6.1, tab 2 is §6.2, tab 3 is §6.3.
- Saleshandy Design System v3 tokens, as used across the canvas.

**Evidence**
- Production behaviour checked live on 7 Oct on sequence 964458: bar, Keep and Remove writes, and the duplicate request.
- Metabase query, Leo db: `sequence` joined to `sequence_setting` on `code = 'text-only-email'`, created ≥ 1 Jul 2026, run 6 Oct.

**Related**
- [Card](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873)
- [Sunny's five points](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10216633873#__recording_10371718163)
- [Spintax styling bug](https://app.basecamp.com/4378325/buckets/22269754/todos/10258397458)

**Superseded**
- [PRD v2.2](https://product-team-sh.github.io/prd-paste-as-plain-text/), with [its prototype](https://product-team-sh.github.io/prd-paste-as-plain-text/prototype/).
