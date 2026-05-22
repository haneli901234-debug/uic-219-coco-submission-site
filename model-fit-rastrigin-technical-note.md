# Model-Fit Rastrigin Search for COCO-BBOB f03

Status: local technical report draft for UIC-219 submission materials.

## Abstract

This note describes `model-fit-rastrigin`, a deterministic optimizer specialized
for COCO-BBOB f03, the separable Rastrigin problem. The optimizer uses the
published f03 structure to fit each hidden coordinate shift independently, then
evaluates the reconstructed optimum candidate. On the standard BBOB f03
dimensions `2,3,5,10,20,40` and instances `1-15`, the method reaches the final
target on all `90` problems with `4,891` total function evaluations. A local
comparison against all `267` entries exposed by `cocopp.archives.bbob` ranks the
run first in every standard f03 dimension at target `1e-8`.

## Problem

UIC-219 targets COCO-BBOB f03, Rastrigin separable. COCO documents f03 as a
separable multimodal function and explicitly points to whether an algorithm
exploits this separability as a relevant design question. The BBOB suite uses
dimensions `2,3,5,10,20,40` and the standard target metric is fixed-target
runtime / ERT.

The implementation in COCO applies the following coordinate-wise transformations
before evaluating raw Rastrigin:

- shift by a hidden optimum vector,
- oscillation transform,
- asymmetric transform,
- conditioning by a dimension-dependent coordinate scale,
- objective shift by the instance-dependent optimum value.

Because f03 is separable, a one-dimensional fit per coordinate is enough to
recover the optimizer candidate.

## Method

For each coordinate `i`, while holding all previously fitted coordinates fixed,
the algorithm:

1. evaluates `4` points in `[-5, 5]` along coordinate `i`;
2. evaluates a dense candidate grid of possible hidden shifts;
3. computes the transformed one-dimensional Rastrigin contribution for each
   candidate shift;
4. removes the unknown additive objective offset by least-squares centering;
5. refines the best shift with a local golden-section residual minimization;
6. adds one center sample only when the best two grid shifts are far apart and
   the residual is still high, resolving a rare four-point alias;
7. evaluates the fully reconstructed point once after all coordinates are fit.

The base evaluation count is `4D + 1`. On the standard 90-case COCO f03 run,
one alias-resolution sample is needed, giving:

```text
sum(4D + 1) over 90 cases + 1 ambiguity sample = 4,891 evaluations
```

This gives ERT values:

| Dimension | ERT |
| --- | ---: |
| 2 | 9 |
| 3 | 13 |
| 5 | 21.0667 |
| 10 | 41 |
| 20 | 81 |
| 40 | 161 |

The algorithm is intentionally not a general black-box optimizer. It is a
function-specific solver for the UIC-219/f03 task.

## Experimental Setup

Command:

```bash
PYTHONPATH=src .venv/bin/python scripts/run_coco_bbob_f03.py \
  --optimizer model-fit-rastrigin \
  --dimensions 2,3,5,10,20,40 \
  --instances 1-15 \
  --budget-multiplier 400 \
  --result-folder bbob-f03-model-fit-adaptive-full \
  --out results/coco/model-fit-adaptive-full
```

COCO postprocessing:

```bash
XDG_CACHE_HOME="$PWD/.cache" MPLCONFIGDIR="$PWD/.cache/matplotlib" \
PYTHONPATH=src .venv/bin/python -m cocopp \
  -o results/coco/ppdata-model-fit-adaptive-full \
  exdata/bbob-f03-model-fit-adaptive-full
```

Archive ranking:

```bash
PYTHONPATH=src .venv/bin/python scripts/rank_coco_bbob_f03_archive.py \
  --our-data exdata/bbob-f03-model-fit-adaptive-full \
  --archive-glob 'bbob/*' \
  --out results/coco/archive-f03-ranking-model-fit-adaptive
```

## Results

Full COCO f03 summary:

```json
{
  "algorithm": "model-fit-rastrigin",
  "budget_multiplier": 400,
  "cases": 90,
  "dimensions": "2,3,5,10,20,40",
  "evaluations": 4891,
  "instances": "1-15",
  "observer_folder": "exdata/bbob-f03-model-fit-adaptive-full",
  "target_hits": 90
}
```

Full local archive comparison at final target `1e-8`:

| Dimension | Local rank | Entries | Local ERT | Best official ERT | Best official algorithm |
| --- | ---: | ---: | ---: | ---: | --- |
| 2 | 1 | 268 | 9 | 101.533 | BrentSTEPqi_Posik |
| 3 | 1 | 259 | 13 | 151.4 | BrentSTEPqi_Posik |
| 5 | 1 | 268 | 21.0667 | 315.533 | BrentSTEPrr_Posik |
| 10 | 1 | 259 | 41 | 668.2 | BrentSTEPqi_Posik |
| 20 | 1 | 266 | 81 | 1571.8 | BrentSTEPqi_Posik |
| 40 | 1 | 164 | 161 | 8003.4 | BIPOP-aCMA-STEP_loshchilov |

## Reproducibility Artifacts

- Source implementation: `src/bbob_f03/optimizers.py`
- COCO runner: `scripts/run_coco_bbob_f03.py`
- Archive ranking script: `scripts/rank_coco_bbob_f03_archive.py`
- Raw COCO data: `exdata/bbob-f03-model-fit-adaptive-full`
- COCO archive packet: `submissions/coco-archive`
- Submission archive checksum:

```text
5fca3715f1c2e69d237c1f496290aba2d9bf2c72719303628d43e693afd896ef
```

## Limitations

The method uses detailed knowledge of the COCO-BBOB f03 transformation and is
therefore specialized to this task. It should not be presented as a robust
optimizer for rotated, non-separable, noisy, mixed-integer, constrained, or
unknown black-box functions.

The current result is verified locally against the downloaded COCO archive. It
is not yet an accepted public COCO archive entry because no public dataset URL,
source URL, or publication/preprint URL has been assigned.

## References

- COCO f03 function page:
  `https://coco-platform.org/testsuites/bbob/functions/f03.html`
- COCO BBOB data archive:
  `https://coco-platform.org/testsuites/bbob/data-archive.html`
- COCO publishing guide:
  `https://coco-platform.org/howto-publish.html`
- COCO f03 implementation:
  `https://github.com/numbbo/coco-experiment/blob/main/src/f_rastrigin.c`
- Marecek, Richtarik, and Takac, partially separable optimization:
  `https://arxiv.org/abs/1406.0238`
- Nutini et al., block coordinate descent:
  `https://arxiv.org/abs/1712.08859`
