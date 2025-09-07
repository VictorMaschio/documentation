# Git Quick Guide
Git is a version control software that allows you to track changes in files, collaborate with others, and manage your codebase more efficiently. Below is a quick reference guide for some of the most common Git commands.

## Usefull Commands
### Clone Repository:
```bash
git clone <repository_url>
```
Copies a remote repository to your local machine.

### Status of Working Directory:
```Bash
git status
```
Shows the status of your working directory and staging area (tracked/unstaged files, branch, etc.).
### Commit Changes:
```bash
git commit -m "commit_message"
```
Records your changes locally with a message describing what was done.
### Push Changes:
```bash
git push
```
Pushes your local commits to the remote repository.
### Pull Latest Changes:
```bash
git pull
```
Fetches and merges changes from the remote repository to your local branch.
### Show Commit History:
```Bash
git log
```
Displays the commit history of the current branch.
### Rebase One Branch onto Another::
```Bash
git rebase <base_branch> <feature_branch>
```
Re-applies commits from one branch onto another, allowing for cleaner commit history. Use -i flag for interactive rebasing.
```Bash
git rebase -i <base_branch> <feature_branch>
```
### Reset Branch to Previous Commit:
```Bash
git reset <branch> HEAD~1
```
Resets the current branch to a previous commit. You can adjust the number (HEAD~1, HEAD~2, etc.) to move further back in history.
### Move Branch to a Specific Commit:
```Bash
git branch -f <branch_name> <commit_hash>
```
Forcefully moves a branch to a specific commit.
### Move HEAD to Commit:
```Bash
git checkout <commit_hash>
```
Moves your HEAD (current working state) to a specific commit.

### Revert Changes in a Commit:
```Bash
git revert <commit_hash>
```
Creates a new commit that undoes the changes made in a specific commit.
### Cherry-Pick Commits onto Another Branch:
```Bash
git cherry-pick <commit1> <commit2> ...
```
Applies specific commits from another branch onto the current branch.
### Create an Annotated Tag:
```Bash
git tag -a <tag_name> -m "tag_message"
```
Shows the status of your working directory and staging area (tracked/unstaged files, branch, etc.).

Displays all tags in the repository.
### Describe Current Commit:
```Bash
git describe
```
Returns the most recent tag, followed by the number of commits since that tag, and the current commit hash. For example: v1.0-2-g2414721. Add --long for a more detailed description.
```Bash
git describe --long
```
### List All Tags:
```Bash
git tag
```