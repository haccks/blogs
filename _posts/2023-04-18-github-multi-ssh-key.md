---
title: How to use multiple github accounts on Mac?
description: If you have two github accounts for personal and work use then you might have some issues using both accounts on same Mac. If this is the case then this article will help you to setup both accounts on your Mac. 
date: 2023-04-18 14:00:00 +0530
categories: [dev-ops, version-control]
tags: [github, git, ssh, mac, github-account, multiple-github-accounts]     # TAG names should always be lowercase
render_with_liquid: false
---

Like many others, I too have two github accounts, one for personal use and another for work. I was having trouble using both accounts on my mac for personal and office projects. I tried some tutorials and none worked out for me not because they were wrong but the instructions were not comprehensive. In this article I am putting together all the necessary steps to make it work.
 

Follow the steps below:  

## 1. Generate SSh keys
Assuming you have two different accounts on github: `personal@domain.com` and `work@domain.com`. First, generate two different ssh keys for both of your accounts (follow the [detailed instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for generating ssh keys)


```bash
ssh-keygen -t ed25519 -C personal@domain.com
ssh-keygen -t ed25519 -C work@domain.com
```

    
This will create public/private key pairs for each of these accounts. They should be located in `~/.ssh`{: .filepath} directory. Assuming generated key files are following

```
~/.ssh/id_ed25519_personal        # Private key for personal account
~/.ssh/id_ed25519_personal.pub    # Public key for personal account
~/.ssh/id_ed25519_work
~/.ssh/id_ed25519_work.pub
```

Once the keys are generated run this command   

```bash
eval "$(ssh-agent -s)"
```

## 2. Modify the `~./ssh/config`{: .filepath} file
You need to modify `/.ssh/config`{: .filepath} file. If it is already there then good else create it by   
```bash
touch ~/.ssh/config
```  

Open `~/.ssh/config`{: .filepath} file and modify it as follows

> <pre><code>#GitHub for personal
Host <b>github.com</b>                    # Notice the host for personal
  HostName github.com
  User git
  IdentityFile <b>~/.ssh/id_ed25519_personal</b>
>
#Github for work
Host <b>github.com-work</b>               # Notice the host for work 
  HostName github.com
  User git
  IdentityFile <b>~/.ssh/id_ed25519_work</b>
</code></pre>

Save the file.
  
## 3. Add keys to keychain
 
```bash
ssh-add -D           # Will delete all keys added previously
ssh-add -K ~/.ssh/id_ed25519_personal
ssh-add -K ~/.ssh/id_ed25519_work
```
     
Check the added keys with `ssh-add -l` if they are added. 

## 4. Add public keys to your corresponding accounts
**Do not skip this step!** 

Add the public key to your account on GitHub. Github already has good documentaion for it. For more information, see [Adding a new SSH key to your GitHub account](https://docs.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account).

## 5. Clone the repo
**Now is the tricky part**.  

If you want to use your `personal` account then copy the remote repo link from github and clone it as
 
> <pre><code>git clone <b>git@github.com</b>:personal/project.git
</code></pre>

or reset if you already have cloned 

> <pre><code>git remote set-url origin <b>git@github.com</b>:personal/project.git
</code></pre>
    
     
if you want to use your `work` account then copy the remote repo url from github and modify the url by *replacing the host **`git@github.com`** by the host **`git@github.com-work`***
  
> <pre><code>git clone <b>git@github.com-work</b>:company/project.git
</code></pre>

Same rule for setting remote `origin` url

> <pre><code>git remote set-url origin <b>git@github.com-work</b>:company/project.git
</code></pre>
    
## 6. Test it out
Create a repo on github and then clone it as instructed above. Change the current working directory to the local repository. Configure the user email and name (you can do globally too but I prefer this way) for this repository.  
If you cloned the repo for `work` account then configure it as

```bash
git config --local user.email "work@domain.com"
git config --local user.name "work"
```
Now you can push/pull to/from github remote repository.
