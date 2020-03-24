# Proposal for metrics/dashboard API

Goals:
* Unified treatment of many different kinds of metrics in dashboard
* A clean separation between dashboard (for visualization) and different kinds of backends (for metric calculation)

Status quo:
* Dashboard currently keeps track of the metric values in the cache dictionary `fairlearn.widget._fairlearn_widget.FairlearnWidget._response`
* The dictionary is updated in `fairlearn.widget._fairlearn_dashboard.FairlearnDashboard._on_request`
* There's a PR out to fill the whole data structure in  `fairlearn.metrics.create_dashboard_dictionary`

Issues with the status quo:
* Code duplication / redundancy / brittleness due to copy-paste errors
* Currently only one kind of a metric (`<metric>_summary`) is supported and hard-wired into the dashboard dictionary

## Proposal

### Part I: More general dashboard dictionary

```
{
  "prediction_type":  "binary_classification" or "probabilistic_binary_classification" or "regression",
  "y_true": float[],
  "y_pred": float[][],
  "sensitive_features": [
    {
      "column_name": String,
      "column_data": int[],
      "value_names": string[]
    },
    ...
  ],
  "metrics": [
    [
      {
        "<metric_fct>" :  <metric_result_dict>,
        ...
      },
      ...
    ]
  ]
}
```

Where `metrics[i][j]["<metric_fct>"]` should correspond to the result of the call
```
<metric_fct>(y_true, y_pred[i], sensitive_features[j])
```
If the result of the call is a dictionary, then we use that dictionary as `<metric_result_dict>`. If the result is a scalar `x`, we could use the dictionary `{ "value": x }`.

#### What's still missing here:

The above is incomplete, because we need to enable caching results for the calls of the form
```
<metric_fct>(y_true, y_pred[i], sensitive_features[j], arg1=val_arg1, arg2=val_arg2, ...)
```
One idea would be to use something like:
```
{
  "<metric_fct>": [
    {
      "arguments": { "arg1": val_arg1, "arg2": val_arg2, ...},
      "result": <metric_result_dict>
    },
    ...
  ]
}
```


### Part II: How to remove duplication

`<TODO>`
