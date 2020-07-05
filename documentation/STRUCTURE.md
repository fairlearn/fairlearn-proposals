# Documentation

For the purpose of this proposal document we consider all of the following as documentation:

- documentation in our code within the Fairlearn repository
- Jupyter notebooks
- project website
- blog posts

Throughout the document we compare with the documentation of other popular
projects such as 

- [scikit-learn](https://scikit-learn.org/)
- [pandas](https://pandas.pydata.org/)

## Goals

The documentation should be

- discoverable, ideally in a single place as opposed to multiple
- clear
- concise when describing individual pieces of functionality
- detailed when describing entire application scenarios, e.g. in the form of
  example notebooks
- available for the latest version, but if possible also for past versions
  ([example](https://scikit-learn.org/dev/versions.html))
- maintainable: it should be simple or at least clear for maintainers how to
  update/validate
- without ads (readthedocs always has ads that are shown alongside our
  documentation)

## Proposal

Like for most projects the website, [fairlearn.org](http://fairlearn.org) will
be the central place to look for documentation.
From there visitors have various paths to explore content depending on what
they are looking for, as detailed in the following subsections.

### Homepage

```
website
|--- About
|--- Quickstart
|--- API reference
|--- User guide
|--- Example Notebooks
|--- Contributor guide
```

### About

This page provides a high-level overview of Fairlearn including

- mission
  - Who does the project serve? Who are current users? 
- project roadmap
- governance structure
- history of the project
- FAQ section

It serves as a primary entrypoint for people who want to understand what
Fairlearn's purpose is (mission), what's coming up (roadmap), and how the
project is set up and controlled (governance). The FAQ section should round
this out by providing answers to frequently asked questions. We already have
lots of reoccurring questions that would make sense there.

### Quickstart

This page provides information on

- Installation
- brief introduction/framing of fairness in ML
  - including basic terminology (perhaps link to more comprehensive section on a
    different page)
- walk-through
  - load data
  - mitigate disparity of an estimator
  - evaluate a few metrics
  - run the dashboard
- links showing where to go next, e.g. links to section of the user guide

Installation should cover various platforms, which should be very
straightforward. Any reoccurring patterns in reported issues should be listed,
as well as how to troubleshoot them. It may end up similar to
[this guide](https://scikit-learn.org/dev/install.html)).

Example:

- [pandas](https://pandas.pydata.org/getting_started.html)
- [scikit-learn](https://scikit-learn.org/dev/getting_started.html)

This should be very similar to what we currently have in our README, so a lot
of the content won't be entirely new.

### API reference

This is simply the generated documentation from our code using docstrings.
Currently we host this in readthedocs, but we want to include it on our
webpage. A good example for this is
[scikit-learn](https://scikit-learn.org/dev/modules/classes.html)

### User guide

The user guide explains all parts of Fairlearn by providing context that
wouldn't fit into the code documentation such as mathematical derivations,
but without using application-specific context (as we'd find it in the
"example notebooks"). The guides are grouped by topic, e.g.

1. What we mean by fairness in ML - should properly frame fairness as a
   sociotechnical challenge incl.
    - considering harms instead of biases
    - why "debiasing" is not possible
    - fairness through unawareness and why it is not sufficient
    - the ML lifecycle and how individual stages can affect fairness
    - AI systems need to be designed around the people they affect
      (specifically subpopulations that may be harmed by a system)
    - some reference to the
      [fairness checklist](https://www.microsoft.com/en-us/research/publication/co-designing-checklists-to-understand-organizational-challenges-and-opportunities-around-fairness-in-ai/)
    - ...
1. Assessment
    1. Fairness definitions
        1. ...
    1. Metrics
        1. ...
    1. Dashboard
        1. ...
1. Mitigation
    1. Postprocessing
        1. Threshold Optimizer
    1. Reductions methods
        1. Exponentiated Gradient
        1. Grid Search

Importantly, the code samples should be minimal. For comprehensive examples
we have the "Example Notebooks" section. In comparison, this section is more
like a tutorial. It's about showing how to use our API while elaborating on
mathemtical background that we can't explain in API documentation

Examples:

- [scikit-learn](https://scikit-learn.org/dev/user_guide.html)
- [pandas](https://pandas.pydata.org/docs/user_guide/index.html)

Of our current notebooks the following may be most suitable as "user guides":

- [Group Metrics](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Group%20Metrics.ipynb) -
  a great example for something that should be a user guide
- [Grid Search for Binary Classification](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Grid%20Search%20for%20Binary%20Classification.ipynb) -
  the purpose is mostly to show Grid Search's functionality; perhaps it may need to be trimmed down to the essentials about Grid Search
- [Grid Search with Census Data](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Grid%20Search%20with%20Census%20Data.ipynb) -
  similar to the previous notebook it covers Grid Search; we could leverage some of this for a user guide for Grid Search, or alternatively for the dashboard visualizations

### Example Notebooks

The purpose of the example notebooks is to walk through an application of
Fairlearn in detail. Any application of a fairness toolkit needs to be done
with great care while taking into account an entire range of concerns due to
the sociotechnical nature of fairness. The showcased notebooks will provide
the space to cover scenarios in depth. The focus is not only on showing
example usage of the Fairlearn toolkit, but on how to approach fairness in ML
in general. We may want to add a scenario even if it contains only few of
Fairlearn's capabilities, but it otherwise demonstrates a great example of
how to build AI responsibly.

All the example notebooks should be downloadable as Jupyter notebooks and
Python source code, and be launchable in [Binder](https://mybinder.org/) or a
similar platform.

Note: [scikit-learn](https://scikit-learn.org/dev/auto_examples/index.html)
refers to these as "Examples". However, they use them to highlight a specific
aspect of a feature/model. For Fairlearn it would be more about a properly
framed example from a fairness point of view.

Of our current notebooks the following would be most closely aligned with
this section:

- [Binary Classification on COMPAS dataset](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Binary%20Classification%20on%20COMPAS%20dataset.ipynb),
  although we should perhaps consider removing it since it may not do
  justice to this complex setup
- [Binary Classification with the UCI Credit-card Default Dataset](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Binary%20Classification%20with%20the%20UCI%20Credit-card%20Default%20Dataset.ipynb)
- [Mitigating Disparities in Ranking from Binary Data](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Mitigating%20Disparities%20in%20Ranking%20from%20Binary%20Data.ipynb)

### Contributor Guide

We want to ensure people know

- ways to contribute
    - It should be clear how people can reach out if they want to contribute
      to Fairlearn, and where they can find small items to get started.
- the respository sturcture / organization of work
- Fairlearn proposals
- how to contribute code
    - Moments
    - ...
- how to contribute notebooks
    - style guide
    - good workflow for editing (from `.ipynb` to `.py` etc.)
    - ...

[This](https://scikit-learn.org/dev/developers/contributing.html) is an
example of how scikit-learn handles it through a contribution guide that is
somewhat similar to ours that we currently have in the repo.

Some projects have a page showing the maintainers as well:

- [scikit-learn](https://scikit-learn.org/stable/about.html#people)
- [pandas](https://pandas.pydata.org/about/team.html)

## Required steps

1. Get GitHub Pages page/repository up and running
1. Set up CI to deploy current documentation there automatically
1. Set up CI to make documentation changes viewable, i.e. the generated
   HTML pages need to be visible (CircleCI)
1. Establish webpage section as outlined above (Quickstart, User guide, etc.)
1. Convert existing content, including reformatting markdown as
   ReST.
    - We already have the examples gallery thanks to Adrin's work on
      `sphinx-gallery`. There will be plenty of work to convert existing
      notebooks to ReST example notebooks (or user guides) as mentioned in
      earlier sections. If this is very laborious we can consider shortcuts
      for the short-term such as linking to GitHub notebooks, or using a
      Jupyter plugin for `sphinx`.
    - Related: Document notebook development process (see separate section
      below)
1. Write remaining content for all of them.
1. We need to find a way to present the dashboard in a website
   where it can't be interactive. Perhaps with screenshots for the user
   guides, but the example notebooks are downloadable as Jupyter notebooks.
   [Could we perhaps pre-calculate all metrics and show the interactive
   dashboard in the example notebooks? There may be a sphinx extension for
   typescript]
1. integrate landing page (will be provided by a designer), everything
   else from the Fairlearn project repositories
1. add style template from pandas
1. remove API doc from readthedocs
1. ensure the navigation from homepage to the other sections works
    - manual testing
1. automated testing of navigation/broken links
1. Add an example notebook to show how to use estimators from various packages
   with our mitigation techniques.

The repository structure should look similar to
[what scikit-learn has](https://github.com/scikit-learn/scikit-learn/tree/master/doc):

- top-level doc directory has all the ReST files for the webpage except for
  API documentation and example notebooks
- API documentation comes directly from the code documentation
- example notebooks live in a separate top-level directory
  (scikit-learn calls it `examples`) as python files

### Less urgent

- Switch to numpy doc format; benefits explained in
  [this issue](https://github.com/fairlearn/fairlearn/issues/314);
  definitely worthwhile, but not as urgent as other items that get the webpage
  started. Proposed solution in that issue was to switch over piece-by-piece.

## Development process

We need to document all the processes around generating documentation.
Specifically, we need to document how one can

- build/generate the documentation and subsequently view it
- develop example notebooks that end up in python files, but perhaps while
  creating them using Jupyter
- get documentation changes into the Fairlearn repository
  (PR with CI generating documentation and storing it as artifacts)

Document exactly which tools/plugins we recommend, e.g. VSCode extensions
or Jupytext, etc.

## Outstanding questions / tasks

Documentation infrastructure related tasks should be tracked through the
corresponding [GitHub project](https://github.com/fairlearn/fairlearn/projects/6)
in the Fairlearn repository.

1. Do we want users to cite us in any way? See
   [this example](https://scikit-learn.org/dev/about.html#citing-scikit-learn).
1. Do we want user testimonials? It definitely provides credibility (assuming
   users are willing)
1. Do we want a "News" section? It could list recent updates such as new
   versions (link to changelog), but also upcoming presentations, references
   to conference papers, blog posts, etc.
1. Do we want a blog? [Example: pandas](https://pandas.pydata.org/community/blog/)
1. Do we want to highlight differences to other fairness toolkits anywhere?
1. Do we want to have an "ecosystem" page where we mention our relationship
   with other projects such as [InterpretML](https://github.com/interpretml)
1. Should we have a glossary?
   [Example: scikit-learn](https://scikit-learn.org/dev/glossary.html)
1. Do we want any kind of website analytics to figure out how users interact
   with the content? Given that this project's goal is to be about more than
   just code, we should have mechanisms to understand whether our educational
   material is actually useful (and used). Any suggestions?
1. [Currently using fairlearn.github.io] deploy through fairlearn.org
    - check that all pages are reachable through fairlearn.org
    - https for fairlearn.org (currently only http works)
