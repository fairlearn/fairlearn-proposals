# Updates for Metrics

This is an update for the existing metrics document, which is being left in place for now as a point of comparison.

## Assumed data

In the following we assume that we have variables of the following form defined:

```python
y_true = [0, 1, 0, 0, 1, 1, ...]
y_pred = [1, 0, 0, 1, 0, 1, ...] # Content can be different for other metrics (see below)
A_1 = [ 'C', 'B', 'B', 'C', ...]
A_2 = [ 'M', 'N', 'N', 'P', ...]
A = pd.DataFrame(np.transpose([A_1, A_2]), columns=['SF 1', 'SF 2'])

weights = [ 1, 2, 3, 2, 2, 1, ...]
```

We actually seek to be very agnostic as to the contents of the `y_true` and `y_pred` arrays; the meaning is imposed on them by the underlying metrics.
Here we have shown binary values for a simple classification problem, but they could be floating point values from a regression, or even collections of classes and associated probabilities.

## Basic Calls

### Existing Syntax

Our basic method is `group_summary()`

```python
>>> result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_1)
>>> print(result)
{'overall': 0.4, 'by_group': {'B': 0.6536, 'C': 0.213}}
>>> print(type(result))
<class 'sklearn.utils.Bunch'>
```
The `Bunch` is an object which can be accessed in two ways - either as a dictionary - `result['overall']` -  or via properties named by the dictionary keys - `result.overall`.
Note that the `by_group` key accesses another `Bunch`.

We allow for sample weights (and other arguments which require slicing) via `indexed_params`, and passing through other arguments to the underlying metric function (in this case, `normalize`):
```python
>>> flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_1, indexed_params=['sample_weight'], sample_weight=weights, normalize=False)
{'overall': 20, 'by_group': {'B': 60, 'C': 21}}
```

We also provide some wrappers for common metrics from SciKit-Learn:
```python
>>> flm.accuracy_score_group_summary(y_true, y_pred, sensitive_features=A_1)
{'overall': 0.4, 'by_group': {'B': 0.6536, 'C': 0.213}}
```

### Proposed Change

We do not intend to change the API invoked by the user.
What will change is the return type.
Rather than a `Bunch`, we will return a `GroupedMetric` object, which can offer richer functionality.

At this basic level, there is only a slight change to the results seen by the user.
There are still properties `overall` and `by_groups`, with the same semantics.
However, the `by_groups` result is now a Pandas Series, and we also provide a `metric` property to record the name of the underlying metric:
```python
>>> result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_1)
>>> result.metric
"sklearn.metrics.accuracy_score"
>>> result.overall
0.4
>>> result.by_groups
B      0.6536
C      0.2130
Name: sklearn.metrics.accuracy_score dtype: float64
>>> print(type(result.by_groups))
<class 'pandas.core.series.Series'>
```
The `metric` property (and using it as the name of the Series) could prove troublesome.
This is because the fully qualified function name, as reconstructed from `__name__`, `__qualname` and `__module__` might not match the user's expectation.
For example:
```python
>>> import sklearn.metrics as skm
>>> skm.accuracy_score.__name__
'accuracy_score'
>>> skm.accuracy_score.__qualname__
'accuracy_score'
>>> skm.accuracy_score.__module__
'sklearn.metrics._classification'
```
We are seeing some of the actual internal structure here of SciKit-Learn, and the user might not be expecting that.

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
B      0.0
C      0.4406
Name: TBD dtype: float64
>>> result.differences(relative_to='min')
B     -0.4406
C      0.0
Name: TBD dtype: float64
>>> result.differences(relative_to='min', abs=True)
B      0.4406
C      0.0
Name: TBD dtype: float64
>>> result.differences(relative_to='overall')
B     -0.2436
C      0.1870
Name: TBD dtype: float64
>>> result.differences(relative_to='overall', abs=True)
B      0.2436
C      0.1870
Name: TBD dtype: float64
>>> result.differences(relative_to='group', group='C', abs=True)
B      0.4406
C      0.0
Name: TBD dtype: float64
```
The arguments introduced so far for the `differences()` method:
- `relative_to=` to decide the common point for the differences. Possible values are `'max'` (the default), `'min'`, `'overall'` and `'group'`
- `group=` to select a group name, only when `relative_to` is set to `'group'`. Default is `None`
- `abs` to indicate whether to take the absolute value of each entry (defaults to false)

The user could then use the Pandas methods `max()` and `min()` to reduce these Series objects to scalars.
However, this will run into issues where the `relative_to` argument ends up pointing to either the maximum or minimum group, which will have a difference of zero.
That could then be the maximum or minimum value of the set of difference, but probably won't be what the user wants.

To address this case, we should add an extra argument `aggregate=` to the `differences()` method:
```python
>>> result.differences(aggregate='max')
0.4406
>>> result.differences(relative_to='overall', aggregate='max')
0.1870
>>> result.differences(relative_to='overall', abs=True, aggregate='max')
0.2436
```
If `aggregate=None` (which would be the default), then the result is a Series, as shown above.

There would be a similar method called `ratios()` on the `GroupedMetric` object:
```python
>>> result.ratios()
B      1.0
C      0.3259
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

