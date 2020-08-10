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

## Tool Selection

We should onboard to using `sphinx-multiversion` going forwards.
However, if we want `v0.4.6` documentation, that will have to be a special case.
It should be relative straightforward to add it to the `fairlearn.github.io` repository manually, once `sphinx-multiversion` has taken over main documentation build.
We will then at least have a URL we can share with those who encounter issues.

Since `sphinx-multiversion` builds every version of the documentation each time and we have some examples which take a while to generate, it is possible that we might start to bump against the time limits of our free CircleCI account.
If this happens, we will have to devise workarounds.

## Required Changes

Moving to versioned documentation will require a number of changes to the workings of our `docs/` directory.

Our static, professionally designed landing page should be removed from the Sphinx build.
The assets should be saved in a separate directory, and copied to the appropriate location by a script after the main documentation has been built.

To access the different versions of the documentation, `sphinx-multiversion` [provides two samples](https://holzhaus.github.io/sphinx-multiversion/master/templates.html) of HTML fragments which can be generated with appropriate links.
Unfortunately, it appears that currently the `pydata-sphinx-theme` we're using does not currently support customised sidebars.
I have [opened an issue on their GitHub repo](https://github.com/pandas-dev/pydata-sphinx-theme/issues/234) about this, and it has been acknowledged.
If no better solution can be found, then I can adjust the generated code from `sphinx-multiversion` to be a full webpage, and have it redirect to the `master` version after a pause of five seconds.
The 'User Guide' links on the static landing page can then point to this version-selection page.
