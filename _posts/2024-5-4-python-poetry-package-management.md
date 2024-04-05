---
layout: post
title: "Using Poetry for Python dependency management"
---

I have been using Python extensively for the last 7 years but using `pip` and `requirements.txt` file for managing dependencies never clicked with me. Maybe it was because during those 7 years, my work required me to use Spring Boot for a few months and that developed in me a want, or shall I say, need in me to go for structure and as much deligating dependency management to a proper tool (`maven` in the case of Spring Boot). That's when I came across Poetry. At first, there was a steep learning curve as to my Pythonic mind, Poetry did not make sense in the beginning. But when a need arise where I had to build packages in a streamlined manner, that's when Poetry clicked. Currently, I use Poetry extensively for dependency management and also to publish and releasing new versions of various Python libraries.

## How to use Poetry

All the package dependencies are entered in poetry.toml. If you have to update a dependency or update the version of the library in which you are working, you can update it here.

On running poetry lock, this creates a poetry.lock file. Just like nodeJS, this file contains the final version of the dependency that is used in the project. And just like nodeJS, you should never updated `poetry.lock` manually because it messes with the hashes of the dependencies.

## Installing and Configuring Python Poetry environment

I use `pyenv` for managing various versions of Python. If you are familiar with this, I suggest you go through this [post](https://realpython.com/intro-to-pyenv/) by RealPython. It's pretty simple - using `pyenv` you can install various Python versions and decide which one to use globally and which one to use locally i.e, in a specific directory.

Now, to get started with Poetry

* First, install Poetry

```bash
pip install poetry
```

* Now you need to tell Poetry which version of Python to use. Whatever version you have configured using `pyenv`, that will be reflected when you run the `python3` command.

```bash
poetry env use python3
```

This will create a .venv file for the virtual environment of this project

* You can find the virtual environment which Poetry is using by doing

```bash
poetry env info -p
```

* The path of virtual environment will generally be something like `$HOME/Library/Caches/pypoetry/virtualenvs`. You can configure Poetry to create venv in the same directory by running `poetry config settings.virtualenvs.in-project true` before you run the command above

* Then start the Poetry shell

```bash
poetry shell
```

* Now if your project already has a `pyproject.lock` file, you can run `poetry install` to install dependencies from it. Or if you only have `pyproject.toml` file, you will need to run `poetry lock` to generate a lock file and then run `poetry install` to install dependencies

> Poetry always uses `poetry.lock` file to install dependencies

* If you are starting a project from scratch, you will need to create `pyproject.toml` file. Here is the guide detailing how you can do it - https://python-poetry.org/docs/basic-usage/


## Publishing a new version of Python Library

If you are working on a library project that you would like to import into your other Python projects, you will need to build and publish it before you can do so.

* First, in your `pyproject.toml` file, update the version.

* Then run `poetry build`. This will build the project's package under the `dist` folder. This will automatically be avaiable (locally) when you import this new project version in your project

* Now, let's say in a different Python project where you have to use this library, you will first have to update the version of the library to the new version in `pyproject.toml` file, then run `poetry update package-name` to update only this package. This will also update the hash in the `poetry.lock` file

> If after updating the version in `pyproject.toml` file you run `poetry lock` this will update all the inner dependencies as well, if they are updated in the new version of the library. This can result in some runtime errors.

## Publishing Python packages to GitLab

Using Poetry, you can also publish the Python libraries as artifacts to GitLab where you can use them in your CI/CD pipelines.

* First, its better to set two environment variables which will indicate your GitLab access token name and access token

```bash
export GITLAB_TOKENNAME="<gitlab_access_token_name>"
export GITLAB_TOKEN="<gitlab_access_token>"
```

* Now in Poetry, you will need to configure them by doing 

```bash
poetry config http-basic.repo ${GITLAB_TOKENNAME} ${GITLAB_TOKEN}
```

Here `repo` will mean the artifact repository you are using in GitLab. So if its name is `saturn`, you will replace it with `poetry config http-basic.saturn`

* After all this is done, you can check the configurations using `poetry config --list`. Here under `repositories.saturn.url` you can find the pypi URL of the artifact repository. This will host all your Python artifacts or packages.

* Now to publish to GitLab, use

```bash
poetry publish --repository saturn
```

## Troubleshooting Common Poetry Issues

### Common Issue #1: Inconsistent poetry.lock and pyproject.toml

This happens when someone updated the `pyproject.toml` file but did not run `poetry lock`. So now when you try to do `poetry install`, Poetry throws an error saying that the version of dependencies the project is using are inconsistent with the ones in toml file. So fix this, just run `poetry lock` and then do `poetry install`

### Common Issue #2: Poetry hash is not supported in the current version

If your local version of Poetry is different from the one in CI/CD pipeline or that `poetry.lock` file was generated by someone who was using a different version of Python or POetry, the hash generated will not be compatible with yours. So in your CI/CD pipelines or in the dev environments of devs in your organisation, make sure the version of Poetry is a new one and stable, and preferrably all are using the same Poetry and Python version.

### Common Issue #3: Updates made to the same version of a package are not being reflected locally

Suppose you made some changes in a library package called `cool-py-lib` and decided to keep the version same, say 0.7. Now you pushed the changes to GitLab, pipeline passed and your projects were able to pick up the changes made in the same version. Then you open a project locally that was already using `cool-py-lib==0.7` and remove the lock file and run `poetry install` again, but you see that your changes are not reflected here. 

Now in a state of fury and whim, you clear Poetry’s cache by doing `poetry cache clear saturn --all` and go berserk and clear PyPi and _default_cache as well by doing `poetry cache clear pypi --all && poetry cache clear _default_cache --all` and do installation again but still the same.

So now you start to lose your mind and decide to go the easy way by using the directory path of the package’s wheel file instead. For this you go to the `pyproject.toml` file and instead of writing the version of the library, you mention its path so that the project picks `cool-py-lib` from this path instead

```bash
# cool-py-lib = {version = "0.7", source = "saturn"} 
cool-py-lib= { path = "$HOME/cool-py-lib/dist/cool_py_lib-0.7-py3-none-any.whl", develop = true }
```

run `poetry install` again and it works! So your commits are now full of commented out `cool-py-lib` which you have to manually remove before every `git push`.

Unfortunately Poetry `cache clear` does not work in an intuitive way but you can fix this in two different ways - 

#### The good but hard way

1. When you made changes in `cool-py-lib`, it publishes a tar file in dist directory (you can find it in the project)

2. Untar this file `tar -xzvf cool_py_libs-0.7.tar.gz` and you will get a directory of your project with the updated code

3. Go inside it to `cool_py_libs-0.7/src/cool_py_libs`

4. Now you need to manually copy the contents of this directory to your project’s virtual environment

5. So figure out where the virtual environment of the project is by doing `poetry env info -p` from inside the project where you have to use this library

6. Now copy all the contents. In the command below, we are inside the `cool-py-lib` directory where we untarred the files. We are copying all of the unpacked files to the `site-packages` of the `importing-py-project` (Python project where `cool-py-lib` is being used)

```bash
cp -r * $HOME/Library/Caches/pypoetry/virtualenvs/importing-py-project-k6HKJt70-py3.10/lib/python3.10/site-packages/axm_py_libs/.
```

#### The easy but ugly way

The easy way is to remove Poetry’s artifacts. Here all the built packages are stored. So if your package’s published version is 0.7, it will be stored here. When you made changes and published the same version again to GitLab, Poetry will not download it again from GitLab if the version is same. Instead it will create a symbolic link of the package in your virtual env of the importing project that points to here. 

So run this **but be warned it might make your Poetry env unstable so you can end up re-installing and re-configuring Poetry**

```bash
rm -rf $HOME/Library/Caches/pypoetry/artifacts/*
```

#### Common Issue #4: Cannot remove in-project virtual environment

If you have configured poetry to create a virtual environment in the project directory by doing `poetry config virtualenvs.in-project true`, then you run into some Python version or some other issue which is forcing you to delete the virtual environment, Poetry will not stop referencing to it even after you delete it. Turns out this is a (bug)[https://github.com/python-poetry/poetry/issues/2124]

Poetry stores it in its cache somewhere and you can easily remove this by doing `poetry env remove --all` but this **REMOVE ALL POETRY THE ENVIRONMENT** from your system. So not recommended.

The ideal way to go about this is to set `poetry config virtualenvs.in-project false` so that Poetry creates the virtual environment in the default path. Then delete the `.venv` folder and create the virtual environment again using `poetry env use python3`. (Make sure version of python3 here is similar to the one in pipeline of the project).



