---
name: Predict from a fitted Ledger Analytics model and retrieve the result triangle
description: Run a prediction from a fitted development, tail or forecast model on the Ledger Analytics API and download the resulting prediction triangle.
api: openapi/ledger-investing-analytics-openapi.yml
operations:
  - listDevelopmentModels
  - getDevelopmentModel
  - predictDevelopmentModel
  - predictTailModel
  - predictForecastModel
  - getTask
  - getTriangle
generated: '2026-07-19'
method: generated
---

# Predict from a fitted model and retrieve the result

Base URL: `https://api.korra.com/analytics`
Auth: `Authorization: Api-Key <LEDGER_ANALYTICS_API_KEY>`

Predictions in the Ledger Analytics API do not return data inline. They spawn an asynchronous task that writes results **back into a new triangle resource**, which you then fetch.

## Steps

1. **Locate the fitted model.** Call `listDevelopmentModels` (`GET /development-model?limit=25`) and match on `results[].name`, or call `getDevelopmentModel` (`GET /development-model/{id}`) directly if you already hold the id. The same pattern applies to tail and forecast models via `listTailModels` / `listForecastModels`.

2. **Submit the prediction.** Call the predict operation for the model class — `predictDevelopmentModel` (`POST /development-model/{id}/predict`), `predictTailModel` (`POST /tail-model/{id}/predict`) or `predictForecastModel` (`POST /forecast-model/{id}/predict`) — with:

   ```json
   {
     "triangle_name": "<triangle to predict on>",
     "prediction_name": "<name for the output triangle>",
     "overwrite": false,
     "predict_config": { "target_triangle": "<optional triangle to project onto>" }
   }
   ```

   `prediction_name` is optional but strongly recommended — without it the output triangle is harder to find later. `predict_config.target_triangle` names an existing triangle to project the predictions onto (for example a squared triangle for a right-edge forecast).

   The response carries `predictions` (the id of the triangle the results will be written to) and `modal_task.id`.

3. **Poll the task.** Call `getTask` (`GET /tasks/{task_id}`) with `modal_task.id`. Wait until `task_response` is non-null, then require `task_response.status == "success"`. On failure read `task_response.error`. Do not fetch the prediction triangle before the task succeeds — it will not be populated.

4. **Retrieve the prediction triangle.** Call `getTriangle` (`GET /triangle/{id}`) with the `predictions` id from step 2.

   **Large-payload handling:** the response may return a `url` field carrying a pre-signed download location **instead of** an inline `triangle_data` object. If `url` is present, follow it with a plain unauthenticated GET and stream the body — do not send the `Api-Key` header to the pre-signed URL. Parse the downloaded content with `bermuda-ledger`.

## Cashflow predictions

For a cashflow model, call `predictCashflowModel` (`POST /cashflow-model/{id}/predict`). Its `predict_config` accepts `initial_loss_name` — the name of an initial loss triangle used to seed the projection — instead of `target_triangle`. The task and retrieval steps are identical.

## Conventions to respect

- Cross-resource references on write are by **name**; reads are by **id**.
- List endpoints paginate with `limit` and return `count` / `results`; there is no cursor.
- No rate limits are documented, but the API is in free beta — poll on a courteous interval rather than tight-looping.
- No webhooks or event callbacks exist. Polling is the only completion signal.