We would also provide the same wrappers such as `accuracy_score_difference()` but expose the extra arguments discussed here.
One question is whether the default aggregation should be `None` (to match the method), or whether it should default to scalar results similar to the existing methods. 

In the section on Conditional Metrics below, we shall discuss one extra optional argument for `differences()` and `ratios()`.

## Intersections of Sensitive Features

### Existing Syntax

Our current API does not support evaluating metrics on intersections of sensitive features (e.g. "black and female", "black and male", "white and female", "white and male").
To achieve this, users currently need to write something along the lines of:
```python
>>> A_combined = A['SF 1'] + '-' + A['SF 2']

>>> accuracy_score_group_summary(y_true, y_pred, sensitive_features=A_combined)
{ 'overall': 0.4, by_groups : { 'B-M':0.4, 'B-N':0.5, 'B-P':0.5, 'C-M':0.5, 'C-N': 0.6, 'C-P':0.7 } }
```
This is unecessarily cumbersome.
It is also possible that some combinations might not appear in the data (especially as more sensitive features are combined), but identifying which ones were not represented in the dataset would be tedious.


### Proposed Change

If `sensitive_features=` is a DataFrame (or list of Series.... exact supported types are TBD), we can generate our results in terms of a MultiIndex. Using the `A` DataFrame defined above, a user might write:
```python
>>> result = group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A)
>>> result.by_groups
SF 1    SF 2
B       M       0.5
        N       0.7
        P       0.6
C       M       0.4
        N       0.5
        P       0.5
Name: sklearn.metrics.accuracy_score, dtype: float64
```
If a particular combination of sensitive features had no representatives, then we would return `None` for that entry in the Series.

The `differences()` and `ratio()` methods would act on this Series as before.

## Conditional (or Segmented) Metrics

For our purposes, Conditional Metrics (alternatively known as Segmented Metrics) do not return single values when aggregation is requested in a call to `differences()` or `ratios()` but instead provide one result for each unique value of the specified condition feature(s).

### Existing Syntax

Not supported.
Users would have to devise the required code themselves

### Proposed Change

We propose adding an extra argument to `differences()` and `ratios()`, to provide a `condition_on=` argument.

