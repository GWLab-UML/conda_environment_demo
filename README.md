# Managing Environments with Conda

Dependency management of software is annoying.
For example, you might be using a Python library that was developed against version 3.8 of Python, but then want to use a different package that requires version 3.10.
Moreover, when working on Unity or other managed systems (or even your own!!!), you may not want to nor be able to install dozens of different packages at the system level that might conflict with each other.

To get around this, we use `virtual environments` that allow us to install packages in isolated containers that are independed of the host system.
[Conda](https://docs.conda.io/en/latest/) is one such environment and package management system widely used in the scientific community.
Conda has a multitude of `channels` for installing software from different collections (think of them as apt sources, or big ol' libraries).
[Bioconda](https://bioconda.github.io/index.html) is a Conda channel that has a good number of packages related to the work we do.

This repo is an example setup for persisting Conda environments _alongside_ the code that uses them.
This way, different users can be assured they are using the same setup when running the same notebooks/tools.
It will also make moving between your personal computer and shared servers more simple.

### Getting Started

Follow the directions to get [get started with Conda](https://docs.conda.io/projects/conda/en/stable/user-guide/getting-started.html).
We recommend following the tutorials for familiarizing yourself with the Conda CLI.

You can optionally setup the [Bioconda](https://bioconda.github.io/index.html#usage) channel.
Though as you will see below, it may be preferable to not configure that source list globally.

### Environment Storage

In a nutshell, when you run something like:

```
$ conda create --name myenv python=3.11
$ conda activate myenv
```

Conda sets up a brand new environment in your default directory (usually the install dir).
For instance, on my machine this new environment, along with all its dependencies are installed in:

```
$ ls ~/miniconda3/envs/myenv/
bin  compiler_compat  conda-meta  include  lib  man  share  ssl  x86_64-conda_cos7-linux-gnu  x86_64-conda-linux-gnu
```

As you can see, just installing one version of python takes up a lot of space!
```
$ du -sh ~/miniconda3/envs/myenv/
228M    /home/eholum/miniconda3/envs/myenv/
```

Installing more packages in this environment folder will just blow that up, e.g:
```
$ conda install fastqc
...
$ du -sh ~/miniconda3/envs/myenv/
689M    /home/eholum/miniconda3/envs/myenv/
```

Moreover, the _details_ of this environment are isolated to my local conda install, as everything is persisted in my home directory.
To get around this, we are going to setup a _local_ environment folder and configuration file that will let us commit the Conda environment right alongside our code.

### Using Prefixes and Yaml

Rather than doing everything from the CLI as one offs, we can use [environment files](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#sharing-an-environment) to save and share  setups.
In this repo, we have setup an [environment.yml](environment.yml) file for just that purpose:

```
channels:
  - conda-forge
  - bioconda
  - defaults
# It's always good to be in the habbit of hardcoding versions, if you can
dependencies:
- python=3.11
- fastqc
```

For this environment we want both `python` and `fastqc`, the latter coming from the `bioconda` channel.
Note the advantage here!
Rather than specifing channels at the _global_ level in `~/.condarc`, you can have them on a per-project basis.

To create an environment and put everything into a local folder, we have to use `prefixes` rather than names.
To create and run the environment:

```
$ conda env create -p ./environment
```

This will create a new environment and put everything into the `./environment` folder at the root of this repo, rather than your home directory.
To activate from the root of the repo run:

```
# From the repo root
$ conda activate ./environment

# Or from somewhere else on the system you must use the full path
$ conda activate <FULL_PATH_TO_REPO>/environment
```

To remove the environment
```
$ conda deactivate
$ conda env remove -p ./environment
```

Now, if you ever need to install new packages you can add it as a line in the `dependencies` section and just run `conda env update`.

**NOTE**: There is a `./gitignore` file in the root of the repo that specifically excludes everything in the `./environment` folder from being committed. It is important to use that prefix!