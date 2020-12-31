# Continuous Integration (CI)


CI Integration allows us to automatically build and test the codebase itself as well as any documentation. This is powerful because it enables us to automatically do the following:

For our base package:
- **BUILD**: Test whether our package is build-able
- **TEST**: Run tests on our package to ensure that it will work as intended
- **DEPLOY**: Automatically generate a PyPI package

For our documentation:
- **BUILD**: Test whether our documentation builds successfully
- **DOCTEST**: We plan on adding `sphinx doctests` in order to validate the example code that we inject into our docstrings
- **DEPLOY**: Deploy in this context refers to publishing our successfully built and tested documentation HTML pages to GitHub Pages as versioned documentation 

For the `niviz` package we'll be using GitHub actions in order to perform CI

## NiViz GitHub Actions Workflows

### Documentation

#### Builds and Doctests

Documentation should be built with every push to the repository, in addition tests should also be performed on every push to the repository

