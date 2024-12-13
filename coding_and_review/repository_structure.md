# Repository Structure Guidelines

## General Repository Structure

- Standardizing project (~repository) structure and code organization further improves code development clarity
- In general, include `README.md` with more information about the code or directory structure as often as necessary
- Not all directories below have to apply to your current project - choose whatever applies
- You can add further subdirectories to help further organize your project

### Recommended Repository Structure

- **`bin`** - additional binaries that are not installed through `conda` or virtual environments or are not run through `Docker`/`Apptainer`. Instruction on installing additional software to `bin` should be described in `src/environment`
- **`configs`** - configuration files for the project and analyses
- **`data`** - raw data, databases, references, and other necessary input data. Prefer linking from shared directories whenever possible to avoid unnecessary large file duplication
    - **`data/raw`** -  raw input data for the project; this data do not change
    - **`data/databases`** - databases used in the project
    - **`data/metadata`** - necessary metadata (for example, sample sheet) for the analyses
    - **`data/preprocessed`** - preprocessed raw files that do not change during the analysis - primary data analysis results necessary for secondary analyses; these files can also be included in **`results/data/preprocessed`** depending on the project structure
    - **`data/references`** - references used in the analysis (for example, reference genomes)
- **`docs`** - additional documentation for the project such
    - **`docs/NOTES.md`** - additional notes worth keeping but not suitable for README
    - **`docs/COMMUNICATION.md`** - copy of the communication with the collaborator(s)
    - **`docs/publication`** copies of relevant PDFs materials
- **`envs`** - environment files to ensure reproducibility - e.g., `conda` environment `YAML` files, `Dockerfiles`, Apptainer (Singularity) `.def` files
    - If your project requires a lot of containers, you can further structure the files into:
        - **`envs/apptainer`**
        - **`envs/conda`**
        - **`envs/docker`**
- **`examples`** - example data and/or outputs of the analysis
- **`logs`** - log files and technical reports from the analysis
- **`notebooks`** - Jupyter or RMarkdown notebooks for exploratory data analysis and prototyping
- **`README.md`** - description of the project and analyses with instructions on how to reproduce the analysis, including but not limited to how to get the input data, build databases and references, create necessary environments, run the analysis, output files description, etc.
- **`publication`** - materials, results, visualization scripts for the plots, and text for the publication results; this can also include a copy of the final publication.
- **`reports`** - summary reports summarizing the analysis results; these reports can also be included in `results/reports`
- **`results`** - analysis results and intermediate files
- **`run`** - scripts necessary to run the analysis - high-level scripts using individual scripts from `src` directories; use `run_` prefix for the most important analysis *run* scripts; scripts should include instructions on how to get the input data, build references and databases, and set up the necessary environments
- **`run.sh`** - the main script to run the whole analysis, calling individual scripts from the `run` directory (e.g., get data, prepare environments, run analysis, make reports)
- **`src`** - scripts for individual analysis steps
    - **`src/bash`** - bash scripts
    - **`src/nf`** - `nextflow` workflow scripts and files
    - **`src/Python`** - Python scripts
    - **`src/R`** - R scripts
    - **`src/Snakemake`** - `snakemake` workflow scripts and files
- **`tests`** - end-to-end and unit tests for the project code
- **`tmp`** - temporary files generated during the analysis; these files can be deleted

Note: For research-heavy projects where we don't have an *exact flow* of the code, it's a good idea to prefix your script names with `01`, `02`, `03` to highlight the order scripts in the analysis/exploration.

## `snakemake`

- Follow the `snakemake` repository organization [recommendations](https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html)
    - For other best practices, follow `snakemake` [recommendations](https://snakemake.readthedocs.io/en/latest/snakefiles/best_practices.html)

## `nextflow`

- `nextflow` recommends using `nf-core pipelines create` to setup a new pipeline [repository structure](https://nf-co.re/docs/guidelines/pipelines/requirements/use_the_template)

## Shared resources

### References, Indexes and Databases

- Unified references and database directory structure help with code readability, shareability, and transferability

#### References Structure

- For should use Latin name for the organisms unless agreed otherwise
- Use: `<organism_latin_name>/<genome_version>/<source_database_and_version>/<reference_file>`
	- For example: `homo_sapiens/hg38/UCSC/genome.fa` 
- Note: Not all databases have to have `...and_version>`
- Keep the original file name and link it to a *simple* final file name
	- For example, for human reference based on UCSC `hg38`, make `homo_sapiens/hg38/UCSC` directory, download the original file from [UCSC](https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz) to `homo_sapiens/hg38/UCSC/` and (optional) soft-link it to the final `genome.fa.gz` 
	- The directory then contains the original downloaded file name (`hg38.fa.gz`) and the final reference name to use in the analysis, `genome.fa.gz`
- For annotations, use the same structure
	- For example, download mouse gene annotation from [Ensembl]([Mus_musculus.GRCm39.113.gtf.gz](https://ftp.ensembl.org/pub/release-113/gtf/mus_musculus/Mus_musculus.GRCm39.113.gtf.gz)) to `mus_musculus/GRCm39/ensembl_v113` directory and (optional) soft-link it (within the same directory) to the final `genes.gtf.gz`
- Don't forget to keep track of from where you downloaded the references, when, and if you did any postprocessing (including the soft-linking) in, for example, `run.sh` script within the references directory
- Optional: You can use references metadata (for example, `YAML`) to store all the information (organism, version, database, link, final file name, date, ...) and parse it automatically (for example, with `yq`)

##### Indexes

- Put indexes into the same reference directory as stated on the [top of the paragraph](#reference-structure)
- Use: `<tool_name_with_version/<index_settings>/<index_file>`
	- For example, `STAR_2.7.4a/150bp_splice/SAindex
- Note: Some tools don't have `<index_settings>`
- Exceptions are indexes that are very common and should be in the same directory as the reference
	- For example, `samtools faidx genome.fa` output `genome.fa.fai` or, `tabix genes.gtf.gz` gene annotation output `genes.gtf.gz.tbi` should be in the *main* directory as `genome.fa` or `genes.gtf.gz` respectively

#### Databases

- Use the same directory structure as in [Reference Structure](#reference-structure)

### Docker/Apptainer Images

- TODO