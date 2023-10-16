---
title: Mac Terminal Customization with Oh-My-ZSH and powerlevel10k
description: ""
date: 2023-10-08 6:00:00 +0530
categories: [mac]
tags: [terminal, iTerm2, powerlevel10k, zsh, oh-my-zsh]     # TAG names should always be lowercase
render_with_liquid: false
---

![banner](/assets/img/media/p10k/iterm-omz-p10k.avif){: width="720" height="600" }

<!-- ### System information

![sys-info](/assets/img/media/p10k/sys-info.png){: width="720" height="600" } -->


## A step by step guide to customize your terminal on Mac

### 1. Install Apple's Developer Tools

Type "terminal" in the spotlight (type <kbd>⌘ command</kbd>+<kbd>space</kbd>) and launch it. Run the command below and install Apple's Command Line Developer Tools: 

```bash
xcode-select --install
```
{: .nolineno}  

### 2. Install a package manager for Mac
You can either install [MacPorts](https://www.macports.org/) or [Homebrew](https://brew.sh/). I prefer MacPorts over homebrew.
Download a MacPorts package from its [official website](https://www.macports.org/install.php) for your operating system.

After installation is complete fire up your terminal and run `port version`. If it is installed successfully it will print the version installed (`Version: 2.8.1`).   

### 3. Install zsh (Only for macOS older than Catalina)

The [default login shell](https://support.apple.com/en-in/guide/terminal/trmlstrtup/2.14/mac/14.0) for newer macOS is `zsh` (started with macOS Catalina). If you are using older version then you can install the latest version of `zsh`  

```bash
sudo port install zsh
```
{: .nolineno}

We are done with terminal here , so close it for now.

### 4. Install iTerm2

<!-- ![iTerm2](/assets/img/media/p10k/iterm2-logo.jpeg)  -->
iTerm2 is a replacement for Mac Terminal. It has lot of [features](https://iterm2.com/features.html) and supports lot of plugins which makes it fun and easy to use. Download iTerm2 from its [official website](https://iterm2.com/) and install it.

### 5. Install [oh-my-zsh](https://ohmyz.sh/)

<!-- ![ohmyzsh](/assets/img/media/ohmyzsh.png){: width="500" height="100" } -->

To install oh-my-zsh, you can either use `curl`  
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
{: .nolineno}

or `wget`  

```bash
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
{: .nolineno}

Open/relaunch iTerm2. Run `ls -l` and it should look like   

![term-omz](/assets/img/media/p10k/iterm-omz.avif){: width="720" height="600" }

### 6. Install [powerlevel10k](https://github.com/romkatv/powerlevel10k)  

Download powerlevel10k theme

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
{: .nolineno}  

Open `~/.zshrc` file and replace the line 

    ZSH_THEME="robbyrussell"

with 

    ZSH_THEME="powerlevel10k/powerlevel10k"

### 7. Download and install a Nerd Font  

Install a *monospaced nerd font* of your choice from [nerd-fonts](https://github.com/ryanoasis/nerd-fonts). My list of top 5 fonts for terminal and programming:  

+ [JetBrains Mono NF (Regular)](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/patched-fonts/JetBrainsMono/Ligatures/Regular/JetBrainsMonoNerdFont-Regular.ttf)
+ [Fira Code NF (Regular)](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/patched-fonts/FiraCode/Regular/FiraCodeNerdFont-Regular.ttf)
+ [Hack NF (Regular)](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/patched-fonts/Hack/Regular/HackNerdFont-Regular.ttf)
+ [Iosevka NF (Regular)](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/patched-fonts/Iosevka/Regular/IosevkaNerdFont-Regular.ttf)
+ [Meslo LGS NF (Regular)](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/patched-fonts/Meslo/S/Regular/MesloLGSNerdFont-Regular.ttf)

You can use any nerd font but for me *Meslo* works best with powerlevel10k (and is recommended by the developer).   

+ Open iTerm2 
+ type <kbd>⌘ command</kbd>+<kbd>,</kbd>
+ navigate to `Profiles > Text`{: .filepath} tab
+ set *Font* to "MesloLGS Nerd Font"  
+ set font size to `14`

![term-font](/assets/img/media/p10k/iterm-font.avif){: width="720" height="600" }

### 8. Configure powerlevel10k
Run   

```bash
p10k configure
```
{: .nolineno} 

and configure it (try out your own combinations!).

![p10k-conf](/assets/img/media/p10k/p10k-config.avif){: width="720" height="600" } 

My choices for the configuration:
<!-- + Does this look like a diamond (rotated square)?: **y** -->
<!-- + Does this look like a lock?: **y** -->
<!-- + Does this look like an upwards arrow?: **y** -->
<!-- + Do all these icons fit between the crosses?: **y** -->
<!-- + Prompt Style: **(3) Rainbow.** -->
<!-- + Character Set: **(1) Unicode.** -->
<!-- + Show current time?: **(1)  12-hour format.** -->
<!-- + Prompt Separators: **(1)  Angled.** -->
<!-- + Prompt Heads: **(1) Sharp.** -->
<!-- + Prompt Tails: **(1) Flat.** -->
<!-- + Prompt Height: **(2) Two Lines.** -->
<!-- + Prompt Connection: **(1)  Disconnected.** -->

| Settings | Choice |
|----------|--------|
| Does this look like a diamond (rotated square)?   | **y** |
| Does this look like a lock? | **y** |
| Does this look like an upwards arrow? | **y** |
| Do all these icons fit between the crosses? | **y** |
| Prompt Style | **3** |
| Character Set | **1** |
| Show current time? | **1** |
| Prompt Separators | **1** |
| Prompt Heads | **1** |
| Prompt Tails | **1** |
| Prompt Height | **1** |
| Prompt Connection | **1** |
| Prompt Frame | **2** |
| Frame Color | **1** |
| Prompt Spacing | **1** |
| Icons | **2** |
| Prompt Flow| **1** |
| Enable Transient Prompt? | **n** |
|  Instant Prompt Mode | **1** |


### 9. Install color schemes for iTerm2  

Download color schemes from [iTerm2-color-schemes](https://github.com/mbadolato/iTerm2-Color-Schemes/zipball/master) or use the git [repo](https://github.com/mbadolato/iTerm2-Color-Schemes). Save it to `~/Download`{: .filepath} folder.

Double click on the downloaded file to unzip it.  

+ Open iTerm2 
+ type <kbd>⌘ command</kbd>+<kbd>,</kbd>
+ navigate to `Profiles > Colors`{: .filepath} tab
+ Click on *Color Presets*
+ Click on *Import*
+ Navigate to the `~/Download/iTerm2-Color-Schemes-master/schemes`{: .filepath} folder 
+ Select the profile `MaterialDesignColors.itermcolors`{: .filepath} and import (you can select any profile you would like to import, give it a try and see what color scheme you like!)

![color-scheme](/assets/img/media/p10k/color-preset.avif){: width="720" height="600" } 

Now relaunch your iTerm2.  

### 10. More Customization

This step will add *context* and a green "❯" on left prompt, *cpu load* and *ram* on right prompt.  
Open `~/.p10k.zsh`{: .filepath} file.   

+ Find element `typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS` and add `context` segment. It should look like this

```bash
  typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
    # =========================[ Line #1 ]=========================
    os_icon                 # os identifier
    context                 # user@hostname
    dir                     # current directory
    vcs                     # git status
    # =========================[ Line #2 ]=========================
    newline                 # \n
    # prompt_char           # prompt symbol
  )
```
{: .nolineno}

+ Similarly, find `typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS` and uncomment `cpu` and `load` segment and comment `context` segment.  

+ Find line `typeset -g POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX='%244F╰─` and replace it with  

```bash
typeset -g POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX='%244F╰─%76F❯'
```
{: .nolineno}

+ Change OS identifier color to change to color of apple logo 

```bash
typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=7
typeset -g POWERLEVEL9K_OS_ICON_BACKGROUND=0
```
{: .nolineno}

+ Change default and privileged context from `user@hostname` to `user`

```bash
  # Context format when running with privileges: user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_ROOT_TEMPLATE='%n'
  # Context format when in SSH without privileges: user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_{REMOTE,REMOTE_SUDO}_TEMPLATE='%n@%m'
  # Default context format (no privileges, no SSH): user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_TEMPLATE='%n'
```
{: .nolineno}  

and comment out this line to always show the context

```bash
typeset -g POWERLEVEL9K_CONTEXT_{DEFAULT,SUDO}_{CONTENT,VISUAL_IDENTIFIER}_EXPANSION=
```

+  Change current directory background color

```bash
 # Current directory background color.
typeset -g POWERLEVEL9K_DIR_BACKGROUND=4
# Default current directory foreground color.
typeset -g POWERLEVEL9K_DIR_FOREGROUND=0
```
{: .nolineno}  

To see the effect run  

```bash
source ~/.p10k.sh
```
{: .nolineno} 

### 11. Install plugins for zsh 

+ Syntax Highlighting Plugin:  

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```
{: .nolineno}  

+ AutoSuggestion Plugin:  

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
{: .nolineno}  

<!-- + Autocomplete Plugin:

```bash
git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git $ZSH_CUSTOM/plugins/zsh-autocomplete
```
{: .nolineno} -->

Open `~/.zshrc` file. Find `plugins` section and install the downloaded plugins  

```bash
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
)
```

### 12. Customize Mac Terminal

+ Navigate to the `~/Download/iTerm2-Color-Schemes-master/terminal`{: .filepath} folder (Downloaded in step 9)
+ Right click on `MaterialDesignColors.terminal`{: .filepath} and open it with Terminal app
+ Type <kbd>⌘ command</kbd>+<kbd>,</kbd> and go to `Profiles`{: .filepath}. You will see `MaterialDesignColors` profile on the left pan. Select it and click *Default* button at the bottom.
+ Change font to "MesloLGS Nerd Font".
+ Relaunch the Terminal app.

### 13. Advanced iTerm2 settings

+ Open iTerm2 
+ Set a hotkey to show/hide all iTerm2 windows. Navigate to `Keys > Hotkey`{: .filepath} and type <kbd>⌘ command</kbd>+<kbd>i</kbd>.
+ *Remove tab bar*: Navigate to `Appearance > General`{: .filepath} and set **Theme** to *minimal*.
+ *Unlimited scrollback*: Navigate to `Profiles > Terminal`{: .filepath} and check *Unlimited scrollback*
+ Type <kbd>⌘ command</kbd>+<kbd>return</kbd> to enter/exit full screen mode
