+++
title = "MacOS Development Setup Guide"
date = "2026-07-26"
tags = [
    "development",
    "guide",
    "mac",
]
+++
---

When I got this MacBook Pro 14" with 24 GB of RAM and 1 TB of storage a few months ago, the first thing I wanted to do was to have a development setup that didn't involve separate version managers for each programming language, running the risk of dependency hell, and constant frustration in managing those.

Things have changed a lot over the last few years since the last time I set up my workflow. Before, I used a combination of `miniconda`, `nvm`, etc. But it's 2026, and what was great a few years ago is now replaced by newer tools that do the same job, but much faster and more efficiently.

In this post, I'll go over each step of setting up a working development environment on MacOS. This tutorial assumes you're starting with a fresh copy of the operating system, a complete clean slate to work with.

It will focus primarily on the essentials, as well as installing VS Code (and Docker), and setting up Git. However, I will also include a section for setting up the Mac to work with local LLMs (which was my rationale for getting this Mac!) later down the road! 😎

{{< toc >}}

## Install The Essentials

### 1. Xcode Command Line Tools

The first thing to install is the **Xcode Command Line Tools** package. This will install Unix developer and compiler tools such as `clang`, `make`, `git`, etc. It is crucial for the next step, which is installing **Homebrew**.

To install it, enter the following command in the Terminal:
```bash
xcode-select --install
```

### 2. Homebrew

**Homebrew** is a command-line interface (CLI) based package manager for macOS (as well as Linux). This is a useful tool that eliminates the need to download `.dmg` files, manage updates and dependencies, and, all in all, helps keep things simple. 

To install **Homebrew**, enter this command in the Terminal (or refer to the [official site](https://brew.sh/)):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Following that, you will need to enter these lines (it should indicate it during your installation).
`your_user` is your Mac user (or Home) folder name in the `/Users` directory.

```bash
echo >> /Users/your_user/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> /Users/your_user/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

### 3. Mise

With **Homebrew** installed, the first package to install is **Mise**. It is a Rust-based utility tool that manages language versions, environments, and a task runner. Previously, this was done using `nvm`, `pyenv`, and so on. With this, we can manage language versions in one place, and we will install `python`, `node`, and `java`.

To install `mise`, enter the following command in the Terminal:

```bash
brew install mise
```

Include the zsh activation command in the `.zshrc`, and run `source ~/.zshrc` to reload the Zsh configuration file in the current Terminal session

```bash
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
source ~/.zshrc
```

To confirm `mise` is installed, run `mise -V` to check which version is installed. If it does not give the intended message, you may need to redo the earlier steps.

Now that the tool is installed, we will install `node`, `python`, and `java`; run the following commands sequentially. You are free to choose whichever version suits you the best, but as of this writing, these are the versions of the language I am using. 

```bash
mise use --global node@lts
mise use --global python@3.13
mise use --global java@corretto-25
```

And to confirm each language is installed, run a `which` for each of the languages to confirm their existence: `which node`, `which python`, `which java`

### 4. uv

**uv** is a Rust-based Python package and version manager that serves as a faster and more efficient alternative to `pip`, `pyenv`, etc. 

It can be installed via **Homebrew**:

```bash
brew install uv
```

To use it, create an example folder `test_uv` on your Desktop (or wherever you prefer), and access it in VS Code (see below to install) or Terminal (via `cd Desktop/test_uv`)

Inside the folder, to initialize uv, run `uv init`. This sets up an initial project scaffolding, as well as `git` version control for your project.

To create a virtual environment, run `uv sync`, and it will use information stored within `pyproject.toml` to install the specified dependencies. Right now, there is nothing there, so we will try to install one dependency, `bs4` (BeautifulSoup4)

Activate the virtual environment (`source .venv/bin/activate`) and run `uv add bs4`, and you should get the following output:

```bash
(test-uv) vkumar@Viveks-MacBook-Pro test_uv % uv add bs4
Resolved 5 packages in 448ms
Prepared 4 packages in 175ms
Installed 4 packages in 4ms
 + beautifulsoup4==4.15.0
 + bs4==0.0.2
 + soupsieve==2.9.1
 + typing-extensions==4.16.0
```

## VS Code & Docker

With the essentials installed, you have everything that is needed for your dev setup on your Mac. To be able to work with those languages, for instance, if you want to set up a Python project, you'd need a code editor or an integrated development environment (IDE). In this tutorial, we will be installing the code editor, **Visual Studio Code. 

To install it, enter the following command. The `--cask` flag tells Homebrew to install and manage the graphical user interface (GUI) applications.

```bash
brew install --cask visual-studio-code
```

This step is completely optional, but it is good to have it for development needs, and that is **Docker**. It is an open-source platform that lets you build, test, and deploy applications in isolated environments called containers. A good use case is if you want to use `MySQL` for your project; Docker avoids the need to install it with brew. This has the benefits of not having background services, and only running when you need to run them. 

Enter the following command to install Docker:

```bash
brew install --cask docker
```

## Git & Github

**Git** is an open-source program that allows you to keep track of your projects in version control. **GitHub** is a platform where you can put those project files tracked with Git on the web for others to see.

`git` should already have been installed if you have done the first step to install **Xcode Command Line Tools**. By default, it will not work with GitHub off the bat; it will require you to set it up.

First, you will need to configure the credentials for Git. Add in your username and email, so replace the placeholder `my_user` with your actual username and email.

```bash
git config --global user.name "my_user"
git config --global user.email "my_user@gmail.com"
```

To make GitHub work with your local git repository, you will need to set up authentication. You can use **Personal Access Tokens (PATs)**, but it is recommended to use **SSH Keys** instead.

```bash
ssh-keygen -t ed25519 -C "my_user@gmail.com"
```

 Run `pbcopy < ~/.ssh/id_ed25519.pub` to get the public key into your clipboard, and go to the [SSH Keys](https://github.com/settings/keys) page on GitHub. Click on **New SSH Key** and paste it.

Using an SSH Key, you will clone GitHub projects using SSH. You can do this automatically in VS Code, or run the following command to clone my GitHub repo `dna-to-amino-translator` (this is just for an example, but it should still work!)

```bash
git@github.com:vkumar825/dna-to-amino-translator.git
```