# Tutorial for GridEngine (SGE/PBS) clusters

1. Download [cromwell](https://github.com/broadinstitute/cromwell) on your `$HOME` directory.
    ```bash
    $ cd 
    $ wget https://github.com/broadinstitute/cromwell/releases/download/38/cromwell-38.jar
    $ chmod +rx cromwell-38.jar
    ```

2. Git clone this pipeline and move into its directory.
    ```bash
    $ cd
    $ git clone https://github.com/ENCODE-DCC/chip-seq-pipeline2
    $ cd chip-seq-pipeline2
    ```

3. Download a SUBSAMPLED (1/400) paired-end sample of [ENCSR936XTK](https://www.encodeproject.org/experiments/ENCSR936XTK/).
    ```bash
    $ wget https://storage.googleapis.com/encode-pipeline-test-samples/encode-chip-seq-pipeline/ENCSR936XTK/ENCSR936XTK_fastq_subsampled.tar
    $ tar xvf ENCSR936XTK_fastq_subsampled.tar
    ```

4. Download pre-built chr19/chrM-only genome database for hg38.
    ```bash
    $ wget https://storage.googleapis.com/encode-pipeline-genome-data/test_genome_database_hg38_chr19_chrM_chip.tar
    $ tar xvf test_genome_database_hg38_chip.tar
    ```

5. Get information about a parallel environment (PE) on your SGE system. If your system doesn't have a PE then ask your admin to add one with name `shm` to SGE master. 
    ```
    $ qconf -spl
    ```

Our pipeline supports both [Conda](https://conda.io/docs/) and [Singularity](https://singularity.lbl.gov/).

## For Conda users

6. [Install Conda](https://conda.io/miniconda.html). Skip this if you already have equivalent Conda alternatives (Anaconda Python). Download and run the [installer](https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh). Agree to the license term by typing `yes`. It will ask you about the installation location. On Stanford clusters (Sherlock and SCG4), we recommend to install it outside of your `$HOME` directory since its filesystem is slow and has very limited space. At the end of the installation, choose `yes` to add Miniconda's binary to `$PATH` in your BASH startup script.
    ```bash
    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    $ bash Miniconda3-latest-Linux-x86_64.sh
    ```

7. Install Conda dependencies.
    ```bash
    $ bash conda/uninstall_dependencies.sh  # to remove any existing pipeline env
    $ bash conda/install_dependencies.sh
    ```

8. Run a pipeline for the test sample. If your parallel environment (PE) found from step 5) has a different name from `shm` then edit the following shell script to change the PE name.
    ```bash
    $ qsub examples/local/ENCSR936XTK_subsampled_chr19_only_sge_conda.sh
    ```

## For singularity users

6. CHECK YOUR SINGULARITY VERSION FIRST AND UPGRADE IT TO A VERSION `>=2.5.2` OR PIPELINE WILL NOT WORK CORRECTLY.
    ```bash
    $ singularity --version
    ```

7. Pull a singularity container for the pipeline. This will pull pipeline's docker container first and build a singularity one on `~/.singularity`.
    ```bash
    $ mkdir -p ~/.singularity && cd ~/.singularity && SINGULARITY_CACHEDIR=~/.singularity SINGULARITY_PULLFOLDER=~/.singularity singularity pull --name chip-seq-pipeline-v1.2.2.simg -F docker://quay.io/encode-dcc/chip-seq-pipeline:v1.2.2
    ```

8. Run a pipeline for the test sample. If your parallel environment (PE) found from step 5) has a different name from `shm` then edit the following shell script to change the PE name.
    ```bash
    $ qsub examples/local/ENCSR936XTK_subsampled_sge_singularity.sh
    ```

## For all users

9. It will take about an hour. You will be able to find all outputs on `cromwell-executions/chip/[RANDOM_HASH_STRING]/`. See [output directory structure](output.md) for details.

10. See full specification for [input JSON file](input.md).

11. You can resume a failed pipeline from where it left off by using `PIPELINE_METADATA`(`metadata.json`) file. This file is created for each pipeline run. See [here](../utils/resumer/README.md) for details. Once you get a new input JSON file from the resumer, then edit your shell script (`examples/local/ENCSR936XTK_subsampled_chr19_only_sge_*.sh`) to use it `INPUT=resume.[FAILED_WORKFLOW_ID].json` instead of `INPUT=examples/...`.

## For singularity users

12. IF YOU WANT TO RUN PIPELINES WITH YOUR OWN INPUT DATA/GENOME DATABASE, PLEASE ADD THEIR DIRECTORIES TO `workflow_opts/sge.json`. For example, you have input FASTQs on `/your/input/fastqs/` and genome database installed on `/your/genome/database/` then add `/your/` to `singularity_bindpath`. You can also define multiple directories there. It's comma-separated.
    ```javascript
    {
        "default_runtime_attributes" : {
            "singularity_container" : "~/.singularity/chip-seq-pipeline-v1.2.2.simg",
            "singularity_bindpath" : "/your/,YOUR_OWN_DATA_DIR1,YOUR_OWN_DATA_DIR2,..."
        }
    }
    ```
