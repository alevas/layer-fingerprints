# Paper points — the ICLR 27 Drafting story

Scope anchor: `notion/Alex Fingerprinting/ICLR 27 Drafting *.md` (the ~31-line source doc).
This file curates only what's actually in scope for *this* paper — not everything the
broader project has produced. Evidence pointers go to `STATUS.md` (claim) and
`PROJECT_PROGRESS.md` (full methodology/numbers) unless noted otherwise.

**The setup, plainly**: an LLM solves tasks, we take its CoT trace, and examine the
token representations produced along the way — what they encode, how they change layer
to layer, whether that change is itself informative, and whether any of it is causally
load-bearing rather than just descriptive.

---

## 1. Token information evolution & coexistence

Representations encode information; clustering them (label-free, no hand-assigned
categories) groups tokens in similar states. Two structural-token classes — `meta`
(discourse-marker-ish: `user`/`wants`/`asked`) and `noise` (`\n`, `<|im_end|>`, etc.) —
cluster reliably across a 5-fold k-sweep, a random-projection seed change, and multiple
datasets; `noise` reaches near-perfect precision at essentially every k tested.

→ `STATUS.md` item 1 (Fingerprint object validity).

## 2. Token groupings / clusters formed

Two independent, unsupervised routes both find the same kind of structure:

- **Per-(token, layer-transition) delta clusters are 89–99% single-transition-specific**
  — tokens mostly occupy one characteristic depth-local niche rather than carrying a
  stable identity across layers. Holds from k=25 to k=120 (a 5× range), and replicates
  *stronger* on a second model 13× larger (Qwen3-8B: 95–99%, above the entire
  Qwen3-0.6B band).
- **ICA recovers a dedicated `<think>`/`</think>` independent component in every
  dataset tested** — a clean, unsupervised separation of one transformation type,
  achieved without ever looking at labels.

→ `STATUS.md` item 4 (89%-layer-locked); `PROJECT_PROGRESS.md`'s ICA entries.

## 3. The "layer fingerprint" — repeating behavioral patterns across layers

The fingerprint is a compressed, label-free summary of a token's per-layer deltas
(random-projected and concatenated across every transition) — built specifically to
answer the doc's own stated compression problem ("how do we compress information from
each layer — concatenating each rep or delta would be too high-dim").

Two lines of evidence that layers *do* exhibit repeating, class-specific behavior:

- **"Wanders and returns"**: `<think>`/`</think>` tokens show a reproducible geometric
  signature — elevated tortuosity (2.62–2.70 vs. a population baseline of 1.84–2.02) and
  a multi-lag directional persistence that never fully decays — in *every* dataset,
  no exceptions. This is itself a transformation-type finding, reached via hand-built
  summary statistics rather than an explicit clustering algorithm — independent
  confirmation of the same conclusion delta-clustering (§2) is supposed to produce.
- **Tortuosity against a random-walk baseline** is the motivating check underneath all
  of this, not a standalone result: population paths are far straighter than an
  isotropic random walk (1.84–2.02 vs. √27≈5.2), meaning there is real, non-random
  structure in the deltas worth clustering at all. Setup, not a finding in its own right.

→ `STATUS.md` item 7 (Open gap 8); the "wanders and returns" / tortuosity entries in
`PROJECT_PROGRESS.md`.

## 4. Do deltas convey information, cluster well, hint at important updates?

Yes to the first two (§§1–3 above). The third — do deltas hint at which updates matter
computationally — is where the paper's actual causal/LRP work (§§5–6) answers directly,
rather than the geometric descriptors alone.

## 5. LRP in this context — the paper's actual headline result

The doc's central, previously-untested question: **does filtering by LRP relevance make
delta-based clustering less noisy than the naive, unfiltered version?** And specifically:
*"if we filter e.g. for the most relevant attention heads before the update of the
residual stream, do we see any different behaviors?"*

