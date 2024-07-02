# Run a Python Script on HPC

Running a Python script on Cheaha is pretty straightforward. One of the easiest ways is to load a pre-installed **Anaconda** module and create a new **conda environment**. The conda environment is a self-contained workspace where you will install all the packages you need for this particular project or program.

As an example, we will install the package [**SCENIC+**](https://scenicplus.readthedocs.io/en/latest/install.html).

## 1. Create conda environment

You only need to do these steps once. Once you've created the new environment, you only need to activate it in order to use it.

These steps are not memory-intensive, so it should be fine to run this on a head node, rather than starting an interactive job.

We will create the environment in our user data directory, which is the best place for personal long(ish)-term storage. To start, `cd` into `$USER_DATA`.

First, load a pre-installed Anaconda module. Unless you need a specific version, just call generic `Anaconda3` and it will load the most current version.

```
module load Anaconda3
```

Now let's create the new conda environment for SCENIC+. This package requires installing specifically with Python v.3.11, so we need to specify that below.

You can name the environment anything you want; here, I'll name it `scenicplus_env` (note that when you actually install SCENIC+ later, it installs in a directory named `scenicplus`, so you don't want to name the environment that if you plan to install it here in the same directory).

```
conda create --name scenicplus_env python=3.11
```

Now that the environment is created, just activate it to use it (as we will do below).

## 2. Install packages

Now let's install SCENIC+.

**Important:** We want to install the packages in our newly created `scenicplus_env` environment, so first we have to activate the environment:

```
conda activate $USER_DATA/scenicplus_env
```

Now follow the instructions on the [SCENIC+ website](https://scenicplus.readthedocs.io/en/latest/install.html) to install the packages:

```
# Clone the git repository
git clone https://github.com/aertslab/scenicplus
cd scenicplus
pip install .
```

### Errors I sometimes encountered

If you receive a **Rust** error, you will need to [install Rust](https://www.rust-lang.org/tools/install) first, then go back and retry the SCENIC+ install.

From the website:

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

If you receive a **gcc** error (can't build wheel for annoy), load the pre-installed GCC v.13 module:

```
module load GCC/13.1.0
```

Then retry `pip install .` for SCENIC+.

## 3. Run Python script

To run a Python script in a job on HPC inside our new conda environment, we need to make sure of two things:

1. In the Python script file, properly set the shebang.
2. In the HPC job submission file, load Anaconda, activate the env, and properly call python3.

### 3a. The Python script

In the Python script, set the **shebang** line at the top of the script to run the Python version set by the active environment:

```
#!/usr/bin/env python3
```

The rest of the script is the same as if you were running it locally. Just be sure all the paths are correct -- any paths to directories or paths to other programs used should point to the correct location on HPC, not your local computer system.

### 3b. The job submission script
In your job submission file, change into your working directory in `$USER_SCRATCH`, load Anaconda, and activate the env:

```
export WORK_DIR=/scratch/cenewman/scenic
cd $WORK_DIR

date

module load Anaconda3
conda activate /data/user/cenewman/scenicplus_env
```

Now simply tell the job to run a single line of code: calling the Python script. We are running the script with python3, so use the command `python3` to tell bash to run the file in that language.

My script is named `scenicplus_Reln_HPC.py`

```
python3 scenicplus_Reln_HPC.py
```
