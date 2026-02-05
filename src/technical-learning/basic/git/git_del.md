# delete remote branch

```

git for-each-ref --format='%(committerdate:short) %(refname:short)' --sort=committerdate refs/remotes | awk '{ if ($1 <= "'$(date -v-3m +%Y-%m-%d)'") { print $2  }}' | awk -F'/' '{if ($NF != "HEAD" && $NF != "master") print $NF}' |  xargs -I{} git push --delete origin {}
```

delete remote branchs which commit large than 3 monthes by now!
