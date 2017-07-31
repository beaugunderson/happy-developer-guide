This is a *work in progress*,  mostly written to share some tips with my
coworkers.

# Beau's guide to being a happy developer

In this guide being happy means:

- using as few keystrokes as possible
- having the right amount of context
- letting your computer remember things for you

## Preqrequisites

- macOS
- homebrew

## Using as few keystrokes as possible

### `bash`

For some of these tips we'll want to use the latest version of bash so let's
upgrade:

```sh
$ brew install bash
```

You'll need to add `/usr/local/bin/bash` to the bottom of `/etc/shells` and
then excecute the `chsh` command to change your login shell to
`/usr/local/bin/bash/`:

```sh
$ sudo vim /etc/shells
$ chsh
```

### `bash` basics

Commands like `git commit` will use the editor specified in your shell's
`$EDITOR` variable, so pick one that you're comfortable with:

```sh
export EDITOR=atom
export EDITOR=charm
export EDITOR=nano
export EDITOR=vim
```

#### Make bash remember everything you've done

```
export HISTFILE=~/.bash_eternal_history
export HISTFILESIZE=
export HISTIGNORE="&:[bf]g:exit:ls:history:clear"
export HISTSIZE=
export HISTTIMEFORMAT="[%F %T] "
export HISTCONTROL="erasedups:ignoreboth"
```

#### Ignore some files for auto-completion

```sh
export FIGNORE=$FIGNORE:.pyc:.o
```

#### Less typing with `ssh-agent`

```sh
export SSH_ENV=$HOME/.ssh/environment

function start_agent {
  echo "Initialising new SSH agent..."
  /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
  chmod 600 "${SSH_ENV}"
  . "${SSH_ENV}" > /dev/null
  /usr/bin/ssh-add ~/.ssh/id_rsa
  /usr/bin/ssh-add ~/.ssh/canvas_ed25519
}

# Source SSH settings, if applicable
if [ -f "${SSH_ENV}" ]; then
  . "${SSH_ENV}" > /dev/null
  ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
    start_agent;
  }
else
  start_agent;
fi
```

#### Turbo-charging bash

Let's enable some options in your `~/.bashrc`:

```sh
shopt -s checkhash
shopt -s checkwinsize
shopt -s autocd 2> /dev/null
shopt -s dirspell 2> /dev/null

# If set, minor errors in the spelling of a directory component in a cd
# command will be corrected. The errors checked for are transposed
# characters, a missing character, and one character too many. If a
# correction is found, the corrected file name is printed, and the command
# proceeds. This option is only used by interactive shells.
shopt -s cdspell 2> /dev/null

shopt -s cmdhist

# append to the history file, don't overwrite it
shopt -s histappend
shopt -s no_empty_cmd_completion
shopt -s globstar 2> /dev/null

export FZF_DEFAULT_OPTS="--extended --select-1 --sort=2500 --inline-info"

[ -f ~/.fzf.bash ] && source ~/.fzf.bash

eval "$(npm completion -)"

[ -f /usr/local/bin/virtualenvwrapper_lazy.sh ] && \
  source /usr/local/bin/virtualenvwrapper_lazy.sh
```

#### Further turbo-charging bash via `~/.inputrc`

```sh
$if Bash
  #do history expansion when space entered
  Space: magic-space
$endif

Control-j: menu-complete
Control-k: menu-complete-backward

"\M-s": menu-complete

# Find these sequences using `read`
"\e[B": history-search-forward
"\e[A": history-search-backward

# Ubuntu via PuTTY
"\eOD": vi-prev-word
"\eOC": vi-next-word

# Local iTerm 2 on OS X
"\e[1;9D": vi-prev-word
"\e[1;9C": vi-next-word

#set keymap vi
#set editing-mode vi

set convert-meta off
set input-meta on
set output-meta on

set colored-completion-prefix on
set colored-stats on

set completion-ignore-case on
set completion-map-case on
set completion-prefix-display-length 2
set completion-query-items 9001

set expand-tilde on

set mark-directories on
set mark-symlinked-directories on

set page-completions off

set show-all-if-ambiguous on
set show-all-if-unmodified on

set visible-stats on

# Include system wide settings which are ignored
# by default if one has their own .inputrc
$include /etc/inputrc
```

#### Aliasing commands

