!Git](./img/OpCards-Git.png)

# Git #

## Task: Manage files in github repository ##
### Prerequisites: ### 
Make sure you have an account with github
Configure your use of git

```
$ git config --global user.name “Your GitName”
$ git config --global user.emal “your@email.com”
```
This will be used to track who made which changes.

## Initialize a repository ##

Every time you create a project you need to initialize a repository

```
$ git init
$ git remote -v
$ git remote set-url origin <repository-url>
```
## Check on the status of your repository ##

```
$ git status
```
Shows the commit status of all the files in the repository.

##Add files into staging##

```
$ git add <files>
```

This will add the files to the staging area so the can be committed with the next commit. So not all changed files will be committed but only the staged ones.

Committing to the branch
```
$ git commit -a “Some explaining text”
```

Show what has been committed 
```
$ git log
```

Go back to a previous commit

```$ git checkout <hash from log>
```

## Branches ##
```
$ git branch <new branch name>
$ git branch 
$ git checkout <branch-name>
$ git push --set-upstream origin <branch-name>
$ git merge <branch-name>
```
##Two Factor Authenticationv
When 2FA is enabled the git commands need a password token instead of a password. To avoid being asked one every time, the password can be cached for a set time period. For example two hours.

```
$ git config --global credential.helper 'cache --timeout=7200’
```

## References ##

:link: [Learn Git in 15 minutes](https://www.youtube.com/watch?v=USjZcfj8yxE)
