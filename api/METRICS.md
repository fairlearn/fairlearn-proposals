# API proposal for metrics

## Example

```python
# For most sklearn metrics, we will have their group version that returns
# the summary of its performance across groups as well as the overall
# performance, represented as a Bunch object with fields
#   * overall: overall metric value
#   * by_group: a dictionary that maps sensitive feature values to metric values

summary = accuracy_score_group_summary(y_true, y_pred, sensitive_features=sf, **other_kwargs)

# Exporting into pd.Series or pd.DataFrame in not too complicated

series = pd.Series({**summary.by_group, 'overall': summary.overall})
df = pd.DataFrame({"model accuracy": {**summary.by_group, 'overall': summary.overall}})

# Several types of scalar metrics for group fairness can be obtained from the group summary via transformation functions

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

## Proposal

*Function signatures*

```python
group_summary(metric, y_true, y_pred, *, sensitive_features, **other_kwargs)
# return the group summary for the provided `metric`, where `metric` has the signature
# metric(y_true, y_pred, **other_kwargs)

make_metric_group_summary(metric)
# return a callable object <metric>_group_summary:
# <metric>_group_summary(...) = group_summary(<metric>, ...)

# Transformation functions returning scalars
difference_from_summary(summary)
ratio_from_summary(summary)
group_min_from_summary(summary)
group_max_from_summary(summary)

