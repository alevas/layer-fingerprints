# Outputs mapping — eligible local output directories for this paper

Cross-referenced `RUN_INDEX.md` (pre-session load-bearing list, broader-project scope)
against `PAPER_POINTS.md` (this paper's actual ICLR 27 Drafting scope), 2026-09-16.
"Eligible" = the directory backs a finding actually cited in `PAPER_POINTS.md`.

## Eligible

| Path | Backs |
|---|---|
| `outputs/alphabet_disjoint/baseline/2026-07-20_00-00-16` | primary dataset — meta/noise validity, necessity test (§6) |
| `outputs/alphabet_disjoint/layer_clamp/clamp_layers_24-26/2026-07-19_23-30-29` | necessity test, clamp comparison (§6) |
| `alex_binding/cluster_res/2026-08-31_14-22-37` | layers-24-26 zero-ablation bypass run (§6) |
| `alex_binding/cluster_res/2026-08-31_14-36-17` | full-28 zero-ablation bypass run (§6) |
| `outputs/2026-09-04_15-12-27` (`alphabet`) | meta/noise, 89%-layer-locked, ICA, LRP-filtering, tortuosity (§§1-5) |
| `outputs/2026-09-04_17-42-30` (`days_of_week`) | LRP-filtering headline result, 89%-layer-locked (§§2,5) |
| `outputs/2026-09-04_15-11-58` (`numbers_letters`) | LRP-filtering (§5) |
| `outputs/2026-09-04_13-54-18` (`countries_capitals`) | joint-necessity original result, LRP-filtering + sweep (§§5-6) |
| `outputs/2026-09-04_17-50-00` (`ioi`, 5-sample) | LRP-filtering (§5) |
| `outputs/2026-09-09_13-45-07` (`ioi50`) | joint-necessity **replication**, LRP-filtering + sweep (§§5-6) |
| `outputs/2026-09-14_14-47-03` (Qwen3-8B `days_of_week`) | 89%-layer-locked cross-model replication, LRP-filtering (§§2,5) |
| `outputs/2026-09-14_14-48-33` (Qwen3-8B `numbers_letters`) | LRP-filtering (§5) |
| `outputs/2026-09-14_15-15-38` (Qwen3-8B `ioi`) | LRP-filtering (§5) |
| `outputs/alphabet/baseline/2026-07-01_17-47-56-alph-no-interv` | successor-circuit / induction-head thread (Open gap 7) — the letter-recitation base run: attention-sink → semantic anchor → preceding-letter (induction-like) → punctuation attribution pattern, and the check ruling out the synthetic layer-16-head-14 induction head as the real driver |
| `alex_binding/cluster_res/2026-05-18_17-00-10` | successor-circuit thread, second ordinal category (`days_of_week`) — the cross-category generalization check (54.3% one-back attribution at layer 26, vs. letters' 53% at layer 16) |

All paths relative to `/home/alevas/Desktop/workspace/playground/` (local) or
`/home/alevas/playground/` (cluster) — same relative structure both places.

**Note on the successor-circuit pair (decided with the user, 2026-09-16):** included as
eligible even though the successor-circuit/induction-head finding isn't one of
`PAPER_POINTS.md`'s six numbered sections outright — it's a real, on-scope mechanistic
result (does the token-transformation structure connect to an identifiable computational
circuit, i.e. induction-like attention heads) and belongs alongside the rest of this
paper's evidence, not held out as a separate direction.

## Not eligible (real, load-bearing for the *broader* project, wrong paper)

`outputs/2026-09-10_18-32-22` (Apertus, parked), any Nemotron/SmolLM3/Phi-4 bridge-model
output, `outputs/math_pilot/*`, the Qwen3-8B 100-sample `countries_capitals` run.

## Corrected (2026-09-16): removed one mischaracterized entry

`outputs/alphabet_disjoint/baseline/2026-07-19_22-54-00` was previously listed as
eligible, "backs the necessity test cross-run check (§6)" — wrong on two counts. It's
actually an early `meta`/`noise` cluster-replication check (unrelated to the necessity/
ablation test), and `RUN_INDEX.md`'s own description marks it explicitly superseded and
confounded ("confounded with the intervention itself... superseded in strength by the
2026-08-14 confound-free check... reference only, not actively used past that one
check"). Not used anywhere in `PAPER_POINTS.md`'s actual text — removed from the eligible
table rather than left as an unfulfilled claim.
