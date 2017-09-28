## Git

* [Merge      ](#Merge      )
* [Rebase     ](#Rebase     )
* [Remote     ](#Remote     )
* [Diff       ](#Diff       )
* [Tags       ](#Tags       )
* [Clean      ](#Clean      )
* [Undo       ](#Undo       )
* [Log        ](#Log        )
* [Stash      ](#Stash      )
* [Worktree   ](#Worktree   )
* [Credentials](#Credentials)
* [Aliases    ](#Aliases    )
* [Github     ](#Github     )
* [Links      ](#Links      )


<a id="Merge"></a>
#### Merge
merge master into the development first
so that if there are any conflicts,
resolve in the development branch itself
and master remains clean
--no-ff - to find out, who and when did the actual merge

    (devbranch)$ git merge master
    (resolve any conflicts)
    git checkout master
    git merge --no-ff development (no conflicts)


<a id="Rebase"></a>
#### Rebase

rebase the current branch onto base

    git rebase base

rebase workflow

    git checkout feature
    git checkout -b temporary-branch
    git rebase -i master
    git checkout master
    git merge temporary-branch

rebase of only last 3 commits

    git rebase -i HEAD~3

returns the commit id of the original base

    git merge-base dev master

rebase merge conflict

    git mergetool
    git rebase --continue

reset: --soft --mixed --hard

    git checkout hotfix
    git reset HEAD~2 //foo.cpp

checkout

    git checkout HEAD~2 //foo.cpp

revert rebase, else.. find the head commit, supose it was 'HEAD@{5}'

    git reflog
    git reset --hard HEAD@{5}


undo merge: if you haven't done anything else after the merge attempt

    git reset --hard HEAD@{1}


from which branch we are rebasing - that will be on top

    git pull --rebase origin master
    git push origin feature --force # ☠
    git pull --rebase origin feature

from master

    git merge feature --no-ff


###### resolve conflicts
    git checkout --theirs <conflicted-file>
    git checkout --ours <conflicted-file>

###### save direct commits history (xz)
   git pull --rebase -> fetch origin -> status -> fsck -> reflog -> rebase -i HEAD~3

or

    git stash -> git pull -> git stash apply -> fix conflicts

or

    git rebase --autostash

or in config

    rebase.autostash


<a id="Remote"></a>
#### Remote

Add remote

    git remote add reponame https://github.com/user/repo.git
    git fetch reponame
    git checkout branchname
    git checkout --track reponame/branchname

Push after rebase - force

    git push -f <reponame> <branchname>

Push to remote branch

    git push [reponame] [localbranch]:[remotebranch]

Delete remote branch

    git push [reponame] :[remotebranch]


<a id="Diff"></a>
#### Diff
    git difftool -d master..devel
    git difftool -d [branchname]


<a id="Tags"></a>
#### Tags
    git tag -a v1.4 -m 'my version 1.4'
    git fetch --tags
    git tag -l "v1.8.5*"
    git show v1.8.5


<a id="Clean"></a>
#### Clean

☠ Use with caution: remove not tracked

    git clean

safer: remove everything but save it in a stash

    git stash --all

remove all the untracked files in your working directory

    git clean -d

-x - remove ignored
-n - dry run
-f - force ☠
-i - interactive

###### List git-ignored files
    git ls-files . --ignored --exclude-standard --others

###### List untracked files
    git ls-files . --exclude-standard --others
    git diff --name-status


<a id="Undo"></a>
#### Undo

revert deleted branch

    git fsck --full --no-reflogs --unreachable --lost-found | grep commit | cut -d\  -f3 | xargs -n 1 git log -n 1 --pretty=oneline >  lost-found.txt
    git config --global alias.rescue '!git fsck --full --no-reflogs --unreachable --lost-found | grep commit | cut -d\  -f3 | xargs -n 1 git log -n 1 --pretty=oneline > .git/lost-found.txt'
    git cat-file -p [commit_chsum]
    git log [commit_chsum]
    git branch commit_rescued [commit_chsum]


<a id="Log"></a>
#### Log
log by files

    git log -- foo.py bar.py

to apply commit to your current branch: git-cherry-pick
pick in gitk and cherry-pick them with right-clicks on the commit

    git cherry-pick <commit>

If you want to go more automatic (with all its dangers)
and assuming all commits since yesterday happened on wss
you could generate the list of commits using

    git log --reverse --since=yesterday --pretty=%H
    for commit in $(git log --reverse --since=yesterday --pretty=%H);
    do
        git cherry-pick $commit
    done

If something goes wrong here (there is a lot of potential)
you are in trouble since this works on the live checkout
so either do manual cherry-picks or use rebase

    git diff $start_commit..$end_commit -- path/to/file
    git blame -w # ignore whitespaces
    git blame -M # ignore text moving
    git blame -C # ignore text moving to other files


<a id="Stash"></a>
#### Stash
    git stash
    git stash list
    git stash apply stash@{2}
    git stash drop stash@{0}
    git stash pop # = apply + drop
    git stash branch [name] # create branch from stash

stash options

    --keep-index - not to stash staged
    --include-untracked or -u - +untracked


<a id="Worktree"></a>
#### Worktree
    git worktree add ../folder name-of-branch
    git worktree list
    git worktree prune


<a id="Credentials"></a>
#### Credentials
cache credentials
If you’re using a Mac, Git comes with an “osxkeychain” mode
, which caches credentials in the secure keychain that’s attached to your system account.
This method stores the credentials on disk, and they never expire
, but they’re encrypted with the same system that stores HTTPS certificates and Safari auto-fills.

If you’re using Windows, you can install a helper called “wincred.”
This is similar to the “osxkeychain” helper described above
, but uses the Windows Credential Store to control sensitive information.

    git config --global credential.helper cache.
    git ls-remote [remote]
    git remote show [remote]
    git fetch origin
    git remote add otherrepo git://git.otherrepo.com
    git fetch othrrepo
    git push <remote> <branch>

    git config --global push.default tracking


<a id="Aliases"></a>
#### Aliases
in gitconfig

    hist = log --pretty=format:'%C(yellow)%h %C(cyan)%ad %Creset| %s%C(green)%d %Creset[%an]' --graph --date=short

in bashrc

    PS1='\[\033]0;$TITLEPREFIX:${PWD//[^[:ascii:]]/?}\007\]\n\[\033[32m\]\u@\h \[\033[35m\]$MSYSTEM \[\033[33m\]\w\[\033[36m\]`__git_ps1`\[\033[0m\]\n--> '


<a id="Github"></a>
## Github

#### URLs

###### public keys
    https://github.com/<user_name>.keys

###### add .diff or .patch to url
    https://github.com/tars/tars/commit/07902a9.diff

###### permanent link to specific commit file - press 'y'
    <permanent link>

###### diff without whitespaces
    ?w=1

###### block highlight 15 to 17 line
    #L15-L17

###### revision diff
    https://github.com/github/linguist/compare/master@%7B2week%7D...master
    master@{1day}...master

#### Hotkeys
    '?' - show all hotkeys
    't' - file search
    'l' - go to line

    'gp' - go pull-requests
    'gi' - go issues
    'gn' - go notifications

#### Tickets/pull requests

###### close issue via commit message use 'fix/resolve/close'
    git commit -m "Fix screwup, fix #12"

#### Special files
    ISSUE_TEMPLATE.md - opening new ticket
    PULL_REQUEST_TEMPLATE.md - will show up when


<a id="Links"></a>
#### Links
    https://github.com/search
    https://github.com/explore
    https://github.com/trending
    https://status.github.com
    https://gist.github.com - drafts
    https://help.github.com
    https://github.com/github/hub
    https://habrahabr.ru/post/129343/ - user scripts
    https://greasyfork.org/en/scripts/by-site/github.com
    https://github.com/jerone/UserScripts/tree/master/Github_Commit_Whitespace
    https://github.com/jerone/UserScripts/tree/master/Github_News_Feed_Filter
    https://github.com/gelstudios/gitfiti
    http://dotfiles.github.io
    https://octodex.github.com
    https://github.com/Kikobeats/awesome-github
    https://git-lfs.github.com - large file storage
    https://github.com/buunguyen/octotree - chrome ext