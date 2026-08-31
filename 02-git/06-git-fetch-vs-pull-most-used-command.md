## Question  
Which command do you use more often — `git fetch` or `git pull`, and why?

### 📝 Short Explanation  
This question explores your Git workflow habits and whether you prefer a manual or automated approach to syncing changes from a remote repository.

## ✅ Answer  
I mostly use `git pull` because it streamlines my workflow by fetching and merging remote changes in one step. It’s convenient for staying up to date quickly, especially when collaborating in fast-moving branches.

### 📘 Detailed Explanation  
`git pull` is essentially a shortcut that performs both a `git fetch` and a `git merge`. Instead of running two separate commands, I prefer to use:

```bash
git pull origin main
```

This makes my routine faster and keeps my local branch synchronized with the remote without extra steps. It's particularly useful when:
- I’m working alone or in a small team where merge conflicts are rare
- I'm contributing to a feature branch that others aren’t modifying
- I want to frequently pull in the latest changes to test or deploy updates

$ git merge <commit-ID>
- Merges the history/changes up to that commit into the current branch.
- If the commit is 3-4 commits ahead, the changes from those commits
- can be brought into the current branch.

$ git cherry-pick <commit-ID>
- Takes only the changes introduced by that specific commit
- and creates a new commit on the current branch.

Of course, I stay cautious by committing or stashing local changes before pulling to avoid conflicts or interrupted workflows. And if I suspect major upstream changes or want a closer look, I’ll temporarily switch to `git fetch`.

But for my day-to-day development, especially in active branches, `git pull` keeps things fast and simple — and that makes it my go-to.

---