# Metric-specific functions returning group summary and scalars
<metric>_group_summary(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_difference(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_ratio(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_min(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_max(y_true, y_pred, *, sensitive_features, **other_kwargs)
```

*Transformations and transformation codes*

|transformation function|output|metric-specific function|code|aif360|
|-----------------------|------|------------------------|----|------|
|`difference_from_summary`|max - min|`<metric>_difference`|D|unprivileged - privileged|
|`ratio_from_summary`|min / max|`<metric>_ratio`|R| unprivileged / privileged|
|`group_min_from_summary`|min|`<metric>_group_min`|Min| N/A |
|`group_max_from_summary`|max|`<metric>_group_max`|Max| N/A |

*Tasks and task codes*

|task|definition|code|
|----|----------|----|
|binary classification|labels and predictions are in {0,1}|class|
|probabilistic binary classification|labels are in {0,1}, predictions are in [0,1] and correspond to estimates of P(y\|x)|prob|
|randomized binary classification|labels are in {0,1}, predictions are in [0,1] and represent the probability of drawing y=1 in a randomized decision|class-rand|
|regression|labels and predictions are real-valued|reg|

*Predefined metric-specific functions*

* variants: D, R, Min, Max refer to the transformations from the table above; G refers to `<metric>_group_summary`.

|metric|variants|task|notes|aif360|
|------|--------|-----|----|------|
|`confusion_matrix` | G | class | sklearn | `binary_confusion_matrix` |
|`false_positive_rate` | G,D,R | class | | &#x2713; |
|`false_negative_rate` | G | class | | &#x2713; |
|`true_positive_rate` | G,D,R | class | | &#x2713; |
|`true_negative_rate` | G | class | | &#x2713; |
|`selection_rate`| G,D,R | class | | &#x2713; |
|`demographic_parity`| D,R | class | `selection_rate_difference`, `selection_rate_ratio` | `statistical_parity_difference`, `disparate_impact`|
|`equalized_odds` | D,R | class | max of difference or min ratio under `true_positive_rate`, `false_positive_rate` | - |
|`accuracy_score`| G,D,R,Min | class | sklearn | `accuracy` |
|`balanced_accuracy_score` | G,Min | class | sklearn | - |
|`precision_score`| G,Min | class | sklearn | &#x2713; |
|`recall_score`| G,Min | class | sklearn | &#x2713; |
|`f1_score`| G,Min | class | sklearn | - |
|`roc_auc_score`| G,Min | prob | sklearn | - |
|`log_loss`| G,Max | prob | sklearn | - |
|`mean_prediction`| G | prob, reg | | - |
|`mean_absolute_error` | G,D,R,Max | class, reg | sklearn | class only: `error_rate` |
|`mean_squared_error`| G,Max | prob, reg | sklearn | - |
|`r2_score`| G,Min | reg | sklearn | - |
|`_mean_overprediction` | G | class, reg | | - |
|`_mean_underprediction` | G | class, reg | | - |
|`_root_mean_squared_error`| G | prob, reg | | - |
|`_balanced_root_mean_squared_error`| G | prob | | - |

# Harmonizing metrics across sub-packages

Various sub-packages of Fairlearn refer to the related fairness concepts in many different ways. The goal of this part of the proposal is to harmonize their API with the one used in `fairlearn.metrics`.

## Notation

A _metric_ refers to any function that can be evaluated on subsets of the data. We use the notation _metric_(\*) for its value on the whole data set and _metric_(_a_) for its value on the subset of examples with sensitive feature value _a_. We write `<metric>` and `<Metric>` for literals representing the metric name.

For example, if _metric_ is _accuracy_score_, then

* _metric_(\*) = P[_Y=h(X)_]
* _metric_(_a_) = P[_Y=h(X)_ | _A=a_]
* `<metric>` = `accuracy_score`
* `<Metric>` = `AccuracyScore`

## Fairness metrics and fairness constraints

Fairness metrics are expressed in terms of _metric_(\*) and _metric_(_a_), and they are used to define fairness constraints. Mitigation algorithms also have a notion of an objective, which is typically equal to _metric_(\*).

### fairlearn.metrics

There are two kinds of fairness metrics in `fairlearn.metrics`:

* Those systematically derived from some base metrics:

  * `<metric>_difference` =
    [max<sub>_a_</sub> _metric_(_a_)] - [min<sub>_a_</sub> _metric_(_a_) ]
  * `<metric>_ratio` =
    [min<sub>_a_</sub> _metric_(_a_)] / [max<sub>_a_</sub> _metric_(_a_) ]
  * `<metric>_group_min` =
    [min<sub>_a_</sub> _metric_(_a_)]
  * `<metric>_group_max` =
    [max<sub>_a_</sub> _metric_(_a_)]

* Additional metrics that are defined in terms of the systematic functions:

  * `demographic_parity_difference` <br>
    = `selection_rate_difference`

  * `demographic_parity_ratio` <br>
    = `selection_rate_ratio`

  * `equalized_odds_difference` <br>
    = max(`true_positive_rate_difference`, `false_positive_rate_difference`)

  * `equalized_odds_ratio` <br>
    = min(`true_positive_rate_ratio`, `false_positive_rate_ratio`)

### fairlearn.postprocessing

**Status quo.** Our postprocessing algorithms use the following calling conventions:

* `constraints` is a string that represents fairness constraints as follows:

  * `"demographic_parity"`: for all _a_, <br>
    _selection_rate_(_a_) = _selection_rate_(\*).

  * `"equalized_odds"`: for all _a_,<br>
    _true_positive_rate_(_a_) = _true_positive_rate_(\*), <br>
    _false_positive_rate_(_a_) = _false_positive_rate_(\*).

* `objective` is always _accuracy_score_(\*).

**Proposal.** The following proposal is an extension of the status quo; no breaking changes are introduced:

* `constraints`: in addition to `"demographic_parity"` and `"equalized_odds"`, also allow:

  * `"<metric>_parity"`: for all _a_,<br>
    _metric_(_a_) = _metric_(\*).

* `objective` is a string of the form:

  * `"<metric>"`: <br>
    goal is then to maximize _metric_(\*) subject to constraints

### fairlearn.reductions

We support two algorithms: `ExponentiatedGradient` and `GridSearch`. They both represent `constraints` and `objective` via objects of type `Moment`.

**Status quo:**

* In both cases, `objective` is automatically inferred from the provided `constraints`.

* `ExponentiatedGradient(estimator, constraints, eps=`&epsilon;`)` considers constraints that are specified jointly by the provided `Moment` object and the numeric value &epsilon; as follows:

  * `DemographicParity()`: for all _a_, <br>
    |_selection_rate_(_a_) - _selection_rate_(\*)| &le; &epsilon;.

  * `DemographicParity(ratio=`_r_`)`: for all _a_, <br>
    _r_ &sdot; _selection_rate_(_a_) - _selection_rate_(\*) &le; &epsilon;, <br>
    _r_ &sdot; _selection_rate_(\*) - _selection_rate_(_a_) &le; &epsilon;.

  * `TruePositiveRateDifference()`: for all _a_, <br>
    |_true_positive_rate_(_a_) - _true_positive_rate_(\*)| &le; &epsilon;.

  * `TruePositiveRateDifference(ratio=`_r_`)`: analogous

  * `EqualizedOdds()`: for all _a_, <br>
    |_true_positive_rate_(_a_) - _true_positive_rate_(\*)| &le; &epsilon;, <br>
    |_false_positive_rate_(_a_) - _false_positive_rate_(\*)| &le; &epsilon;.

  * `EqualizedOdds(ratio=`_r_`)`: analogous

  * `ErrorRateRatio()`: for all _a_, <br>
    |_error_rate_(_a_) - _error_rate_(\*)| &le; &epsilon;.

  * `ErrorRateDifference(ratio=`_r_`)`: analogous

  * all of the above constraints are descendants of `ConditionalSelectionRate`

  * the `objective` for all of the above constraints is `ErrorRate()`

* `GridSearch(estimator, constraints)` considers constraints represented by the provided `Moment` object. The behavior of the `GridSearch` algorithm does not depend on the value of the right-hand side of constraints, so it is not provided to the constructor. In addition to the `Moment` objects above, `GridSearch` also supports the following:

  * `GroupLossMoment(<loss>)`: for all _a_, <br>
    _loss_(_a_) &le; &zeta;

    * where `<loss>` is a _loss evaluation_ object; it supports an interface that takes `y_true` and `y_pred` as the input and returns the vector of losses evaluated on individual examples

    * the `objective` for `GroupLossMoment(<loss>)` is `AverageLossMoment(<loss>)`

  * both `GroupLossMoment` and `AverageLossMoment` are descendants of `ConditionalLossMoment`

**Proposal.** The proposal introduces some breaking changes:

* `constraints`:

  * `<Metric>Parity(difference_bound=`&epsilon;`)`: for all _a_, <br>
    |_metric_(_a_) - _metric_(\*)| &le; &epsilon;.

  * `<Metric>Parity(ratio_bound=`_r_`, ratio_bound_slack=`&epsilon;`)`: for all _a_, <br>
    _r_ &sdot; _metric_(_a_) - _metric_(\*) &le; &epsilon;, <br>
    _r_ &sdot; _metric_(\*) - _metric_(_a_) &le; &epsilon;.

  * `DemographicParity` and `EqualizedOdds` have the same calling convention as `<Metric>Parity`

  * `BoundedGroupLoss(<loss>, bound=`&zeta;`)`: for all _a_, <br>
    _loss_(_a_) &le; &zeta;

    _Alternative proposals_:
    * `LossParity(<loss>, group_max_bound=`&zeta;`)` <br>
    * `<Loss>Parity(group_max_bound=`&zeta;`)`


* `objective`:
  * `<Metric>()` (for classification moments)

  * `AverageLoss(<loss>)` (for loss minimization moments) <br>

    _Alternative proposals_:
    * `MeanLoss(<loss>)`
    * `OverallLoss(<loss>)`
    * the object `<loss>` doubles as (1) the loss evaluator and (2) the moment implementing the objective

* the loss evaluation object `<loss>` needs to support the following API:

  *  `<loss>(y_true, y_pred)` returning the vector of losses on each example
  *  `<loss>.min_loss` the minimum value the loss evaluates to
  *  `<loss>.max_loss` the maximum value the loss evaluates to
