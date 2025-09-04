---
layout: page
title: Guidebook - Installing Antelope
permalink: /guidebook-installation
---

# Installation

Antelope is offered as a set of packages on the [Python Package Index](https://pypi.org/) and can be installed using `pip`.

## Virtual Environments

It is always recommended to use [virtual environments](https://docs.python.org/3/library/venv.html) in order to simplify installation and maintenance of projects.  A virtual environment is simply a folder that contains a local copy of the python execution environment. This local copy has an environment-specific library of packages, so you don't have to worry about different projects using different versions of the same package.

There are several different ways to create virtual environments If you are a regular python user, you probably have your own virtual environment solution already set up. But if you're not, the simplest way to start out is to use `venv` which is built in to all modern versions of python (after 3.3).

### Creating your virtual environment

To use venv, open a terminal window and navigate to a directory that contains your project work.  Then pick a name for your virtual environment and remember it. For the purposes of this demo, the environment name is `wwwxffj` but yours should probably be more descriptive.  Be sure to use your custom name instead of `wwwfxxj` wherever it appears.

```bash
python -m venv wwwxffj
```
This will create a folder named `wwwxffj` in your current directory which contains a local copy of `python` and `pip`.

### Activating your virtual environment

To activate a virtual environment, you need to be in the same directory as you were in when you created it.

Now, if you are on a normal computer, enter:

```bash
source wwwxffj/bin/activate
```
and the environment will be "activated".  The environment name will appear as part of the command prompt to remind you. But if you have the misfortune to be on Windows, then you need to use one of the following commands instead:

(on Windows command prompt)
```cmd
wwwxffj\scripts\activate
```

(in Windows PowerShell)
```powershell
.\wwwfxxj\Scripts\Activate.ps1
```

Once you're done with your work, you can exit the virtualenv by typing the command `deactivate`.

### Adding your venv to Jupyter

If you are planning on using Jupyter notebooks for your work, you need to add your virtual environment as a "kernel" so that you can select it.


```bash
(wwwxffj) $ pip install jupyter ipykernel
(wwwxffj) $ python -m ipykernel install --user --name=wwwxffj
```

## Installing Antelope

Once you're in your virtual environment, installing Antelope is simple:

```bash
(wwwxffj) $ pip install antelope-foreground
```

If you are planning on performing LCA computations using local background data, you may need some additional packages:

 * `antelope-background` for LCI matrix construction and inversion
 * `lxml` for accessing EcoSpold or ILCD data (soon I will add `antelope-xml` instead, but it does not exist yet)
 * `antelope-reports` containing support tools for constructing and running models.

Then, open up `ipython` and run:

```python
import antelope_foreground
```
and you're off to the races!

[Home](/guidebook/)
