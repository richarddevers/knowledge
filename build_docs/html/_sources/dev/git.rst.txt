git
****

<git init> init a repo

<git status> get status of what has been staged or not...

<git rm --cached <file>...> to unstage

"git add <file>...." add a file to staging

"git add ." add all files to staging

"git commit -m "revision one"" commit files with revision file

"git log" get commit log...

"git checkout <file_name>" discard changes in working directory

"git reset HEAD <file>"  remove commit from the staging environment

"git add --all" add all modification to the staging environment

"git checkout <hash>" rollback to a specified commit

"git branch" see branches list( local only)

"git branch -a" see all branches list

"git checkout <branch>"  go to specified branch

"git branch <name> <hash>" create a new branch to the specified hash states

"git merge <branch>" merge the <branch> to the current one

"git branch -m <branch>" <new_name_branch> rename a branch 

"git branch -D <branch>" delete a local branch

"git push origin --delete <branch_name>" delete a remote branch

"git reset --hard <hash>" specify new head

"git checkout -b <new_local_branch> <remote_branch>" 
-> Example: "git checkout -b 02_01 origin/02_01"  -> Clone the remote branch "origin/02_01" on the local branch "02_01"

"git clone -b <branch> clone a specific branch

"git stash" mettre en mémoire les modification actuelle et checkout

"git stash apply" reappliquer ces modifs