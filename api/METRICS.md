# API proposal for metrics

## Goals

1. Metrics should have simple functional signatures that can be passed to `make_scorer`.

1. Metrics that return structured objects&mdash;such as evaluating accuracy in each group&mdash;should be easy to turn into a `DataFrame` or `Series`.

1. It should be possible to implement caching of intermediate results (at some point in future). For example, many metrics can be derived from the set of group-level metric values across all groups, so it would be nice to avoid re-calculating the group summaries.

1. Metrics that are derived from existing `sklearn` metrics should be recognizable.

## Proposal in the form of an example

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

## Proposal details

The items that are not implemented yet are marked as `[TODO]`.

### Metrics engine

```python
group_summary(metric, y_true, y_pred, *, sensitive_features, **other_kwargs)
# return the group summary for the provided `metric`, where `metric` has the signature
# metric(y_true, y_pred, **other_kwargs)

make_group_summary(metric) [TODO]
# return a callable object <metric>_group_summary:
# <metric>_group_summary(...) = group_summary(metric, ...)

# Transformation functions returning scalars (the definitions are below)
difference_from_summary(summary)
ratio_from_summary(summary)
group_min_from_summary(summary)
group_max_from_summary(summary)

derived_metric(metric, transformation, y_true, y_pred, *, sensitive_features, **other_kwargs) [TODO]
# return the value of a metric derived from the provided `metric`, where `metric`
# has the signature
#   * metric(y_true, y_pred, **other_kwargs)
# and `transformation` is a string 'difference', 'ratio', 'group_min' or 'group_max'.
#
# Alternatively, `metric` can be a metric group summary function, and `transformation`
# can be a function one of the functions <transformation>_from_summary.

make_derived_metric(metric, transformation) [TODO]
# return a callable object <metric>_<transformation>:
# <metric>_<transformation>(...) = derived_metric(metric, transformation, ...)

# Predefined metrics are named according to the following patterns:
<metric>_group_summary(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_difference(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_ratio(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_min(y_true, y_pred, *, sensitive_features, **other_kwargs)
<metric>_group_max(y_true, y_pred, *, sensitive_features, **other_kwargs)
```

### Definitions of transformation functions

|transformation function|output|derived metric name|code|aif360|
|-----------------------|------|------------------------|----|------|
|`difference_from_summary`|max - min|`<metric>_difference`|D|unprivileged - privileged|
|`ratio_from_summary`|min / max|`<metric>_ratio`|R| unprivileged / privileged|
|`group_min_from_summary`|min|`<metric>_group_min`|Min| N/A |
|`group_max_from_summary`|max|`<metric>_group_max`|Max| N/A |

> _Note_: The ratio min/max should evaluate to `np.nan` if min<0.0, and to 1.0 if min=max=0.0.

### List of predefined metrics

* In the list of predefined metrics, we refer to the following machine learning tasks:

  |task|definition|code|
  |----|----------|----|
  |binary classification|labels and predictions are in {0,1}|class|
  |probabilistic binary classification|labels are in {0,1}, predictions are in [0,1] and correspond to estimates of P(y\|x)|prob|
  |randomized binary classification|labels are in {0,1}, predictions are in [0,1] and represent the probability of drawing y=1 in a randomized decision|class-rand|
  |regression|labels and predictions are real-valued|reg|

* For each _base metric_, we provide the list of predefined derived metrics, using D, R, Min, Max to refer to the transformations from the table above, and G to refer to `<metric>_group_summary`. We follow these rules:
  * always provide G (except for demographic parity and equalized odds, which do not make sense as group-level metrics)
  * provide D and R for confusion-matrix-derived metrics
  * provide Min for score functions (worst-case score)
  * provide Max for error/loss functions (worst-case error)
  * for internal API metrics starting with `_`, only provide G


|metric|variants|task|notes|aif360|
|------|--------|-----|----|------|
|`confusion_matrix` | G | class | sklearn | `binary_confusion_matrix` |
|`false_positive_rate` | G,D,R | class | | &#x2713; |
|`false_negative_rate` | G,D,R | class | | &#x2713; |
|`true_positive_rate` | G,D,R | class | | &#x2713; |
|`true_negative_rate` | G,D,R | class | | &#x2713; |
|`selection_rate`| G,D,R | class | | &#x2713; |
|`demographic_parity`| D,R | class | `selection_rate_difference`, `selection_rate_ratio` | `statistical_parity_difference`, `disparate_impact`|
|`equalized_odds` | D,R | class | max difference or min ratio under `true_positive_rate`, `false_positive_rate` | - |
|`accuracy_score`| G,D,R,Min | class | sklearn | `accuracy` |
|`zero_one_loss` | G,D,R,Max | class | sklearn | `error_rate` |
|`balanced_accuracy_score` | G,Min | class | sklearn | - |
|`precision_score`| G,Min | class | sklearn | &#x2713; |
|`recall_score`| G,Min | class | sklearn | &#x2713; |
|`f1_score` [TODO]| G,Min | class | sklearn | - |
|`roc_auc_score`| G,Min | prob | sklearn | - |
|`log_loss` [TODO]| G,Max | prob | sklearn | - |
|`mean_absolute_error` | G,Max | reg | sklearn | - |
|`mean_squared_error`| G,Max | prob, reg | sklearn | - |
|`r2_score`| G,Min | reg | sklearn | - |
|`mean_prediction`| G | prob, reg | | - |
|`_mean_overprediction` | G | class, reg | | - |
|`_mean_underprediction` | G | class, reg | | - |
|`_root_mean_squared_error`| G | prob, reg | | - |
|`_balanced_root_mean_squared_error`| G | prob | | - |

