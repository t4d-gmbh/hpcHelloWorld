# Exo 2: Automating with a Slurm Batch Script

Now that we have a working, HPC-compatible Apptainer environment, we need to transition from running interactive tests to submitting asynchronous "batch" jobs.
This is how actual research is done on an HPC cluster.

We have a script located at `scripts/drafts/say_hello.py`.
If you look at the source code, you will notice it requires two pieces of information to run: a configuration file path and an output directory.
It is designed to look for these in the environment variables (`CONFIG_PATH` and `OUTPUT_DIR`).

We need to write a Slurm submission script that sets up this environment, binds our project directory into the container, and executes the code.

#### Your Tasks:

* **Create the Configuration File:** Create a simple JSON file (e.g., `config.json` inside a `config/` directory) that contains `{"name": "Your Name"}`.
* **Create the `.env` File:** In the root of your project repository on the cluster, create a file named `.env`.
* Add the line `CONFIG_PATH=config/config.json`.
* Add the line `OUTPUT_DIR=results/`.


* **Create the Slurm Script:** Create a file named `submit_hello.sh` inside the `scripts/slurm/` directory.
* **Write the Slurm Directives:** At the top of `submit_hello.sh`, add your `#SBATCH` headers.
  Request 1 CPU, 4GB of memory, and 10 minutes of runtime.
  Make sure to define an output log file (e.g., `#SBATCH --output=results/hello_%j.log`).
* **Write the Execution Command:** In the Slurm script, write the `apptainer exec` command to run the Python script.
* You must use the `--env-file .env` flag to inject the variables so `hello.py` doesn't crash.
* You must use the `--bind "$(pwd):/app"` and `--pwd /app` flags.
  This ensures the container has permission to read your live `config.json` file and write to the `results/` folder on the host system.
* Call the script naturally: `python scripts/drafts/hello.py`.

* **Submit the Job:** From the root of your project, submit the script to the scheduler using:
  ```bash
  sbatch scripts/slurm/submit_hello.sh
  
  ```

* **Verify:** Use `squeue -u $USER` to check if your job is running.
  Once it finishes, check the `results/` folder to see if your output file and the Slurm `.log` file were successfully generated.

#### Hints & Best Practices:

* **Seamless Imports:** Because your `container.def` installed the project defined in your `pyproject.toml`, `mypkgs` is treated as a native Python package inside the container. You don't need any messy `PYTHONPATH` hacks or module flags; the container's Python environment already knows exactly where `mypkgs` is.
* **Directory Creation:** The `hello.py` script relies on a utility `prepare_output_dir` to make the `results/` folder if it doesn't exist. Because we bound `$(pwd)` to `/app`, the container utilizes your user's exact host permissions to create that folder safely on the cluster's shared filesystem.
