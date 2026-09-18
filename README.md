# IT & Cybersecurity Knowledge Base

A practical, continuously developed knowledge base covering IT infrastructure, system administration, networking, cloud platforms, cybersecurity, identity management, monitoring, automation, governance, and day-to-day IT operations.

The purpose of this repository is to collect technical knowledge in a format that is:

- easy to read
- easy to search
- easy to maintain
- easy to update
- independent from any specific documentation platform
- available directly from GitHub
- usable locally with tools such as Obsidian
- version controlled with Git

All knowledge articles are written primarily in **Markdown (**`**.md**`**)**.

---

# How to Use This Knowledge Base

There are two main ways to use this repository.

## 1. Read Directly on GitHub

You do not need to install anything.

Simply browse the repository using GitHub and open the Markdown files directly in your browser.

GitHub automatically renders Markdown, including:

- headings
- tables
- lists
- code blocks
- links
- images
- formatting

This is the easiest option if you only want to read the documentation.

---

## 2. Use the Knowledge Base Locally

For the best experience, I recommend downloading the repository and opening it with **Obsidian**.

Obsidian provides:

- fast full-text search
- easy navigation between documents
- Markdown editing
- backlinks
- graph view
- tags
- local offline access
- quick navigation between related topics
    

Obsidian is optional.

The repository uses standard Markdown wherever possible, so the documentation can still be read using:

- GitHub
- VS Code
- Neovim
- any Markdown editor
- any text editor
    

---

# Repository Structure

The repository is intentionally kept simple.

```
knowledge-base/
├── README.md
├── content/
└── templates/
```

## `content/`

Contains the actual knowledge base.

The high-level structure is organized by technical domain.

```
content/
├── 01-architecture/
├── 02-identity-access/
├── 03-networking/
├── 04-systems/
├── 05-endpoint-management/
├── 06-cloud-platforms/
├── 07-collaboration-productivity/
├── 08-cybersecurity/
├── 09-monitoring-soc/
├── 10-backup-recovery/
├── 11-automation-scripting/
├── 12-governance-compliance/
├── 13-it-operations/
├── 14-projects-migrations/
└── 90-reference/
```

Topics are grouped by their primary technical purpose rather than by vendor.

For example:

```
content/
└── 03-networking/
    └── firewalls/
        └── fortigate/
```

rather than creating a top-level folder for FortiGate.

This makes the structure easier to understand and keeps it useful even when technologies or vendors change.

---

# Topic Structure

A topic will normally contain:

```
topic-name/
├── topic-name.md
└── manifest.yml
```

The `topic-name.md` file contains the actual documentation intended for readers.

The `manifest.yml` file contains machine-readable information about the topic and is primarily used by the repository itself.

It may support future functionality such as:

- website generation
- navigation
- categorization
- search
- tags
- topic relationships
- document status
- publishing controls
    

If you are using this repository only for learning or as a technical reference, you do **not** need to read or understand the `manifest.yml` files.

## Do I Need the `manifest.yml` Files?

No.

The `manifest.yml` files are not required to understand the knowledge base.

All important learning material is stored inside the Markdown files.

You can simply ignore the manifests and focus on the `.md` files.

### Removing Manifest Files

If you prefer to remove the manifest files completely from your local copy, you can do so.

They are not required for reading the knowledge base.

#### Linux

From inside the repository:

```
find . -type f -name "manifest.yml" -delete
```

#### macOS

From inside the repository:

```
find . -type f -name "manifest.yml" -delete
```

#### Windows PowerShell

From inside the repository:

```
Get-ChildItem -Path . -Recurse -Filter "manifest.yml" | Remove-Item -Force
```

This will remove all files named:

```
manifest.yml
```

from your local copy.

### Important Note About Git

The manifest files are tracked by Git.

If you remove them manually, Git will detect them as locally deleted files.

Running:

```
git status
```

may therefore show entries similar to:

```
deleted: content/03-networking/example/manifest.yml
```

This does not damage the knowledge base, but it means your local repository is no longer identical to the original repository.

For the cleanest update experience, keeping the files and simply ignoring or hiding them is recommended.

If you previously removed the manifest files and want to restore them, run:

```
git restore .
```

Be aware that this command restores **all tracked files with local modifications**, not only manifests.

---

# Templates

The `templates/` directory contains reusable templates for creating new documentation.