Answered directly, not inferred: for each token, at each layer, each attention head's
`hook_z` output is projected through that head's own `W_O` slice and summed — the exact,
literal decomposition of the attention block's contribution to the residual-stream
update. **Unfiltered** = sum over all heads (the model's real update). **Filtered** =
sum over only the top-K most LRP-relevant heads. Cluster both, compare `meta`/`noise`
best-cluster recovery.

**Result, across 9 dataset/model combinations (6 task families, 2 model scales):**

| | improves under filtering | flat | degrades under filtering |
|---|---|---|---|
| `meta` | 6/9 | 2/9 | 1/9 (negligible) |
| `noise` | 1/9 | 5/9 | 3/9 (substantial: -0.25 to -0.42) |

Real, but **asymmetric, not a clean win** — reported honestly as such. `noise`
specifically degrades on `countries_capitals`/`ioi`/`ioi50`, all single-fact or
short-form, non-templated task families; it's flat everywhere in the ordinal/
sequence-continuation family (`alphabet`/`days_of_week`/`numbers_letters`). Working
explanation, not yet confirmed: `noise` tokens may derive distinctiveness from diffuse
activity spread across *many* heads (a structural signal), which concentrating on the
most task-relevant few heads washes out; `meta` tokens may already be concentrated in a
handful of relevant heads, so filtering sharpens rather than dilutes them.

Reproducible end to end: `lrp_head_filtered_clustering.py` → one JSON per
dataset/top-K → `plot_lrp_head_filtering_results.py` → CSV + plot. No hand-transcription
anywhere in the chain (raw JSON results and the regenerated CSV/plot are committed
alongside the script).

**Correction (2026-09-16): the `noise`-degradation pattern is NOT robust.** Swept
`top_k` in {2,4,6,8} on the three affected datasets: `countries_capitals` actually
*helps* at every `top_k` except 4 (which looks like an outlier, not the trend); `ioi50`
flips sign between top4 and top6; only `ioi` shows a consistent (shrinking) hurt across
the range. Treat the "asymmetric, `meta` 6/9 vs. `noise` 3/9" framing above as a
single-point (`top_k=4`) result, not a settled property — worth re-checking `meta`
across the same `top_k` range before trusting that side of the finding either.

**Still open, explicitly named in the doc, not yet addressed**: does relevance
correlate with which delta-cluster a token actually lands in, specifically (not just
aggregate cluster quality)? Design spec (2026-09-16, not yet built): correlate a
token's z-scored layerwise delta profile against its relevance profile, using either
relevance that *led to* this token or relevance *driven from* it — both require the raw
per-sample `*_lrp_cache.h5` caches (cross-token attribution), not
`relevance_scores_all_tokens` (which is self-relevance — a token's own heads'
contribution to its own output, a different quantity). Full spec in
`PROJECT_PROGRESS.md`'s "Design spec" entry (2026-09-16).

→ `STATUS.md`'s scope-correction note; `PROJECT_PROGRESS.md`'s two "LRP head-filtered
clustering" entries (2026-09-15).

**Complementary mechanistic evidence: the successor circuit (added 2026-09-16, decided
with the user).** LRP-filtering and delta-clustering both ask *which heads matter* for a
token's update; this answers the doc's related "second step" question instead — *how*
is a specific update actually implemented via self-attention message-passing? Via direct
attribution inspection (not clustering), a real head set (13, 3, 7, 9, 8) drives
letter-succession — with a genuine negative control: the "textbook" synthetic induction
head (layer 16, head 14) scores high on the standard induction diagnostic but is *not*
the real driver on actual instances. Generalizes to a second ordinal category
(`days_of_week`, comparable concentration: 54.3% one-back attribution at layer 26 vs.
letters' 53% at layer 16) but *relocates in depth* — a partial touch on §3's "repeating
behavioral patterns," with the depth shift itself an open, unexplained question.

→ `STATUS.md` item 6 (Open gap 7); `OUTPUTS_MAPPING.md` for the two backing output
directories (`outputs/alphabet/baseline/2026-07-01_17-47-56-alph-no-interv`,
`alex_binding/cluster_res/2026-05-18_17-00-10`).

## 6. Robustness under perturbation

The validation step, once structure is established: if fingerprint/clustering structure
holds up, test whether the layers it implicates are actually load-bearing.

- **Necessity**: hard (zero-)ablating layers 24–26 collapses the CoT trace's final
  answer while reasoning stays coherent. Three independent signals agree — behavioral
  collapse, the otherwise-pure `noise` cluster degrading into an undifferentiated mess,
  and a logit-lens next-token-prediction check dropping from 89% (clean) to ~11%
  (ablated).
- **Joint necessity, causally validated with escalating controls**: layers 24–26 are
  *individually* low-importance and redundant (bypassing any one alone ranks
  21st/25th/26th of 28 by causal KL — near the bottom) but *jointly* indispensable —
  bypassing all three collapses the answer far more than a matched-magnitude injection
  control, surviving progressively harder controls (isotropic noise → a same-subspace
  permutation control using another token's real activity).
- **Replicated today on a second, structurally unrelated, well-powered dataset**
  (`ioi50` — short, non-templated, procedurally varied, vs. `countries_capitals`'s
  single-fact-recall): bypass beats the harder control in **41 of 41 measurable
  samples**, $p = 9.1\times10^{-13}$ — stronger on every metric than the original
  result ($p=1.86\times10^{-9}$, 30/30). This converts the paper's single strongest
  positive result from a single-dataset finding into a replicated one.
- **Single-layer geometric/causal metrics fail to even detect this known effect** —
  layers 24–26 don't rank highly by any single-layer importance measure. A validity
  problem for single-layer analysis specifically, not a null result about the group
  effect — and a concrete motivation for why the group/filtered analyses in §§5–6 matter
  more than single-layer descriptors alone.

→ `STATUS.md` item 3 (Necessity); the joint-necessity / causal-geometry entries in
`PROJECT_PROGRESS.md`, including the `ioi50` replication (2026-09-15).

---

## Honest stock-take: what actually works (2026-09-16)

An explicit strength ranking, asked for directly and worth keeping current rather than
letting the strongest-sounding prose win by default. Re-derive this whenever a new sweep
or replication lands — the LRP-filtering entry below is a live example of a result that
looked strong on a single point and got visibly weaker under its own robustness check.

**Strongest — real, replicated, decisive:**

1. **Joint-necessity causal validation (§6)** — layers 24-26 individually redundant,
   jointly indispensable, survived escalating controls, and now replicated on `ioi50`
   with an *even stronger* effect (41/41 samples, $p=9\times10^{-13}$, vs. the original
   30/30, $p=1.9\times10^{-9}$). Two structurally unrelated task families, same effect,
   strengthening under replication rather than weakening. The single most decisive result
   in the paper.
2. **89%-layer-locked clustering (§2/§3)** — 89-99% single-transition-specificity, holds
   across a 5x range of k, replicates *stronger* on a second model (Qwen3-8B: 95-99%,
   above the entire 0.6B band).
3. **ICA's `<think>` component (§2)** — dedicated, distinctive, in every dataset tested,
   no exceptions. The project's own standing description: "the most robust finding of
   this thread."
4. **"Wanders and returns" (§3)** — reproducible archetype, no exceptions across 6
   datasets.
5. **The successor circuit (§5)** — real head set, a genuine negative control that
   actually ruled something out (the textbook synthetic induction head), replicates on a
   second ordinal category.

**Foundational, solid but not headline-grade:** meta/noise label-free clustering (§1) —
reliable, but it's validity infrastructure for the rest, not a standalone finding.

**Shaky — needs honest treatment, not overweighting:**

- **LRP head-filtering (§5)** — currently written up as this paper's centerpiece
  (including in the rewritten abstract), but the `top_k` sweep showed the `noise` result
  isn't robust (flips sign across the range). `meta`'s robustness across the same range
  hasn't been checked yet and may have the same problem. Right now this is the
  *least*-supported claim in the paper, not the strongest — the abstract overweighted it
  before the sweep exposed the fragility. **Action item: sweep `meta` across the same
  `top_k` range before the abstract's framing is trusted as final.**
- **Open gap 6's causal half** — a real null (fingerprint purity doesn't predict causal
  importance), reframed as motivating the paper's more sophisticated methods rather than
  standalone evidence. Useful context, not a "works."

## Explicitly out of scope for this paper (real work, wrong paper)

- **LRP-conservation checks** — verification that the LRP machinery is trustworthy on a
  given model/architecture, requested specifically to confirm the models work correctly.
  Infrastructure, never paper content; methods-appendix mention at most.
- **The broader cross-model infrastructure sprint** (Phi-4/Nemotron/SmolLM3 bridge
  loading, Apertus) — real engineering, but only the one direct, on-scope replication
  check (the 89%-layer-locked finding on Qwen3-8B, §2) belongs in this paper's main
  narrative. The rest is future-work/appendix material at most.
- **The sentence-category position-confound audit** (Thought-Anchors-style probing is
  substantially confounded by sequence position) — a real, rigorous result, but
  tangential to this paper's actual mechanism. Supporting material, not core.
- **MATH-benchmark integration** — a separate, still-unfinished thread (Level 1/2
  screened, both too easy; Level 3 not yet run). Not blocking this paper's core story;
  status still explicitly `\todo`'d in the paper itself.
- **The delta-vs-state "founding premise" test** — retracted as inconclusive (3
  datasets favor delta, 1 coin-flip, 2 favor state including the best-powered one).
  Settled: deltas are a tool for building the fingerprint object, not a claim that
  they beat raw state at a downstream task. Kept in the paper's Limitations, not
  the abstract or headline results.

## The story, end to end

Representations and their layer-to-layer deltas carry real, non-random, clusterable
structure (§§1–4) — this isn't assumed, it's shown against a random-walk baseline and
confirmed by two independent unsupervised routes (spectral clustering, ICA). That
structure is class-specific and reproducible across datasets and model scale (§§2–3).
Filtering that structure by what the model itself considers relevant (LRP, at the
literal point of the residual-stream update) changes what's recoverable — genuinely, but
asymmetrically, which is itself the honest finding rather than a inconvenience to
smooth over (§5). And at least one piece of this structure — three layers that look
individually unimportant — is demonstrably, causally, repeatedly load-bearing once you
test the right unit (a group, not a single layer) with the right controls (§6). That's
the paper: real structure, an honest test of whether filtering it by relevance helps,
and causal teeth on the piece that matters most.
