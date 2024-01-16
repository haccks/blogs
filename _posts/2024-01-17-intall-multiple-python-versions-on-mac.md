---
title: How to install multiple Python versions on macOS?
description: "A comprehensive tutorial on how to manage and use multiple versions of Python on Mac"
date: 2024-01-17 00:15:00 +0530
categories: [mac, python, devops]
tags: [python3, programming]     # TAG names should always be lowercase
render_with_liquid: false
---

## A step by step tutorial for beginners to install and use multiple versions of Python on macOS

<sub>If you are absolute beginner and just started Python programming and want to setup Python coding environment on you Mac then you probably don't need multiple versions. In that case I would suggest you to follow this tutorial instead: [How to setup Python coding environment on macOS](https://www.haccks.com/posts/python-setup-on-mac/).</sub>

### 1. Introduction

There are plenty of tutorials on how to use Python right way or manage multiple versions of python on Mac. And then comes the swarm of other python packages and dependencies to work with Python, namely `pyenv`, `pyvenv`, `virtualenvwrapper`, `pip-tool`, `poetry`, `anaconda` etc. Well, this seem overwhelming for beginners. In no way I am against using any of these packages but things can be simplified and you can manage multiple versions of Python and use virtual environment without using any of the packages mentioned earlier, specially *without using `pyenv`*. The only package you will need is a package manager for macOS and then `pip` (to install Python packages) and `venv` (to isolate project-specific dependencies from a shared Python installation) will be suffice for any Python project for beginners.

### 2. Install a Package Manager

Install [MacPorts](https://www.macports.org/) if you don't have it already on your Mac. Follow this detailed tutorial on [How to install MacPorts on Mac?](https://www.haccks.com/posts/macports-install-and-usage/).

### 3. Install Python

Let's check all the available versions of Python

```bash
port search --name --line --regex '^python\d*$'
```
{: .nolineno}

it will return something like this

```
python26	2.6.9	lang	An interpreted, object-oriented programming language
python27	2.7.18	lang	An interpreted, object-oriented programming language
python32	3.2.6	lang	An interpreted, object-oriented programming language
python33	3.3.7	lang	An interpreted, object-oriented programming language
python34	3.4.10	lang	An interpreted, object-oriented programming language
python35	3.5.10	lang	An interpreted, object-oriented programming language
python36	3.6.15	lang	An interpreted, object-oriented programming language
python37	3.7.17	lang	An interpreted, object-oriented programming language
python38	3.8.18	lang	An interpreted, object-oriented programming language
python39	3.9.18	lang	An interpreted, object-oriented programming language
python310	3.10.13	lang	An interpreted, object-oriented programming language
python311	3.11.7	lang	An interpreted, object-oriented programming language
python312	3.12.1	lang	An interpreted, object-oriented programming language
```

For this tutorial we will install Python versions `3.11.7` and `3.12.1`. Along with these Python versions we need to install pip packages for both versions.  

```bash
sudo port install python311 python312 py311-pip py312-pip
```
{: .nolineno}

It will give the list for dependencies and ask you if you want to continue.

```
--->  Computing dependencies for python311
The following dependencies will be installed: 
 bzip2
 expat
 gettext
 gettext-runtime
 gettext-tools-libs
 gperf
 libedit
 libffi
 libiconv
 libtextstyle
 openssl
 openssl3
 pkgconfig
 python3_select
 python3_select-311
 python_select
 python_select-311
 sqlite3
 xz
 zlib
Continue? [Y/n]: y
```

After installation is complete, checkout the versions installed  

```bash
python3.11 --version; python3.12 --version 
```
{: .nolineno}

You should see this output  

```
Python 3.11.7
Python 3.12.1
```

### 4. Switch between different versions of installed Python

You can switch between these installed versions by using the command `python3.11` or `python3.12`. Type these command one by one in the terminal and seeit in action. 

`which python3` will give the system's Python `/usr/bin/python3` and this is the default Python on your Mac.  

You can set any of these installed versions as the default version. Below command will set `python3.12` as default version.  

```bash
 sudo port select --set python3 python312
```
{: .nolineno}

If you will type `which python3` then it will show `/opt/local/bin/python3` (the one installed and set default by `port`).


>If you have installed multiple versions of Python using `port`, e.g. python310, python311 and python312, then each of these will be active and you can switch between them by using the command `python` with their version suffix, e.g. `python3.10`, `python3.11` and `python3.12` respectively, in the terminal. You can make any these versions as the default python by running the `sudo port select --set python3 python<version>` command.
{: .prompt-info}

You can use `pip-<version>` command to switch between `pip` for each version. For example, `pip-3.11` or `pip-3.12`.

### 5. Create virtual environment and install project related dependencies

For each Python application create a new virtual environment  

```bash
 python3.12 -m venv myvenv
```
{: .nolineno}

Activate this environment using the command  

```bash
source myvenv/bin/activate
```
{: .nolineno}

Once activated, the default Python for this environment will be `python3.12`, that means you can use just `python` or `python3` instead of `python3.12` in this environment even if Python 3.12 is set as the default version. You can check it anytime by running `which python3` command

```bash
which python
```
{: .nolineno}

```
/Users/dankhan/demo-venv/bin/python3
```

To install project level dependencies you can use either `pip` or `pip3` in this environment (both are same in the virtual environment). Outside the environment you have to use `pip-3.12` for Python 3.12 version.

```bash
pip install numpy pandas scipy matplotlib
```

Run `deactivate` to deactivate the virtual environment. 

>Note that most of the IDEs or text editors offers feature to create or select virtual environment for your project. Once you create a virtual environment through CLI, you can select this environment and use it in your favorite IDE or text editor. 