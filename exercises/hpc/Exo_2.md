# Exo 2: Minimal HPC Workflow

Now that we have a working, HPC-compatible Apptainer environment, we need to transition from running interactive tests to submitting asynchronous "batch" jobs.
This is how actual research is done on an HPC cluster.

In order to adhere to a stirct separation between environment, configuration, code and data we need to have a closer look at how these 4 pillars can come together on an HPC cluster and in an Apptainer container in particular.

Following earlier exercises we have produced a script located at `scripts/drafts/say_hello.py`.
You will notice that this scirpt imports some functions form the `mypgks` package (Pillar 3), requires two pieces of information to run: a configuration file (Pillar 2) path and an output directory (Pillar 4).
It is designed to look for these in the environment variables (`CONFIG_PATH` and `OUTPUT_DIR`) (Pillar 1).

Before we can combine all of this we need to make sure everything is set up properly:

### Epilog

#### Your Tasks:
- Build a container to run the script in (see previous exercise).
- Decide where you want the intermediary data to be stored and where the final output data should be stored.
- Declare the `OUTPUT\_DIR` path variable such that it can be loaded as environment variable.
- Define the `CONFIG\_PATH` environment variable. Make sure your script will be able to access the `config/hello_to.json` configuration file **inside** the container.
- Write your slurm sumbission script 
  - Adapt the `scripts/slurm/submit_template.sh` script, request
    - 1 CPU
    - 4GB of memory
    - 10 minutes of runtime.
  - Use the `apptainer exec` command to run a specific script.
  - Provide the environment variables defined in `.env`
  - Make sure container can write to the persisten output location.

### Execution

#### Your Tasks:

- Submit your job using `sbatch`
- Check the status of your job.

### Prolog

#### Your Tasks:

- Verify the output was written to the correct location.
- Remove/cleanup unused scratch data (we don't have any).
- Remove the container (.sif) again.


#### Hints & Best Practices:

* In this simple example we can also not dstinguish between `interim` and `final` data and directly write to the (permanent) output location.
* Content that is direclty submitted in the project repository (e.g. `config/*`) will be present inside the container.
  Since the scripts run inside the container, also the environment variables are read inside the container, so relative paths like `./config` will be relative to the location inside the container.
* Use the `apptainer exec` command to run a specific script.
* Use the `--env-file` option to inject environment variables into a container run.
* The option `--bind` allows to mount a location of the host filesystem inside the container.
* Checking queued or running jobs:
  - `squeue -u <username>`
  - `sacct -u <username>`
  You can add extra foramtting options, e.g. `--format=JobID,JobName,State,ExitCode`.