Suppose we have a DataFrame, `A_3` with three sensitive features: SF 1, SF 2 and Income Band (the latter having values 'Low' and 'High').
This could represent a loan scenario where discrimination based on income is allowed, but within the income bands, other sensitive groups must be treated equally.
When `differences()` is invoked with `condition_on=`, the result will not be a scalar, but a Series.
A user might make calls:
```python
>>> result = accuracy_score_group_summary(y_true, y_test, sensitive_features=A_3)
>>> result.differences(aggregate=min, condition_on='Income Band')
Income Band
Low                 0.3
High                0.4
Name: TBD, dtype: float64
```
We can also allow `condition_on=` to be a list of names:
```python
>>> result.differences(aggregate=min, condition_on=['Income Band', 'Sex'])
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
On possible solution is to have lists, with indices corresponding to the list of functions supplied to `group_summary()`
For example, for `index_params=` we would have:
```python
indexed_params = [['sample_weight'], ['sample_weight']]
```
In the `**kwargs` a single `extra_args=` argument would be accepted (although not required), which would contain the individual `**kwargs` for each metric:
```python
extra_args = [ 
    {
        'sample_weight': [1,2,1,1,3, ...],
        'normalize': False
    },
    {
        'sample_weight': [1,2,1,1,3, ... ],
        'pos_label' = 'Granted'
    }
]
```
If users had a lot of functions with a lot of custom arguments, this could get error-prone and difficult to debug.

## User Conveniences

We can consider allowing the metric function given to `group_summary()` to be represented by a string.
We would provide a mapping of strings to suitable functions.
This would make the following all equivalent:
```python
>>> r1 = group_summary(sklearn.accuracy_score, y_true, y_pred, sensitive_features=A_1)
>>> r2 = group_summary('accuracy_score', y_true, y_pred, sensitive_features=A_1)
>>> r3 = accuracy_score_group_summary( y_true, y_pred, sensitive_features=A_1)
```
We would also allow mixtures of strings and functions in the multiple metric case.

## Generality

Throughout this document, we have been describing the case of classification metrics.
However, we do not actually require this.
It is the underlying metric function which gives meaning to the `y_true` and `y_pred` lists.
So long as these are of equal length (and equal in length to the sensitive feature list - which _will_ be treated as a categorical), then `group_summary()` does not actually care about their datatypes.
For example, each entry in `y_pred` could be a dictionary of predicted classes and accompanying probabilities.
Or the user might be working on a regression problem, and both `y_true` and `y_pred` would be floating point numbers (or `y_pred` might even be a tuple of predicted value and error).
So long as the underlying metric understands the datastructures, `group_summary()` will not care.

There will be an effect on the `GroupedMetric` result object.
Although the `overall` and `by_groups` properties will work fine, the `differences()` and `ratios()` methods may not.
After all, what does "take the ratio of two confusion matrices" even mean?
We should try to trap these cases, and throw a meaningful exception (rather than propagating whatever exception happens to emerge from the underlying libraries).
Since we know that `differences()` and `ratios()` will only work when the metric has produced scalar results, this should be a straightforward test.

## Pitfalls

There are some potential pitfalls which could trap the unwary.

The biggest of these are related to missing classes in the subgroups.
To take an extreme case, suppose that the `B` group speciified by `SF 1` were always being predicted classes H or J, while the `C` group was always predicted classes K or L.
The user could request precision scores, but the results would not really be comparable between the two groups.
With intersections of sensitive features, cases like this become more likely.

Metrics in SciKit-Learn usually have arguments such as `pos_label=` and `labels=` to allow the user to specify the expected labels, and adjust their behaviour accordingly.
However, we do not require that users stick to the metrics defined in SciKit-Learn.

If we implement the convenience strings-for-functions piece mentioned above, then _when the user specifies one of those strings_ we can log warnings if the appropriate arguments (such as `labels=`) are not specified.
We could even generate the argument ourselves if the user does not specify it.
However, this risks tying Fairlearn to particular versions of SciKit-Learn.

Unfortunately, the generality of `group_summary()` means that we cannot solve this for the user.
It cannot even tell if it is evaluating a classification or regression problem.

## The Wrapper Functions

In the above, we have assumed that we will provide both `group_summary()` and wrappers such as `accuracy_score_group_summary()`, `accuracy_score_difference()`, `accuracy_score_ratio()` and `accuracys_score_group_min()`. Do these wrappers add value, or do they end up just polluting our namespace and confusing users?

The wrappers such as `demographic_parity_difference()` and `equalized_odds_difference()` are arguably useful, since they are specific metrics used in the literature (although even then we might want to add the extra `relative_to=` and `group=` arguments).
The case for `accuracy_score_group_summary()` and related functions is less clear.

## Methods or Functions

Since the `GroupMetric` object contains no private members, it is not clear that it needs to be its own oject.
We could continue to use a `Bunch` but make the `group_by` entry/property return a Pandas Series (which would embed all the other information we might need).
In the multiple metric case, we would still return a single `Bunch` but the properties would both be DataFrames.

The question is whether users prefer:
```python
>>> diff = group_summary(skm.recall_score, y_true, y_pred, sensitive_features=A).difference(aggregate='max')
```
or
```python
>>> diff = difference(group_summary(skm.recall_score, y_true, y_pred, sensitive_features=A), aggregate='max')
```
