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
  selected="$(fzf --multi --reverse)"
  case "$selected" in
    "") echo "cancelled fzf";;
    *) eval "$EDITOR" "$selected";;
  esac
}
```


# Other Awesome FZF Plugins

[FZF Git Fuzzy](https://github.com/bigH/git-fuzzy)
