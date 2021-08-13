# A *semi-complete* list of fzf functions

*Please note, consider this repo pre-alpha right now.*
*There is work to be done to make sure these functions are portable and the syntax is concise and easy to understand. Also adding support for $FZF_DEFAULT_OPTS.*
*If you wish to contribute, pull requests are highly appreciated!*


[//]: # (NOTE FOR CONTRIBUTERS, you can make a gif of screen recording with below site)
[//]: # (https://ezgif.com/video-to-gif)
[//]: # (And then upload it here to be hosted permanently)
[//]: # (https://imgbb.com/)


# How to use these functions.

- Copy any of these functions to your zshrc or preferred location
- Either rename the function stub to your liking, or create an alias e.g
    `alias man="fzf-man"`
- Alternatively clone this repo or curl the awesome-fzf.\* file of your preference
- Source the file in your zshrc etc e.g. `source /path/to/awesome.fzf.zsh`
- All the commands will be available on your shell, similarly you can create aliases of your choosing


## Enhanced rm
```bash
function fzf-rm() {
  if [[ "$#" -eq 0 ]]; then
    local files
    files=$(find . -maxdepth 1 -type f | fzf --multi)
    echo $files | xargs -I '{}' rm {} #we use xargs so that filenames to capture filenames with spaces in them properly
  else
    command rm "$@"
  fi
}
```
![enhanced rm](https://i.ibb.co/n04XVWF/ezgif-1-9930c7cd8903.gif)


## Enhanced man
```bash
# Man without options will use fzf to select a page
function fzf-man(){
	MAN="/usr/bin/man"
	if [ -n "$1" ]; then
		$MAN "$@"
		return $?
	else
		$MAN -k . | fzf --reverse --preview="echo {1,2} | sed 's/ (/./' | sed -E 's/\)\s*$//' | xargs $MAN" | awk '{print $1 "." $2}' | tr -d '()' | xargs -r $MAN
		return $?
	fi
}
```


## Eval commands on the fly within fzf
```bash
function fzf-eval(){
echo | fzf -q "$*" --preview-window=up:99% --preview="eval {q}"
}
```

## Search list of your aliases and functions
```bash
function fzf-aliases-functions() {
    CMD=$(
        (
            (alias)
            (functions | grep "()" | cut -d ' ' -f1 | grep -v "^_" )
        ) | fzf | cut -d '=' -f1
    );

    eval $CMD
}
```


## File Finder (Open in $EDITOR)
```bash
function fzf-find-files(){
  local file=$(fzf --multi --reverse) #get file from fzf
  if [[ $file ]]; then
    for prog in $(echo $file); #open all the selected files
    do; $EDITOR $prog; done;
  else
    echo "cancelled fzf"
  fi
}
```


## Find Dirs
```bash
function fzf-cd() {
  local dir
  dir=$(find ${1:-.} -path '*/\.*' -prune \
                  -o -type d -print 2> /dev/null | fzf +m) &&
  cd "$dir"
  ls
}
```

## Find Dirs + Hidden
```bash
function fzf-cd-incl-hidden() {
  local dir
  dir=$(find ${1:-.} -type d 2> /dev/null | fzf +m) && cd "$dir"
  ls
}
```

## Change into the directory of the selected file
```bash
function fzf-cd-to-file() {
   local file
   local dir
   file=$(fzf +m -q "$1") && dir=$(dirname "$file") && cd "$dir"
   ls
}
```

## Change to selected parent directory
```bash
function fzf-cd-to-parent() {
  local declare dirs=()
  get_parent_dirs() {
    if [[ -d "${1}" ]]; then dirs+=("$1"); else return; fi
    if [[ "${1}" == '/' ]]; then
      for _dir in "${dirs[@]}"; do echo $_dir; done
    else
      get_parent_dirs $(dirname "$1")
    fi
  }
  local DIR=$(get_parent_dirs $(realpath "${1:-$PWD}") | fzf-tmux --tac)
  cd "$DIR"
  ls
}
```

## Search Environment Variables
```bash
function fzf-env-vars() {
  local out
  out=$(env | fzf)
  echo $(echo $out | cut -d= -f2)
}
```


## Kill process
```bash
function fzf-kill-processes() {
  local pid
  pid=$(ps -ef | sed 1d | fzf -m | awk '{print $2}')

  if [ "x$pid" != "x" ]
  then
    echo $pid | xargs kill -${1:-9}
  fi
}
```

## Enhanced Git Status (Open multiple files with tab + diff preview)
```bash
fzf-git-status() {
    git rev-parse --git-dir > /dev/null 2>&1 || { echo "You are not in a git repository" && return }
    local selected
    selected=$(git -c color.status=always status --short |
        fzf --height 50% "$@" --border -m --ansi --nth 2..,.. \
        --preview '(git diff --color=always -- {-1} | sed 1,4d; cat {-1}) | head -500' |
        cut -c4- | sed 's/.* -> //')
            if [[ $selected ]]; then
                for prog in $(echo $selected);
                do; $EDITOR $prog; done;
            fi
    }
```

## Checkout to existing branch or else create new branch
- Falls back to fuzzy branch selector list powered by fzf if no args.
- Replacement for both git checkout and git branch commands
- gco - brings up fzf list of existing branches to checkout to on selection
- gco <somenewbranch> - create and checkout to new branch
- gco <someexistingbranchremoteorlocal> - checkout to existing branch
```bash
fzf-checkout(){
    if git rev-parse --git-dir > /dev/null 2>&1; then
        if [[ "$#" -eq 0 ]]; then
            local branches branch
            branches=$(git branch -a) &&
            branch=$(echo "$branches" |
            fzf-tmux -d $(( 2 + $(wc -l <<< "$branches") )) +m) &&
            git checkout $(echo "$branch" | sed "s/.* //" | sed "s#remotes/[^/]*/##")
        elif [ `git rev-parse --verify --quiet $*` ] || \
             [ `git branch --remotes | grep  --extended-regexp "^[[:space:]]+origin/${*}$"` ]; then
            echo "Checking out to existing branch"
            git checkout "$*"
        else
            echo "Creating new branch"
            git checkout -b "$*"
        fi
    else
        echo "Can't check out or create branch. Not in a git repo"
    fi
}
```

## (Bonus) List all awesome-fzf functions
```bash
#List Awesome FZF Functions
function fzf-awesome-list() {
if [[ -f $AWESOME_FZF_LOCATION ]]; then
    selected=$(grep -E "(function fzf-)(.*?)[^(]*" $AWESOME_FZF_LOCATION | sed -e "s/function fzf-//" | sed -e "s/() {//" | grep -v "selected=" | fzf --reverse --prompt="awesome fzf functions > ")
else
    echo "awesome fzf not found"
fi
    case "$selected" in
        "");; #don't throw an exit error when we dont select anything
        *) "fzf-"$selected;;
    esac
}
```

# Other Awesome FZF Plugins

[FZF Git Fuzzy](https://github.com/bigH/git-fuzzy)
