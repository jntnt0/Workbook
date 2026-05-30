Yes. The clean answer is: use **autostash**, then commit and push. `git pull --rebase` fetches remote changes and rebases your local work on top of them, and `--autostash` temporarily stashes your dirty working tree before the rebase and reapplies it afterward. Git stash with `-u` includes untracked files, which mattered in your case because you had new folders/files. ([Git](https://git-scm.com/docs/git-pull/2.17.0?utm_source=chatgpt.com "Git - git-pull Documentation"))

Use this as your normal “sync before push” block:

```powershell
cd "C:\Users\mini\Documents\Obsidian Vault\Workbook"

git status

git pull --rebase --autostash origin main

git status

git add .

git commit -m "Update workbook notes"

git push origin main
```

The one catch: `--autostash` handles modified tracked files well, but I would still use the fuller “belt and suspenders” version when you have **new folders or new files** sitting around:

```powershell
cd "C:\Users\mini\Documents\Obsidian Vault\Workbook"

git status

git stash push -u -m "pre-sync local workbook changes"

git pull --rebase origin main

git stash apply 'stash@{0}'

git status

git add .

git commit -m "Update workbook notes"

git push origin main
```

After you verify GitHub looks right:

```powershell
git stash drop 'stash@{0}'
```

For your actual workflow, I would save this as the safest repeatable version:

```powershell
cd "C:\Users\mini\Documents\Obsidian Vault\Workbook"

# 1. See what local changes exist
git status

# 2. Save all local work, including new untracked files/folders
git stash push -u -m "pre-sync local workbook changes"

# 3. Pull GitHub web changes into local repo
git pull --rebase origin main

# 4. Restore your local work on top of the updated repo
git stash apply 'stash@{0}'

# 5. Review what came back
git status

# 6. Stage, commit, and push everything
git add .
git commit -m "Update workbook notes"
git push origin main

# 7. Only after confirming GitHub looks correct, remove the backup stash
git stash drop 'stash@{0}'
```

Why this works: GitHub rejects non-fast-forward pushes when the remote has commits your local branch does not have, because accepting the push could lose commits. Pulling/rebasing first is the correct fix. ([GitHub Docs](https://docs.github.com/en/get-started/using-git/dealing-with-non-fast-forward-errors?utm_source=chatgpt.com "Dealing with non-fast-forward errors"))

One more improvement so you do not get dragged through that mess as often:

```powershell
git config --global pull.rebase true
git config --global rebase.autoStash true
```

Then your future pull can usually be just:

```powershell
git pull origin main
```

But I would still use the explicit stash block when you added new Obsidian folders/files and are not 100% sure what is tracked yet.