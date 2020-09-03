# Testing Other ML Packages

In our documentation, we state that the mitigation algorithms in Fairlearn are compatible with any package which presents an interface similar to SciKit-Learn.
Specifically, so long as there's some sort of 'estimator' concept which provides `fit()` and `predict()` methods (the former with a `sample_weight=` argument), then those estimators can be used with our mitigation algorithms.
However, we do not currently have tests proving that we do offer this sort of compatibility with a variety of popular ML packages.

## Target Packages

We should show compatibility with the following ML packages:
  - XGBoost
  - LightGBM
  - TensorFlow
  - PyTorch

This is not an exhaustive list, but is a good starting point.
In all cases, these packages either offer a SciKit-Learn compatible API, or there is a SciKit-Learn compatible wrapper available (such as `skorch` for PyTorch).

## Required Tests

