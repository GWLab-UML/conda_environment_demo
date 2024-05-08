# Managing Environments with Conda

Dependency management of software is annoying.
For example, you might be using a Python library that was developed against version 3.8 of Python, but then want to use a different package that requires version 3.10.
Moreover, when working on Unity or other managed systems (or even your own!!!), you may not want to nor be able to install dozens of different packages at the system level that might conflict with each other.

To get around this, we use `virtual environments` that allow us to install packages in isolated containers that are independed of the host system.
[Conda](https://docs.conda.io/en/latest/) is one such environment and package management system widely used in the scientific community.
Conda has a multitude of `channels` for installing software from different collections (think of them as apt sources, or big ol' libraries).
[Bioconda](https://bioconda.github.io/index.html) is a Conda channel that has a good number of packages related to the work we do.
[conda-forge](https://conda-forge.org) is another that has a huge number of packages, including many pre-compiled R binaries.

This repo is an example setup for persisting Conda environments _alongside_ the code that uses them.
This way, different users can be assured they are using the same setup when running the same notebooks/tools.
It will also make moving between your personal computer and shared servers more simple.

## Getting Started

Follow the directions to get [get started with Conda](https://docs.conda.io/projects/conda/en/stable/user-guide/getting-started.html).
We recommend following the tutorials for familiarizing yourself with the Conda CLI.

There are many different distrubutions, but I personally recommend using [miniconda3](https://docs.anaconda.com/free/miniconda/index.html) since it's quick and easy to setup.

## Building and Using Environments

Conda environments are folders that contain specific collections of packages that are installed and managed with `conda`.

### Environment Storage

In a nutshell, when you run something like:

```
$ conda create --name myenv python=3.12
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

### Configuring Environments with Yaml and Prefixes

Rather than doing everything from the CLI as one offs, we can use [environment files](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#sharing-an-environment) to save and share setups.
It is **strongly** encouraged to use yaml to configure environments since it makes sharing, debugging, and reproducing work much, much easier.

In this repo, we have setup an [environment.yml](environment.yml) file for just that purpose.

```
# Recommend using channel_priority: strict to avoid conflicts
channels:
  - conda-forge
  - bioconda
  - defaults
# When possible, we recommend hardcoding versions
dependencies:
- python=3.12

# Use the latest version of jupyter
- jupyter=1.0.0
- ipykernel=6.28.0

# We're going to be using R in a juypter environment, so setup necessary packages
- r-base=4.3.3
- r-irkernel=1.3.2

# Install other packages as necessary, when using conda this is preferable for
# avoiding dynamic linking issues
- r-tidyverse=2.0.0
- r-biocmanager=1.30.22
- bioconductor-enhancedvolcano=1.20.0
```

In this demo we're going to setup an isolated jupyter and R environment in a local folder.
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
For instance, if you needed both `fastqc` and `multiqc` then add that line:

```
$ cat environment.yml
...
dependencies:
- python=3.12
...
- fastqc
- multiqc
$ conda env update
```

**NOTE**: There is a `./gitignore` file in the root of the repo that specifically excludes everything in the `./environment` folder from being committed.
It is important to use that prefix!

## Using Conda with Jupyter and R

Once you've built and activated your environment, you can install the required R kernel and run `jupyter lab` to check the demo.

```
# First install the Kernel
$ R -e 'IRkernel::installspec()'

# Start Jupyter and checkout the `Sample-R` notebook.
# jupyter lab
```

### Gotchas for MacOS

On later versions of MacOS, you may run into compiler or dynamic linking issues compiling R packages from source.
For example, when using `install.packages(...)` or `BiocManager::install(...)`, you may see errors like:

```
In file included from ./data.table.h:2:
./dt_stdio.h:30:12: fatal error: 'stdio.h' file not found
   30 |   #include <stdio.h>
```

If possible, we recommend installing any required R packages from pre-built binaries using Conda to avoid the need to compile dynamically linked libraries from scratch.
For instance in this repo we have installed `tidyverse` and `biocmanager` directly from the conda-forge channel,
and `EnhancedVolcano` directly from the bioconda channel.
This way you do not need to bother mucking around with commands above.

Reminder that most R packages are prefixed by `r-`, and to search for a specific package:

```
$ conda search -c <channel-name> <package name>

# E.g.
$ conda search -c conda-forge *tidyverse*
```

This will give you all available versions.

**NOTE**: It is certainly possible that a particular package/version that you need for an analysis was not released to any channel.
If that is the case you may be stuck building for source, and your options for doing that in a Conda environment may be limited.
