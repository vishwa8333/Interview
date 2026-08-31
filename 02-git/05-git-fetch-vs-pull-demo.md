# Git Fetch vs Git Pull Demo

Please watch the Udemy video for this question. No additional information is required.

#So I did a changes in direct UI in branch like added some text, after when I type git fetch I was not able to see anything because it is in local.

$git branch -a  ## checking braches in command line

$git log remotes/origin/updateDescription  ### you will see changes what changes done

$ git diff updateDescription origin/updateDescription   #this will show what exact change is done

$git merge 1299948f9d908e77d81539165de36011f1080e21 ##by using commit ID we can merge that particular commit

$git merge remote/updateDescription  ## also you can run on branch also, example if there are 4 commits made. then all 4 commits will be pushed

$ git pull origin updateDescription 
# Fetches the latest changes from the "updateDescription" branch on the remote 
# repository and then merges those changes into the current local branch.
# In simple terms: git pull = git fetch + git merge


$ git rebase <branch-name>
# Reapplies your current branch's commits on top of the latest commit
# from the specified branch, creating a clean and linear commit history.
# Example: git rebase main








