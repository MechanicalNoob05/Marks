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

2) Neo-vim Setup