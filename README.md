# How to snippets

A collection of snippets on how to do various computer-y things.

## Snippets

### Git

#### Pushed 2 branches, the 1st one is merged, and the 2nd one has the same changes as the first branch, plus some additional changes

- To avoid this scenario to begin with:
  You need to pull/ fetch the master branch,
  before adding commits to your 2nd branch.
- If you have already done this:
  Fix by doing a pull/ fetch from the master branch,
  and then merging that into your new branch:
  ```shell
  git checkout my-2nd-branch
  git fetch origin master:master
  git merge --no-ff master
  # now my-2nd-branch contains the commits from my-1st-branch
  # that have already been merged to master
  git push origin my-2nd-branch
  ```

## Licence

GPL-3.0

## Author

[Brendan Graetz](http://bguiz.com/)
