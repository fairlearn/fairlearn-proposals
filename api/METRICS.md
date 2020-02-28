# API proposal for metrics

## Example

```python
# For most sklearn metrics, we would have their group version that returns a Bunch with fields
#   * overall: overall metric value
#   * by_group: a dictionary that maps sensitive feature values to metric values

summary = accuracy_score_by_group(y_true, y_pred, sensitive_features=sf, **other_kwargs)

# Exporting into pd.Series or pd.DataFrame in not too complicated

series = pd.Series({**summary.by_group, 'overall': summary.overall})
df = pd.DataFrame({"model accuracy": {**summary.by_group, 'overall': summary.overall}})

# Several types of scalar metrics for group fairness can be obtained from `summary` via transformation functions

acc_difference = difference_from_summary(summary)
acc_ratio = ratio_from_summary(summary)
acc_group_min = group_min_from_summary(summary)

# Most common disparity metrics should be predefined

demo_parity_difference = demographic_parity_difference(y_true, y_pred, sensitive_features=sf, **other_kwargs)
demo_parity_ratio = demographic_parity_ratio(y_true, y_pred, sensitive_features=sf, **other_kwargs)
eq_odds_difference = equalized_odds_difference(y_true, y_pred, sensitive_features=sf, **other_kwargs)

# For predefined disparities based on sklearn metrics, we adopt a consistent naming conventions

acc_difference = accuracy_score_difference(y_true, y_pred, sensitive_features=sf, **other_kwargs)
acc_ratio = accuracy_score_ratio(y_true, y_pred, sensitive_features=sf, **other_kwargs)
acc_group_min = accuracy_score_group_min(y_true, y_pred, sensitive_features=sf, **other_kwargs)
```

## Functions

*Function signatures*

```python
metric_by_group(metric, y_true, y_pred, *, sensitive_features, **other_kwargs)
# return the summary for the provided metrics

make_metric_by_group(metric)
# return a callable object <metric>_by_group:
# <metric>_by_group(...) = metric_by_group(<metric>, ...)

# Transformation functions returning scalars
difference_from_summary(summary)
ratio_from_summary(summary)
group_min_from_summary(summary)
group_max_from_summary(summary)

# Metric-specific functions returing summary and scalars
<metric>_by_group(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_difference(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_ratio(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_min(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_max(y_true, y_pred, *, sensitive_features, **other_kwargs)
```

*Summary of transformations*

|transformation function|output|metric-specific function|code|aif360|
|-----------------------|------|------------------------|----|------|
|`difference_from_summary`|max - min|`<metric>_difference`|D|unprivileged - privileged|
|`ratio_from_summary`|min / max|`<metric>_ratio`|R| unprivileged / privileged|
|`group_min_from_summary`|min|`<metric>_group_min`|Min| N/A |
|`group_max_from_summary`|max|`<metric>_group_max`|Max| N/A |

*Supported metric-specific functions*

|metric|variants|task|notes|aif360|
|------|--------|-----|----|------|
|`selection_rate`| G,D,R,Min | class | | &#x2713; |
|`demographic_parity`| D,R | class | `selection_rate_difference`, `selection_rate_ratio` | `statistical_parity_difference`, `disparate_impact`|
|`accuracy_score`| G,D,R,Min | class | sklearn | `accuracy` |
|`balanced_accuracy_score` | G | class | sklearn | - |
|`mean_absolute_error` | G,D,R,Max | class,reg | sklearn | class only: `error_rate`
|`false_positive_rate` | G,D,R | class | | &#x2713; |
|`false_negative_rate` | G | class | | &#x2713; |
|`true_positive_rate` | G,D,R | class | | &#x2713; |
|`true_negative_rate` | G | class | | &#x2713; |
|`equalized_odds` | D,R | class | max of difference or ratio under `true_positive_rate`, `false_positive_rate` | - |
|`precision_score`| G | class | sklearn | &#x2713; |
|`recall_score`| G | class | sklearn | &#x2713; |
|`f1_score`| G | class | sklearn | - |
|`roc_auc_score`| G | prob | sklearn | - |
|`log_loss`| G | prob | sklearn | - |
|`mean_squared_error`| G | prob,reg | sklearn | - |
|`r2_score`| G | reg | sklearn | - |
