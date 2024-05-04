---
layout: post
title: "How to run Jupyter Notebook inside Python virtualenv"
---

Yesterday I came across a problem where Langchain ran into some issues with my globally installed Pydantic and hence, I wasn't able to run the code I had written to leverage LLM. So this is an article detailing how you can run Jupyter Notebook inside a Python virtualenv. I am assuming you already have Jupyter Notebook installed.

### Step 1 - Initialize virtualenv

Go to the directory where you want to run Jupyter from and create a virtual environment.

```bash
python3 -m venv .venv
```

Activate the environment

```bash
source .venv/bin/activate
```

### Step 2 - Install ipykernel

To manage the kernel, you will need to install `ipykernel`. 

```bash
pip install ipykernel
```

Now to check which kernels you have, you can run `jupyter kernelspec list`

![Jupyter config list](/assets/jupyter_config.png)

As you can see there is only one kernel currently configured.


### Step 3 - Setup iPython kernel

Now, to install a new kernel, use this command

```bash
python -m ipykernel install --user --name notebooks --display-name "Notebooks"
```

This will configure a new kernel in your Jupyter Notebook. The list will now look like 


![Kernel installed list](/assets/install_kernel.png)

If you go to the path mentioned here, you will find a `kernel.json` file which will contain the path to the venv file.


![Kernel path](/assets/kernel_installed.png)


### Step 4 - Switch to this kernel in Jupyter Notebook

Run Jupyter Notebook from the venv using

```bash
jupyter notebook
```

Now, open a Notebook and you can find the new kernel and switch to it.

![Kernel path](/assets/kernel_final.png)
