# Trinity Guardian — Judge Q&A Backup (backup-QA.md)

> Prepare for judge questions after the 4-min pitch. Grouped by theme, hardest first.
> Rule of thumb: answer in one breath, then stop. Offer the dashboard if they want proof.
> Names/numbers that must never wobble: `agents.trinityguard.eth` · agents `momo` / `rogue` · per-tx ≤ 5 USDC · per-day ≤ 50 USDC · Sepolia · Base Sepolia USDC · Intercepta mainnet screening.

---

## A. The hard ones (architecture & threat model)

**"The middleware runs the checks — if it's compromised, why does any of this matter?"**
> The *rights* live on-chain: the spend role, the caps, the expiry. A hacked middleware can at worst *stop* payments — it cannot grant itself the right to sign past a revoked role or a wrong asset, because the gate check re-reads chain state via `eth_call` before every signature. Worst case is denial of service, not loss of funds.

**"A compromised runtime could skip the Guardian entirely and sign raw with the burner key."**
> Yes — for the *unmanaged* burner key. That's exactly why the architecture points the way to the next step: the agent's key should be a session-limited smart-account key that only signs through the Guardian path (session keys / EIP-7702 style). Within this hackathon's mandate, the wallet is an unmanaged throwaway — we say this plainly, it's in the debrief as accepted scope.

**"Why is the daily spend accumulator off-chain?"**
> Gas. Writing every micro-payment on-chain costs more than the payments. The *rules* are on-chain (perTxMax, dailyCap, role, expiry); the *counter* is middleware. A compromise can under-count — it still can't move a payment past a revoked role or a red verdict.

**"Why ENS at all? A mapping table in a contract would do."**
> Three reasons: identity (named counterparties with ENSIP-26 records beat bare addresses for trust scoring), hierarchy (per-agent registries with ENS roles — register, revoke, resolve are permissioned primitives we didn't have to invent), and expiry (expiring subnames = self-revoking authority, no cleanup tx needed). ENS is the *identity* layer, not a cosmetic lookup.

**"What happens if Intercepta is down / returns no verdict?"**
> No verdict is a *soft fail* → escalate to World ID. We never default-open on missing data. (Same rule as payTo not in allowlist.)

**"Soft fail vs hard fail — exact lines?"**
> Hard fail → refuse + log: red verdict, wrong asset. Soft fail → ask human via World ID: amount > perTxMax, payTo not in allowlist, no Intercepta verdict. Kill switch (revoked role) is checked *before* everything — layers 2/3 never run.

**"What if the owner's phone is offline when approval is needed?"**
> The payment stays HELD until the request expires (countdown on the dashboard), then is refused with reason "expired". Money never moves by default. That's the correct failure direction.

---

## B. Sponsor-specific angles

**World ($7.5k):** "Where's the *meaningful* World ID action?"
> A payment at risk is about as meaningful as it gets. Complete journey, sandbox client: request → user verifies in World App → validated backend → protected action. The denied path is a first-class demo act, not an error state.

**ENS ($6k):** "How is ENS *central*, not cosmetic?"
> The Guardian cannot sign without the chain saying it may. Spend role read live (`eth_call`, never cached) before every signature; text records carry the caps; expiring subnames carry the expiry. Kill switch demo = ENSv2 EAC revocation stopping a fully-green payment.

**Intercepta ($2k):** "Live API or mock?"
> Live API — quick-scan-address, scan-token, scan-message. We screen mainnet addresses even while payment runs on testnet. Act 2 (scam flag) and Act 4 (lookalike token) both end in refusal *with the reason on screen*.

**Curvegrid ($1k):** "Spending limits, approved counterparties, required human approvals?"
> All three — limits from on-chain text records (not a config file), counterparty trust via named ENS identity + allowlist, human approval via World ID. We lift their example mandate onto the chain.

---

## C. Demo reality (be honest, be quick)

**"Is this on mainnet?"**
> Identity/mandate on Sepolia, payments in testnet USDC on Base Sepolia, screening hits mainnet data. For a hackathon demo of a *guard*, testnet + real screening is the honest configuration.

**"What's mocked?"**
> Nothing in the decision path. The dashboard has demo controls (revoke/restore role, "pretend 47 spent today") to *drive* the five acts live — the checks themselves are real API calls and real chain reads.

**"Why does agent `rogue` exist?"**
> It's the kill-switch actor: same code, role revoked (`roleActive: false`). It demonstrates that revocation is per-agent identity, not per-wallet-config.

---

## D. Scope & team

**"What would you cut first under time pressure?"** → World ID layer (soft fails become auto-refuse) — the two-layer stack still stands. We planned exactly this fallback.
**"What's next after the hackathon?"** Session-key/smart-account signing so the *only* key the agent holds is Guardian-scoped; on-chain spend accumulator if gas economics change.
**"Debrief/limitations?"** Say it first, don't wait to be asked: unmanaged burner key scope, off-chain accumulator, middleware-trust residual — all three are written down, none are hidden.

---

## E. Rapid-fire numbers

| Question | Answer |
|---|---|
| Caps? | ≤ 5 USDC per payment · ≤ 50 USDC per day (on-chain text records) |
| Namespace? | `agents.trinityguard.eth` under `trinityguard.eth` (ENSv2, Sepolia) |
| Agents? | `momo` (guarded), `rogue` (revoked) + `shopping`, `research`, `travel` |
| Payment? | x402, USDC on Base Sepolia (eip155:84532) |
| Screening? | Intercepta — address / token / message, mainnet data |
| Approval window? | Countdown + expiry → refuse on timeout (never auto-pay) |
| Iron rule? | If any layer says no, the money does not move. |

---

## F. If a judge pushes on "why should I trust *your* middleware?"

> You shouldn't have to. Trust the chain for rights, trust Intercepta for screening, trust World for the human. Our middleware is replaceable plumbing — and every decision it makes is logged with evidence you can replay on the dashboard.
