---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


> [!danger] Golden Rules of Secret Management
> 1. **Never commit secrets to git.** Once committed, a secret lives in the repository history forever.
> 2. **Never put real passwords in `appsettings.json`.** This file is committed and shared.
> 3. **Never put secrets in `launchSettings.json`.** This file is usually committed.
> 4. **Never log secrets.** Check that your logging configuration does not dump full configuration objects.
> 5. **Never embed secrets in code.** Hardcoded connection strings in C# files are just as dangerous as in JSON.
> 6. **Never share secrets via email, Slack, or Teams.** Use a secret manager or encrypted channel.

### If a Secret Was Already Committed

If you accidentally committed a secret, simply deleting it in a new commit is **not enough**. The secret remains in git history.

**Immediate steps**:

1. **Rotate the secret** -- generate a new password/key and invalidate the old one
2. **Remove from history** using `git filter-repo` or BFG Repo Cleaner (advanced)
3. **Force push** the cleaned history (requires team coordination)

```bash
# Using BFG Repo Cleaner to remove a file from all history
java -jar bfg.jar --delete-files secrets.json
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

> [!warning] Warning
> Force-pushing rewrites history and affects all team members. Always coordinate with your team before doing this. And always rotate the compromised secret first -- removing it from history does not guarantee nobody already copied it.

### .gitignore Best Practices

Ensure these patterns are in your `.gitignore`:

```gitignore
# Secrets and local config
.env
.env.*
*.local
secrets.json
**/appsettings.Local.json

# User Secrets are already outside the project, but be safe
```

> [!summary] Section Summary
> Never commit secrets to git, embed them in code, or log them. If a secret is accidentally committed, rotate it immediately and then clean the git history. Maintain a thorough `.gitignore`.
