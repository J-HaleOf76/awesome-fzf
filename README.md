# A *complete* list of fzf functions

# How to use these functions.

- Copy any of these functions to your zshrc or preferred location
- Either rename the function stub to your liking, or create an alias e.g
    `alias man="fzf-man"`

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

## Eval commands on the fly within fzf
```bash
function fzf-eval(){
echo | fzf -q "$*" --preview-window=up:99% --preview="eval {q}"
}
```

## File Finder (Open in $EDITOR)
```bash
function fzf-filefinder() {
  execute-fzf "" $EDITOR
}
```
