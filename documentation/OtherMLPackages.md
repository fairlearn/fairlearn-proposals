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

The primary goal of this testing is to show that other ML packages can be used with our mitigation algorithms.
This does not require in-depth testing, but more a smoke test.
The tests should verify that, with a reasonable dataset, estimators from each package (possibly only a single example estimator) can be used with our mitigation algorithms, and produce reasonable results.
A sample test might look a lot like those in `test_exponentiatedgradient_smoke.py` but on a more realistic dataset, and with fewer combinations of constraints and error bounds.

## Concerns for Testing

The packages listed above could be fairly described as 'not lightweight.'
Requiring developers to install all of them as part of the standard developer setup (i.e. listing in `requirements-dev.txt`) will introduce friction to the process.
Furthermore I am concerned that
  - Some of these packages might have incompatible dependencies
  - Some might be best installed with `conda` rather than `pip`

Even if there aren't incompatible dependencies, moving to requiring `conda` for development work would be a significant change from the current `pip`-only setup (even if most developers probably `pip install` into a fresh `conda` environment).

## Proposed Setup

Because they will have extra dependencies, these tests should not go into our current `tests/` directory.
We instead have a new directory `othermlpackages/` at the top level and then arrange it along these lines:
```
othermlpackages/
    lightgbm/
        conda-env.yml
        test_lightgbm_expgrad.py
        test_lightgbm_gridsearch.py
    pytorch
        conda-env.yml
        test_pytorch_expgrad.py
        test_pytorch_gridsearch.py
```
There would be new build pipelines which would run these tests.
The tests for each package would be run in a separate job (so that they can be isolated), and the pipeline would:
  1. Create a `conda` environment using the appropriate `conda-env.yml`
  1. Install Fairlearn and its prerequisites into that environment
  1. Run the tests in the appropriate directory

Whether there would be one pipeline to run all the jobs, or whether we would have one pipeline for each ML package we're testing would have to be decided.

## Examples in the user guide

Adopting the proposal above would make it impossible to incorporate examples into the user guide in the way we have done to date - the separate installation of the ML packages would mean that `sphinx` would not be able to run any sample code.
Since our goal is to demonstrate that the packages _can_ work with Fairlearn, this is not a major concern.
We can still write up some small code samples, perhaps incorporating code directly from the tests; we just won't be able to generate the results 'live' as part of the documentation build.