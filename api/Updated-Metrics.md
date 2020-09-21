# Updates for Metrics

This is an update for the existing metrics document, which is being left in place for now as a point of comparison.

We are having meetings to discuss this proposal.
Please reach out to `riedgar@microsoft.com` if you would like to join.

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
>>> result = flm.group_summary(skm.accuracy_score,
                               y_true, y_pred,
                               sensitive_features=A_1)
>>> print(result)
{'overall': 0.4, 'by_group': {'B': 0.6536, 'C': 0.213}}
>>> print(type(result))
<class 'sklearn.utils.Bunch'>
```
The `Bunch` is an object which can be accessed in two ways - either as a dictionary - `result['overall']` -  or via properties named by the dictionary keys - `result.overall`.
Note that the `by_group` key accesses another `Bunch`.

We allow for sample weights (and other arguments which require slicing) via `indexed_params`, and passing through other arguments to the underlying metric function (in this case, `normalize`):
```python
>>> flm.group_summary(skm.accuracy_score,
                      y_true, y_pred,
                      sensitive_features=A_1,
                      indexed_params=['sample_weight'],
                      sample_weight=weights, normalize=False)
{'overall': 20, 'by_group': {'B': 60, 'C': 21}}
```

We also provide some wrappers for common metrics from SciKit-Learn:
```python
>>> flm.accuracy_score_group_summary(y_true, y_pred, 
                                     sensitive_features=A_1)
{'overall': 0.4, 'by_group': {'B': 0.6536, 'C': 0.213}}
```

### Proposed Change

We propose to introduce a new object, the `GroupedMetric` (name discussion below).
Users will compute metrics by passing arguments into the constructor:
```python
>>> metrics = GroupedMetrics(skm.accuracy_score,
                             y_true, y_pred,
                             sensitive_features=A_1)
<class 'GroupedMetric'>
>>> metrics.overall
          accuracy_score
overall   0.4
>>> metrics.by_group
   accuracy_score
B             0.6536
C             0.213
```
Both properties are now Pandas DataFrames, with the column name set to the `__name__` property of the given metric function.
The rows of the `by_group` property are set to the unique values of the `sensitive_feature=` argument.

Extra parameters for the metric function are passed in via the `sample_params=` and `params=` arguments.
The `params=` arguments are broadcast, while the `sample_params=` arguments will be sliced along with the `y_true` and `y_pred` arguments in accordance with the sensitive feature vector.
```python
>>> metrics = GroupedMetrics(skm.accuracy_score,
                             y_true, y_pred,
                             sensitive_features=A_1,
                             sample_params={'sample_weight': weight},
                             params={'normalize': False})
```
A key which appears in both `sample_params=` and `params=` will be an error.
We would _not_ provide the basic wrappers such as `accuracy_score_group_summary()`.

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

### Proposed Change

Although the functionality of the `group_max_from_summary()` and `group_min_from_summary()` can be accessed by calling `metrics.by_group.min()` and `metrics.by_group.max()`, we will provide `.group_min()` and `.group_max()` methods for completeness.

For differences and ratios, we will provide `.difference()` and `.ratio()` methods.
These will take an optional argument of `method=`, to indicate how the values are to be calculated (defaulting to `minmax`)
First for computing the difference:
```python
>>> metrics.difference()
      accuracy_score
all   0.4406
dtype: float64
>>> metrics.difference(method='to_overall')
accuracy_score   0.2563 # max(abs(0.6536-0.4), abs(0.213-0.4))
dtype: float64
```
Note that the result type is a Series (for reasons which will become clear below).
The `ratio()` method would behave in a similar way:
```python
>>> metrics.ratio()
        accuracy_score
all     0.3259
>>> metrics.ratio(method='to_overall')
       accuracy_score
all    0.6120 # min(abs(0.4/0.6536), abs(0.213/0.6536))
```

## Intersections of Sensitive Features

### Existing Syntax

Our current API does not support evaluating metrics on intersections of sensitive features (e.g. "black and female", "black and male", "white and female", "white and male").
To achieve this, users currently need to write something along the lines of:
```python
>>> A_combined = A['SF 1'] + '-' + A['SF 2']

>>> accuracy_score_group_summary(y_true, y_pred,
                                 sensitive_features=A_combined)
{ 'overall': 0.4, by_groups : { 'B-M':0.4, 'B-N':0.5, 'B-P':0.5, 'C-M':0.5, 'C-N': 0.6, 'C-P':0.7 } }
```
This is unecessarily cumbersome.
It is also possible that some combinations might not appear in the data (especially as more sensitive features are combined), but identifying which ones were not represented in the dataset would be tedious.


### Proposed Change

If `sensitive_features=` is a DataFrame (or list of Series.... exact supported types are TBD), we can generate our results in terms of a MultiIndex. Using the `A` DataFrame defined above, a user might write:
```python
>>> result = group_summary(skm.accuracy_score,
                           y_true, y_pred,
                           sensitive_features=A)
