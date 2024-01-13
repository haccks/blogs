---
title: A beginner guide to setup Python coding environment on macOS
description: "A step by step beginner guide to setting up your mac for python programming"
date: 2024-01-07 1:20:00 +0530
categories: [mac, python, programming]
tags: [python3, vscode, git, github]     # TAG names should always be lowercase
render_with_liquid: false
---

## A step by step beginner guide to setting up your Mac for python programming 

### 1.Prerequisite

Make sure following tools are installed

+ Apple's Developer Tools
+ MacPorts Package Manager

If you don't know what are these then don't worry. Follow [this beginner tutorial to  install Apple's Developer Tools and MacPorts](https://www.haccks.com/posts/macports-install-and-usage/).

<!-- Open spotlight by typing <kbd>⌘ command</kbd>+<kbd>space</kbd> and then type "terminal" in the spotlight and launch the terminal. Run the command below and install Apple's Command Line Developer Tools: 

```bash
xcode-select --install
```
{: .nolineno} -->

Once installed, launch your terminal and type `python3 --version` (for older MacOS version < 12.3 run `python --version`) and it will show the system python version (Python 3.9.6 on Sonoma 14.2.1).

If you run `which python3` (or `which python` for older versions) in the terminal then it will return `/usr/bin/python3`. This is the default python installed for macOS. This python is used by apple system. This is **the system python and let's forget about this python and and promise yourself to never look back at it again**. 

<!-- ### 2. Install a package manager for Mac 

You can either install [MacPorts](https://www.macports.org/) or [Homebrew](https://brew.sh/). I prefer MacPorts over homebrew and will use it for this tutorial.
Download a MacPorts package from its [official website](https://www.macports.org/install.php) for your operating system.

After installation is complete fire up your terminal again and run `port version`. If it is installed successfully it will print the version installed (`Version: 2.8.1`).    -->

### 2. Install and setup python

#### 2.1 Install python

There are many ways to install python on your Mac, but in this tutorial I will use MacPort package manager to install a new version of python.   

>Do not use [python.org](https://www.python.org/downloads/macos/) to install python! 
{: .prompt-warning}

You must be asking by now *why not use system python instead of using a package manager to install a new one?*
The reason is simple. You may want to use different python version and update it in the future or use multiple python versions for your application development. You don't want to mess with system python. The MacPort package manager will help to manage python (install, update, use multiple versions etc.) and other packages.

Let's use MacPort. To check the available versions of python, run 

```bash
port search --name --line --regex '^python\d*$'
```
{: .nolineno}

It will return the output similar to this  

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

Choose the version you want to install. For this tutorial I will choose the 3.11 version. Run below command to install python 3.11

```bash
sudo port install python311
```
{: .nolineno}

It will ask for your password and then the permission to install all other dependencies for python. 

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
Type `y` and wait for it to install (it will take a while). Once its installed, in the terminal run

```bash
python3.11 --version 
```
and you will see the version installed by python (*Python 3.11.7*).  

Let's make this new version the *default python* by executing

```bash
 sudo port select --set python3 python311
```
{: .nolineno}

Run `which python3` in the terminal and you will see that the default python is now `/opt/local/bin/python3`.

In the terminal type `python3` (or `python3.11`) and hit <kbd>return</kbd>. You should see python REPL (Read, Evaluate, Print, Loop)

```
Python 3.11.7 (main, Jan  5 2024, 00:17:30) [Clang 15.0.0 (clang-1500.1.0.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
Type `exit()` or `quit()` to get out of REPL.

#### 2.2 Install `pip`

`pip` is a package manager for python. Let's install it 

```bash
sudo port install py311-pip
```
{: .nolineno}

To install a python package, for e.g. *Django*, you can run

```bash
pip-3.11 install django
```
{: .nolineno}

To make `pip-3.11` the default `pip3`, run

```bash
sudo port select --set pip3 pip311
```
{: .nolineno}

Now you can use just `pip3` instead of `pip-3.11`

```bash
pip3 install django
```
{: .nolineno}


#### 2.3 Install python virtual environment wrapper  

It is a best practice to use virtual environments when working on python projects. This gives you the flexibility to use different versions of same package/library for different python projects/applications. (Read more on [official doc](https://docs.python.org/3/tutorial/venv.html)).  

`venv` is installed by default with python. To confirm, run   
```bash
python3 -m venv -h
``` 
{: .nolineno} 

To manage virtual environment we will install another python package called `virtualenvwrapper`. We will install `virtualenvwrapper` using `pip3` 

```bash
pip3 install virtualenvwrapper
```
{: .nolineno}

Open `~/.bash_profile` file  

```bash
nano ~/.bash_profile
```
{: .nolineno}

and paste these lines at the end of the file

```bash
########## Start 'virtualenvwrapper' Setup ############

export VIRTUALENVWRAPPER_PYTHON=/opt/local/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_VIRTUALENV=~/Library/Python/3.11/bin/virtualenv
source ~/Library/Python/3.11/bin/virtualenvwrapper.sh

########## End 'virtualenvwrapper' Setup   ############
```
{: .nolineno}

Press keys <kbd>⌃ control</kbd> + <kbd>X</kbd> and then press `y` to save. Relaunch the terminal and then run `mkvirtualenv myvenv` to create a virtual environment `myvenv`. To start working with this virtual environment run the command `workon myvenv`. If everything is installed properly then this virtual environment will be activated. To deactivate it, run `deactivate`. Read more about `virtualenvwrapper` and how to use it on [official doc](https://virtualenvwrapper.readthedocs.io/en/latest/index.html).

### 3. Install Visual STudio Code

Download Visual Studio Code source code editor from its [official site](https://code.visualstudio.com/). Extract the executable and move it to the application folder.    
Create a folder named `hello`{: .filepath} and navigate to it and launch VSC (By starting VS Code in a folder, that folder becomes your "workspace". Read more on [official doc](https://code.visualstudio.com/docs/python/python-tutorial#_start-vs-code-in-a-workspace-folder)) 

```bash
mkdir hello
cd hello
code .
```

![vsc1](/assets/img/media/py-install/vsc-welcome.png){: width="720" height="600" }

VS Code comes with hundreds of free extensions. For this tutorial we will install only two extensions, *Python* and *Pylance*. Click on extension button on the activity bar and install *Python* and *Pylance* extensions.

![vsc2](/assets/img/media/py-install/vsc-ext.avif){: width="720" height="600" }

You can create new virtual environments in VSC using Command Palette, but we have already created one in step #3 and therefore we will use that one. Type <kbd>⌘ command</kbd>+<kbd>shift</kbd>+<kbd>P</kbd> to open Command Palette. Type `Python: Select Interpreter` and select it. Then choose the python interpreter from `myvenv` (*Python 3.11.7 ('myvenv')*).

![vsc2](/assets/img/media/py-install/vsc-interpreter.png){: width="720" height="600" }

Now create a [new code file](https://code.visualstudio.com/docs/python/python-tutorial#_create-a-python-source-code-file) `hello.py` in `hello`{: .filepath} directory and write a line `print("Hello, World!)` in it. Click the [play button](https://code.visualstudio.com/docs/python/python-tutorial#_run-python-code) in the top-right side of the editor to run the python code.

Congratulations! You have just created your first python project. 

> Visual Studio Code comes with integrated source control and includes git support. Alternatively you can download GitHub Desktop (https://desktop.github.com/) that also comes with integrated git. If you are using any one of these then skip step #5
{: .prompt-info}  


<!-- ### 4. Install Git

> Follow this step only if you want to use git command line.
{: .prompt-warning} 

In step #1 we installed *Apple’s Developer Tools*. It installs a set of tools for development and along with other tools it also install git. In the terminal, run 

```bash
git --version
```
{: .nolineno}
and it will return something like  

```
git version 2.39.3 (Apple Git-145)
```

We will use this git. If you want to use the latest version then you can install it using `port`  

```bash
sudo port install git
```
{: .nolineno}

Setup your git [username](https://docs.github.com/en/get-started/getting-started-with-git/setting-your-username-in-git) and commit [email address](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address).

```bash
git config --global user.name "username"
git config --global user.email "useremail@host.com"
```

Replace `username` and `useremail@host.com` with your's.

Now it's time to learn version control using [git](https://rogerdudler.github.io/git-guide/) -->

<!-- ### 5. Setting up a GitHub account -->

<!-- Create a [GitHub account](https://github.com/) to host your code on cloud and share it other developers.  

After that you need to connect your git to GitHub. You can either connect over HTTPS os SSH. For beginner I would recommend SSH. Follow the following docs:  

1. [Generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key) and adding to he ssh-agent

2. [Add the generated key to your github account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

3. [Add your local repo hosted code to GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github#adding-a-local-repository-to-github-using-git)

To use multiple github accounts on Mac follow this [tutorial](https://www.haccks.com/posts/github-multi-ssh-key/) -->

### 4. Setting up a GitHub account

Follow this step by step tutorial to setup git and github on Mac: [How to setup git and github on Mac OS?](https://www.haccks.com/posts/setup-git-and-github-on-mac/).