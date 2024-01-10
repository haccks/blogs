---
title: How to install MacPorts on Mac? 
description: "A beginner tutorial on how to install MacPorts on Mac and its usage as a package manager."
date: 2023-08-14 5:00:00 +0530
categories: [mac]
tags: [macports]  # TAG names should always be lowercase
# render_with_liquid: false
# pin: true
---

## A step by step tutorial to install MacPorts on Mac and use it to to manage packages.

There are two popular package manager for Mac, [Macports](https://www.macports.org/) and [Homebrew](https://brew.sh/). Both package managers are great and have their own strength. This blog is all about installation and usage of MacPorts. 

### 1. Introduction

MacPorts is a package manager for Mac operating system. It is an easy to use system for *compiling*, *installing*, and *managing* open source software.  

> A package is a collection of files that are bundled together and can be installed and removed as a group.
{: .prompt-tip }
<!-- A package manager lets you load packages into memory. A package is a set of routines and data types that is stored as a resource of type 'PACK'. -->

Conceptually Macports is divided into two parts:
+ MacPorts Base: The software that we will install (the package manager)
+ Set of available *ports*: A port is a set of specifications contained in a [Portfile](https://guide.macports.org/#development.introduction) that defines an application, its characteristics, and any files or special instructions required to install it.   
This allows you to use a single command to tell MacPorts to automatically download, compile, and install applications and libraries
(Ref: [doc](https://guide.macports.org/#using))

### 2. Prerequisite

+ You need to install Apple's Developer Tools

Open spotlight by typing <kbd>⌘ command</kbd>+<kbd>space</kbd> and then type "terminal" in the spotlight and launch the terminal. Run the command below and install Apple's Command Line Developer Tools: 

```bash
xcode-select --install
```
{: .nolineno}

+ Few ports require [xcode installation](https://developer.apple.com/xcode/) but for now you can skip this step. You can install it if MacPorts tells you to do so.

### 3. Installing MacPorts on Mac

There are two ways to install MacPorts:
+ Install a binary package (recommended)
+ Install it from source
+ Git install

We will use binary package to install it. It is easy and sets all the shell environment variables need to run port. 

1. Download the [latest version](https://www.macports.org/install.php) of MacPorts from it's official page  
2. Double click the downloaded installer (.pkg file) 
3. Once installation is finished, launch the terminal and run

```bash
port version
```
{: .nolineno}  

You can run below command to check if the shell environment variables are set  

```bash
echo $PATH
```
{: .nolineno}  

You should see   

```
/opt/local/bin:/opt/local/sbin:$PATH
```
at the beginning of the path.

### 4. Using MacPorts

Let's Install a port (a package) using MacPorts. If you want to install a version of python or MySQL then first you have to check if that version is available. We can `search` action with `--regex` switch to search all the available versions of python

```bash
port search --name --line --regex '^python\d*$'
```
{: .nolineno} 

This will return an output like this 

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

To install python311, run 

```bash
sudo port instal python311
```
{: .nolineno}

To check the installation, launch the terminal again and type  

```bash
python3.11 --version
```
{: .nolineno}

and you will get `Python 3.11.7`.

MacPorts install it's binary in `/opt/local/bin`. Upon running command  

```bash
which python3.11
```
{: .nolineno}

It will return `/opt/local/bin/python3.11`.

> You can also install multiple ports together. For example, below command will install php 8.1, MySQL 8.1 and apache 2
{: .prompt-tip }
```bash
sudo port instal php81 mysql81 apache2
```


### 5. Common MacPorts Commands
+ port help 
+ sudo port selfupdate
+ port list
+ port installed
+ port outdated
+ sudo port upgrade
+ port info
+ port deps
+ port notes
+ sudo port clean
+ sudo port uninstall


### 6. Troubleshoot

