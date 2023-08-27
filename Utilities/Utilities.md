1) Git save function for bash

```
save(){
  git status
  git add .
  git commit -m "$1"
  if [ $2 == 1 ]; then
    git push
    echo "Commited and Pushed"
  else
    echo "Commited"
  fi
}
```

2) Fuzzy find directories
```bash
bind '"\C-f":"cd_with_fzf\n"'

cd_with_fzf(){
  cd $HOME && cd "$(fd -t d | fzf --preview="tree -L {}" --bind="space:toggle-preview" --preview-window=:hidden)" && echo "$PWD" && tree -L 2
}
```

3) 