
## conception
* index
* work area
* HEAD
* branch point

work area > index > repo
git add
git commit

git checkout $(log name)
git checkout $(branch name)


## forward and backward

```
git reset HEAD@{1}
git reset HEAD^
git reset HEAD~1

```


## soft,mix and hard

soft:just move HEAD and branch point to the commit
mixed:change index but not commit
hard:change index and working copy(work area)

## revert
revert the changes that related patches introduce
