# Data Card — Militarized-Framing Seed Dataset (synthetic)

## What this is
A **44-example synthetic seed** of imitated EU institutional speech about European/liberal
values, generated against `generation_prompt.md`. Each example is labeled on four
independent binary discourse axes. Intended as a schema demonstration and a starter set
for prompt/labeling validation — **not** a training corpus and **not** evidence about the
real EU.

## Files
- `militarization_seed_dataset.jsonl` — one JSON object per line (canonical).
- `militarization_seed_dataset.csv` — same data, flat.
- `generation_prompt.md` — the generator prompt + run protocol.

## Schema (per row)
| field | type | notes |
|---|---|---|
| `id` | str | m001–m044 |
| `value_theme` | enum | democracy, rule_of_law, human_rights, solidarity, freedom, european_unity, strategic_autonomy, none |
| `register` | enum | formal_address, press_statement, parliamentary_debate, op_ed, informal_remarks |
| `securitization` | 0/1 | SEC axis |
| `militarization` | 0/1 | MIL axis (focal) |
| `geopoliticization` | 0/1 | GEO axis (control) |
| `affective_polarization` | 0/1 | POL axis (control) |
| `text` | str | 55–150 words |

## Axis definitions (operational)
- **SEC — securitization.** Referent object framed as facing an *existential* threat justifying *exceptional* measures beyond normal politics. (Buzan, Wæver & de Wilde 1998.) Not: ordinary risk/concern.
- **MIL — militarization (focal).** *Legitimizes* military force / rearmament / mobilization, or fuses values-identity with martial strength. Not: merely *mentioning* defense/NATO/weapons as a topic; not metaphorical "battle/warfare."
- **GEO — geopoliticization.** Inter-state *power rivalry*: great-power competition, spheres of influence, strategic autonomy, blocs, relative gains, named rival powers.
- **POL — affective polarization.** Antagonistic *in-group/out-group* sorting, delegitimization of opponents — typically domestic. An external state adversary alone is GEO, not POL, unless moralized us/them sorting is added.

**Focal hypothesis class** = `value_theme != none` AND `SEC=1` AND `MIL=1`.

## Design rationale
The four axes exist to establish **discriminant validity**. The recurring reviewer attack on
this kind of work is "your detector just picks up us/them polarization or great-power-rivalry
talk and calls it militarism." Measuring GEO and POL separately lets you show MIL carries
independent signal. The dataset therefore over-samples **boundary cells** that isolate one
axis from MIL:
- SEC=1/MIL=0 (securitized but non-militarized — e.g. rule-of-law defended juridically)
- GEO=1/MIL=0 (geopoliticized but non-militarized — economic/strategic rivalry)
- POL=1/SEC=0/MIL=0 (polarized but non-securitized — domestic contempt)
- MIL=1/SEC=0 (defense-as-normal-policy — force legitimated without threat dramatization)
- all-0 topical (defense as administrative *topic*, no framing)

Several rows are deliberate hard cases (e.g. m026 uses metaphorical "warfare by other means"
but is MIL=0; m036 mentions munitions procurement but is MIL=0). These teach the boundary
that mention ≠ militarization.

## Balance (n=44)
Per-axis positives: SEC 21, MIL 16, GEO 13, POL 10.
Focal (SEC&MIL) 12 · SEC&¬MIL 9 · MIL&¬SEC 4 · all-zero 10.
Focal rate is roughly constant across registers (2/5, 2/8, 3/13, 2/8, 3/10) — i.e. register
does not leak the label.

## Critical limitations
1. **Single generator, no agreement filter.** This seed was produced by one model without
   the cross-model blind re-labeling gate. Before use at scale, apply the gate in the prompt's
   run protocol; without it the labels encode one model's construct interpretation.
2. **Too clean.** Synthetic EU speech is more explicit and on-the-nose than real discourse,
   where militarized framing is subtle and deniable. Expect a detector trained only on this to
   over-rely on lexical tells and fail on real text.
3. **Markers are theoretical commitments.** If the axis definitions are wrong, the data is
   wrong. Validate them against a small human-coded set of *real* EU speeches (target
   inter-annotator κ ≥ ~0.7 per axis) before trusting any classifier output.
4. **No empirical claim follows from synthetic-only data.** Use as augmentation/robustness on
   top of a real, human-coded seed and test set. Literature consensus: best mix is majority
   synthetic + a real anchor (~80/20 in one controlled study), never 100% synthetic.

## Suggested next steps
- Scale via the cell-grid loop in `generation_prompt.md` (≥2 generators, temperature sweep 0.3–1.0, dedup → coverage selection → cross-model agreement filter).
- Build a real human-coded validation set (sampling frame: institution × period) — the part currently parked.
- Baseline: fine-tune DistilBERT/BERT with a 4-output sigmoid head, BCE loss; report per-axis F1 and, crucially, MIL performance on the SEC=1/MIL=0 and POL=1 boundary subsets.
