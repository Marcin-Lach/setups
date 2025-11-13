Example git config.  
Make sure 
- to instal `git lfs` or omit it from the config.  
- that vs code is added to PATH
- `core.autocrlf` matches your system (`true` for windows, `false` for others)
  
```ini
[filter "lfs"]
	process = git-lfs filter-process
	required = true
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
[user]
	name = <>
	email = <>
[core]
	editor = code --wait --new-window
	autocrlf = true
[fetch]
	prune = false
[pull]
	rebase = true
[push]
	default = simple
	followTags = true
[rebase]
	autoStash = false
[rerere]
	enabled = 1
	autoUpdate = 1
[alias]
	f = fetch origin
	fr = "!f() { echo -e '\\033[1;40;32mFetching...\\033[0m'; git f; echo -e '\\033[1;40;33mRebasing...\\033[0m'; git rebase --quiet; echo -e '\\033[2;49;36mDone.\\033[0m';}; f"
	p = push
	pf = push --force
	mt = mergetool
	rpo = remote prune origin
	edit-global-config = config --global --edit
	rh = reset --hard
	cp = cherry-pick
	cpc = cherry-pick --continue
	cpa = cherry-pick --abort
	cm = "!f() { commit -m \\"$@\\";}; f"
	cam = commit --amend
	caam = "!f() { git aa; git cam;}; f"
	aa = add --all
	s = status
	ahead = !git --no-pager log --pretty=oneline @{u}..HEAD
	behind = "!f() { echo -n \\"You are: \\"; git rev-list ..@{u} --count --pretty=oneline | tr -d \\"\\n\\"; echo \\" commits behind\\"; git log --pretty=oneline --format=format:'%C(bold blue)%h%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset) %C(bold green)(%ar) %C(reset)%C(bold cyan)%aD%C(reset)' -10 ..@{u};}; f"
	c = checkout
	chm = checkout master
	chd = checkout dev
	graph-n = !sh -c \"git log -$1 --date=short --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset)%C(bold yellow)%d%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset) %C(bold green)(%ar) %C(reset)%C(bold cyan)%aD%C(reset)' $2 \" -
	g = !sh -c \"git graph-n 10\" -
	r = rebase --rebase-merges
	rbc = rebase --continue
	rba = rebase --abort
	rebase-interactive = !sh -c \"git rebase -i HEAD~$1\" -
	ri = !sh -c \"git rebase-interactive $1\" -
	delete-gone = !"git fetch -p && for branch in $(git for-each-ref --format '%(refname) %(upstream:track)' refs/heads | awk '$2 == \"[gone]\" {sub(\"refs/heads/\", \"\", $1); print $1}'); do git branch -D $branch; done"
```

## to review

.

## Git book

<https://git-scm.com/book/en/v2>

## So You Think You Know Git (Part 2)

<https://www.youtube.com/watch?v=Md44rcw13k4&ab_channel=GitButler>

- Switch + Restore instead of Checkout
- Hooks
- .gitattributes
- Smudge and Clean (LFS)

- merge workflow vs rebase workflow
    - commit --fixup=<commitSha>
        - `git commit --fixup ":/base types" ` # The :/ prefix means "try to find a commit containing the phrase base types". It will stop at the first commit it matches and use that. But be warned: the search is case-sensitive, and if the search doesn't match a commit it won't error, it will just use the provided string.
    - rebase --autosquash <branch> # looks for "special" commit messages prefixed with fixup!, squash!, or amend! to order commits in a way that allows for easier squashing when in rebase interactive mode
        - `git config --global rebase.autosquash true` # enable autosquash by default
- stacked branch / stacked diff workflow
    - one feature branch originate from other feature branch which originate from yet another feature branch, creating a stack of branches
    - when rebasing the top-most branch with `main`, you probably want to also rebase the older branches to limit diff
    - `git config --global rebase.updateRefs true` # move references for the whole stack of branchs
- working with large repositories (milions of files, gigabytes)
    - instead of cloning with `git clone` command, clone with `scalar clone` command
    - use sparse-checkout
- worktrees
    - work on more than one branch at a time
    - when you need to patch something while working on some other feature, instead of stashing/commiting not finished work and changing branches, you can use worktree to create a new working directory for each branch


## So You Think You Know Git - FOSDEM 2024 (Part 1)

<https://www.youtube.com/watch?v=aolI_Rz0ZqY>

