# Become a VScodeR
## Zellerlab Group meeting – 25-05-20

### Motivation
A very common everyday workflow...
- **Receive dataset** with raw data and metadata; process raw data (mostly on the cluster via Bash/Nextflow/Python)
- **Process metadata** (using R/Python)
- **Import processed data** (in R or Python), run analysis, and create data visualizations
- **Export results** (to PDFs, Markdown, etc.)

### Pre-VSCode era
What I used before:
- **R**: RStudio (local) & RStudio Server (cluster) (>80% of my time)
- **Python/Conda**: PyCharm (local), Jupyter (browser), JupyterHub (cluster) (~5%)
- **Bash/Nextflow**: Terminal (on the cluster) or simple editors (Sublime Text, Atom) (~15%)

### Since I switched to VSCode:
**One IDE – One setup – For both local AND remote workflows.**

---

## About VS Code
Visual Studio Code (VS Code) as a fast, lightweight editor that you can turn into a full-blown **IDE for R** (and **dozens of other languages**) simply by installing extensions
- **Extensive extension gallery** for languages, linters, debuggers, and theming
- **Built-in SSH** for seamless server/cluster access
- **Integrated terminal** that respects Conda environments (both Python & R)
- **Git integration** and **GitHub Copilot** support ;)
- All **settings** managed via **txt** files, making setup easy to share, change, and rebuild

---

## 1. Setup VSCode for R 
Disclaimer: Everything below is based on my personal experience and preferences. 
I think with this setup you should be able to use R in VSCode after a few minutes. However I'd not expect to be producive from day 1. For me it took a few days of getting used to the new environment, fix bugs and google weird error messages. However: Once you overcome the initial hurdles, you will love the experience!
### 1.1 Setup a conda environment for R
1) (Install Miniforge/Miniconda/Anaconda)
2) For EMBL users: 
Remove defaults channel from `condarc` and keep only `conda-forge` and `bioconda`
modify:
```
# ~/.condarc
channels:
  - conda-forge
  - bioconda
```
Don't update VSCode >1.85 due to incopatibilities with `amur` and `nile` (Disable autoupdate)

3) Create and conda environment for R and activate: 
`conda create -n r_env_4.3.3` 
`conda activate r_env_4.3.3`
[Suggestion for ARM Mac users in their local installation: force environment to consider only osx-86 packages: ``conda config --env --set subdir osx-64`` (prevents conflicts in my personal experience and most packages are available for osx-64)]
4) Download R: `mamba install -c conda-forge r-base=4.3.3`
5) Pin R version (to prevent changes when updating packages etc) `echo "r_base=4.3.3" > <PathToCondaEnv>/conda-meta/pinned`
6) Install necessary packages:
```
mamba install -c conda-forge \
r-languageserver \ # support R language
radian \ # better R console
r-httpgd # to forward/display plots
```
- In general: Manage your R libraries all via conda. If you want to install a new package, go for the conda version and then install it into the conda environment. 
E.g. `mamba install -c conda-forge r-tidyverse r-devtools`
Some R packages might not be available in conda. Within the R session one can also install packages directly from GitHub or other sources: 
`devtools::install_git('https://git.embl.de/grp-zeller/ggembl.git')`

### 1.2 Modify `~/.Rprofile`

```
# ~/.Rprofile
if (interactive() && Sys.getenv("TERM_PROGRAM") == "vscode") {
  if ("httpgd" %in% .packages(all.available = TRUE)) {
    options(vsc.plot = FALSE)
    options(device = function(...) {
      httpgd::hgd(silent = TRUE)
      .vsc.browser(httpgd::hgd_url(), viewer = "Beside")
    })
  }
}

if (interactive() && Sys.getenv("RSTUDIO") == "") {
  source(file.path(Sys.getenv(if (.Platform$OS.type == "windows") "USERPROFILE" else "HOME"), ".vscode-R", "init.R"))
}
```

### 1.2 In VSCode
1) Install the R excension from the extensions marketplace
2) Other suggested extensions: `Remote-SSH`,`Path Autocomplete`,`Better Comments`, `Color Highlight`, `Git Graph`, `Github Copilot`
3) Set paths correctly in `settings.json` of VSCode
```
# settings.json
    "r.rterm.mac": "/usr/local/Caskroom/miniconda/base/envs/r_env_x86_4.3.1/bin/radian",
    "r.rpath.mac": "/usr/local/Caskroom/miniconda/base/envs/r_env_x86_4.3.1/bin/R",
    "r.rpath.linux": "/g/scb/<PathToYourCondaInstallation>/miniforge3/envs/r_env_4.3.3/bin/R",
    "r.rterm.linux": "/g/scb/<PathToYourCondaInstallation>/miniforge3/envs/r_env_4.3.3/bin/radian", 
    "r.bracketedPaste": true,
    "r.alwaysUseActiveTerminal": true,
    "r.sessionWatcher": true,
    "r.lsp.debug": true,
    "r.lsp.diagnostics": true,
    "r.plot.useHttpgd": true,
    "[rmd]": {
        "editor.defaultFormatter": "REditorSupport.r",
        "editor.formatOnSave": true
      },    
    "r.rmarkdown.knit.useBackgroundProcess": false,
```
4) Add some convenient shortcuts equivalent to RStudio:
Open `keybindings.json` and add:
<details>
<summary>Click to unfold</summary>

