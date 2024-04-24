---
layout: post
title: "How to run Jupyter Notebook inside Python virtualenv"
---

Yesterday I came across a problem where Langchain ran into some issues with my globally installed Pydantic and hence, I wasn't able to run the code I had written to leverage LLM. So this is an article detailing how you can run Jupyter Notebook inside a Python virtualenv

### Step 1 - Initialize virtualenv

Go to the directory where you want to run Jupyter from and create a virtual environment.

```bash
python3 -m venv .venv
```

Activate the environment

```bash
source .venv/bin/activate
```

### Step 2 - Install Jupyter Notebook

Using pip, install Jupyter Notebook

```bash
pip install notebook
```

Then install all the dependencies you want.

### Step 3 - Setup iPython kernel

Your virtualenv name is usually the directory name. As you can see from the image, on the right hand side, the venv name comes as `personal` which is same as my current working directory name.

![Venv name](/assets/venv_name.png)

Now, run this command

```bash
ipython kernel install --user --name=personal
```

This will configure a new kernel in your Jupyter Notebook

### Step 4 - Switch to this kernel in Jupyter Notebook

Run Jupyter Notebook from the venv using

```bash
jupyter notebook
```

Now, open a Notebook and you can find the new kernel and switch to it.

![Kernel switch](/assets/kernel.png)
