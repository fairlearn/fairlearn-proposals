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

We propose to introduce a new object, the `MetricFrame` (name discussion below).
Users will compute metrics by passing arguments into the constructor:
```python
>>> metrics = MetricFrame(skm.accuracy_score,
                          y_true, y_pred,
                          sensitive_features=A_1)
<class 'MetricFrame'>
>>> metrics.overall
accuracy_score   0.4
>>> metrics.by_group
   accuracy_score
B             0.6536
C             0.213
```
The `overall` property is a Pandas Series, indexed by the name of the underlying metric.
The `by_group` property is a Pandas DataFrame, with a column named by the underlying metric.
The rows of the `by_group` property are set to the unique values of the `sensitive_feature=` argument.

Sample based parameters (such as sample weights) can be passed in using the `sample_params=`
argument:
```python
>>> metrics = MetricFrame(skm.accuracy_score,
                          y_true, y_pred,
                          sensitive_features=A_1,
                          sample_params={'sample_weight': weight})
```
If the underlying requires other arguments (such as the `beta=` argument to `fbeta_score()`),
then `functools.partial()` must be used.

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
These will take an optional argument of `method=`, to indicate how the values are to be calculated.
For now, the valid values of this argument will be `between_groups` (indicating that just the values in the
`by_groups` property should be used) and `to_overall` (indicating that all results should be calculated
relative to the appropriate values in the `overall` property).
First for computing the difference:
```python
>>> metrics.difference(method='between_groups')
accuracy_score   0.4406
dtype: float64
>>> metrics.difference(method='to_overall')
accuracy_score   0.2563 # max(abs(0.6536-0.4), abs(0.213-0.4))
dtype: float64
```
Note that the result type is a Series (for reasons which will become clear below).
The `ratio()` method would behave in a similar way:
```python
>>> metrics.ratio(method='between_groups')
accuracy_score     0.3259
dtype: float64
>>> metrics.ratio(method='to_overall')
accuracy_score    0.6120 # min(abs(0.4/0.6536), abs(0.213/0.6536))
dtype: float64
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

If `sensitive_features=` is a DataFrame (or list of Series, or list of numpy arrays, or a 2D numpy array etc.), we can generate our results in terms of a MultiIndex. Using the `A` DataFrame defined above, a user might write:
```python
>>> result = MetricFrame(skm.accuracy_score,
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
If a particular combination of sensitive features had no representatives, then we would return `NaN` for that entry in the Series.
Although this example has passed a DataFrame in for `sensitive_features=` we should aim to support lists of Series and `numpy.ndarray` as well.

The `differences()` and `ratios()` methods would act on this DataFrame as before.

## Control Metrics

Control Metrics (alternatively known as Conditional Metrics) are specified separately from the sensitive features, since
the aggregation functions discussed above do not act across them.
Within the `by_group` property, they behave like additional sensitive features.

### Existing Syntax

Not supported.
Users would have to devise the required code themselves

### Proposed Change

The `MetricFrame` constructor will need an additional argument `control_features=` to specify the control features.
It will accept similar types to the `sensitive_features=` argument.
Suppose we have another column called `income_level` with unique values 'Low' and 'High'
```python
>>> metric = MetricFrame(skm.accuracy_score,
                         y_true, y_pred,
                         sensitive_features=A_1,
                         control_features=income_level)
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
The `overall` property is now a DataFrame, with rows corresponding to the unique values of the control feature(s).
Similarly, the result DataFrame now uses a Pandas MultiIndex for the columns, giving one column for each (combination of) control feature.

Note that it is possible to have multiple sensitive features, and multiple control features.
Operations such as `.group_max()` and `.difference()` will act on each combination of control feature values, and aggregate across the sensitive features.
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

We allow a dictionary of metric functions in the call to group summary.
The properties then extend themselves:
```python
>>> result = MetricFrame({'accuracy':skm.accuracy_score, 'precision':skm.precision_score},
                         y_true, y_pred,
                         sensitive_features=A_1)
>>> result.overall
accuracy       0.3
precision      0.5
dtype: float64
>>> result.by_groups
      accuracy  precision
'B'        0.4        0.7
'C'        0.6        0.75
```
Note that we use the dictionary keys, rather than the function names in the output.
This should generalise to the other methods described above.

When users wish to use the `sample_params=` arguments, then they should pass in a dictionary of dictionaries, matching the functions by key:
```python
metric_fns = { 'accuracy':skm.accuracy_score, 'precision':skm.precision_score}
sample_params = { 'accuracy':{'sample_weight':weight}, 'precision':{'sample_weight':weight}}
result = MetricFrame(metric_fns,
                     y_true, y_pred,
                     sensitive_features=A_1,
                     sample_params=sample_params)
```
The outer set of dictionary keys given to `sample_params=` should be a subset of the keys of the metric function dictioary.
This is somewhat repetitious (see the `sample_weight` above), but trying to share some arguments between functions is likely to lead to a worse mess.

## Generality

Throughout this document, we have been describing the case of classification metrics.
However, we do not actually require this.
It is the underlying metric function which gives meaning to the `y_true` and `y_pred` lists.
So long as these are of equal length (and equal in length to the sensitive feature list - which _will_ be treated as a categorical), then `MetricFrame` does not actually care about their datatypes.
For example, each entry in `y_pred` could be a dictionary of predicted classes and accompanying probabilities.
Or the user might be working on a regression problem, and both `y_true` and `y_pred` would be floating point numbers (or `y_pred` might even be a tuple of predicted value and error).
So long as the underlying metric understands the data structures, `MetricFrame` will not care.

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

Unfortunately, the generality of `MetricFrame` means that we cannot solve this for the user.
It cannot even tell if it is evaluating a classification or regression problem.
