# Versioned Documentation

We are starting to run into issues with people trying to use Fairlearn installed from PyPI with the documentation on [our website](https://fairlearn.github.io/).
This is an issue because we have made and are making a number of breaking changes to our API.
The solution to this is to provide versioned documentation, so that users can read the docs associated with the version they have installed.
This is the pattern used by projects such as [SciKit-Learn](https://scikit-learn.org).

The basic pattern is to change the layout of the `fairlearn.github.io` repository so that there is a directory for each version of the documentation.
There is then a root level `index.html` to redirect users to an appropriate default directory (either latest release, or the development branch).
See the [SciKit-Learn documentation repo](https://github.com/scikit-learn/scikit-learn.github.io) for an example.

There are tools to help with this, such as:
- [sphinx-multiversion](https://pypi.org/project/sphinx-multiversion/)
- [sphinxcontrib-versioning](https://pypi.org/project/sphinxcontrib-versioning/)

The latter of these last had a PyPI release in 2016. The last repo commit was in 2019, but there was little activity after 2016.
As such, it is probably not a wise choice.

The `sphinx-multiversion` tool has more recent updates, and the author has been very helpful with debugging issues the Fairlearn documentation.
Some of these have resulted from Windows, and the author has issued patches for them.
Unfortunately, there is a more fundamental issue: we have substantially changed the `examples/` directory, and this means that the latest `conf.py` does not work with `v0.4.6`.
This is not compatible with the way `sphinx-multiversion` works.

## Proposal

We should onboard to using `sphinx-multiversion` going forwards.
However, if we want `v0.4.6` documentation, that will have to be a special case.
It should be relative straightforward to add it to the `fairlearn.github.io` repository manually, once `sphinx-multiversion` has taken over main documentation build.

Aside from the `v0.4.6` issue, the risk in this approach is that each version will build all the examples.
That could take us over the time limit for the CircleCI free tier.