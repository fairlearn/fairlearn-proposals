# Releases & Compatibility

To date, we have been somewhat cavalier about making breaking changes in Fairlearn.
Although we are currently pre-`v1.0.0` and hence without particular commitments to compatibility (as generally understood - see e.g. the [Semantic Versioning Scheme](https://semver.org/)), we should work to reduce the number of breaking changes we make.

## The Problem

At the time of writing, we are about to release `v0.4.6`, which has some substantial breaking changes in the Metrics area of Fairlearn.
Before it, `v0.4.5` reworked a smaller subset of this functionality, which caused smaller breakages.
There are also further changes to Metrics planned, which may well cause further breaks (although these should be more minor).
All this is obviously undesirable from a user standpoint - these look like 'patch' level releases, but are actually breaking their code.
Concretely, we've had users install Fairlearn with `pip`, and then find themselves unable to run Notebooks from our GitHub project - not because the functionality was missing, but because it had been renamed.

We do not need deprecation policies as elaborate of those of [SciKit-Learn](https://numpy.org/neps/nep-0023-backwards-compatibility.html) or [NumPy](https://numpy.org/neps/nep-0023-backwards-compatibility.html) - indeed, policies such as those would be ridiculous given the size of our code and user bases.
However, we do need to start to move in that direction, or we will not be able to grow our user base due to chaos in the code.

## Proposed Solution

Starting with `v0.5.0` we should make a commitment that anything which works at `v0.n.m_0` will also work for `v0.n.m` so long as `m >= m_0`.
However, we do *not* guarantee compatibility between `n` and `n+1` in this scheme (although we would seek to minimise breakage).

We will enforce this using builds which run the notebooks currently in the repository against versions of Fairlearn published on PyPI.
We already have a build which checks that our notebooks can run against the latest published version of Fairlearn (due to the metrics changes, this build is a steady red); this would be an extension of that functionality.
For full compliance with the above, we will have to devise a means of allowing each notebook to specify its `m_0`.
Our notebooks do not provide comprehensive coverage of Fairlearn, but enforcing a constraint like this will be a dramatic improvement over the current situation (particularly since new users will likely try our notebooks first).

In order to support less mature functionality, we should also add a `fairlearn.experimental` package (see [a similar namespace in SciKit-Learn](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.experimental)).
Anything in there will be subject to breaking at any time.
There would be a parallel subdirectory of notebooks which rely on such functionality.
We will also automatically check that any notebook in that directory depends on `fairlearn.experimental`.
Using an `experimental` namespace would not have helped with our current set of breaks, since the changes were being made to core functionality.
Rather, this is to give future developers a space where they can get feedback on new functionality without immediately being committed to supporting their speculative design decisions.