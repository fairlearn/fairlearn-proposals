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

At this basic level, there is only a slight change to the results seen by the user.
There are still properties `overall` and `by_groups`, with the same semantics.
However, the `by_groups` result is now a Pandas Series, and we also provide a `metric` property to record the name of the underlying metric:
```python
>>> result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_sex)
>>> result.metric
"sklearn.metrics.accuracy_score"
>>> result.overall
0.4
>>> result.by_groups
Male      0.6536
Female    0.2130
Name: sklearn.metrics.accuracy_score dtype: float64
>>> print(type(result.by_groups))
<class 'pandas.core.series.Series'>
```
We would continue to provide convenience wrappers such as `accuracy_score_group_summary` for users, and support passing through arguments along with `indexed_params`.
There is little advantage to the change at this point.
This will change in the next section.

## Obtaining Scalars

### Existing Syntax

We provide methods for turning the `Bunch`es returned from `group_summary()` into scalars:
```python
>>> difference_from_summary(result)
0.4406
>>> ratio_from_summary(result)
0.3259
>>> group_max_from_summary(result)
0.6536
>>> group_min_from_summary(result)
0.2130
```
We also provide wrappers such as `accuracy_score_difference()`, `accuracy_score_ratio()` and `accuracy_score_min()` for user convenience.

One point which these helpers lack (although it could be added) is the ability to select alternative values for measuring the difference and ratio.
For example, the user might not be interested in the difference between the maximum and minimum, but the difference from the overall value.
Or perhaps the difference from a particular group.

### Proposed Change

The `GroupedMetric` object would have methods for calculating the required scalars.
First, let us consider the differences.

We would provide operations to calculate differences in various ways (all of these results are a Pandas Series):
```python
>>> result.differences()
Male      0.0
Female    0.4406
Name: TBD dtype: float64
>>> result.differences(relative_to='min')
Male     -0.4406
Female    0.0
Name: TBD dtype: float64
>>> result.differences(relative_to='min', abs=True)
Male      0.4406
Female    0.0
Name: TBD dtype: float64
>>> result.differences(relative_to='overall')
Male     -0.2436
Female    0.1870
Name: TBD dtype: float64
>>> result.differences(relative_to='overall', abs=True)
Male      0.2436
Female    0.1870
Name: TBD dtype: float64
>>> result.differences(relative_to='group', group='Female', abs=True)
Male      0.4406
Female    0.0
Name: TBD dtype: float64
```
The arguments introduced so far for the `differences()` method:
- `relative_to=` to decide the common point for the differences. Possible values are `'max'` (the default), `'min'`, `'overall'` and `'group'`
- `group=` to select a group name, only when `relative_to` is set to `'group'`. Default is `None`
- `abs` to indicate whether to take the absolute value of each entry (defaults to false)

The user could then use the Pandas methods `max()` and `min()` to reduce these Series objects to scalars.
However, this will run into issues where the `relative_to` argument ends up pointing to either the maximum or minimum group, which will have a difference of zero.
That could then be the maximum or minimum value of the set of difference, but probably won't be what the user wants.

To address this case, we should add an extra argument `aggregate` to the `differences()` method:
```python
>>> result.differences(aggregate='max')
0.4406
>>> result.differences(relative_to='overall', aggregate='max')
0.1870
>>> result.differences(relative_to='overall', abs=True, aggregate='max')
0.2436
```

There would be a similar method called `ratios()` on the `GroupedMetric` object:
```python
>>> result.ratios()
Male      1.0
Female    0.3259
Name: TBD dtype: float64
```
The `ratios()` method will take the following arguments:
- `relative_to=` similar to `differences()`
- `group=` similar to `differences()`
- `ratio_order=` determines how to build the ratio. Values are
   - `sub_unity` to make larger value the denominator
   - `super_unity` to make larger value the numerator
   - `from_relative` to make the value specified by `relative_to=` the denominator
   - `to_relative` to make the value specified by `relative_to=` the numerator
- `aggregate=` similar to `differences()`

## Intersections of Sensitive Features

### Existing Syntax

Our current API does not support evaluating metrics on intersections of sensitive features (e.g. "black and female", "black and male", "white and female", "white and male").
To achieve this, users currently need to write something along the lines of:
```python
>>> A_combined = A['Sex'] + '-' + A['Race']

>>> accuracy_score_group_summary(y_true, y_pred, sensitive_features=A_combined)
{ 'overall': 0.4, by_groups : { 'Female-Black':0.4, 'Female-Hispanic':0.5, 'Female-White':0.5, 'Male-Black':0.5, 'Male-Hispanic': 0.6, 'Male-White':0.7 } }
```
This is unecessarily cumbersome.
It is also possible that some combinations might not appear in the data (especially as more sensitive features are combined), but identifying which ones were missing would be tedious.


### Proposed Change

If `sensitive_features=` is a DataFrame, we can generate our results in terms of a MultiIndex:
```python
>>> result = group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A)
>>> result.by_groups
Sex     Race
Male    Black       0.5
        White       0.7
        Hispanic    0.6
Female  Black       0.4
        White       0.5
        Hispanic    0.5
Name: sklearn.metrics.accuracy_score, dtype: float64
```
If a particular combination of sensitive features had no representatives, then we would return `None` for that entry in the Series.

The `differences()` and `ratio()` methods would act on this Series as before.

## Segmented (or Conditional) Metrics

For our purposes, Segmented Metrics (alternatively known as Conditional Metrics) do not return single values when aggregation is requested in a call to `differences()` or `ratios()` but instead provide one result for each unique value of the specified segmentation feature(s).

### Existing Syntax

Not supported.
Users would have to devise the required code themselves

### Proposed Change

We propose adding an extra argument to `differences()` and `ratios()`, to provide a `segment_by=` argument.

Suppose we have a DataFrame, `A_3` with three sensitive features: Sex, Race and Income Band (the latter having values 'Low' and 'High').
This could represent a loan scenario where discrimination based on income is allowed, but within the income bands, other sensitive groups must be treated equally.
When `differences()` is invoked with `segment_by=`, the result will not be a scalar, but a Series.
A user might make calls:
```python
>>> result = accuracy_score_group_summary(y_true, y_test, sensitive_features=A_3)
>>> result.differences(aggregate=min, segment_by='Income Band')
Income Band
Low                 0.3
High                0.4
Name: TBD, dtype: float64
```
We can also allow `segment_by=` to be a list of names:
```python
>>> result.differences(aggregate=min, segment_by=['Income Band', 'Sex'])
Income Band     Sex
Low             Female  0.3
Low             Male    0.35
High            Female  0.4
High            Male    0.5
```

## Multiple Metrics

Finally, we can also allow for the evaluation of multiple metrics at once.

### Existing Syntax

This is not supported.
Users would have to devise their own method

### Proposed Change

We allow a list of metric functions in the call to group summary.
Results become DataFrames, with one column for each metric:
```python
>>> result = group_summary([skm.accuracy_score, skm.precision_score], y_true, y_pred, sensitive_features=A_sex)
>>> result.overall
      sklearn.metrics.accuracy_score  sklearn.metrics.precision_score
   0                             0.3                              0.5
>>> result.by_groups
      sklearn.metrics.accuracy_score  sklearn.metrics.precision_score
'Female'                        0.4                            0.7
'Male'                          0.6                            0.75
```
This should generalise to the other methods described above.

One open question is how extra arguments should be passed to the individual metric functions, including how to handle the `indexed_params=`.