>>> result.by_groups
           accuracy_score
SF 1 SF 2
B    M               0.50
     N               0.40
     P               0.55
C    M               0.45
     N               0.70
     P               0.63
```
If a particular combination of sensitive features had no representatives, then we would return `None` for that entry in the Series.
Alternatively, we could skip that entry in the DataFrame - this point is **still TBD**.
Although this example has passed a DataFrame in for `sensitive_features=` we should aim to support lists of Series and `numpy.ndarray` as well.

The `differences()` and `ratios()` methods would act on this DataFrame as before.

## Conditional Metrics

For our purposes, Conditional Metrics (alternatively known as Segmented Metrics) are specified separately from the sensitive features, since for users, they add columns to the result.
Mathematically, they behave like additional sensitive features.

### Existing Syntax

Not supported.
Users would have to devise the required code themselves

### Proposed Change

The `GroupedMetric` constructor will need an additional argument `conditional_features=` to specify the conditional features.
It will accept similar types to the `sensitive_features=` argument.
Suppose we have another column called `income_level` with unique values 'Low' and 'High'
```python
>>> metric = GroupedMetric(skm.accuracy_score,
                           y_true, y_pred,
                           sensitive_features=A_1,
                           conditional_features=income_level)
>>> metric.overall
       accuracy_score
High            0.46
Low             0.61
dtype: float64
>>> metric.by_group
          accuracy_score
High   B            0.40 
       C            0.55
Low    B            0.55
       C            0.65
```
The `overall` property now has rows corresponding to the unique values of the conditional feature(s).
Similarly, the result DataFrame now uses a Pandas MultiIndex for the columns, giving one column for each (combination of) conditional feature.

Note that it is possible to have multiple sensitive features, and multiple conditional features.
Operations such as `.group_max()` and `.difference()` will act on each combination of conditional feature values, and aggregate across the sensitive features.
So for example
```python
>>> metric.difference(method='minmax')
        accuracy_score
High              0.15
Low               0.10
>>> metric.difference(method='overall')
        accuracy_score
High              0.09
Low               0.06
```

If it users found it more convenient to have the conditional features be sub-columns on the metrics, then the `unstack()` method of the pandas DataFrame can be used.

## Multiple Metrics

Finally, we can also allow for the evaluation of multiple metrics at once.

### Existing Syntax

This is not supported.
Users would have to devise their own means.

### Proposed Change

We allow a list of metric functions in the call to group summary.
The properties then add columns to their DataFrames:
```python
>>> result = GroupedMetric([skm.accuracy_score, skm.precision_score],
                            y_true, y_pred,
                            sensitive_features=A_1)
>>> result.overall
         accuracy_score  precision_score
overall             0.3              0.5
>>> result.by_groups
      accuracy_score  precision_score
'B'              0.4              0.7
'C'              0.6              0.75
```
This should generalise to the other methods described above - extra metric functions add extra columns to the resultant DataFrames (this is why we made all results to be DataFrames, even if the actual result was a scalar).

When users wish to use the `sample_params=` and `params=` arguments, then they should pass in lists of dictionaries, matching the functions by index:
```python
metric_fns = [skm.accuracy_score, skm.precision_score]
sample_params = [{'sample_weight':weight}], [{'sample_weight':weight}]]
params = [{ 'normalize': False }, {'pos_label' = 'Granted'}]
result = GroupedMetric(metric_fns,
                       y_true, y_pred,
                       sensitive_features=A_1,
                       sample_params=sample_params,
                       params=params)
```
The length of the lists would be required to match.
This is somewhat repetitious (see the `sample_weight` above), but trying to share some arguments between functions is likely to lead to a worse mess.

## Naming

The name `GroupedMetric` is not especially inspired.
Some possible alternatives:
  - `GroupedMetric`
  - `GroupMetricResult`
  - `MetricByGroups`
  - ?

Other names are also up for debate.
Arguments such as `method='minmax'` are important, but narrower in impact.

## User Conveniences

In addition to having the underlying metric be passed as a function, we can consider allowing the metric function given to the `GroupedMetric` constructor to be represented by a string.
We would provide a mapping of strings to suitable functions.
This would make the following equivalent:
```python
>>> r1 = GroupedMetric(sklearn.accuracy_score,
                       y_true, y_pred, 
                       sensitive_features=A_1)
>>> r2 = GroupedMetric('accuracy_score',
                       y_true, y_pred, 
                       sensitive_features=A_1)
