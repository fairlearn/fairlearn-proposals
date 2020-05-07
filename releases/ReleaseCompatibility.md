# Releases & Compatibility

To date, we have been somewhat cavalier about making breaking changes in Fairlearn.
Although we are currently pre-`v1.0.0` and hence without particular commitments to compatibility (as generally understood - see e.g. the [Semantic Versioning Scheme](https://semver.org/)), we should work to reduce the number of breaking changes we make.

At the time of writing, we are about to release `v0.4.6`, which has some substantial breaking changes in the Metrics area of Fairlearn.
Before it, `v0.4.5` reworked a smaller subset of this functionality, which caused smaller breakages.
There are also further changes to Metrics planned, which may well cause further breaks (although these should be more minor).
All this is obviously undesirable from a user standpoint - these look like 'patch' level releases, but instead are breaking their code.
Concretely, we've had users install Fairlearn with `pip`, and then find themselves unable to run Notebooks from our GitHub project - not because the functionality was missing, but because it had been renamed.

We do not need deprecation policies as elaborate of those of [SciKit-Learn](https://numpy.org/neps/nep-0023-backwards-compatibility.html) or [NumPy](https://numpy.org/neps/nep-0023-backwards-compatibility.html) - indeed, policies such as those would be ridiculous given the size of our code and user bases.
However, we do need to start to move in that direction, or we will not be able to grow our user base due to chaos in the code.