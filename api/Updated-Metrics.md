# Updates for Metrics

This is an update for the existing metrics document, which is being left in place for now as a point of comparison.

## Assumed data

In the following we assume that we have variables of the following form defined:

```python
y_true = [0, 1, 0, 0, 1, 1, ...]
y_pred = [1, 0, 0, 1, 0, 1, ...]
A_sex = [ 'male', 'female', 'female', 'male', ...]
A_race = [ 'black', 'white', 'hispanic', 'black', ...]
A = pd.DataFrame(np.transpose([A_sex, A_race]), columns=['Sex', 'Race'])

weights = [ 1, 2, 3, 2, 2, 1, ...]
```

We actually seek to be very agnostic as to the contents of the `y_true` and `y_pred` arrays.
Meaning is imposed on them by the underlying metrics.

## Basic Calls

### Existing Syntax

Our basic method is `group_summary()`

```python
>>> result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_sex)
>>> print(result)
{'overall': 0.4, 'by_group': {'male': 0.6536, 'female': 0.213}}
>>> print(type(result))
<class 'sklearn.utils.Bunch'>
```
The `Bunch` is an object which can be accessed in two ways - either as a dictionary - `result['overall']` -  or via properties named by the dictionary keys - `result.overall`.
Note that the `by_group` key accesses another `Bunch`.

We allow for sample weights (and other arguments which require slicing) via `indexed_params`, and passing through other arguments to the underlying metric function (in this case, `normalize`):
```python
>>> flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_sex, indexed_params=['sample_weight'], sample_weight=weights, normalize=False)
{'overall': 20, 'by_group': {'male': 60, 'female': 21}}
```

We also provide some wrappers for common metrics from SciKit-Learn:
```python
>>> flm.accuracy_score_group_summary(y_true, y_pred, sensitive_features=A_sex)
{'overall': 0.4, 'by_group': {'male': 0.6536, 'female': 0.213}}
```

### Proposed Change

We do not intend to change the API invoked by the user.
What will change is the return type.
Rather than a `Bunch`, we will return a `GroupedMetric` object, which can offer richer functionality.

At this basic level, there is only a slight change to the results seen by the user. There are still properties `overall` and `by_groups`, with the same semantics.
However, the `by_groups` result is now a Pandas Series:
```python
>>> result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_sex)
>>> result.overall
0.4
>>> result.by_groups
Male      0.6536
Female    0.2130
dtype: float64
>>> print(type(result.by_groups))
<class 'pandas.core.series.Series'>
```
We would continue to provide convenience wrappers such as `accuracy_score_group_summary` for users, and support passing through arguments along with `indexed_params`.
There is little advantage to the change at this point.
This will change in the next section.