```
We would also allow mixtures of strings and functions in the multiple metric case.

## Generality

Throughout this document, we have been describing the case of classification metrics.
However, we do not actually require this.
It is the underlying metric function which gives meaning to the `y_true` and `y_pred` lists.
So long as these are of equal length (and equal in length to the sensitive feature list - which _will_ be treated as a categorical), then `GroupedMetric` does not actually care about their datatypes.
For example, each entry in `y_pred` could be a dictionary of predicted classes and accompanying probabilities.
Or the user might be working on a regression problem, and both `y_true` and `y_pred` would be floating point numbers (or `y_pred` might even be a tuple of predicted value and error).
So long as the underlying metric understands the data structures, `GroupedMetric` will not care.

There will be an effect on the `difference()` and `ratio()` methods.
Although the `overall` and `by_groups` properties will work fine, the `differences()` and `ratios()` methods may not.
After all, what does "take the ratio of two confusion matrices" even mean?
We should try to trap these cases, and throw a meaningful exception (rather than propagating whatever exception happens to emerge from the underlying libraries).
Since we know that `difference()` and `ratio()` will only work when the metric has produced scalar results, which should be a straightforward test using [`isscalar()` from Numpy](https://numpy.org/doc/stable/reference/generated/numpy.isscalar.html).

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

Unfortunately, the generality of `GroupedMetric` means that we cannot solve this for the user.
It cannot even tell if it is evaluating a classification or regression problem.

## Creating Derived Metrics

Rather than a `GroupedMetric` object, users will often want to have a function which yields a scalar.

### Existing Syntax

We currently provide a `make_derived_metric()` function which can build a callable object which does this:
```python
fhalf_score = functools.partial(skm.fbeta_score, beta=0.5)

custom_difference1 = make_derived_metric(
    difference_from_summary,
    make_metric_group_summary(fhalf_score))
```
Notice that we have had to put a wrapper around `fbeta_score` since `beta=` is a required parameter, but we do not support anything beyond `y_true`, `y_pred` and `sensitive_features` as arguments.

We use this to provide helper functions such as `accuracy_score_difference()` and `accuracy_score_group_min()`

### Proposed Syntax

We should be able to provide a function builder of the following form:
```python
fbeta_diff = make_derived_metric(
    'difference',
    skm.fbeta_score,
    sample_param_names=['sample_weight']
)

print(fbeta_diff(y_true, y_pred, sensitive_features=A_1, sample_weight=weights, beta=0.5))
```
Since the goal of this function is to produce scalars, we would not support supplying multiple underlying metrics.
We also do not propose to support conditional features at the present time (the appropriate behaviour is not clear).
The derived metrics would correspond to the various methods described above which compute scalars from the `GroupedMetrics` object.


## Support for `make_scorer()`

SciKit-Learn has the concept of 'scorer' functions, which take arguments of `y_true` and `y_pred` and return a scalar score.
These are used in higher level algorithms such as `GridSearchCV`.
In order to use these in combination with metrics which can use other arguments (such as the `normalize=` argument on `accuracy_score` above), SciKit-Learn has a `make_scorer()` function , which takes a metric function, along with a list of other arguments, and binds them together to provide a function which just accepts `y_true` and `y_pred` arguments, but will invoke the underlying metric function with the specified extra arguments.
The higher level algorithms take folds of the input data, and ask the generated scoring function to evaluate these.

There is one problem with this: if a user has a per-sample input (such as sample weights), how do we select the correct values to match the fold?
When the generated scorer is invoked, the `y_true` and `y_pred` arrays will be a subset of the `sample_weights` bound into the scorer by `make_scorer()`, so the problem is to work out the subset.
Currently, there is no good way to do this through SciKit-Learn (although a proposed solution is under development).
There is a [work around described by Adrin on StackOverflow](https://stackoverflow.com/questions/49581104/sklearn-gridsearchcv-not-using-sample-weight-in-score-function), which relies on DataFrames being sliced 'in-place' by SciKit-Learn.
If all arguments are DataFrames (or Series) when when the generated scorer is invoked, the `index` property of `y_true` can be examined, and used as a mask on the sample weights column (which is bound into the generated scorer).
A more general solution is [under discussion within the SciKit-Learn community](https://scikit-learn-enhancement-proposals.readthedocs.io/en/latest/slep006/proposal.html).

Our grouped metrics will always face this problem, since we always have the sensitive feature which will need to be passed along.
We can provide a `make_grouped_scorer()` method with a signature like:
```python
make_grouped_scorer(metric_function,
                    sensitive_feature,
                    indexed_params,
                    params,
                    disparity_measure='difference',
                    relative_to='min')
```
We will only support a single sensitive feature for this, since we need to produce a single scalar result.
The function will verify that it has been passed a Pandas Series or DataFrame, so that the `index` is available.
The `disparity_measure=` argument specifies whether the disparity should be measured via the difference or ratio (corresponding to the methods on the `GroupedMetric` object).
Similarly, the `relative_to=` argument can also be set to `overall` - although in this case it is important to note that this will be the overall value for the fold, and *not* the overall value on the entire dataset.


## Convenience Functions

We currently provide functions for evaluating common fairness metrics (where `X` can be `ratio` or `difference`):

- `demographic_parity_X()`
- `true_positive_rate_X()`
- `false_positive_rate_X()`
- `equalized_odds_X()`

We will continue to provide these wrappers, based on `GroupedMetric` objects internally.
