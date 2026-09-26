# Trinity Guardian — Design Spec (関所 redesign)

Visual + layout redesign of `packages/ui`. No scope change: every existing feature, field and state is kept.
Mockups: `Trinity Guardian Redesign.dc.html` (artboard ids 1a–1t referenced below).

## 1. Principles
- **5-second read:** a payment flows through three gates (ENS → Intercepta → World ID) and lands on PAID / REFUSED / HELD. The Pipeline band makes this literal.
- **One primary action:** while an approval is pending, the ApprovalCard sits full-width directly under the Pipeline.
- **Ma (間):** generous whitespace; few borders; quiet grid.
- **Never colour alone:** every status = glyph + word (✓ pass · ! soft/hold · ✕ fail · ‖ waiting · – skipped/idle · ● live).
- **Projector first:** body ≥ 15px, key headings 22–30px, stamps up to 30–44px.
- **Japanese, not kitsch:** hanko seals, torii line glyphs, washi/sumi neutrals, kanji as quiet aria-hidden accents only.

## 2. Tokens (`app/globals.css`)
Same names as today → existing Tailwind classes (`text-deny`, `bg-hold-soft`…) keep working. Light in `:root`, dark in `@media (prefers-color-scheme: dark)`.

| Token | Light | Dark | Role |
|---|---|---|---|
| `--paper` | `#F2EEE6` | `#141619` | page ground (washi) |
| `--sheet` | `#FBF9F4` | `#1C1F23` | cards, panels |
| `--ink` | `#1F2327` | `#ECE7DD` | primary text (sumi) |
| `--ink-2` | `#484E54` | `#B6B0A5` | secondary text |
| `--ink-3` | `#686D72` | `#8F8A81` | meta, labels |
| `--line` | `#DDD7CB` | `#34383D` | borders, meter track |
| `--dot` | `rgba(31,35,39,.06)` | `rgba(255,255,255,.045)` | 24px dot texture |
| `--guard` | `#23457A` | `#93AEDB` | ai 藍 — trust, coin, focus, links |
| `--guard-soft` | `#E3E8F0` | `#1F2939` | busy / hover |
| `--allow` | `#2C6647` | `#82C49D` | PAID, pass |
| `--allow-soft` | `#E1ECE3` | `#172A1F` | pass chip ground |
| `--hold` | `#835400` | `#E2B35F` | kincha — HOLD, soft fail |
| `--hold-soft` | `#F4E6C8` | `#33280F` | approval header band |
| `--deny` | `#B2361B` | `#F28C70` | shu 朱 — REFUSED, TG seal |
| `--deny-soft` | `#F6E0D8` | `#3B1D15` | fail chip ground |

Contrast (verified): all text roles ≥ 4.5:1 on `--sheet` and `--paper` in both themes (lowest `--ink-3`: 4.5 light / 4.8 dark). Accent-on-soft chips 4.8–7.7:1.

### Type
Fonts via `next/font/google`: **Zen Kaku Gothic New** 400/500/700/900 → `--font-zen-kaku` (replaces Plex Sans); **IBM Plex Mono** 400/500/600 → `--font-plex-mono`.
`@theme inline { --font-sans: var(--font-zen-kaku); --font-mono: var(--font-plex-mono); }`

| Style | Size / weight | Use |
|---|---|---|
| display | 30 / 900 | ApprovalCard headline (22 on mobile) |
| flow title | 28 / 900 | Pipeline band title (21 mobile) |
| h2 | 22 / 700 | panel titles |
| gate | 19 / 700 | gate names; ticket title (latest 22/900) |
| body-lg | 17 / 400 | reasons, scenario titles (700) |
| body | 15 / 400 | default |
| small | 14 / 400–700 | details, chips |
| label | 12 / 700, +.14em, uppercase | eyebrows ("Layer 2 · 二") |
| mono | 13–15 / 500 | addresses, hashes, amounts |
| code | 30 / 600, +.12em mono | World ID user code |

### Spacing · radii · focus
- Spacing: 4 · 8 · 12 · 16 · 24 · 32 · 40 · 56. Page padding 36/40 desktop, 16/14 mobile; column gap 28.
- Radii: 6 chip-square · 10 control/stamp-md · 14 card · 18 pipeline band · 999 pill.
- Focus: `button, a, summary, input, [tabindex]:focus-visible { outline: 3px solid var(--guard); outline-offset: 2px }`.
- Hit targets: 44px min for primary actions (40px demo controls).
- Motion: stamp-in 260ms and hold-gate pulse (1.6s); global `prefers-reduced-motion` reset kept.

## 3. Components
### Stamp (hanko) — replaces `.stamp*`
Props `verdict: paid|refused|hold|failed|screening`, `size: sm|md|lg|xl`, `kanji = true`.
- Border solid currentColor (2/3/4/5px) + inner rule via `box-shadow: inset 0 0 0 Npx var(--sheet), inset 0 0 0 Mpx currentColor`; `rotate(-4deg)`; weight 900, tracking .07em.
- Kanji under a 1px rule, `aria-hidden`: PAID 済 · REFUSED 否 · HOLD 保留 · NOT SETTLED 未済 · SCREENING 審査中.
- Colour: allow / deny / hold / ink-2 / ink-3. SCREENING = dashed, unrotated, no inner rule.
- `role="img" aria-label="{English}"`.

