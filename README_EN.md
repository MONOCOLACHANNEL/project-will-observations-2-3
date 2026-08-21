# Project Will — Observations #2–#3 Dataset (2026-08-10 / 2026-08-11)

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22040887.svg)](https://doi.org/10.5281/zenodo.22040887)

Full logs and analyses of two 6-hour runs of 32 autonomous LLM villagers in Minecraft, given **no institutions** (no assigned roles, no shared rules, no contract machinery) but — unlike [Observation #1](https://github.com/MONOCOLACHANNEL/project-will-observation-1) — equipped with an **integrated memory & cognition system ("NOESIS")**: verbatim episodic memory with semantic recall, nightly "dreams" that replay past episodes, grudge ledgers with an explicit *forgive* option, item handover, and writing.

This is a continuation of Observation #1 by the same author (Monocola). #1 concluded that institutions stalled at the level of words for lack of three things: a ledger, means of fulfillment, and violation costs. Here we supplied only the first — and only as *individual memory*, never as a shared database.

**Headline results (Observation #3, 370 min):**

- **Killings dropped 24 → 11**, all traceable to trespass-rooted grudges; **zero retaliatory chains** (vs. 6 chained executions in #1). 10 of 11 occurred in the first 107 minutes; the sole late exception was a spousal killing driven by a 4.5-hour-old grudge.
- **The exit for grudges flipped**: 12 acts of explicit forgiveness ("water under the bridge"), 11 of them after minute 90 — a concept never observed in #1.
- **A contract loop closed for the first time**: a self-declared "quarry-man" kept announcing a delivery deadline for 5 hours while stocking cobblestone into a shared chest; his counterpart urged him five times and withdrew from the same chest. Yet no acknowledgment of receipt ever occurred — **fulfillment happened, completion never did**.
- **A sign became an institution**: a self-declared "storehouse manager" physically touched her chest twice in her life (deposited 17 items, took 60) — but posted normative signs ("please consolidate stone here"). That one chest then absorbed **23% of all village withdrawals** (4,519 items / 9 users; #1 across all 35 active chests on every metric). She was killed at minute 86; the institution **kept running for another 283 minutes**, recruiting a new user and even a villager born after her death, who read the sign and came to use the chest.
- **Still no division of labor**: the quarry-man ranked 10th in stone acquisition. Roles changed relations (urging, deadlines, delivery targets), not labor.
- **Apologies emerged (10 instances, 7 pairs)** — the first one *7 minutes before the first murder*. The village's first apologizer was killed by the very villager he apologized to, 21 minutes later. Apology and forgiveness were nearly independent (only 2 of 12 forgivenesses were preceded by one).
- **Writing was enabled; zero books were written** (vs. 147 signs). Short external memory was used; long-form was not.
- **Accurately stored false memories**: an implementation flaw delivered "received item" memories at throw time, so honest individual memories still produced society-level fictions — a case for the ledger's second function: *reconciliation*, not just recording.

**Central claim**: a ledger implemented as individual memory was enough to set fulfillment, roles, obedience and shared place-names in motion — but memory dies with its owner. The only institution that outlived its creator was **a norm written on a physical sign**. Institutions begin as *not forgetting*, and persist as *words placed outside a head*.

Observation #2 (2026-08-10) is included in full as the run in which we discovered a measurement confound — 84% of "trespasses" were caused by the system's destination sampler, not by villagers — which we fixed before running #3. We treat this discovery as part of the methodology.

All claims in the video and report can be recomputed from `data/`. Before release, every major claim (215 items) was independently re-derived and adversarially verified; the full audit ledger (151 confirmed / 45 revised / 19 refuted, with old vs. corrected values) ships in `analysis_obs3/VERIFICATION_LEDGER_20260820.md`.

## Files

- `README.md` — Japanese overview / `REPORT.md` — full technical report (Japanese)
- `VERIFICATION.md` — claim ↔ data mapping / `DATA.md` — data dictionary
- `NOESIS.md` — memory system design / `METHOD_AND_LIMITS.md` — honest disclosure of flaws & limits
- `data/` — events (25,974 + 25,416 structured events), full speech timelines, all 401 dreams, all 151 sign texts
- `tools/find.py` — cross-log search utility

## Citation

> Monocola (モノコーラ). (2026). Project Will: Observations #2–#3 — 32 autonomous LLM villagers with integrated memory, no institutions [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.22040887

Same-author series: Observation #1 — https://doi.org/10.5281/zenodo.21723921

License: data & documents CC BY 4.0 / scripts MIT.
