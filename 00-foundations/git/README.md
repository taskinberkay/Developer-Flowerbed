# Git cheat sheet

The commands I actually use, plus the "how do I undo that" section I always end up searching for.

## The mental model

Three places your changes live:

```
working directory  --git add-->  staging area  --git commit-->  local repo  --git push-->  remote
```

- A **commit** is a snapshot, not a diff. It points to its parent.
- A **branch** is just a movable label pointing at a commit.
- `HEAD` is "where I am right now" (usually the tip of the current branch).

## Setup (once per machine)

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false      # or true, if I prefer a linear history
git config --global core.autocrlf true     # Windows: avoids the LF/CRLF warnings
```

## Daily loop

```bash
git status                  # what changed?
git diff                    # unstaged changes
git diff --staged           # what I'm about to commit
git add path/to/file        # stage a file (git add -p to pick hunks)
git commit -m "feat: add binary search"
git push
git pull                    # fetch + merge
```

## Branching

```bash
git switch -c feature/stack      # create and switch
git switch main                  # switch back
git branch                       # list local branches
git branch -d feature/stack      # delete (safe, only if merged)
git merge feature/stack          # merge into current branch
```

**Merge vs rebase**

| | merge | rebase |
|---|---|---|
| History | keeps the real shape, adds a merge commit | rewrites commits into a straight line |
| Safe on shared branches | yes | **no**, never rebase what others already pulled |
| Good for | integrating finished work | tidying my own local branch before opening a PR |

```bash
git switch feature/stack
git rebase main              # replay my commits on top of main
```

Conflict during merge/rebase: fix the files, `git add` them, then `git merge --continue` or `git rebase --continue`. Bail out with `--abort`.

## Looking around

```bash
git log --oneline --graph --decorate --all
git log -p path/to/file          # history of one file with diffs
git blame path/to/file           # who changed each line
git show <commit>                # one commit in detail
git reflog                       # every place HEAD has been (the safety net)
```

## Undoing things

| I want to... | Command |
|---|---|
| Discard changes in a file (not staged) | `git restore file` |
| Unstage a file, keep the changes | `git restore --staged file` |
| Fix the last commit message | `git commit --amend` (only if not pushed) |
| Add a forgotten file to the last commit | `git add file && git commit --amend --no-edit` |
| Undo a commit, keep the changes staged | `git reset --soft HEAD~1` |
| Undo a commit, keep the changes unstaged | `git reset HEAD~1` |
| Undo a commit and throw the changes away | `git reset --hard HEAD~1` ⚠️ |
| Undo a commit that's already pushed | `git revert <commit>` (adds a new "undo" commit) |
| Get back something I "lost" | `git reflog`, then `git reset --hard <hash>` or `git switch -c rescue <hash>` |

Rule of thumb: **`reset` rewrites history, `revert` adds to it.** If it's pushed and shared, use `revert`.

## Stash (park work temporarily)

```bash
git stash push -m "wip: graph traversal"
git stash list
git stash pop            # apply and drop
git stash apply          # apply and keep
```

## Remotes

```bash
git remote -v
git remote add origin <url>
git push -u origin main          # first push, sets upstream
git fetch                        # download, don't merge
git push --force-with-lease      # safer than --force
```

## Commit messages

Format: `type: short summary` (imperative, under ~70 chars).

`feat` new thing · `fix` bug · `docs` docs only · `test` tests · `refactor` no behaviour change · `chore` tooling/housekeeping

Small commits that do one thing. Future me reading `git log` should understand the story.

## `.gitignore`

- Create it **before** the first commit. Ignoring a file that's already tracked doesn't work until I run `git rm --cached file`.
- Never commit secrets. If I do, rotating the key matters more than rewriting history.
- Check what's ignored and why: `git check-ignore -v path`

## GitHub workflow

1. Branch off `main`: `git switch -c feat/thing`
2. Commit small and often
3. Push the branch, open a pull request
4. Review my own diff first (the "Files changed" tab catches a lot)
5. Merge (squash for messy branches), delete the branch
6. `git switch main && git pull`

## Gotchas

- **Detached HEAD:** I checked out a commit instead of a branch. Make a branch (`git switch -c name`) before committing or the work gets orphaned.
- **"Updates were rejected":** remote has commits I don't. `git pull` first (or `--rebase`), then push.
- **Empty folders aren't tracked:** that's why there are `.gitkeep` files around.
- **Case-only renames on Windows** can confuse Git: use `git mv`.
- **Line endings** (LF vs CRLF) cause "everything changed" diffs. Set `core.autocrlf` once and stop thinking about it.

## Further reading

- Pro Git book (free): https://git-scm.com/book
- Oh Shit, Git!?!: https://ohshitgit.com
- Visual practice: https://learngitbranching.js.org