For example:

```
templates/
├── topic.md
├── manifest.yml
└── ...
```

Templates help keep documentation consistent across the entire knowledge base.

---

# Recommended Setup: Obsidian

Obsidian is the recommended way to use this repository locally.

You can download Obsidian from:

[https://obsidian.md/](https://obsidian.md/)

After cloning the repository, open Obsidian and select:

**Open folder as vault**

Then select the cloned repository.

For example:

```
knowledge-base/
```

Obsidian will treat the repository as a vault and allow you to navigate and search all Markdown documents.

You do not need to move the files into another Obsidian folder.

The Git repository itself can be the Obsidian vault.

---

# Downloading the Knowledge Base

The recommended way to download this repository is using Git.

Using Git has one major advantage over downloading a ZIP file:

**you can easily download future updates without downloading the entire repository again.**

---

# Installing Git

Before cloning the repository, make sure Git is installed.

Check using:

```
git --version
```

If Git is installed, you should see something similar to:

```
git version 2.x.x
```

---

# Linux

## Install Git

### Arch Linux

```
sudo pacman -S git
```

### Ubuntu / Debian

```
sudo apt update
sudo apt install git
```

### Fedora

```
sudo dnf install git
```

### openSUSE

```
sudo zypper install git
```

Verify the installation:

```
git --version
```

## Clone the Knowledge Base

Open a terminal and move to the directory where you want to store the repository.

For example:

```
cd ~/Documents
```

Clone the repository:

```
git clone https://github.com/MrGulczu/Knowledge-Base
```

Then enter the repository:

```
cd Knowledge-Base
```

You can now open this directory using Obsidian or any Markdown editor.

---

# Windows

## Install Git

Download **Git for Windows** from:

[https://git-scm.com/download/win](https://git-scm.com/download/win)

Install it using the default options unless you have a reason to change them.

After installation, open one of the following:

- PowerShell
- Windows Terminal
- Command Prompt
- Git Bash
    

Verify that Git works:

```
git --version
```

## Clone the Knowledge Base

For example, to clone it into your Documents folder:

```
cd $HOME\Documents
```

Then:

```
git clone https://github.com/MrGulczu/Knowledge-Base
```

Enter the directory:

```
cd Knowledge-Base
```

You can now open this folder as an Obsidian vault.

---

# macOS

Git may already be available on macOS.

Check using:

```
git --version
```

If macOS asks you to install the Command Line Developer Tools, accept the installation.

Alternatively, if you use Homebrew:

```
brew install git
```

Verify:

```
git --version
```

## Clone the Knowledge Base

Open Terminal.

Move to the directory where you want to keep the repository:

```
cd ~/Documents
```

Clone it:

```
git clone https://github.com/MrGulczu/Knowledge-Base
```

Enter the repository:

```
cd Knowledge-Base
```

You can now open the repository as an Obsidian vault.

---

# Updating the Knowledge Base

One of the biggest advantages of cloning the repository with Git is that updates are very easy.

You do **not** need to clone the repository again.

Open a terminal inside the repository.

For example:

```
cd ~/Documents/Knowledge-Base
```

Then run:

```
git pull
```

Git will download the newest changes and update your local copy.

On Windows:

```
cd $HOME\Documents\Knowledge-Base
git pull
```

On Linux and macOS:

```
cd ~/Documents/Knowledge-Base
git pull
```

That is normally all you need.

---

# Typical Update Workflow

After the repository has been cloned once, future updates usually look like this:

```
cd Knowledge-Base
git pull
```

Then open Obsidian and continue reading.

Obsidian will automatically see the updated Markdown files.

---

# Check Whether Updates Are Available

You can retrieve information about the latest version from the remote repository without immediately changing your local files:

```
git fetch
```

Then:

```
git status
```

Git may display something similar to:

```
Your branch is behind 'origin/main' by 5 commits, and can be fast-forwarded.
```

You can then download the updates using:

```
git pull
```

---

# Local Changes

If you only use the repository for reading, updating should normally be as simple as:

```
git pull
```

However, if you modify the Markdown files locally, Git may prevent an update if your changes conflict with newer versions from the repository.

Check your local changes using:

```
git status
```

You can inspect what has changed with:

```
git diff
```

If you want to keep your own notes, consider maintaining them separately or creating your own fork of the repository.

---

# Download Without Git

If you do not want to use Git, GitHub also allows you to download the repository as a ZIP archive.

On the repository page:

1. Click **Code**.
    
2. Select **Download ZIP**.
    
3. Extract the downloaded archive.
    
4. Open the extracted directory in Obsidian or another Markdown editor.
    

This method works, but it is not recommended for regular use because future updates need to be downloaded manually.

Using Git is easier in the long term.

---

# Searching the Knowledge Base

## GitHub

GitHub can be used to search the repository directly.

Use the search field while browsing the repository to search for:

- technologies
- commands
- error messages
- configuration options
- protocols
- products
    

## Obsidian

Obsidian provides very fast local search.

Use:

```
Ctrl + Shift + F
```

or on macOS:

```
Cmd + Shift + F
```

to search across the entire vault.

This can be especially useful when searching for technologies, commands, configuration options, or specific error messages.

---

# Navigating the Knowledge Base

The content is organized primarily by domain.

A typical path may look like:

```
Networking
    ↓
VPN
    ↓
IPsec
    ↓
IKEv2
```

Another example:

```
Identity & Access
    ↓
Microsoft Entra ID
    ↓
Conditional Access
```

If you are unsure where something belongs, think about its primary purpose rather than the vendor that created it.

---

# Markdown

All documentation is written in Markdown.

A Markdown file is simply a text file using lightweight formatting.

Example:

````
# Active Directory

## Overview

Active Directory is a directory service developed by Microsoft.

## Useful Commands

```powershell
Get-ADUser -Filter *
```
````

Markdown can be read directly as plain text, but platforms such as GitHub and Obsidian render it as formatted documentation.

---

# Why Markdown?

Markdown was selected as the primary documentation format because it provides:

- excellent Git integration
- readable version history
- easy editing
- portability
- GitHub compatibility
- Obsidian compatibility
- website integration
- simple linking
- code blocks
- low complexity
- long-term maintainability
    

Markdown files are the primary source of knowledge in this repository.

Other formats such as PDF may be generated for specific purposes, but should not replace the Markdown source unless necessary.

---

# Learning From This Repository

The knowledge base is intended both as:

- technical reference documentation
- learning material
    

You do not necessarily need to read it from beginning to end.

You can start with the area that interests you.

For example, someone learning infrastructure administration could explore:

```
Architecture
↓
Networking
↓
Systems
↓
Identity & Access
↓
Cloud Platforms
↓
Cybersecurity
```

Someone working primarily with security might instead follow:

```
Identity & Access
↓
Cybersecurity
↓
Monitoring & SOC
↓
Governance & Compliance
↓
Backup & Recovery
```

The structure is intended to allow different learning paths without requiring a fixed course order.

---

# AI Usage & Transparency

AI tools are used during the creation and maintenance of this knowledge base as **writing assistants**.

Their purpose is to help with tasks such as:

- organizing existing notes and ideas
- improving wording and readability
- summarizing information already collected by the author
- restructuring documentation
- correcting grammar and formatting
- expressing technical thoughts more clearly and consistently
    

AI is **not treated as a source of technical knowledge** for this repository.

The technical content is based on the author's own knowledge, professional experience, research, testing, vendor documentation, technical standards, and other appropriate source material.

AI-generated technical claims are not intentionally added to the knowledge base simply because an AI system produced them. AI assistance is primarily used to improve the presentation of information that has already been understood, researched, tested, or otherwise verified.

The author remains responsible for reviewing and validating the final content before it becomes part of the knowledge base.

---

# Content Status

This knowledge base is continuously developed.

Some areas may contain significantly more documentation than others.

Existing articles may also be expanded, corrected, reorganized, or updated as technologies change.

For that reason, keeping your local repository updated is recommended:

```
git pull
```

---

# Disclaimer

This repository is intended for educational and technical reference purposes.

Commands, configurations, security settings, and procedures should always be reviewed before being used in production environments.

Infrastructure differs between organizations, operating systems, software versions, and security requirements.

Always understand a command or configuration change before applying it.

---

# Quick Start

If you already have Git installed, the entire process is:

```
git clone https://github.com/MrGulczu/Knowledge-Base
cd Knowledge-Base
```

Open the directory in Obsidian.

Later, update it using:

```
git pull
```

That is all that is required to keep a local copy of the knowledge base.