```sh
alias outdated="pip list -l -o --format=columns"

alias git=hub

alias grep="grep --color=auto"
alias fgrep="fgrep --color=auto"
alias egrep="egrep --color=auto"

alias tree="tree -C -I node_modules -I \*.pyc"
alias treed="tree -C -I node_modules -I \*.pyc --du --si --sort=size --dirsfirst"

alias dc=docker-compose
alias dr="docker-compose run --service-ports --rm web"
alias drn="docker-compose run --rm web"
alias drm="docker-compose run --rm web ./manage.py"
alias drms="docker-compose run --service-ports --rm web ./manage.py"

if [ -z `which gls 2> /dev/null` ] && shell_is_osx
then
  LS_OPTIONS="-G"
else
  LS_OPTIONS='--group-directories-first --color=auto --time-style="+%D %r" --quoting-style=shell'
fi

[ ! -z `which gls 2> /dev/null` ] && LS=gls

alias l="$LS -hF $LS_OPTIONS"
alias ls="$LS -hF $LS_OPTIONS"
alias l1="$LS -hF1 $LS_OPTIONS"
alias ll="$LS -hlF $LS_OPTIONS"
alias lrt="$LS -hlFrt $LS_OPTIONS"
alias lx="$LS -hlFX $LS_OPTIONS"
alias lll="$LS -ahlF $LS_OPTIONS"

alias gs="git status"
alias gs.="git status ."
alias gd="git diffr"
alias gda="git diff"
alias gp="git push"

alias df="df -h"

alias ..="cd .."
alias ...="cd ../.."

alias cd..="cd .."
alias cd...="cd ../.."

alias dc='popd'

function cd() {
  if [ $# -eq 0 ]; then
    DIR="${HOME}"
  else
    DIR="$1"
  fi

  builtin pushd "${DIR}" > /dev/null
}

shell_is_osx && \
  [ -f `brew --prefix`/etc/bash_completion ] && \
    source `brew --prefix`/etc/bash_completion

alias work-ontologies="cd ~/p/canvas/ontologies && source $WORKON_HOME/ontologies/bin/activate"
alias work-pcp="cd ~/p/pcp && source $WORKON_HOME/pcp/bin/activate"
alias work-pharmacy="cd ~/p/canvas/pharmacy && source $WORKON_HOME/pharmacy/bin/activate"
alias work-science="cd ~/p/canvas/science && source $WORKON_HOME/science/bin/activate"
alias work-web="cd ~/p/canvas-web && source $WORKON_HOME/canvas-web/bin/activate"
```

#### Increasing context with your bash prompt

```sh
# Create Prompt
if shell_is_linux
then
  BOX_COLOR="\[\e[0;36m\]"
else
  BOX_COLOR="\[\e[0;32m\]"
fi

# WHITE="\[\e[1;37m\]"
RED="\[\e[1;31m\]"
GRAY="\[\e[38;05;239m\]"
LIGHT_GRAY="\[\e[38;05;235m\]"
MEDIUM_GRAY="\[\e[38;05;244m\]"
GREEN="\[\e[38;05;34m\]"
YELLOW="\[\e[38;05;172m\]"
NOTHING="\[\e[0m\]"

# If I use literal Unicode characters then it wedges bash on OS X
BOX=$(echo -e "\xE2\x97\xBC")
SLASH="/"

if (($SHLVL > 1))
then
  BOX_COLOR=$MEDIUM_GRAY
fi

function git_branch {
  command git branch --no-color 2> /dev/null | \
    sed -e '/^[^*]/d' -e 's/* \(.*\)/\1/'
}

function git_clean {
  command git diff --quiet HEAD 2> /dev/null
}

function git_status {
  GIT_BRANCH=$(git_branch)
  COLOR=$RED

  if [[ $GIT_BRANCH ]]
  then
    git_clean && COLOR=$GREEN
    GIT_BRANCH=" $COLOR$GIT_BRANCH"
  fi

  echo -e "$GIT_BRANCH"
}

function virtualenv_status {
  if [[ $VIRTUAL_ENV ]]
  then
    echo -e " $YELLOW$(basename $VIRTUAL_ENV)"
  fi
}

function _prompt_command {
  # Save history after each command
  history -a

  # Set window title for time-tracking purposes
  # https://timingapp.com/help/terminal
  echo -ne "\033]0;${USER}@${HOSTNAME%%.*}:${PWD/#$HOME/~}\007"

  PS1="\[$(iterm2_prompt_mark)\]"
  PS1="$PS1 $BOX_COLOR$BOX$GRAY \h$LIGHT_GRAY$SLASH$GRAY\u$(virtualenv_status)"
  PS1="$PS1 $MEDIUM_GRAY\w$(git_status)$NOTHING "
}

PROMPT_COMMAND=_prompt_command
```

#### `tag`: never copy and paste another search result

```sh
if hash ag 2>/dev/null; then
  tag() {
    command tag "$@"; source ${TAG_ALIAS_FILE:-/tmp/tag_aliases} 2>/dev/null
  }

  alias ag=tag
fi

alias agpy='ag -G ".py$"'
```

#### Adding some color

```sh
man() {
  env \
    LESS_TERMCAP_md=$'\e[1;36m' \
    LESS_TERMCAP_me=$'\e[0m' \
    LESS_TERMCAP_se=$'\e[0m' \
    LESS_TERMCAP_so=$'\e[1;41m' \
    LESS_TERMCAP_ue=$'\e[0m' \
    LESS_TERMCAP_us=$'\e[1;32m' \
    man "$@"
}
```

