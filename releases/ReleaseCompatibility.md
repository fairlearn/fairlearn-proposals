# Releases & Compatibility

To date, we have been somewhat cavalier about making breaking
changes in Fairlearn.
Although we are currently pre-`v1.0.0` and hence without particular
commitments to compatibility (as generally understood - see e.g.
the [Semantic Versioning Scheme](https://semver.org/)), we should
work to reduce the number of breaking changes we make.

## The Problem

At the time of writing, we have just released `v0.4.6`, which has
some substantial breaking changes in the Metrics area of Fairlearn.
Before it, `v0.4.5` reworked a smaller subset of this functionality, 
caused smaller breakages.
There are also further changes to Metrics planned, which may well
cause further breaks (although these should be more minor).
All this is obviously undesirable from a user standpoint - these look
like 'patch' level releases, but are actually breaking their code.
Concretely, we've had users install Fairlearn with `pip`, and then
find themselves unable to run Notebooks from our GitHub project - not
because the functionality was missing, but because it had been
renamed.
Moving our notebooks to being generated as part of the documentation
[in our examples directory](https://github.com/fairlearn/fairlearn/tree/master/examples)
will help with this, since this will result in the notebooks being
versioned.
However, improved backwards compatibility is still desirable.

We do not need deprecation policies as elaborate of those of
[SciKit-Learn](https://numpy.org/neps/nep-0023-backwards-compatibility.html)
or [NumPy](https://numpy.org/neps/nep-0023-backwards-compatibility.html) - indeed,
policies such as those would be excessive for Fairlearn, given the
size of our code and user bases.
However, we do need to start to move in that direction, or we will
not be able to grow our user base due to chaos in the code.

## Support Policy

Starting with `v0.5.0` we should make a commitment that anything which works at `v0.n.m_0` will also work for `v0.n.m` so long as `m >= m_0`.
However, we do *not* guarantee compatibility between `n` and `n+1` in this scheme (although we would seek to minimise breakage).
If we have to do `v.n.m.post[i]` releases, we will only support the final `post` release in the chain.

In order to support less mature functionality, we should also add
a `fairlearn.experimental` package (see [a similar namespace in
SciKit-Learn](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.experimental)).
Anything in there will be subject to breaking at any time.
Using an `experimental` namespace would not have helped with our
current set of breaks, since the changes were being made to core
functionality.
Rather, this is to give future developers a space where they can
get feedback on new functionality without immediately being
committed to supporting their speculative design decisions.

At the time of writing, it appears that the dashboard code does
not use namespaces.
However, namespaces are supported in TypeScript, and as we develop
the UX code, we should introduce a similar distinction.

To monitor the required backwards compatibility, each new release
will have a build pipeline associated with it.
This pipeline will run the *tests associated with the release tag*
against the version of Fairlearn in `master`
This will become part of the PR Gate for `master`, except when
we are moving from `v0.n` to `v0.n+1`.

## Revised Branching and Release Policy

Our recent releases have been made from `master`, without
making a release branch.
While this approach has desirable properties, using release branches
will work better with the more robust support policy described above.