```
# add to `keybindings.json`
{
    "key": "Cmd+Shift+m",
    "command": "type",
    "args": { "text": " %>% " },
    "when": "editorTextFocus && editorLangId == r"
  },
  {
    "key": "Alt+-",
    "command": "type",
    "args": { "text": " <- " },
    "when": "editorTextFocus && editorLangId == r"
  },
  // keybindings for Rmarkdown
  {
    "key": "Cmd+Shift+m",
    "command": "type",
    "args": { "text": " %>% " },
    "when": "editorTextFocus && editorLangId == rmd"
  },
  {
    "key": "Alt+-",
    "command": "type",
    "args": { "text": " <- " },
    "when": "editorTextFocus && editorLangId == rmd"
  },
  // keybindings for R terminal (radian included)
  {
    "key": "Ctrl+Shift+m",
    "command": "workbench.action.terminal.sendSequence",
    "args": { "text": " %>% " },
    "when": "terminalFocus"
  },
  {
    "key": "Alt+-",
    "command": "workbench.action.terminal.sendSequence",
    "args": { "text": " <- " },
    "when": "terminalFocus"
  },

  //Rstudio convenience features (commenting blocks)
  {
    "key": "shift+cmd+c",
    "command": "editor.action.commentLine",
    "when": "editorTextFocus && !editorReadonly"
  },
  {
    "key": "cmd+i",
    "command": "editor.action.formatSelection",
    "when": "editorHasDocumentSelectionFormattingProvider && editorTextFocus && !editorReadonly"
  },
  {
    "key": "ctrl+alt+left", 
    "command": "workbench.action.previousEditor"
  },
  {
    "key": "ctrl+alt+right", 
    "command": "workbench.action.nextEditor"
  },
  //Use CMD Shift R to insert comment sections recognized by RStudio
  {
    "key": "cmd+shift+r",
    "command": "editor.action.insertSnippet",
    "when": "editorLangId == r && editorTextFocus",
    "args": {
        "snippet": "#* ${TM_SELECTED_TEXT}$0 ----"
    }
  }
```
asd
</details>


### Now you're all set!

## Useful links
[How to not get lost in VSCode-R](https://statnmap.com/2021-10-09-how-not-to-be-lost-with-vscode-when-coming-from-rstudio/)

[RSessionWatcher](https://github.com/REditorSupport/vscode-R/wiki/R-Session-watcher#advanced-usage-for-self-managed-r-sessions)

[Suggested Settings](https://renkun.me/2020/04/14/writing-r-in-vscode-working-with-multiple-r-sessions/)

## 2 Work with R in VSCode 
### 2.1 Local R session
[Demo 1]
1) Open VSCode.
2) Open the search bar with `cmd + shift + p` and type "Open new terminal".
`cmd + shift + p` is one of the most important shortcuts in VSCode. 
It opens the search bar (on top) and lets you control almost the entire program with the keybord:
Open the settings file: `cmd + shift + p` -> search for "settings"
Show/Hide sidebar: `cmd + b`.
3) Activate you conda environment with the R packages: `conda activate r_env_4.3.3`.
4) Start R (radian) by typing `radian` into the terminal.
5) Open/Create an R Script and start coding/executing the code.

### 2.2 SSH on remote server
[Demo 2]
1) On bottom left click on the two arrows.
2) Select "connect to host".
3) Select "Add New SSH Host" and enter credentials e.g. `yourID@login1.cluster.embl.de`, `yourID@nile.embl.de`.
4) Enter password or select file with ssh-key pair.
5) Done - You are connected! 
6) Use **tmux** to generate R sessions that stay active even when you disconnect:
Create new tmux session `tmux new -s zellerGM`; detach with `cmd+b +d`, reconnect with `tmux attach -t zellerGM`.
7) Within the tmux session, activate your conda environment and start R by launching `radian` just as before.

**Interactive Cluster Jobs** with R:
1) Connect to cluster `ssh emblID@login1.cluster.embl.de`
2) Open Project folder and terminal
3) Start `tmux` session
4) Request an interactive slurm job
  `srun -p bigmem -c 4 --mem-per-cpu=4G --pty -J "resssion" -t 24:00:00 /bin/bash`.
5) Wait until resources were allocated and you are connected.
6) Activate the R conda environment, launch radian, start working.

### 2.3 GitHub Copilot: :rocket:
[Demo 3]
1) Register on Github.com to GitHub Pro (free for acedemics/students)
2) Get Copilot extension for VSCode and login with GitHub credentials
3) (A privacy note: On Github.com/settings/copilot/features uncheck "Allow GitHub to use my data for product improvements")
4) Get inline code suggestions with Tab-autocomplete or the chat feature. Set instrucitons via prompting directly inside the script in the comments.
5) :exclamation: **ALWAYS double check! Never trust blindly!** :exclamation:
Outsourse boring tasks (generation of color codes, cosmetic changes in plots, code completion for loop structures etc.) and never accept code that you don't understand!