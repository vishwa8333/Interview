# Git Fork in Action

Please watch the Udemy video for this question. No additional information is required.

$git clone https://github.com/iam-veeramalla/shell.git  ### This will create a clone copy of repo

$Git checkout -b updateDescription  ### this will create a branch

$git status ## this will show which branch and commit status

$ vim textfilesample.txt ### just edit some thing for testing

$git add .  ## this will update the changes

$git commit -am "chore: updated description" ## This is committing a changes with description

$ git push origin
# Pushes your committed changes to the remote repository.
# If the branch isn't specified, Git may ask you to set the upstream branch.

$ git remote -v
# Shows the remote repositories connected to your local repository.
# "origin" normally points to the GitHub repository you cloned from.

$ git push origin updateDescription
# Pushes the "updateDescription" branch to the "origin" remote repository.
# This publishes your local branch and its commits to GitHub.













