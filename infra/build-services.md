# Choice of Build Pipelines

Currently, our builds run on [Azure Dev Ops Pipelines](https://dev.azure.com/responsibleai/fairlearn/_build).
This was an easy default choice for a Microsoft-sponsored project.
However, it could create a false perception as to the degree of control Microsoft exerts over Fairlearn.
We want Fairlearn to be a good open source citizen.

## An Alternative: GitHub Actions

GitHub offers a similar feature to ADO pipelines, named [GitHub Actions](https://github.com/features/actions).
We already 