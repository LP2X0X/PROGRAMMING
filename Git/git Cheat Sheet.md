#### To clone with another user name and password, use:
```bash
git clone https://user.name:pass.word@git/repo/link
```

```ad-note
In case the error "Could not resolve host: https" appear, remove the https from the git repo link.
```

---

#### To exclude file from being track with git
1 - Add file path to `.git/info/exclude`
2 - git update-index --skip-worktree \<file-list>

--- 

#### To soft reset a commit (delete the commit but keep the local changes) use 
```bash
git reset --soft HEAD~
```
---

#### To create a patch
- If you haven't yet commited the changes, then:

```bash
git diff > mypatch.patch
```

- But sometimes it happens that part of the stuff you're doing are new files that are untracked and won't be in your `git diff` output. So, one way to do a patch is to stage everything for a new commit (`git add` each file, or just `git add .`) but don't do the commit, and then:

```bash
git diff --cached > mypatch.patch
```

- Add the 'binary' option if you want to add binary files to the patch (e.g. mp3 files):

```bash
git diff --cached --binary > mypatch.patch
```

- You can later apply the patch:

```bash
git apply mypatch.patch
```
