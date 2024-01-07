---
title: How to setup git and github on Mac OS?
description: "A step by step guide to setup git and github on mac"
date: 2024-01-07 1:20:00 +0530
categories: [mac, devops]
tags: [git, github]     # TAG names should always be lowercase
render_with_liquid: false
---

## A step by step guide to setup git and github on mac

### 1. Install Apple's Developer Tools

Open spotlight by typing <kbd>⌘ command</kbd>+<kbd>space</kbd> and then type "terminal" in the spotlight and launch the terminal. Run the command below and install Apple's Command Line Developer Tools: 

```bash
xcode-select --install
```
{: .nolineno}

### 2. Install a package manager for Mac 

You can either install [MacPorts](https://www.macports.org/) or [Homebrew](https://brew.sh/). I prefer MacPorts over homebrew and will use it for this tutorial.
Download a MacPorts package from its [official website](https://www.macports.org/install.php) for your operating system.

After installation is complete fire up your terminal again and run `port version`. If it is installed successfully it will print the version installed.

### 3. Install Git

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

Now it's time to learn version control using [git](https://rogerdudler.github.io/git-guide/)

### 4. Setting up a GitHub account

Create a [GitHub account](https://github.com/) to host your code on cloud and share it other developers.  

### 5. Connect git with GitHub

You need to connect your git to GitHub. You can either connect over HTTPS os SSH. For beginner I would recommend SSH. Follow the following docs:  

1. [Generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key) and adding to he ssh-agent

2. [Add the generated key to your github account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

### 6. Create a local repo and host it on GitHub
Todo:
[Add your local repo hosted code to GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github#adding-a-local-repository-to-github-using-git)


To use multiple github accounts on Mac follow this [tutorial](https://www.haccks.com/posts/github-multi-ssh-key/)