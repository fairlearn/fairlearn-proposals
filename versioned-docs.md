# Versioned Documentation

We are starting to run into issues with people trying to use Fairlearn installed from PyPI with the documentation on [our website](https://fairlearn.github.io/).
This is an issue because we have made and are making a number of breaking changes to our API.
The solution to this is to provide versioned documentation, so that users can read the docs associated with the version they have installed.
This is the pattern used by projects such as [SciKit-Learn](https://scikit-learn.org).

Getting to something as sophisticated as the SciKit-Learn setup will take time.
As a first step, we propose:

1. Change the CirceCI integration to push to a `main` directory (in preparation for the expected rename of the `master` branch) within the [`fairlearn.github.io`](https://github.com/fairlearn/fairlearn.github.io) repository, rather than pushing to the root of the repository.

1. Update links on the custom-designed landing page to go to this new `main` directory.

1. Create a `v0.4.6` directory in `fairlearn.github.io`, and do a documentation build corresponding to the `v0.4.6` tag in this directory. This may well have to be done manually.

The last stage will at least give us a set of documentation corresponding to the version of Fairlearn on PyPI, which we can share with people who run into difficulties.

For the future:

1. Automate the creation of the `vx.y.z` subdirectories

1. Figure out a navigation control for switching between versions, similar to that used by SciKit-Learn