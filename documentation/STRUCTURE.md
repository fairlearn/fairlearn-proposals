# Documentation

For the purpose of this proposal document we consider all of the following as documentation:

- documentation in our code within the Fairlearn repository which is currently also hosted on readthedocs
- Jupyter notebooks
- project website
- blog posts

Throughout the document we compare with the documentation of other popular projects such as 

- [scikit-learn](https://scikit-learn.org/)
- [pandas](https://pandas.pydata.org/)

## Goals

The documentation should be

- discoverable, ideally in a single place as opposed to multiple
- clear
- concise when describing individual pieces of functionality
- detailed when describing entire application scenarios, e.g. in the form of case studies
- available for the latest version, but if possible also for past versions ([example](https://scikit-learn.org/dev/versions.html))
- maintainable: it should be simple for maintainers to update/validate
- without ads (readthedocs always has ads that are shown alongside our documentation)

## Proposal

Like for most projects the website, [fairlearn.org](http://fairlearn.org) will be the central place to look for documentation.
From there visitors have various paths to explore content depending on what they are looking for, as detailed in the following subsections.

### Homepage

```
website
|--- Getting Started
|--- API reference
|--- User guide
|--- Case studies
|--- Community
```

### Getting Started

This page provides information on
- Installation
- terminology (or at least a link to it)
- where to go next, e.g. tutorials, videos, articles

Installation should cover various platforms, which should be very straightforward. Any reoccurring patterns in reported issues should be listed, as well as how to troubleshoot them. It may end up similar to [this guide](https://scikit-learn.org/dev/install.html)).

Example:

- [pandas](https://pandas.pydata.org/getting_started.html)
- [scikit-learn](https://scikit-learn.org/dev/getting_started.html)

### API reference

This is simply the generated documentation from our code. Currently we host this in readthedocs, but we want to include it on our webpage. A good example for this is [scikit-learn](https://scikit-learn.org/dev/modules/classes.html)

### User guide

The user guide explains all parts of Fairlearn by providing context that wouldn't fit into the code documentation such as mathematical derivations, but without using application-specific context (as we'd find it in the "case studies"). The guides are grouped by topic, e.g.

1. Assessment
    1. Metrics
        1. ...
    1. Visualizations
        1. ...
1. Mitigation
    1. Reductions methods
        1. Exponentiated Gradient
        1. Grid Search
    1. Postprocessing
        1. Threshold Optimizer

Examples:

- [scikit-learn](https://scikit-learn.org/dev/user_guide.html)
- [pandas](https://pandas.pydata.org/docs/user_guide/index.html)

Of our current notebooks the following may be most suitable as "user guides":

- [Group Metrics](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Group%20Metrics.ipynb) - a great example for something that should be a user guide
- [Grid Search for Binary Classification](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Grid%20Search%20for%20Binary%20Classification.ipynb) - the purpose is mostly to show Grid Search's functionality; perhaps it may need to be trimmed down to the essentials about Grid Search
- [Grid Search with Census Data](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Grid%20Search%20with%20Census%20Data.ipynb) - similar to the previous notebook it covers Grid Search; we could leverage some of this for a user guide for Grid Search, or alternatively for the dashboard visualizations

### Case studies

The purpose of fairness case studies is to walk through an application of Fairlearn in detail. Any application of a fairness toolkit needs to be done with great care while taking into account an entire range of concerns due to the sociotechnical nature of fairness. The showcased case studies will provide the space to cover scenarios in depth. The focus is not only on showing example usage of the Fairlearn toolkit, but on how to approach fairness in ML in general. We may want to add scenario even if it contains only few of Fairlearn's capabilities, but it otherwise demonstrates a great example of how to build AI responsibly.

All the case studies should be downloadable as Jupyter notebooks and/or Python source code, and be launchable in [Binder](https://mybinder.org/) and perhaps other platforms.

Note: [scikit-learn](https://scikit-learn.org/dev/auto_examples/index.html) refers to these as "Examples".

Of our current notebooks the following would be most closely aligned with this section:

- [Binary Classification on COMPAS dataset](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Binary%20Classification%20on%20COMPAS%20dataset.ipynb) - although we should perhaps consider removing it since it may not do justice to this complex setup
- [Binary Classification with the UCI Credit-card Default Dataset](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Binary%20Classification%20with%20the%20UCI%20Credit-card%20Default%20Dataset.ipynb)
- [Mitigating Disparities in Ranking from Binary Data](https://github.com/fairlearn/fairlearn/blob/master/notebooks/Mitigating%20Disparities%20in%20Ranking%20from%20Binary%20Data.ipynb)

### Community

This page should provide basic information about how to reach out in order to ask questions, provide feedback, report bugs or request features. Similarly, it should be clear how people can reach out if they want to contribute to Fairlearn, and where they can find small items to get started. [This](https://scikit-learn.org/dev/developers/contributing.html) is an example of how scikit-learn handles it through a contribution guide that is somewhat similar to ours that we currently have in the repo.

We could also write up the history of the project, its roadmap, and how we envision governance to work. The code of conduct should certainly be mentioned as well.

Some projects have a page showing the maintainers as well:

- [scikit-learn](https://scikit-learn.org/stable/about.html#people)
- [pandas](https://pandas.pydata.org/about/team.html)

## Required steps

To properly format math, text, and links across the pages we need to use ReST. Some of our existing documentation may have to be converted from Markdown to ReST, e.g. the terminology page.

We need to find a way to present the dashboard in a website where it can't be interactive. Perhaps with screenshots for the user guides, but the case studies are downloadable as Jupyter notebooks. [Could we perhaps pre-calculate all metrics and show the interactive dashboard in the case studies?]

We already have the examples gallery thanks to Adrin's work on sphinx-gallery: https://fairlearn.readthedocs.io/en/latest/index.html
There will be plenty of work to

- convert existing notebooks to case studies (or user guides)
- write all user guides
- write the installation guide

The repository structure should look similar to [what scikit-learn has](https://github.com/scikit-learn/scikit-learn/tree/master/doc):

- top-level doc directory has all the ReST files for the webpage except for API documentation and case studies
- API documentation comes directly from the code documentation
- case studies live in a separate top-level directory (scikit-learn calls it `examples`) as python files

Putting it all together:

- deploy through fairlearn.org
  - landing page will be provided by a designer, everything else from the Fairlearn project repositories
  - CI should generate documentation and make it available at PR-time for reviewers to check
  - all pages are reachable through fairlearn.org
  - https for fairlearn.org (currently only http works)
  - remove API doc from readthedocs
- ensure the navigation from homepage to the other sections works
  - manual testing?
  - automated testing? (also check for broken links)

## Development process

We need to document all the processes around generating documentation. Specifically, we need to document how one can

- build/generate the documentation and subsequently view it
- develop case studies that end up in python files, but perhaps while creating them using Jupyter
- get documentation changes into the Fairlearn repository (PR with Ci generating documentation and storing it as artifacts)

## Outstanding questions

1. Do we want users to cite us in any way? See [this example](https://scikit-learn.org/dev/about.html#citing-scikit-learn).
1. Do we want user testimonials? It definitely provides credibility (assuming users are willing)
1. Do we want a "News" section? It could list recent updates such as new versions (link to changelog), but also upcoming presentations, references to conference papers, blog posts, etc.
1. Do we want a blog? [Example: pandas](https://pandas.pydata.org/community/blog/)
1. Do we want to highlight differences to other fairness toolkits anywhere?
1. Do we want some sort of "Getting started" page?
1. Should we have a glossary? [Example: scikit-learn](https://scikit-learn.org/dev/glossary.html)
1. Should we have a FAQ section? [Example: scikit-learn](https://scikit-learn.org/dev/faq.html)
