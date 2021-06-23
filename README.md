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

## (Bonus) List all awesome-fzf functions
```bash
AWESOME_FZF_LOCATION="/path/to/awsome-fzf.zsh"

function fzf-awesome-list() {
if [[ -f $AWESOME_FZF_LOCATION ]]; then
    selected=$(grep -E "(function fzf-)(.*?)[^(]*" $AWESOME_FZF_LOCATION | sed -e "s/function fzf-//" | sed -e "s/() {//" | fzf --reverse --prompt="awesome fzf functions")
else
    echo "awesome fzf not found please try updating the path in awesome-fzf.zsh"
fi
    case "$selected" in
        "");; #don't throw an exit error when we dont select anything
        *) echo $selected;;
    esac
}
```

# Other Awesome FZF Plugins

[FZF Git Fuzzy](https://github.com/bigH/git-fuzzy)
