# Trinity Guardian — 4-Minute Pitch Script

> For live pitch with the deck: https://ethglobaltokyo2026.github.io/trinity-guardian-presentation/
> ~580 words at 145 wpm = 4:00. Cue format: **[SLIDE]** → say the block. Iron rule lands 3×.
> Tone: calm, confident. Let the demo speak. No hype words.

---

## 0:00 — OPEN

**[SLIDE 1 — title, pause 2s on the stamp]**

> AI agents can now buy things on their own — one line of code, their own wallet, no one watching.
>
> We built Trinity Guardian so that money can still be *yours*.
>
> Three gates sit between the agent and every payment. One rule holds everything together:

**[beat]** **If any layer says no — the money does not move.**

---

## 0:20 — THE SHIFT

**[SLIDE 2 — chat wireframe]**

> This is already normal. You brief your agent in one message — "get me premium flight data." It finds sellers, picks one, pays, subscribes. Agent pays agent, no human in the loop.
>
> That's the agent economy. It's fast — and it's *unguarded*.

---

## 0:40 — THE PROBLEM

**[SLIDE 3 — problem]**

> The agent holds its own keys and its own USDC. A prompt-injected or buggy agent pays *anyone who asks* — scammers included.
>
> And if your safety rules live in a config file — they live inside the very machine an attacker already owns.
>
> Whatever stops a payment must live **outside** the runtime, and outlive it.

---

## 1:00 — THE CONCEPT

**[SLIDE 4 — three gates]**

> Three gates, in order. **ENS** — the right to pay lives on-chain. **Intercepta** — the counterparty gets screened, live. **World ID** — when risk is real, a human decides.
>
> None of them can overrule the others. That's not a limitation — that's the security model.

**[SLIDE 5 — flow animation]**

> Every payment runs the gates *before a signature exists*. Quote → gate → checkpoint → human → outcome.

---

## 1:20 — THE THREE LAYERS

**[SLIDE 6 — ENS]**

> Layer one, ENS. Each agent has an on-chain name under `agents.trinityguard.eth` — with a spend role, spending limits, and an expiry date. Our agent `momo` is guarded; agent `rogue` had its role revoked. We read this live, on Sepolia, before *every* signature — never cached.
>
> Revoke the role, and the agent physically cannot sign. That's a kill switch no middleware can argue with.

**[SLIDE 7 — Intercepta]**

> Layer two, Intercepta. Every payTo, token, and auth message gets screened on mainnet — not a mock list. Green and under cap → auto-pay. Red → hard stop.

**[SLIDE 8 — World ID]**

> Layer three, World ID. Soft fails — over-cap, unknown counterparty — pause and wait for the owner's phone. Approve once, or deny with a reason. A meaningful action, not a login screen.

---

## 2:05 — THE DEMO

**[SLIDE 9 — five acts]**

> Five acts, one dashboard. Same button — different world state.

**[SLIDE 10 — Act 1]**

> Act one: clean purchase. Role active, verdict green — the agent pays alone. **Fully autonomous.**

**[SLIDE 11 — Act 2]**

> Act two: the seller's address carries a scam flag. Intercepta turns red — hard stop. No override, no human needed. **The Guardian refused it itself.**

**[SLIDE 12 — Act 3]**

> Act three: eight dollars against a five-dollar mandate. Payment pauses, the owner's phone lights up, they approve — *this payment only*. Tomorrow the cap is five again. Nobody edited anything.

**[SLIDE 13 — Act 4]**

> Act four: a lookalike USDC. Wrong asset — refused before a signature exists.

**[SLIDE 14 — Act 5 — slow down here]**

> Act five, the headline. Everything is green — but this agent is `rogue`, and its role was revoked on-chain. Watch: the gate never opens. Intercepta never runs. World ID never runs.
>
> **Even if every check were green — the identity layer has closed the gate.**

---

## 3:20 — CLOSE

**[SLIDE 15 — stack]**

> Boring where it can be — TypeScript, viem, x402. On-chain where it must be — ENSv2 contracts on Sepolia. Live screening, real sandbox approvals.

**[SLIDE 16 — closing stamp]**

> ENS keeps the right to pay. Intercepta keeps the checkpoint. World ID keeps the human. None of them trusts the others — that's what makes it work.
>
> Trinity Guardian — three gates, one rule: **if any layer says no, the money does not move.**
>
> The dashboard is live — send our agent shopping.

---

## DELIVERY NOTES

- **Land the iron rule the same way all 3×** (0:12, 3:48) — same pace, same pause after.
- **Act 5 is the money shot** — slow down, point at L2/L3 "never ran".
- If running long: cut Act 4 to one sentence ("a lookalike token — refused, no signature") and trim Layer 2–3 to a line each. Never cut Act 5.
- **3-minute cut:** drop THE SHIFT (slide 2 VO, keep it on screen), single-line problem, merge Acts 3+4.
- Numbers that must be exact: `agents.trinityguard.eth`, `momo` / `rogue`, 5 USDC per-tx, 50 USDC per-day, $8 quote, Sepolia.
- Q&A ammo: soft vs hard fail (red verdict = hard stop; over-cap/no-verdict = ask human) · spend accumulator is off-chain by design (gas) · mandate expiry = self-revoking authority.
