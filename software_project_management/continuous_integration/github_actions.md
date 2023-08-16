---
name: GitHub Actions
dependsOn: [
]
tags: [github]
---

## Overview

The automated testing we've done so far only takes into account the state of the repository we have on our own machines. In a software project involving multiple developers working and pushing changes on a repository, it would be great to know holistically how all these changes are affecting our codebase without everyone having to pull down all the changes and test them. If we also take into account the testing required on different target user platforms for our software and the changes being made (to potentially many repository branches), the effort required to conduct testing at this scale can quickly become intractable for a research project to sustain.

Continuous Integration (CI) aims to reduce this burden by further automation, and automation - wherever possible - helps us to reduce errors and makes predictable processes more efficient. The idea is that when a new change is committed to a repository, CI clones the repository, builds it if necessary, and runs any tests. Once complete, it presents a report to let you see what happened.

There are many CI infrastructures and services, free and paid for, and subject to change as they evolve their features. We'll be looking at [GitHub Actions](https://github.com/features/actions) - which unsurprisingly is available as part of GitHub.


## CI With GitHub Actions

### A Quick Look at YAML

YAML is a text format used by GitHub Action workflow files.
It is also increasingly used for configuration files and storing other types of data,
so it's worth taking a bit of time looking into this file format.

[YAML](https://www.commonwl.org/user_guide/yaml/)
(a recursive acronym which stands for "YAML Ain't Markup Language")
is a language designed to be human readable.
A few basic things you need to know about YAML to get started with GitHub Actions are
key-value pairs, arrays, maps and multi-line strings.

So firstly, YAML files are essentially made up of **key-value** pairs,
in the form `key: value`, for example:

~~~
name: Kilimanjaro
height_metres: 5892
first_scaled_by: Hans Meyer
~~~
{: .language-yaml}

In general, you don't need quotes for strings,
but you can use them when you want to explicitly distinguish between numbers and strings,
e.g. `height_metres: "5892"` would be a string,
but in the above example it is an integer.
It turns out Hans Meyer isn't the only first ascender of Kilimanjaro,
so one way to add this person as another value to this key is by using YAML **arrays**,
like this:

~~~
first_scaled_by:
- Hans Meyer
- Ludwig Purtscheller
~~~
{: .language-yaml}

An alternative to this format for arrays is the following, which would have the same meaning:

~~~
first_scaled_by: [Hans Meyer, Ludwig Purtscheller]
~~~
{: .language-yaml}

If we wanted to express more information for one of these values
we could use a feature known as **maps** (dictionaries/hashes),
which allow us to define nested, hierarchical data structures, e.g.

~~~yml
...
height:
  value: 5892
  unit: metres
  measured:
    year: 2008
    by: Kilimanjaro 2008 Precise Height Measurement Expedition
...
~~~

So here, `height` itself is made up of three keys `value`, `unit`, and `measured`,
with the last of these being another nested key with the keys `year` and `by`.
Note the convention of using two spaces for tabs, instead of Python's four.

We can also combine maps and arrays to describe more complex data.
Let's say we want to add more detail to our list of initial ascenders:

~~~yml
...
first_scaled_by:
- name: Hans Meyer
  date_of_birth: 22-03-1858
  nationality: German
- name: Ludwig Purtscheller
  date_of_birth: 22-03-1858
  nationality: Austrian
~~~

So here we have a YAML array of our two mountaineers,
each with additional keys offering more information.

GitHub Actions also makes use of `|` symbol to indicate a multi-line string
that preserves new lines. For example:

~~~yml
shakespeare_couplet: |
  Good night, good night. Parting is such sweet sorrow
  That I shall say good night till it be morrow.
~~~

They key `shakespeare_couplet` would hold the full two line string,
preserving the new line after sorrow.

As we'll see shortly, GitHub Actions workflows will use all of these.

## Defining our First CI Workflow

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

