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

```python
result = flm.group_summary(skm.accuracy_score, y_true, y_pred, sensitive_features=A_sex)
print(result)
>>> {'overall': 0.4, 'by_group': {'male': 0.6536, 'female': 0.213}}
print(type(result))
>>> <class 'sklearn.utils.Bunch'>
```