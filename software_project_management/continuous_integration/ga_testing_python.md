---
name: Using GitHub Actions with Python
dependsOn: [
    software_project_management.continuous_integration.github_actions
]
tags: [github]
---

## Creating a Python-specific workflow

Now let's do something more useful.

Navigate to the GitHub *Actions* tab and then click *New Workflow* (near the top left).
This will let us start with a preset workflow containing many of the elements we are interested in.

Search for "Python package" sand select the following workflow by pressing *Configure*:

~~~
Python package
By GitHub Actions

Create and test a Python package on multiple Python versions.
~~~

This takes us into the web editor.
We will make the following changes to the workflow:

1. change the name to "Python versions", and the filename to `python_versions.yml`

1. add the `workflow_dispatch` trigger, just like in the basic file

1. Change the last line to `python -m pytest -m tests/test_models.py`, so it runs

Then use the web interface to commit the changes.
Go over to the *Actions* tab to see it running.

Let's go through what is happening in this workflow:

- The name of this workflow is `Python versions`. It runs whenever there's a push to the `main` branch, a pull request targeting the `main` branch, or on a `workflow_dispatch` trigger.

- This workflow only has one job, named `build`, and it runs on the latest version of Ubuntu.

- This job utilizes a strategy called a matrix, which allows you to run the same job with different configurations.
In this case, it's set to run the job with three different Python versions - "3.8", "3.9", and "3.10".
The `fail-fast` option is set to `false`, which means that if one version fails, the other versions will continue to run.

This job consists of a series of steps:

1. **Checkout Code:** The first step uses an action, `actions/checkout@v3`, which checks out your repository's code onto the runner, so the job can access it.

2. **Set Up Python:** The next step uses another action, `actions/setup-python@v3`, to set up a Python environment with the version specified in the matrix.

3. **Install Dependencies:** The third step runs a series of shell commands to install Python dependencies. It first updates pip, setuptools, and wheel, then installs `flake8` and `pytest`. Next, if a `requirements.txt` file is present (which there should be in our current repository), then it loads those dependencies using `pip` as well.

4. **Lint with flake8:** The fourth step runs the `flake8` linter to check the code for styling errors. [flake8](https://flake8.pycqa.org/en/latest/) is a tool for enforcing Python's PEP 8 style guide, and it can find many different types of common problems with your code. You can check the `flake8` configuration for this project in the `.flake8` file in the repository.

5. **Test with pytest:** The last step runs `pytest` to execute the tests.

Take a look at the job under the `Actions` tab of the repository, and drilling down into the Pytest step output, you can see the
results of the tests having been run.

:::callout
## What about other Actions?

Our workflow here uses standard GitHub Actions (indicated by `actions/*`).
Beyond the standard set of actions,
others are available via the
[GitHub Marketplace](https://docs.github.com/en/developers/github-marketplace/github-marketplace-overview).
It contains many third-party actions (as well as apps)
that you can use with GitHub for many tasks across many programming languages,
particularly for setting up environments for running tests,
code analysis and other tools,
setting up and using infrastructure (for things like Docker or Amazon's AWS cloud),
or even managing repository issues.
You can even contribute your own.
:::

## Create a workflow on multiple operating systems

Now let's create a similar workflow in a file called `os_versions.yml` that tests the code on a fixed python version, but this time on multiple operating systems. Add the following to this new file and save it:

~~~yml
# This workflow will install Python dependencies, run tests and lint with a variety of Python versions
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python

name: Operating systems

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:

    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]

    runs-on: ${{ matrix.os }}
  
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python 3.10
      uses: actions/setup-python@v3
      with:
        python-version: "3.10"
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip setuptools wheel
        python -m pip install -r requirements.txt
    - name: Test with pytest
      run: |
        python -m pytest tests/test_models.py
~~~

So here, we do something similar with a matrix, but apply it to operating systems instead.

::::challenge{id="multiple-python-os" title="Multiple OS's and Multiple Python Versions"}

Amend the workflow above (or create a new one) to also run over the same three versions of Python we had in our previous workflow.
How many jobs will this create?

:::solution

~~~yml
name: OSs and Python versions

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:

    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.9", "3.10", "3.11"]
        os: [ubuntu-latest, macos-latest, windows-latest]

    runs-on: ${{ matrix.os }}

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v3
      with:
        python-version: "${{ matrix.python-version }}"
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip setuptools wheel
        python -m pip install -r requirements.txt
    - name: Test with pytest
      run: |
        python -m pytest tests/test_models.py
~~~

A total of 9 jobs will run, since we are using two variables in our matrix, each with three entries.
Hence, our code will run the tests for each specified version of Python for each of the specified operating systems.
Far easier than testing this manually!
:::

::::

## Next steps

1. \[optional\] read more about [GitHub's hosted runners](https://docs.github.com/en/free-pro-team@latest/actions/reference/specifications-for-github-hosted-runners).

1. \[optional\] read more about the [syntax for GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions).
