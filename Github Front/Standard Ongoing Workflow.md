Use this after editing notes in Obsidian
```
cd "C:\Users\mini\Documents\Obsidian Vault\Workbook"
git status
git pull --rebase origin main
git add .
git commit -m "Update workbook notes"
git push
```



Use this when you see:
`! [rejected] main -> main (fetch first)`

```
cd "C:\Users\mini\Documents\Obsidian Vault\Workbook"
git fetch origin
git status
git pull --rebase origin main
git push -u origin main
```

## Safety Rule

Do not run this unless you intentionally want to overwrite GitHub:
```
git push --force
```

For your resume portfolio repo, the normal workflow should be:

`status → pull --rebase → add → commit → push`