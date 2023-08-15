---
name: GitHub Actions
dependsOn: [
]
tags: [github]
---

## Overview

With a GitHub repository there's a very easy way to set up CI that runs when your 
repository changes: simply add a [.yml file](https://learnxinyminutes.com/docs/yaml/) to your repository in the directory 

~~~
.github/workflows
~~~

Each file in this directory represents a *workflow* and will, when triggered, spin up a virtual machine and run the sequence of commands in the file.

Information about the specifications of these VMs can be found [here](https://docs.github.com/en/free-pro-team@latest/actions/reference/specifications-for-github-hosted-runners).
At the time of writing, each VM will have a 2-core CPU, 7GB of RAM and 14 GB of SSD space available, and each workflow can run for up to 6 hours.

These resources are all free for public repositories, and for private repositories you have a monthly quota of VM-minutes before any payment is required.

In this section you will create several workflows by using the wizard and built-in editor on the GitHub website.

## Creating a basic workflow

We will start with a minimal example to demonstrate various features of a GitHub Actions workflow. In your repository
root directory, create a new directory `.github`, and a new directory `workflows` within that, e.g. in Bash:

~~~bash
mkdir .github
mkdir .github/workflows
~~~

Next, create a new file 'basic.yml' in this directory, copy the following into it and save it, then commit and push the changes to GitHub.

~~~yml
name: Basic GitHub Actions Workflow

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  basic-job:
    runs-on: ubuntu-latest

    steps:
    - name: Run a one-line script
      run: echo "Hello, world!"
~~~

Here's a brief breakdown of this basic workflow:

1. The `name` field gives your workflow a name. This name will be displayed in the Actions tab of your GitHub repository.

2. The `on` field specifies when your workflow should run. In this case, it's configured to run on `push` events, `pull_request` events, and `workflow_dispatch` events.

3. The `jobs` field describes the jobs that make up your workflow. This workflow includes a single job called `basic-job`.

4. The `runs-on` field inside the `basic-job` job specifies the type of machine to run the job on. Here it is set to run on the latest available version of Ubuntu.

5. The `steps` field is a list of tasks to perform in the job. Each item in the list is a separate task. Here, we have one task that runs a basic shell command (`echo "Hello, world!"`). The `name` field gives the step a name that will be displayed in the GitHub Actions UI.

6. The `run` field specifies the command to run. Here, it's just echoing "Hello, world!" to the console.

If you now navigate to the *Actions* tab on your GitHub repository, you should see that this workflow has run and succeeded.
You can explore the run of this job by selecting `Basic workflow` in the list, then `basic-job` on the sidebar.
You can then drill down into each of the steps of the job and see the output at each of its stages.

In this case it was run because we just pushed a change.
We can also trigger this workflow by opening a pull request, or by navigating navigating to the workflow via the *Actions* tab and then selecting the *Run Workflow" dropdown (this is the `workflow_dispatch` trigger).

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
