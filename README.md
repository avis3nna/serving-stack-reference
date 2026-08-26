# serving-stack

The one system this course builds. Your team creates this repository once from
the template, and every lab from week 2 to graduation is a change to it. There
is no week where you start again.

## What is here

```
app/        empty. Your service goes here, starting week 2 day 2
docs/       the API contract the Agentic AI cohort integrates against
scripts/    verify-env.sh, which checks your machine against what the labs need
PINS.md     every version this course depends on
setup.md    how to work in this repository
```

That is the whole repository, and the shortness of that list is the point. You
are not given a finished system to read. You build one, a day at a time, and by
week 6 another cohort's agents are calling it.

## What you add, and when

| Week | Day | What you add |
|---|---|---|
| 2 | Mon | `app/` behind an OpenAI-compatible `/v1` on CPU |
| 2 | Tue | `Dockerfile`, and your image on Docker Hub |
| 2 | Wed | `Dockerfile.gpu`, the same code on a GPU |
| 2 | Thu | `compose.yaml`, the stack described rather than run by hand |
| 3 | Thu | `bench/`, the harness that measures all of it |

Each one is a lab, and each one starts from files that day hands you. Lab
instructions, decks and quizzes are on the course Drive, one folder per week.
This repository is your code.

## Start here

```bash
./scripts/verify-env.sh     # checks your machine, writes verify-env-report.json
```

Then read `setup.md`. It is short, and it covers the two things that go wrong:
committing a key, and committing a model.

---

## W2D1: first contact (reference)

Measured on a Colab T4, `Qwen/Qwen2.5-1.5B-Instruct`, 128 generated tokens.

| dtype | predicted GB | measured GB | observed bytes/param | tokens/s |
|---|---|---|---|---|
| fp16 | 3.0 | 3.29 | 2.19 | 31.9 |
| int8 | 1.5 | 1.87 | 1.25 | 6.0 |
| int4 | 0.75 | 1.24 | 0.83 | 15.3 |

Three things the numbers say that the formula does not:

- **Measured always exceeds predicted.** Parameters times bytes is weights only.
  The CUDA context, the framework and the activations are the rest, and they do
  not shrink when the weights do.
- **The quantised rows overshoot hardest.** int4 predicted 0.75 GB and measured
  1.24 GB, so observed bytes per parameter is 0.83 rather than 0.5. Quantisation
  is applied to most of the weights, not all of them, and the fixed overhead is
  now a larger share of a smaller number.
- **int8 is slower than int4, and both are slower than fp16.** Smaller weights do
  not mean faster. bitsandbytes int8 dequantises on the fly through a path with
  no fused kernel, so it saves memory and costs speed. That gap is the plant for
  week 3 Thursday: the bits were never the problem, the kernels were.

Files: `generate.py`, `results.json`.
