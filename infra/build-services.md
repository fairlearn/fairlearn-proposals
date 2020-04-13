# Choice of Pipelines

Currently, our pipelines run on [Azure Dev Ops Pipelines](https://dev.azure.com/responsibleai/fairlearn/_build) (hereafter ADO).
This was an easy default choice for a project growing out of work from Microsoft Research.
We were already familiar with ADO, and when we need more resources, linking to subscriptions owned by the AzureML team is straightforward.
However, using ADO could create a false perception as to the degree of control Microsoft exerts over Fairlearn.
We want Fairlearn to be a good open source citizen, and despite recent improvements Microsoft is not always perceived well in this regard.
Furthermore, while ADO and GitHub have federated authentication, ADO is still a separate site to visit, which can be jarring to users.

## An Alternative: GitHub Actions

GitHub offers a similar feature to ADO pipelines, named [GitHub Actions](https://github.com/features/actions).
We already use an Action to auto-assign reviewers, and have recently created a proof-of-concept Actions pipeline to run our current notebooks against our PyPI package (to catch breaking changes).

Being part of GitHub, switching to GitHub Actions will likely improve the perception of Fairlearn as a open source project (which happens to have major contributions from Microsoft).
One major barrier to this move is the lack of templating in defining Actions pipelines.
In ADO, we make heavy use of templating to eliminate duplicate code and have testing pipline which run our tests against multiple platforms and Python versions.
Not only does Actions lack templating, but there is no clear roadmap as to when they will be supported.

## Long Term Plan: Move to GitHub Actions

In the long term, moving our build and release pipelines to GitHub Actions appears to be the logical choice.
We will be a better open source citizen while keeping the same functionality.

## Short Term Plan: Planning the move

There are two basic ways of organising the move, both with their own advantages and disadvantages:
1. Leave existing ADO pipelines as-is, and add any new pipelines in GitHub Actions.
When Actions supports templates, migrate off ADO pipelines
1. Stick to ADO until GitHub Actions supports templates

Creating all new pipelines in GitHub Actions avoids creating more ADO debt to convert later.
However, this leaves us supporting two pipeline systems for an indefinite period.
Also, if the new pipelines have significant common steps, then Actions will require us to duplicate those until Actions has templating.

Keeping to ADO will have all our pipelines in a single place.
However, we will also have more pipelines to switch when we make the change.