```sh
if echo hello|grep --color=auto l >/dev/null 2>&1; then
  export GREP_COLOR="1;32"
fi
```

```sh
# Fancy LS_COLORS
DIRCOLORS=dircolors

[ ! -z `which gdircolors 2> /dev/null` ] && DIRCOLORS=gdircolors

eval $($DIRCOLORS -b $HOME/.dircolors)
```

### git

```
[user]
	name = Beau Gunderson
	email = beau@beaugunderson.com
	username = beaugunderson
[core]
	whitespace = trailing-space,space-before-tab,tab-in-indent,tabwidth=2
	excludesfile = ~/.gitignore
	untrackedCache = true
	disambiguate = commit
[color "branch"]
	current = 214 reverse
	local = 214
	remote = 10
[color "diff"]
	commit = 227 bold
	frag = magenta bold
	meta = 227
	new = green bold
	old = red bold
	whitespace = red reverse
[color "diff-highlight"]
	newHighlight = green bold 22
	newNormal = green bold
	oldHighlight = red bold 52
	oldNormal = red bold
[color "status"]
	added = 214
	changed = 10
	untracked = 196
[color]
	branch = auto
	diff = auto
	grep = auto
	interactive = auto
	showbranch = auto
	status = auto
	ui = auto
[diff]
	renames = copies
	noprefix = true
	indentHeuristic = true
	# wordRegex
[grep]
	lineNumber = true
	extendedRegexp = true
[alias]
	diff = diff -w
	diffr = diff --relative -w
	# Show files ignored by git:
	ignored = ls-files -o -i --exclude-standard
	lg = log --graph --decorate --all --pretty=format:'%Cred%h%Creset%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
	ls = ls-files
	# git reauthor $START..$END
	reauthor = !sh -c 'eval `git log --reverse --topo-order --pretty=format:\"git cherry-pick %H && git commit --amend -C %H --author=\\\"%aN <%aE>\\\" && \" $0 `"echo success"'
	update-submodules = submodule foreach git pull --no-rebase origin master
	wdiff = diff --color-words
[pull]
	rebase = true
[rebase]
	stat = true
[mergetool "splice"]
	cmd = "nvim -f $BASE $LOCAL $REMOTE $MERGED -c 'SpliceInit'"
	trustExitCode = true
[mergetool "vimdiff3"]
	cmd = nvim -f -d \"$LOCAL\" \"$MERGED\" \"$REMOTE\"
[mergetool "fugitive"]
	cmd = nvim -f -c \"Gdiff\" \"$MERGED\"
[merge]
	tool = fugitive
	conflictstyle = merge
[rerere]
	enable = 1
[url "git@github.com:"]
	insteadOf = git://github.com/
	pushInsteadOf = git://github.com/
	insteadOf = https://beaugunderson@github.com/
	pushInsteadOf = https://beaugunderson@github.com/
	insteadOf = https://github.com/
	pushInsteadOf = https://github.com/
[push]
	default = current
[credential]
	helper = cache
[pager]
	diff = diff-so-fancy | less --tabs=4 -RFX
	show = diff-so-fancy | less --tabs=4 -RFX
```

### `hub`

### `pbcopy` and `pbpaste`

### psql

https://www.citusdata.com/blog/2017/07/16/customizing-my-postgres-shell-using-psqlrc/

### Django

### Vim

Knowing just a tiny bit of `vim` will help you when you're in an environment
where it's the only editor (like when `vi` is available by default inside an
Alpine Linux docker image).

Vim is a modal editor, and the two modes you'll need to know are *normal* and
*insert*. Vim starts out in *normal* mode. To get to *insert* mode, where you
can write and delete text like a normal editor, you'll use one of these
commands:

- <kbd>i</kbd> enter insert mode before the cursor
- <kbd>a</kbd> enter insert mode after the cursor
- <kbd>I</kbd> enter insert mode at the start of the line
- <kbd>A</kbd> enter insert mode at the end of the line

To leave *insert* mode you'll hit the <kbd>esc</kbd> key.

These are the most useful *normal* mode commands for moving around or changing
a file:

- <kbd>^</kbd> move to the start of the line
- <kbd>$</kbd> move to the end of the line
- <kbd>w</kbd> move forward one word
- <kbd>b</kbd> move backward one word
- <kbd>x</kbd> delete the character under the cursor
- <kbd>d</kbd><kbd>d</kbd> delete the current line

And of course you'll want to be able to save and quit:

- <kbd>:</kbd><kbd>w</kbd> save
- <kbd>:</kbd><kbd>q</kbd> quit
- <kbd>:</kbd><kbd>q</kbd><kbd>!</kbd> quit and ignore changes
- <kbd>:</kbd><kbd>w</kbd><kbd>q</kbd> save and quit

## Monorepo-specific tips
