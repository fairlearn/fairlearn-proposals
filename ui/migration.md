Proposal to move the existing UI code to a separate repo
========================================================

Current situation
-----------------

The existing UI in the `visualization` directory is published to `npm` as
`fairlearn-dashboard` and consumed in the `fairlearn.widget` module.
Users can view the Fairlearn dashboard through Python code in a Jupyter
notebook.

Problems
-------

1. The current setup with both Python and Typescript code in the same
   repository has lead to confusion for developers who assumed that changes
   to the visualization code would automatically be reflected in the widget.
2. Several members of the community have voiced concerns with the dashboard
   living alongside the "core" Python code. These included:
   - dashboard is written in Typescript, not Python
   - dashboard has lots of dependencies that are unrelated to Python
   - dashboard shouldn't be part of Fairlearn at all
   - even if it's part of Fairlearn, it should be a dependency and not live in
     the same repo
3. The dashboard was built with much more complex requirements in mind than
   perhaps required for just the open source package. This is due to its use
   within Azure Machine Learning Studio.
4. The Python code has been much more open to contributions than the
   Typescript code. The Typescript code lacks documentation and contributor
   guide.
5. Several people mentioned the need to have context-specific visualizations
   as opposed to a fixed dashboard. From a project/website perspective this
   would allow us to think of appropriate visualizations per example notebook
   as opposed to providing a general purpose dashboard with the package.

Proposal
--------

To address the concerns outlined above we propose to

1. Remove the `visualization` code from Fairlearn.
2. Remove the `fairlearn.widget` module from the `fairlearn` package.
3. Move these pieces into a public repository under the `microsoft`
   organization on GitHub (with MIT license as before).
4. Publish a `fairlearn-dashboard` package from the repo in the `microsoft`
   organization that can be installed separately if desired.

In general, this opens up Fairlearn to be the "core" library around which
anyone can build any type of UI that they find useful.
Additionally, the new UI repository will be open source and accept
contributions as long as they comply with the somewhat stricter requirements
due to the cloud integration.
