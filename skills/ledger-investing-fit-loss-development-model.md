---
name: Fit a loss development model on a Ledger Analytics triangle
description: Upload an insurance loss triangle to the Ledger Analytics API and fit a Bayesian loss development model against it, polling the asynchronous task to completion.
api: openapi/ledger-investing-analytics-openapi.yml
operations:
  - createTriangle
  - listTriangles
  - listDevelopmentModelTypes
  - fitDevelopmentModel
  - getTask
  - getDevelopmentModel
generated: '2026-07-19'
method: generated
---

# Fit a loss development model

Base URL: `https://api.korra.com/analytics`

## Authenticate

Every request carries the API key in the Authorization header using the `Api-Key` scheme — **not** `Bearer`:

```
Authorization: Api-Key <LEDGER_ANALYTICS_API_KEY>
```

Keys are requested during the beta by emailing `analytics@ledgerinvesting.com` and managed at `https://ldgr.app/api-keys`. A key without permission for an action returns `403`.

## Steps

1. **Check whether the triangle already exists.** Call `listTriangles` (`GET /triangle?limit=25`). The response is a `count`/`results` envelope. Triangles are addressed by a human-assigned `name` that is unique per account, so match on `results[].name`.

2. **Upload the triangle if it is missing.** Call `createTriangle` (`POST /triangle`) with:

   ```json
   {
     "triangle_name": "<name>",
     "triangle_data": { "...": "bermuda.Triangle.to_dict() output" },
     "overwrite": false
   }
   ```

   `triangle_data` must be the Bermuda triangle dictionary representation — build it with the `bermuda-ledger` package rather than hand-assembling it. Set `overwrite: true` only when you intend to replace an existing triangle of the same name. **There is no Idempotency-Key header** — a retried create against an existing name fails unless `overwrite` is true, so treat `overwrite: true` as the safe-retry form. Read the created `id` from the response.

3. **Confirm the model type is available.** Call `listDevelopmentModelTypes` (`GET /development-model-type`). Development model types are `ChainLadder`, `TraditionalChainLadder`, `ManualATA`, `MeyersCRC` and `GMCL`. Do not pass a type that is not in the returned list.

4. **Submit the fit.** Call `fitDevelopmentModel` (`POST /development-model`) with:

   ```json
   {
     "triangle_name": "<name>",
     "model_name": "<model name>",
     "model_type": "ChainLadder",
     "overwrite": false,
     "model_config": {}
   }
   ```

   The model references the triangle by **name**, not id. `model_config` is model-specific — common members include `loss_family`, `seed`, `use_multivariate`, `informed_priors_version`, `priors` and `autofit_override`. The response returns `model.id` and `modal_task.id`.

5. **Poll the task.** Call `getTask` (`GET /tasks/{task_id}`) with the `modal_task.id`. While `task_response` is `null` the fit is still running. When it is populated, check `task_response.status`:
   - `success` — the model is fitted.
   - anything else — read `task_response.error` for the failure detail.

   Poll on a short interval and give up after a sensible budget; the first-party client defaults to 300 seconds. **A failed fit still returns a 2xx on the submit call** — the failure only surfaces in the polled task body, so never treat the 2xx as success.

6. **Read the fitted model.** Call `getDevelopmentModel` (`GET /development-model/{id}`). `modal_task_info.task_args` echoes back the `model_type` and the resolved `model_config` including defaults.

## Cancelling

If a fit is running longer than expected, call `terminateDevelopmentModel` (`POST /development-model/{id}/terminate`) and re-poll until the task status reads `terminated`. Termination is only meaningful while the status is `created` or `pending`.

## Errors

| Status | Meaning | Action |
|---|---|---|
| 400 | Payload rejected — usually a malformed triangle or invalid `model_config` | Fix the payload; do not retry unchanged |
| 403 | Key lacks permission | Check the `Api-Key` header form and key state |
| 404 | Endpoint or resource not found | Check the host and the resource id |
| 500 | Internal server error | Retry with backoff |

Errors are plain HTTP status codes with an undocumented JSON body — there is no `application/problem+json` envelope and no stable error-code registry. See `errors/ledger-investing-problem-types.yml`.
