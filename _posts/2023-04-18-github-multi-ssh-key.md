---
title: How to use multiple GitHub accounts on Mac?
description: If you have two GitHub accounts for personal and work use then you might have some issues using both accounts on same Mac. If this is the case then this article will help you to setup both accounts on your Mac. 
date: 2023-04-18 14:00:00 +0530
categories: [mac, dev-ops, version-control]
tags: [github, git, ssh, github-account, multiple-github-accounts]     # TAG names should always be lowercase
render_with_liquid: false
---

Like many others, I too have two GitHub accounts, one for personal use and another for work. I was having trouble using both accounts on my mac for personal and office projects. I tried some tutorials and none worked out for me not because they were wrong but the instructions were not comprehensive. In this article I am putting it together all the necessary steps to make it work.  

> This solution works for macOS Monterey (Version 12.x.x) and later. 
{: .prompt-info }
 

Follow the steps below:  

## 1. Generate SSh keys
Assuming you have two different accounts on GitHub: `personal@domain.com` and `work@domain.com`. First, generate two different ssh keys for both of your accounts (follow the [detailed instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for generating ssh keys)


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

```conf
#GitHub for personal
Host github.com                              # Notice the host for personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal    # Private key for personal account

#Github for work
Host github.com-work                         # Notice the host for work 
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work        # Private key for work account

Host *
   # UseKeychain yes                           # If you choose a password to generate keys
   AddKeysToAgent yes                        # Load the keys from keychain into ssh-agent automatically
   IdentitiesOnly yes                        # Tells the ssh-agent server to use the IdentityFiles specified above for each host
```

Save the file. If you choose to add a password to generate your ssh keys, then uncomment line no. 14 in the above config file.

> Do not add any identity file in `Host *` as suggested [here](https://apple.stackexchange.com/a/333547). We are using same host (`github.com`) for both account. This will make the server to always choose the key for the first added identity to the `ssh-agent`. Adding `IdentitiesOnly yes` will prevent the server to make the connection using the first available key and force it to use the respective keys placed under different hosts above.
{: .prompt-tip}
  
## 3. Add keys to macOS Keychain
 
```bash
ssh-add -D           # Will delete all identities added previously from the ssh agent
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_personal
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_work
```
`--apple-use-keychain` option will store keys to the keychain and it will make keys persistent boot to boot.  

> If `ssh-add -d` is used to remove identities then each keys from keychain will be removed. (Read the man page for `ssh-add` for more detail).
{: .prompt-warning }
     
Check the added keys with `ssh-add -l` if they are added. 

>At this point if you reboot your system then keys will be loaded to ssh-agent automatically from the keychain!
{: .prompt-info}

## 4. Add public keys to your corresponding accounts
> Do not skip this step!
{: .prompt-warning }

Add public keys to your account on GitHub. Github already has good documentation for it. For more information, see [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=mac).

## 5. Test the connection  
Test the connection for both accounts  

```bash
ssh -T git@github.com
ssh -T git@github.com-work
```
If you followed everything to this point, this should return  
```
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

## 6. Clone the repo
**Now is the tricky part**.  

If you want to use your `personal` account then copy the remote repo link from GitHub and clone it as
 
```bash
git clone git@github.com:personal/project.git
```

or reset if you already have cloned 

```bash
git remote set-url origin git@github.com:personal/project.git
```
    
     
if you want to use your `work` account then copy the remote repo url from GitHub and modify the url by *replacing the host **`git@github.com`** by the host **`git@github.com-work`***
  
```bash
git clone git@github.com-work:company/project.git
```

Same rule for setting remote `origin` url

<!-- > <pre lang="bash"><code>git remote set-url origin <b>git@github.com-work</b>:company/project.git
</code></pre> -->

```bash
git remote set-url origin git@github.com-work:company/project.git
```
    
## 7. Test it out
Create a repo on GitHub and then clone it as instructed above. Change the current working directory to the local repository. Configure the user email and name (you can do globally too but I prefer this way) for this repository.  
If you cloned the repo for `work` account then configure it as

```bash
git config --local user.email "work@domain.com"
git config --local user.name "work"
```
Now you can push/pull to/from GitHub remote repository.
