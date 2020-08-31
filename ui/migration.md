# Proposal regarding the future of UI development in the Fairlearn project

## Current situation

The UI lives in the Fairlearn repository alongside the Python code.
The existing UI in the `visualization` directory is published to `npm` as
`fairlearn-dashboard` and consumed in the `fairlearn.widget` module of the
`fairlearn` Python package.
Users can view the Fairlearn dashboard through Python code in a Jupyter
notebook.
Notably, this is currently the only UI for Fairlearn and is also used in
Azure Machine Learning Studio.

## Problems

1. Several members of the community have voiced concerns with the dashboard
   living alongside the "core" Python code. These included:
   - dashboard is written in Typescript, not Python
   - dashboard has lots of dependencies that are unrelated to Python
   - dashboard shouldn't be part of Fairlearn at all
   - even if it's part of Fairlearn, it should be a dependency and not live in
     the same repo
   - developers assumed that changes to the visualization code would
     automatically be reflected in the widget (which they're not)
2. The dashboard was built with much more complex requirements in mind than
   perhaps required for just the open source package. This is due to its use
   within Azure Machine Learning Studio. Some of these requirements include:
   - localization (support for various languages)
   - accessibility requirements
   - styling (Office Fabric UI) and light/dark/high contrast modes
3. The Python code has been much more open to contributions than the
   Typescript code. The Typescript code lacks documentation and contributor
   guide.
4. Several people mentioned the need to have context-specific visualizations
   as opposed to a fixed dashboard. From a project/website perspective this
   would allow us to think of appropriate visualizations per example notebook
   as opposed to providing a general purpose dashboard with the package.

## Proposal

To address the concerns outlined above we propose to remove the existing
interactive visualization dashboard from the Fairlearn repository and package.
This opens up Fairlearn to be the base library that defines metrics around
which anyone can build any type of UI that they find useful.
This supports the idea that fairness is context-specific and allows people
to build the kind of UI that makes sense in their application context.

Update: Kevin created a [PR](https://github.com/fairlearn/fairlearn/pull/561)
to add `matplotlib` based plots to Fairlearn which provide some of the
functionality of the existing interactive visualizations.
Any more complex type of visualization should live in a separate repo and
can build on top of the Fairlearn package.
Such a repo can live within the `fairlearn` organization on GitHub provided
the authors are committed to maintaining it, which includes taking care of

- PRs
- issues
- releases
- communication / coordination

As usual anyone can always fork any existing repositories and contribute
improvements back.
However, having a shared repository for visualization development could
enhance collaboration within the community.
For that reason we encourage anyone who's interested to reach out via
[Gitter](https://gitter.im/fairlearn/community).

### The future of the existing interactive visualizations

We plan to carry out the following steps for the existing visualization code:

1. Remove the code in the existing `visualization` directory from Fairlearn.
2. Remove the `fairlearn.widget` module from the `fairlearn` package.
3. Remove existing Fairlearn dashboard from Fairlearn website, unless any
   example notebooks require it. See *endorsements* at the end of the
   proposal for a more general view on UIs and notebooks.
4. Move these pieces into a public open source repository under the
   `microsoft` organization on GitHub (with MIT license as before).
   This is to better signal that the package needs to abide by somewhat
   stricter Microsoft cloud integration requirements.
   That said, contributions would still be welcomed and Roman will add
   clear guidance on how to contribute as well as any additional requirements.
5. Publish a Python package (Name TBD) from the repo in the `microsoft`
   organization that can be installed separately if desired.

### Endorsements

The Fairlearn project could endorse certain UIs and perhaps highlight them on
its webpage through example notebooks.
The community can decide to set up a set of requirements for such endorsements
if it's considered beneficial.
This might include that the visualizations have to render properly on the
website and provide utility beyond that the basic `matplotlib` visualizations.
