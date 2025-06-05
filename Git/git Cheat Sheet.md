#### To do a soft reset (keep the changes but delete the commit on local)
```bash
git reset --soft head~1
```

---

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

---

#### To use submodule

Here’s how to do it cleanly:

##### ✅ **Create or prepare the submodule repo**

```bash
cd /tmp/path/to/submodule/folder
git init
git add .
git commit -m "Initial commit for submodule"
git remote add origin https://github.com/you/thing.git
git push -u origin master
```


##### ✅ **Add the submodule into your main repo**

Go back to the root of your main repo:

```bash
cd /path/to/main-repo
git submodule add https://github.com/you/thing.git submodule/name
```

This will:
- Clone the repo into `submodule/name`
- Create a `.gitmodules` file or update it
- Track the commit of the submodule


##### ✅  **Commit the submodule addition**

```bash
git add .gitmodules submodule/name
git commit -m "Add submodule"
```

---

#### To revert multiple commits

To **revert multiple commits** in Git, you have several options depending on what exactly you want:


##### ✅ **Option 1: Revert multiple commits without rewriting history**

This is safe and preserves history (ideal for shared branches).

```bash
git revert <oldest_commit_hash>^..<newest_commit_hash>
```

🔸 This will create **new commits** that undo the effects of each commit in the range.

**Example:**

```bash
git revert abc123^..def456
```

This reverts all commits from `abc123` to `def456` (inclusive).

> 💡 The `^` includes the starting commit in the range.

##### ✅ **Option 2: Hard reset to an earlier commit (rewrites history)**

This will **delete commits** after a certain point — dangerous on shared branches!

```bash
git reset --hard <commit_hash>
```

**Example:**

```bash
git reset --hard abc123
```

> ⚠️ This removes all commits after `abc123` and resets your working directory.


##### ✅ **Option 3: Interactive rebase (edit, drop, squash commits)**

Great for editing history before pushing.

```bash
git rebase -i <commit_hash>^
```

Then in the editor:

- Replace `pick` with `drop` for commits you want to remove
- Or `edit`, `squash`, etc., as needed
 
##### ✅ **Option 4: Revert specific commits by hash**

```bash
git revert <commit_hash1> <commit_hash2> <commit_hash3>
```

Each hash you specify will be reverted (undone) in the order you list them.
Git will create one new revert commit per original commit, unless you use the --no-commit flag.

---

#### To use auto as a sign to auto prepare commit

Add this content to .git\hook folder as prepare-commit-msg file:

```bash
#!/bin/sh

MSG_FILE="$1"

# Read the first line of the commit message and trim whitespace
MSG=$(head -n 1 "$MSG_FILE" | tr -d '\r' | awk '{$1=$1; print}')

# Check if the message is exactly "auto" (case-insensitive)
if printf "%s\n" "$MSG" | grep -iqx "auto"; then
  DATE=$(date "+%Y-%m-%d %H:%M:%S")
  echo "vault backup: $DATE" > "$MSG_FILE"
fi
```

---

#### To force Git to show login prompt for re-authentication

**Step 1**: Use this command and note down the resulting output value somewhere for use later.

```bash
git config --system credential.helper
```

**Step 2**: Now run the below command which will unset the existing credentials.

```bash
git config --system --unset credential.helper
```

**Step 3**: If you now run `git fetch`, Git should show you the login prompt. Enter the new token and you should be good.

**Step 4**: We'll now restore the value we had copied earlier in Step 1.

```bash
git config --system credential.helper <copied value>
```

---

#### To earch commits that touched the submodule
- You can filter commits that affected the submodule path:

```cpp
git log -- <submodule-path>
```

- This lists only the commits where the submodule path (`CppLib/`) was updated in the main repo.