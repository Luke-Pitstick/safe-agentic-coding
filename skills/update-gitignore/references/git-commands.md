# Git Command Recipes

Run from the repository root unless noted.

## Repo and Inventory

```sh
git rev-parse --show-toplevel
git status --short --ignored
git ls-files
git ls-files --others --exclude-standard
git config --get core.excludesFile
```

Use `rg --files -uu` to inspect ignored files when needed. For very large repos, pair it with targeted path or extension filters.

## Check Ignore Rules

```sh
git check-ignore -v -- path/to/file
git ls-files -ci --exclude-standard
```

`git ls-files -ci --exclude-standard` lists tracked files that now match ignore rules. These are candidates for untracking.

## Untrack Without Deleting Local Files

```sh
git rm --cached -- path/to/file
git rm -r --cached -- path/to/directory
```

After untracking, verify the files still exist locally when important:

```sh
test -e path/to/file
test -d path/to/directory
```

## Check Whether Files Were Committed or Pushed

```sh
git log --oneline -- path/to/file
git rev-parse --abbrev-ref --symbolic-full-name @{u}
git log @{u} --oneline -- path/to/file
```

If `@{u}` is missing, say no upstream branch is configured rather than guessing whether the file was pushed.

For sensitive files, "committed" or "pushed" means the secret may still be recoverable from history. Recommend credential rotation and an explicit history-cleanup task.

## Validation

```sh
git diff -- .gitignore
git status --short --ignored
git ls-files -ci --exclude-standard
```

The final command should usually produce no output. If it still lists files, explain why each remaining path was left tracked.