- aliases 
    - `git config --global alias.a 'stash --all'` # add new command that does some other git commands
    - `git config --global alias.c !bash-script.sh` # executes bash script instead of git command
- conditional configs
    - add configurations based on directory, useful for having personal and corporate email/user separation
    - `[includeIf "gitdir:~/projects/work/"] <newLine> path = ~/projects/work/.gitconfig`
- `git blame -L x,y path/to/file` - show blame for selected lines
    - `-w` # ignore whitespace
    - `-C` # detect code moved or copied in the same commit and ignore it
    - `-C -C` # the same as above plus ignore if done in commit that created the file
    - `-C -C -C` # ignore code movement in any commit at all, so it shows the person that introduced the line of code instead of the one that was just moving/copying the code, also cover for changes to file name 
- `git log -L x,y:path/to/file` - show list of changes to specified lines of file
    - `git log -L :MethodName:path/to/file` #show changes for method named MethodName in specified file

- `git log -S` # the "pickaxe"; show the code that might have now be removed, but git will still find it in history and show you when it was moved
- `git reflog` # list of operations with references for each operation
- `git diff --word-diff` # show diff but based on words and not lines
- `git config --global rerere.enabled true` # REuse REcorded REsolution; useful when rebasing, if the merge conflict has already been solved, git will automatically apply the resolution

- git branch --column # list branches in columns and not in list
    - `git config --global column.ui auto` # git will list things in columns
    - `git config --global branch.sort -committerdate` # list branches by most recently modified, instead of alphabetical

- `git push --force-with-lease` # use force push only if remote does not have any commits that are unknown to local
    - `git push --force-if-includes` # fixes a corner case, when you rebase feature branch without pulling changes
        - `git config --global push.useForceIfIncludes true` 
- commit signing with ssh instead of gpg
    - `git config --global gpg.format ssh` # use ssh for signing
    - `git config --global user.signingkey ~/.ssh/key.pub` # path to public key
    - `git commit -S` # commit with signing

- git maintainance start # add cron job for git that maintain the repository, like running gc, pruning, update commit graph
    - `git config --global fetch.writeCommitGraph true` # update the cache of commit graph on each fetch, will make log operations faster

- Git file system monitor - when running `git status` git must check all files in the repository to know if those were modified
    - configure git fsmonitor to record if file was changed or not to speed up the git status command
        - `git config --global core.fsmonitor true` # turn on the fsmonitor for repositories

- `git clone --filter=blob:none` # clone without blobs, might be useful for repositories with games that does not use LFS for some reason; executes 2 fetches, first for commits and trees, second for WD (Working Directory) Blobs
- `git clone --filter=tree:0` # clone only HEAD, without any trees; useful for CI or large repostories
    - when running git blame, git will have to fetch every version of blob on demand, one-by-one



## git config

- `git config --global rebase.autosquash true` # enable autosquash by default
- `git config --global rebase.updateRefs true` # move references for the whole stack of branchs
- `git config --global rerere.enabled true` # REuse REcorded REsolution; useful when rebasing, if the merge conflict has already been solved, git will automatically apply the resolution
- `git config --global push.useForceIfIncludes true` 
- `git config --global column.ui auto` # git will list things in columns
- `git config --global branch.sort -committerdate` # list branches by most recently modified, instead of alphabetical
- `git config --global fetch.writeCommitGraph true` # update the cache of commit graph on each fetch, will make log operations faster

# https://blog.gitbutler.com/how-git-core-devs-configure-git/


```ini
# clearly makes git better

[column]
        ui = auto
[branch]
        sort = -committerdate
[tag]
        sort = version:refname
[init]
        defaultBranch = main
[diff]
        algorithm = histogram
        colorMoved = plain
        mnemonicPrefix = true
        renames = true
[push]
        default = simple
        autoSetupRemote = true
        followTags = true
[fetch]
        prune = true
        pruneTags = true
        all = true

# why the hell not?

[help]
        autocorrect = prompt
[commit]
        verbose = true
[rerere]
        enabled = true
        autoupdate = true
[core]
        excludesfile = ~/.gitignore
[rebase]
        autoSquash = true
        autoStash = true
        updateRefs = true

# a matter of taste (uncomment if you dare)

[core]
        # fsmonitor = true
        # untrackedCache = true
[merge]
        # (just 'diff3' if git version < 2.3)
        # conflictstyle = zdiff3 
[pull]
        # rebase = true
```
