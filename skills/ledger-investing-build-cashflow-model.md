---
name: Build a Ledger Analytics cashflow model from a development and tail model
description: Compose a cashflow model on the Ledger Analytics API from a fitted loss development model and a fitted tail model, then project cashflows.
api: openapi/ledger-investing-analytics-openapi.yml
operations:
  - fitDevelopmentModel
  - fitTailModel
  - getTask
  - createCashflowModel
  - getCashflowModel
  - predictCashflowModel
  - getTriangle
generated: '2026-07-19'
method: generated
---

# Build a cashflow model

Base URL: `https://api.korra.com/analytics`
Auth: `Authorization: Api-Key <LEDGER_ANALYTICS_API_KEY>`

A cashflow model is not fit from data directly — it is **composed** from two models that must already be fitted and named: one development model and one tail model.

## Steps

1. **Fit the development model.** Call `fitDevelopmentModel` (`POST /development-model`) with `triangle_name`, `model_name` and a `model_type` from `ChainLadder`, `TraditionalChainLadder`, `ManualATA`, `MeyersCRC`, `GMCL`. Poll `getTask` (`GET /tasks/{task_id}`) until `task_response.status == "success"`.

2. **Fit the tail model.** Call `fitTailModel` (`POST /tail-model`) with the same triangle and a `model_type` from `GeneralizedBondy`, `Sherman`, `ClassicalPowerTransformTail`. Poll `getTask` to success.

3. **Compose the cashflow model.** Call `createCashflowModel` (`POST /cashflow-model`) with:

   ```json
   {
     "name": "<cashflow model name>",
     "dev_model_name": "<name from step 1>",
     "tail_model_name": "<name from step 2>",
     "model_config": {}
   }
   ```

   Both source models are referenced by **name**, not id. If either name does not resolve to a fitted model the request fails. Poll the returned `modal_task.id` via `getTask`.

4. **Confirm the composition.** Call `getCashflowModel` (`GET /cashflow-model/{id}`). The response echoes `development_model` and `tail_model` as the ids of the two source models — verify they are the ones you intended before projecting.

5. **Project cashflows.** Call `predictCashflowModel` (`POST /cashflow-model/{id}/predict`) with:

   ```json
   {
     "triangle_name": "<triangle to project on>",
     "prediction_name": "<name for the output triangle>",
     "overwrite": false,
     "predict_config": { "initial_loss_name": "<optional initial loss triangle name>" }
   }
   ```

   `predict_config.initial_loss_name` names an existing triangle used to seed the projection. Poll `getTask` to success, then fetch the output with `getTriangle` (`GET /triangle/{id}`) using the `predictions` id from the predict response — following the pre-signed `url` if one is returned instead of inline data.

## Ordering and failure rules

- The two source models must reach `success` **before** step 3. Composing against a model whose fit task is still pending or failed will not work.
- Every one of these calls returns 2xx on submission regardless of whether the underlying compute succeeds. `task_response.status` is the only authoritative outcome; `task_response.error` carries the reason on failure.
- To retry a composition under the same name, set `overwrite: true` on the create — there is no idempotency key.
- Delete order matters when cleaning up: remove the cashflow model (`deleteCashflowModel`) before deleting the development or tail models it references.