### StatusChip / Pill
- Chip: pill, 4×11 padding, 14/700, soft bg + accent text, leading glyph. Tones pass ✓ / soft ! / fail ✕ / skip – (`--line` bg, `--ink-2`).
- Pill (header): same shape, 5×12, glyph ● live / ✓ ok / ! warn / ✕ off.

### GateGlyph (new)
Torii line SVG (viewBox 48×44, stroke 2.6, round caps): kasagi curve, nuki bar, two posts. Revoked variant adds a horizontal bar across the posts. Colour = currentColor.

### Pipeline (new, `components/dashboard/Pipeline.tsx`) — 1a, 1m
Nodes: Quote (x402) → Layer 1 ENS gate → Layer 2 Intercepta → Layer 3 World ID → Outcome (Stamp md).
- Row layout ≥ lg (node 172w, 96px tile, radius 18); column layout below (56px tile, 18px vertical connectors).
- Gate status from the latest Run: `ens` → L1; `policy`+`screen` (worst level wins) → L2; `approval` → L3. Not-reached gates are dashed `--ink-3` "never ran"; L3 on auto-pay = "not needed".
- Connectors: solid accent up to the stopping point, dashed after. Coin (indigo pill with amount) sits on the stopping node; hold adds a pulse ring.
- Caption: "If any layer says no, the money does not move."

## 4. Screens
### Layout (`Dashboard.tsx`)
Desktop: Header → Pipeline band → ApprovalCard (only when pending, full width) → `grid lg:grid-cols-[360px_minmax(0,1fr)_380px]`: ScenarioPanel · CheckpointLog · MandatePanel.
Mobile (390): Header (compact pills) → Pipeline (column) → ApprovalCard → ScenarioPanel → CheckpointLog → MandatePanel.
Background: `--paper` + dot texture.

### Header — 1n
TG hanko seal in `--deny`; 24px title + subtitle; pills (feed, ENS, Intercepta, World ID); "owner unlocked" = guard pill button. Token form: full-width row, 44px mono input + solid Unlock. Rejected: form tinted `--deny-soft`, "✕ That owner token was rejected."

### ScenarioPanel — 1o
Agent address copy button + "(unfunded throwaway key)" in hold. Cards: 32px mono number tile, 17/700 title, expect chip (✓ pays / ✕ blocked / ! asks owner), description. Busy: 2px guard border, guard-soft bg, "running the agent now…"; others 50% + not-allowed. Scenario 5: "↺ Restore spend role" link below its button (not nested).

### ApprovalCard — 1g–1j
3px `--hold` border, radius 16. Header band `--hold-soft`: Stamp lg HOLD, eyebrow "Layer 3 · 三 — World ID approver", headline "The agent wants to pay $X — waiting for the owner", "to {payTo} for {resource}".
Body: "Held because" list (! bullets) + failed-check chips; fine print; ✕ Cancel request.
QR panel (`--paper`, 300w): QR 196px on white, caption, user code (code style), approval link, poll status, countdown bar + label.
- **Expiring** (< 30s): bar + label `--deny`, weight 900, "⚠ expires in 0:18".
- **Dev bypass**: dashed hold box, ✓ Approve (dev) / ✕ Deny (dev).
- **Redacted**: lock glyph + "Enter the owner token to show the World ID QR." (no QR/code).

### CheckpointLog — 1a, 1b, 1p
Latest ticket: Stamp lg, 22/900 title, summary, meta (time · steps · owner status · tx link), gate trail chips (ENS / Intercepta / World ID). Phases on a timeline rail: 26px status disc (glyph by level) + `<details>`; bodies = event rows, policy chips, screening rows (chip + detail + "› show evidence" JSON), decision line.
Older tickets: Stamp sm in a fixed column, title, 2-line summary, gate trail, time, caret.
Empty: dashed card, torii glyph, "Nothing screened yet. Pick a purchase on the left to send the agent shopping."

### MandatePanel — 1k, 1l
On-chain box (2px allow or deny border, soft fill): GateGlyph + `momo.payguard.eth` + role chip; revoked adds "Gate closed. Layer 1 refuses every payment until the spend role is restored." dl: Registry, Resolver (Etherscan links), subname expiry, agent-endpoint[web], read latency.
Policy dl (agent, asset/network, per payment, per day, allowlist ENS chips, ask-a-human rules). Spend meter 12px: allow < 60% ≤ hold < 90% ≤ deny, labelled 60/90 ticks. Mandate hash copy. Owner controls (Revoke / Restore — solid allow when gate is off / Renew 7 days, busy labels). Demo controls.

### Cross-cutting — 1p
Skeleton blocks on `--line`; full-page error (2px deny, ✕ title, message, ↻ Retry); disconnected banner (hold, `role=status`); inline error banner (deny, `role=alert`, Retry).

## 5. Implementation order
1. Tokens + fonts (`globals.css`, `layout.tsx`), global focus ring.
2. `primitives.tsx`: Stamp, StatusChip/Pill glyphs, GateGlyph.
3. `Pipeline.tsx` (new).
4. `Dashboard.tsx` layout + mobile order.
5. `ApprovalCard.tsx`.
6. `RunLog.tsx` (timeline rail, gate trail).
7. `ScenarioPanel.tsx`.
8. `MandatePanel.tsx`.
9. `Header.tsx`.

Each step ships independently; steps 1–2 alone restyle the whole app.

## 6. Additions to confirm (easy to drop)
Gate-trail chips on tickets · "Gate closed" sentence · 60/90% meter labels · countdown bar.