# Harmonizing metrics across modules

Various modules of Fairlearn refer to the related fairness concepts in many different ways. The goal of this part of the proposal is to harmonize their API with the one used in `fairlearn.metrics`.

## Notation

A _metric_ refers to any function that can be evaluated on subsets of the data. We use the notation _metric_(\*) for its value on the whole data set and _metric_(_a_) for its value on the subset of examples with sensitive feature value _a_. We write `<metric>` and `<Metric>` for literals representing the metric name.

For example, if _metric_ is _accuracy_score_, then

* _metric_(\*) = P[_Y=h(X)_]
* _metric_(_a_) = P[_Y=h(X)_ | _A=a_]
* `<metric>` = `accuracy_score`
* `<Metric>` = `AccuracyScore`

## Fairness metrics and fairness constraints

Fairness metrics are expressed in terms of _metric_(\*) and _metric_(_a_). Mitigation algorithms have a notion of _fairness constraints_, expressed in terms of fairness metrics, and a notion of an _objective_, typically equal to _metric_(\*).

### fairlearn.metrics

There are two kinds of fairness metrics in `fairlearn.metrics`:

* Fairness metrics derived from base metrics:

  * `<metric>_difference` =
    [max<sub>_a_</sub> _metric_(_a_)] - [min<sub>_a'_</sub> _metric_(_a'_) ]
  * `<metric>_ratio` =
    [min<sub>_a_</sub> _metric_(_a_)] / [max<sub>_a'_</sub> _metric_(_a'_) ]
  * `<metric>_group_min` =
    [min<sub>_a_</sub> _metric_(_a_)]
  * `<metric>_group_max` =
    [max<sub>_a_</sub> _metric_(_a_)]

* Other fairness metrics:

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

### fairlearn.reductions [TODO]

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

  * `ErrorRateRatio(ratio=`_r_`)`: analogous

  * all of the above constraints are descendants of `ConditionalSelectionRate`

  * the `objective` for all of the above constraints is `ErrorRate()`

* `GridSearch(estimator, constraints)` considers constraints represented by the provided `Moment` object. The behavior of the `GridSearch` algorithm does not depend on the value of the right-hand side of the constraints, so it is not provided to the constructor. In addition to the `Moment` objects above, `GridSearch` also supports the following:

  * `GroupLossMoment(<loss>)`: for all _a_, <br>
    _loss_(_a_) &le; &zeta;

    * where `<loss>` is a _loss evaluation_ object; it supports an interface that takes `y_true` and `y_pred` as the input and returns the vector of losses evaluated on individual examples

    * the `objective` for `GroupLossMoment(<loss>)` is `AverageLossMoment(<loss>)`

  * both `GroupLossMoment` and `AverageLossMoment` are descendants of `ConditionalLossMoment`

**Proposal.** The proposal introduces many breaking changes:

* `constraints`:

  * `<Metric>Parity(difference_bound=`&epsilon;`)`: for all _a_, <br>
    |_metric_(_a_) - _metric_(\*)| &le; &epsilon;.

  * `<Metric>Parity(ratio_bound=`_r_`, ratio_bound_slack=`&epsilon;`)`: for all _a_, <br>
    _r_ &sdot; _metric_(_a_) - _metric_(\*) &le; &epsilon;, <br>
    _r_ &sdot; _metric_(\*) - _metric_(_a_) &le; &epsilon;.

  * `DemographicParity` and `EqualizedOdds` have the same calling convention as `<Metric>Parity`

  * rename `ConditionalSelectionRate` to `UtilityParity`

  * `BoundedGroupLoss(<loss>, upper_bound=`&zeta;`)`: for all _a_, <br>
    _loss_(_a_) &le; &zeta;

  * remove `ConditionalLossMoment`


* `objective`:
  * `<Metric>()` (for classification moments)

  * `MeanLoss(<loss>)` (for loss minimization moments) <br>

* the loss evaluator object `<loss>` needs to support the following API:

  * `<loss>(y_true, y_pred)` returning the vector of losses on each example
  * `<loss>.min_loss` the minimum value the loss evaluates to
  * `<loss>.max_loss` the maximum value the loss evaluates to

  _Constructors_:

  * `SquareLoss(max_loss=...)`
  * `AbsoluteLoss(max_loss=...)`
  * `LogLoss(max_loss=...)`
  * `ZeroOneLoss()`
