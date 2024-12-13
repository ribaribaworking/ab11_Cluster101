# Code Reproducibility Guidelines

- Code reproducibility is an important part of the code development
- Since we use various development environments, we have to do our best to make the environments and code as reproducible as possible
- The most important thing is - **document everything**
    - This includes downloading databases, making references, setting up the environment, and *how-to* run the analysis or pipelines, etc.
- In an optimal case, we would use container images wherever possible - these are the closest things we can use for true code reproducibility
- **Avoid using locally** installed software on the server
    - They are usually tailored to the particular machine, and we do not have information about how they were installed, the dependencies, the exact version, etc.
    - This includes pre-installed packages on the CeMM cluster. It is very difficult to reproduce the environment on other machines but the CeMM cluster, such as when CeMM updates their servers to a different operating system, hardware, etc.
    - It is ok to use common Unix command-line tools, but even here, you have to be careful - not all Unix-based operating systems are built the same and follow the best practices (looking at you, MacOS) and they often do not have the same software versions (again, looking at you, MacOS)
- Control version of **everything**
    - Document your **code history** - version control (in our case, git and GitHub; other possibilities are GitLab or Bitbucket) as well as the **software version** you are using
        - For non-standard software, track the software version as detailed as possible - [Zenodo](https://zenodo.org/)/[Figshare](https://figshare.com/) DOIs, download links, git commits hashes, GitHub releases, versions, tags
    - Keep in mind the developers can change the releases, update tags, or delete the software completely - in such cases, it's a good idea to make a local copy and include this in the `git`
        - It is safer to use Zenodo/Figshare DOIs because these don't change after they are published and are hosted on an independent platform (sort of) - it is more difficult to remove the code
        - On GitHub, for example, the code owner can rename the repository, delete it, change tags, etc., but also other *called* dependencies can be deleted, broken links, etc. - getting code from GitHub is not a safe way for reproducible and version-controlled code development
    - If, for some reason, you struggle with using a specific software version combination, keep a particular version for the most essential tools and remove the version or the *not-so-critical* software (for example, recording the `wget` version might not be as crucial as `bwa`) - use with caution!
        - This is sometimes a problem in `conda` environments

## High-level Host Information

- Record *high-level* host information in the README:
    - On Linux: `cat /etc/os-release` shows the operating system version
- Record *main* software versions (unless documented otherwise) in the README:
    - `docker --version`
    - `apptainer --version`
    - `conda --version`
- Record the status of the *actual* development environment as part of the output/environment files:
    - R: `sessionInfo()` (at the end of the R script) save the list of loaded packages and their versions
    - `conda`: `conda env export --no-builds > environment.yml` to export all the installed software versions including automatically installed dependencies or `conda env export --from-history > environment.yml` to export only *manually* installed software versions
	    - Note: `conda`: `conda env export > environment.yml` to export **detailed** software version including builds in the current `conda` environment

## Docker, Apptainer, Singularity Images

- Currently, the most common containerization tools are Docker and Apptainer (ex Singularity)
- For full (almost) reproducibility, the **complete** development environment should be defined in `Dockerfile`/`.def` files with instructions on how to build the image
- Running containers has to be supported by the infrastructure, which is often the biggest issue on locally hosted servers
    - For example, [Rootless mode for Docker](https://docs.docker.com/engine/security/rootless/), [podman](https://podman.io/), or [Fakeroot for Apptainer](https://apptainer.org/docs/user/main/fakeroot.html)
- On systems supporting Docker/Apptainer images, we might encounter container/image management problems
    - Both Docker and Apptainer tend to keep many temporary, dangling images, unused containers, cache files, etc, causing storage space issues
    - Note: [kubernetes](https://kubernetes.io/) is the industry standard for large-scale container management that can handle a lot of management issues
- Containers can also be run remotely on the cloud
    - Properly set `Dockerfile/.def` files can be used to create [IDE container-based cloud environments](https://www.youtube.com/watch?v=bHhYBt1BYaU) in [GitHub Codespaces](https://github.com/features/codespaces/), [Gitpod](https://www.gitpod.io/) or [Devpod](https://devpod.sh/)
    - These make it much easier to run, test, and distribute the code because everybody recreates the environment *from scratch* on an independent system
    - The environments can be accessed by multiple users from different access points, making it very easy to use and interact with
- Keep a certain level of granularity - it is often better to have multiple smaller images than one large one

### Base Image

- It is beneficial to decide on **one primary base image** and use it as often as possible
    - The same applies to other development environment, such as R
- Optimally, the base image(s) should be supported by [GitHub hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners#standard-github-hosted-runners-for--private-repositories) to provide future compatibility with testing
- When specifying base images, prefer [`sha256` digest method](https://docs.docker.com/reference/cli/docker/image/pull/#pull-an-image-by-digest-immutable-identifier) (also called **pins**) over image tags
    - This is much more *stable* than using Dockerhub tags - tags can be updated, changed, or deleted by the author
    - "pins" *pin* an image to a specific version in time. Docker does, therefore, not pull updated versions of an image, which may include updates. You must change the digest accordingly if you want to pull an updated image
    - Unfortunately, there is no way to get the digest hash for the previous pushes - you have to record it at the time when you put together the `Dockerfile`
- Using:

```dockerfile
FROM rocker/tidyverse@sha256:e8ecf00ce5fa2fe45a5bb9be6b9afa0b7ac7658354de9be4d17352ae30504e5a AS build
```

- is **strongly preferred** over:

```dockefile
FROM rocker/tidyverse:4.4
```

#### Current Base Images

- **Primary base image** (general code development, `Python`, `snakemake`, `nextflow`): **[Ubuntu 22.04 LTS](https://hub.docker.com/layers/library/ubuntu/22.04/images/sha256:0e5e4a57c2499249aafc3b40fcd541e9a456aab7296681a3994d631587203f97)**  (for R and RStudio base image compatibility - see below)
    - `ubuntu:22.04@sha256:0e5e4a57c2499249aafc3b40fcd541e9a456aab7296681a3994d631587203f97`
- **R and RStudio**: **[rocker/tidyverse:4.4](https://hub.docker.com/layers/rocker/tidyverse/4.4/images/sha256-dc30c36557a708982795a5cbde5cd36db4a2663dc464686031bfd73c19f2af77)**
    - `rocker/tidyverse:4.4@sha256:e8ecf00ce5fa2fe45a5bb9be6b9afa0b7ac7658354de9be4d17352ae30504e5a`

### Software and Dependencies

- Base images usually contain only the most important code base
    - This means we have to install additional dependencies (additional software)
- It is recommended to keep additional software tools and their versions separately from the main `Dockerfile`/`.def` code
    - This makes it easier to reuse the base code for new projects without any (~many) changes and automate builds
    - It makes it easier to modify and add the dependencies without modifying the main code
    - It is much easier to keep the code history
    - The `Dockerfile` code is much cleaner and easier to read

#### System Packages

- Keep the added software dependencies **including the specific versions** in the [**`packages.txt`**](examples/packages.txt) file including the software versions
- In Linux, prefer **`release`** (**`main`** or **`universe`** with **no *prefix***) software versions - these are *frozen* and do not change in additional releases
    - Note: `security`,  `updates`,  `proposed`, and  `backport` might change over time
        - More information about the differences [here](https://askubuntu.com/a/1280725/1013568)
- To get the right software version (`release` - `main` or `universe`), you can search `apt-get update` caches in the interactively open Docker container
    - For `python3`, the desired package version in `Ubuntu 22.04 LTS` is **`python3=3.10.4-0ubuntu2`**:

```shell
# In the Docker base image interactive window
apt-get update
apt-cache madison python3
```

results in:

```shell
Reading package lists... Done
   python3 | 3.10.6-1~22.04.1 | http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages # Do NOT use this one - updates package
   python3 | 3.10.6-1~22.04 | http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages # Do NOT use this one - security package
   python3 | 3.10.4-0ubuntu2 | http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages # USE this one - main (~release) package
```

- Sometimes, you have **collisions** in package versions and you have to specify additional software versions
    - This has to be done manually based on the output of the `apt-get install` command
    - If this still doesn't work, you can add `--allow-downgrades` to the `apt-get install` command, but you should try to avoid this if possible
- **Do not** install system recommended but unused packages
    - Use **`--no-install-recommends`** as part of the `apt-get install` command
- Copy the `packages.txt` file during the build and install the additional software with:

```dockerfile
COPY packages.txt .
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive xargs apt-get install -y --no-install-recommends < /packages.txt
```

#### Python Requirements and Libraries

- Keep the added Python packages in a separate file [**`requirements.txt`**](examples/requirements.txt)
    - This makes the build more flexible and clean
    - It also makes it compatible with [tests on GitHub](https://docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-python#requirements-file)
- Copy the `requirements.txt` file during the build and install the additional requirements and libraries with:

```dockerfile
COPY requirements.txt .
RUN pip3 install --upgrade pip && \
    pip3 install --no-cache-dir -r /requirements.txt
```

#### Other Software

- Keep all the additional software (not included in the standard repos, not part of Python or R requirements in a separate file [**`soups.txt`**](./examples/soups.txt)
    - Note: SOUP = [**SO**ftware of **U**nknown **P**edigree](https://en.wikipedia.org/wiki/Software_of_unknown_pedigree)
- Installing is more elaborate, but you can use for example:

```dockerfile
RUN bcftools_version=$(grep -w bcftools /soups.txt | cut -d "=" -f2) && \
    wget -O /bcftools-${bcftools_version}.tar.bz2 "https://github.com/samtools/bcftools/releases/download/${bcftools_version}/bcftools-${bcftools_version}.tar.bz2" && \
    tar -xvjf bcftools-${bcftools_version}.tar.bz2 && rm bcftools-${bcftools_version}.tar.bz2 && \
    cd /bcftools-${bcftools_version}/ && \
    ./configure --prefix=/usr/local && make && make install && cd / ...
```

#### R Packages

- Keep all R packages, Bioconductor packages, and R-related package requirements in separate files:
    - Software requirements for **CRAN R libraries**: [**`requirements_R.txt`**](./examples/requirements_R.txt)
    - **CRAN R required packages**: [**`packages_R.txt`**](./examples/packages_R.txt)
    - Software requirements for **Bioconductor libraries**: [**`requirements_R_bioc.txt`**](./examples/requirements_R_bioc.txt)
    - **Bioconductor required packages**: [**`packages_R_bioc.txt`**](./examples/packages_R_bioc.txt)
    - **GitHub-hosted R** libraries: [**`requirements_R_github.txt`**](./examples/requirements_R_github.txt)
    - **Other R** libraries: [**`requirements_R_soap.txt`**](./examples/requirements_R_soap.txt)
- Keeping R-related software package dependencies separately helps to reuse the *main* packages (non-R-related) in other image builds

## Conda

- `conda` (or `mamba`) is a great tool to quickly create development environments
- Unfortunately, `conda` is not particularly good in reproducible and transferable environment setups as it heavily depends on the host environment and uses many host files (libraries)
- We can improve on this by using *strict* settings while setting up `conda` environments

### Environment Settings Recommendations

- Here are some recommendations to make `conda` environments setup more robust:

 1. **Disable** `conda` **auto-updates** `conda config --set auto_update_conda false`
 2. Set **strict channel priority** for the `base` environment (see example below) or install with `conda` channels directly in the command line `--strict-channel-priority --override-channels -c conda-forge -c bioconda`

```shell
conda config --add channels r
conda config --add channels anaconda
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge

conda config --set channel_priority strict
```

 3. **Prefer online packages** instead of local ones `conda config --set offline false`
 4. Keep the original `conda create` command with installed package versions

- Exported `conda` environment `YAML` is often very host-specific and it is not possible to reproduce the environment on a different host

 5. Set `conda` to **copy packages** instead of linking them from another environment with `conda config --set always_copy true` or by adding `--copy` to `conda create`

- This is more to help with `conda` environment installation breaking while updating some other `conda` packages during the development

## R

- Consider using [`{renv}`](https://rstudio.github.io/renv/articles/renv.html) for independent reproducible R software environments
	- For example, you can have a single standardized R docker image with `{renv}`, install and track	packages with `{renv}` for each project. Once the project is done and you need to preserve the software versions, you can create a separate docker image with all the software preinstalled
	- Note: Not all R packages are always preserved and sometimes they are even removed from CRAN/Bioconductor - only using `{renv}` is **not enough** for future reproducibility
	- Alternative to `{renv}` is [`{groundhog}`](https://groundhogr.com/)
		- *tldr; academics should probably use groundhog, corporate data scientists should arguably use renv.*
		- Full comparison of `{renv}` and `{grounhog}` [here](https://groundhogr.com/renv/) 
- Consider using [`{usethis}`](https://usethis.r-lib.org/index.html) for automatization of repetitive tasks and project setup
	- `{usethis}` automates repetitive tasks that arise during project setup and development, both for R packages and non-package projects
- You should also use *classic* `sessionInfo()` to report the currently loaded packages
	- `sessionInfo() |> report::report()`
- For RStudio users - use [`.Rproj`](https://support.posit.co/hc/en-us/articles/200526207-Using-RStudio-Projects) files and combine it with [`{here}`](https://here.r-lib.org/) package
	- This make your *analysis* directory into *project* directory for better path tracking and setting
- Nice summary of reproducible R development [Building reproducible analytical pipelines with R](https://raps-with-r.dev/)

## Workflow Managers

- Another step close to full reproducibility is to use workflow managers
- The most used (bioinformatics) workflow managers are [`snakemake`](https://snakemake.readthedocs.io/) and [`nextflow`](https://nextflow.io/)
- Both `snakemake` and `nextflow` are often based on `conda` environments and Docker/Apptainer images, so the same rules as in [`conda`](#conda) apply here as well

### `snakemake`

- TODO

### `nextflow`

- TODO
