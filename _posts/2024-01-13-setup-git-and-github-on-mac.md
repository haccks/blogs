---
title: How to setup git and github on Mac OS?
description: "A step by step guide to setup git and github on mac"
date: 2024-01-13 1:05:00 +0530
categories: [mac, devops]
tags: [git, github]     # TAG names should always be lowercase
render_with_liquid: false
---

## A step by step guide to setup git and github on mac

### 1.Prerequisite

Make sure that following tools are installed on your Mac

+ Apple's Developer Tools
+ MacPorts Package Manager

If you don't know what are these then don't worry. Follow [this beginner tutorial to  install Apple's Developer Tools and MacPorts](https://www.haccks.com/posts/macports-install-and-usage/).

### 2. Install Git

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

### 3. Setting up a GitHub account

Create a [GitHub account](https://github.com/) to host your code on cloud and share it other developers.  

### 4. Connect git with GitHub

You need to connect your git to GitHub. You can either connect over HTTPS os SSH. For beginner I would recommend to connect over SSH. Follow the following docs:  

#### 4.1 Generate a new SSH key

Open your terminal and run the command below after replacing `"demo@example.com"` with your email address you used to create your GitHub account in step #3

```bash
ssh-keygen -t ed25519 -C "demo@example.com"
```
{: .nolineno}

This will prompt you for a file to save keys (press <kbd>enter</kbd> for default file location) and then asks for password (choose a password or press <kbd>enter</kbd> for empty password) 

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/xxxx/xxxx/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
```

This will create a pair of private and public keys `id_ed25519` and `id_ed25519.pub` in `~/.ssh` directory.  
Once these keys are generated successfully, run the ssh-agent in the background  

```bash
eval "$(ssh-agent -s)"
```
{: .nolineno}

#### 4.2 Modify your `~./ssh/config` file

Check if `~./ssh/config` exists, else create one

```bash
touch ~./ssh/config
```
{: .nolineno}

Open `~./ssh/config` in your favorite text editor and add these lines

```conf
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519

Host *
   # Uncomment below line if you choose a password to generate keys
   # UseKeychain yes
   AddKeysToAgent yes
   IdentitiesOnly yes
```

#### 4.3 Add SSH keys to your macOS Keychain

```bash
ssh-add -D           # Will delete all identities added previously from the ssh agent
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

Check the added key with `ssh-add -l`.

#### 4.4 Add the generated key to your github account

+ Copy the generated public SSH key on clipboard

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

+ Go to your GitHub account and click on your the avatar on top right corner and then click on **Settings**

+ On the left sidebar, click on **SSH and GPG keys** and then click on **New SSH Key**

+ Add some title if you wish (like "My MacBook's SSH Key")

+ In **Key Type** section, select **Authentication Key**

+ Paste the public key in the **Key** section.

#### 4.5 Test the Connection

```bash
ssh -T git@github.co
```
{: .noloneno}

If you setup your git and GitHub account correctly then the above command should return the below message 

```
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

### 5. Create a local repo and host it on GitHub

Create a new repository `demo`{: .filepath} on GitHub. After creating the repo successfully you will see three options to clone your repo: **HTTPS**, **SSH** and GitHub **CLI**. Choose **SSH** and copy the remote url.

![img](assets/img/media/git-setup/repo-clone.png){: width="720" height="600" }

<!-- If you already have an existing repo then click on the repo and then click on **<>Code** and choose SSH

[!img]() -->

#### 5.1 Clone a Remote Repository

Open your terminal and clone this repo typing `git clone` and then paste your repo remote url. 

```bash
git clone git@github.com:haccks/demo.git
```

Now you can go to the `demo`{: .filepath} directory and start making changes and push it to the GitHub.

#### 5.2 Host a Local Repository

Create a local repo named `demo`{: .filepath}   

```bash
mkdir demo
```
{: .nolineno}

and perform these actions

```bash
cd demo
git init
git add README.md
git commit -m "First commit"

# Connect your local repo with the remote repo
git remote add origin git@github.com:haccks/demo.git

git branch -M main
git push -u origin main
```

If you already have a local repo then ignore the first four commands and use the last three only.

We are done here!

>If you want to use multiple github accounts on Mac then follow this tutorial: [How to use multiple github accounts on Mac?](https://www.haccks.com/posts/github-multi-ssh-key/)
{: .prompt-info}

------------------------
<sub>References:</sub>  
<sub>1. [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
</sub>  
<sub>2. [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).
</sub>