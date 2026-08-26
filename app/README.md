# app/ — W2D2 reference

The `/v1` service as the lab asks for it: the five starter files with both
TODOs filled in. Scope is exactly the day's objective and nothing further.

| Route | |
|---|---|
| `GET /health` | given in the starter |
| `GET /v1/models` | TODO 1 |
| `POST /v1/chat/completions`, non-streaming | TODO 2 |
| `POST /v1/chat/completions`, streaming | the delta step, optional |

This is deliberately **not** the finished multi-week service. That one is 392
lines and carries the engine backends and the GPU path from weeks 3 to 5.
Reading it now would answer four labs at once.

## Run it

```bash
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

Then in another shell:

```bash
python client_test.py
```

Verified 2026-08-24 against the lab's own `verify.py`, on
`Qwen/Qwen2.5-0.5B-Instruct`: green check passes, streaming shows SSE chunks
and a `[DONE]` terminator, and an unmodified `openai` client works against it
with only a `base_url` change.

## Five things to check in your own version

The first two fail the green check, so you already know if you hit them. The
last three **pass** it and are still wrong, which makes them the ones worth
looking for by hand.

1. **Decoding the whole output instead of the new tokens.** The reply comes back
   with your prompt echoed in front of it, and `completion_tokens` counts the
   prompt. Slice `out[0][prompt_tokens:]` first.
2. **`total_tokens` not equal to prompt plus completion.** The verifier checks
   the arithmetic, not just that the field is present.
3. **`finish_reason` hardcoded to `"stop"`.** It is `"length"` when generation
   stopped because it hit `max_tokens`.
4. **A constant completion `id`.** Fine today, breaks the agentic client in
   week 4. Use a fresh `uuid4` per call.
5. **Serving whatever `model` id the caller asks for.** The contract rejects an
   unknown id with a 400 `model_not_found`, and the consumer compares the id it
   gets back character for character.

See `requests.md` for a working curl per route.
