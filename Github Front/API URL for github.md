For your repo:

```text
https://github.com/jntnt0/Workbook
```

The GitHub API URL is:

```text
https://api.github.com/repos/jntnt0/Workbook
```

For repo contents, use:

```text
https://api.github.com/repos/jntnt0/Workbook/contents
```

For a specific folder, example `1. Networking`:

```text
https://api.github.com/repos/jntnt0/Workbook/contents/1.%20Networking
```

For `1. Networking/1. Command Reference`:

```text
https://api.github.com/repos/jntnt0/Workbook/contents/1.%20Networking/1.%20Command%20Reference
```

Spaces become:

```text
%20
```

### Where to find/build it

Take your normal GitHub repo URL:

```text
https://github.com/jntnt0/Workbook
```

Replace:

```text
github.com
```

with:

```text
api.github.com/repos
```

So it becomes:

```text
https://api.github.com/repos/jntnt0/Workbook
```

### For Git cloning, that is different

HTTPS clone URL:

```text
https://github.com/jntnt0/Workbook.git
```

SSH clone URL:

```text
git@github.com:jntnt0/Workbook.git
```

So use the **API URL** for apps/Shortcuts/scripts reading repo data, and use the **clone URL** for Git push/pull.