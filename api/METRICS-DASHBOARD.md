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

```python
{ 
  "prediction_type":  "binary_classification" or "probabilistic_binary_classification" or "regression",
  "array_bindings": {   # all 1D arrays, including features and predictior vectors, are here
    "<array_key>" : {   # the keys can be arbitrary strings; not sure we need to force any convention, but see examples below
      "name": string,   # the name of a feature would be the feature name, of a prediction vector would be the model name
      "values": number[],  
      "value_names": string[],       # an optional field to encode categorical data   
    },
    "sensitive_feature gender" : {   # an example feature
      "name": "gender",
      "values": [0, 1, 0, 0, 2],
      "value_names": ["female", "male", "non-binary"],
    },
    "y_pred model0" : {      # an example prediction vector             
      "name": "model0",
      "values": [0, 0, 1, 1, 0],
    },
    "y_true": {
      "name": "y_true",
      "values": [0, 1, 1, 1, 0],
    },
    "sample_weight": {
      "name": "sample_weight",
      "values": [0.1, 0.3, 1, 0.9],
    }
    ...
  },
  "cache" : [
    {
      "function": string,   # python function name; we could either limit to fairlearn.metrics
                            # or use fully qualified names
      "arguments": {
        "<array_argument>": "<array_key>" or null,   # array-valued arguments are matched with array bindings
        "<numeric_argument>": number or null,        # we should also support numeric arguments, strings, booleans
        "<string_argument>": string or null,         # null corresponds to None
        "<boolean_argument>": boolean or null,
      },
      "return_value": number or string or boolean or null or dict,
                                                     # dict could be encoded as { "keys": any[], "values": any[] }
    },
    { # an example
      "function": "fbeta_score_group_summary",
      "arguments": {
        "y_true": "y_true",
        "y_pred": "y_pred model0",
        "sensitive_features": "sensitive_feature gender",
        "sample_weight": "sample_weight",
        "beta": 0.3,
      },
      "return_value": {
        "overall": 0.11,      
        "by_group": {
          "keys": [0, 1, 2],
          "values": [0.15, 0.04, 0.03],
        }
      },
    },
    ...
  ]
}


### Part II: How to remove duplication

`<TODO>`
