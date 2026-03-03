# Exo 1: The HPC-Compatible Environment

We want to build a runtime environment that uses the exact Python version and dependencies declared in our `pyproject.toml`.

While using `uv` (and `uv sync` in particular) locally would perform all the necessary steps in one go, deploying that standard virtual environment on an HPC cluster is not ideal for several reasons:

* **Metadata Hammering:** The generated `.venv` directory will contain thousands of individual files.
  Reading/writing these small files severely degrades the performance of HPC shared network filesystems (like CephFS or Lustre).
* **Portability Issues:** Within the `.venv` directory, various configurations and scripts use absolute paths, effectively breaking the virtual environment whenever the project folder is moved or renamed.
* **Strictly Limited to Python:** A standard virtual environment does not include any information on system-level packages (like C++ compilers or external libraries like GDAL).
  If the cluster is missing a system dependency, your code will fail.

Therefore, we are going to create a "containerized environment," relying on [Apptainer's "Integration First" philosophy](https://apptainer.org/user-docs/master/introduction.html#why-use-apptainer).

#### Your Tasks:

* Make sure the resulting binary container files (`*.sif`) are ignored and not accidentally tracked by Git (add them to your `.gitignore`).
* Create a `env.def` file in the appropriate location in this repository.
* Declare a container recipe that sets up our environment properly (use `uv` inside the `%post` section of the container for this).

  You might want to start with a small base image and add only what you need.

  Here is a basic layout to get you started:

  ```
  Bootstrap: docker 
  # Choose your image
  From: python:3.13-slim
  
  %files 
      # Copy your files from the host into the container 
      # NOTE: Placing files inside the `/app` folder is a common practice for containers
      pyproject.toml /app/
      ...
   
  %post 
      # In here, perform every operation that is needed after (i.e. post) the
      # base image has been bootstrapped 
      # e.g. system updates and installs:
      apt-get update && apt-get install -y curl
      rm -rf /var/lib/apt/lists/*
  
      # Install uv
      ...
  
      # Move into the app directory and setup the python environment
      ...
   
      # Install dependencies directly from the pyproject.toml 
      ...
   
  %environment 
      # Here you define environment variables that set when running the container
      # e.g. ensure the container uses a specific python env:
      export VIRTUAL_ENV=/opt/venv
      export PATH="/opt/venv/bin:$PATH"
  
  ```
* Build the `env.sif` file locally on your machine and try it out with the [`apptainer shell`](https://apptainer.org/docs/user/main/cli/apptainer_shell.html) command.
* Log in to the cluster, checkout your repository, and try building the `env.sif` file directly on the cluster.
* "Enter" the created container environment on the cluster using `apptainer shell container/env.sif`.
  Once inside:
  * Check which Python executable are you using now?
  * Verify the `numpy` version specified in your `pyproject.toml` is indeed installed (e.g., run `python -c "import numpy; print(numpy.__version__)"`).

* Run the `scripts/drafts/say_hello.py` script interactively from inside the container shell.
* Exit the container (type `exit`), and try running the exact same script non-interactively from your host terminal using the `apptainer exec` command.

#### Hints & Best Practices:

* **Respect the Login Node:** Avoid running heavy builds or computational workloads on the login nodes. Grab an interactive session on a compute node first:
  ```bash
  srun --cpus-per-task=1 --mem=4G --time=00:20:00 --pty bash
  ```

* **Check your Environment:** The `which python` command is your best friend to verify whether you are using the host's default Python or the container's baked-in Python.
* **Manage Secrets:** Use the `--env-file .env` flag with your Apptainer commands to securely pass environment variables (like API keys or paths) from a local `.env` file into the context of the container.
* **Bypassing Cluster Build Errors:** If your `apptainer build` command crashes on the cluster with a `fakeroot` or similar error, try adding the `--ignore-fakeroot-command` flag.
  If that still fails, build the `.sif` on your laptop and `scp` it to the cluster!
