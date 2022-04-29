# Git Proficiency

This README contains information on how to do some unusual and advanced stuff in git.

## Changing the remote HTTPS URL to an SSH URL

Get the remote URL list of the repo

```bash
git remote -v
```

You should get a list showing that the origin is at some HTTPS URL.
To change the URL for the origin remote, you can use the set-url command.
The SSH URL should be available from your VCS provider.

```bash
git remote set-url origin git@github.com:username/repositoryname.git
```

You can run the `git remote -v` command again to see that the URL has been updated.

## Clean repository of all non-tracked artifacts

Project repositories can quickly consume disk space with build artifacts,
third-party dependencies, and non-tracked files.
Folder bloat can especially be true for long-running projects
that have seen many iterations on your machine.
A trick you can use to get back to a "clean" state in your repository folder
is to use the `git clean` command.

**Warning: Be sure all files you want to retain are committed;
this is a destructive command with no recovery.**

Run

```bash
git clean -xdf
```

Let's look at what each of the flags means:

- x : Normally, the clean command will ignore files and folders found in the
.gitignore file of the repository. This flag tells the command to target all
untracked files, even if they happen to be ignored.
- d : Inform the command to recurse through all directories,
whether they are tracked or untracked.
- f : force the clean command to delete files and directories.

If you feel weary of running the **clean** command without knowledge
of what it will do to your repository,
then add the **n** flag along with the others to perform a dry-run.

```bash
git clean -xdfn
```

## Fix your last commit message

Sometimes you make a mistake in your commit message,
or you just want to add more information.

To edit you last commit message:

```bash
git commit --amend
```

This opens an editor where you edit your message.
To directly change the message run:

```bash
git commit --amend -m "New message"
```

## Add files to your last commit

Just like you need to sometimes change your last commit message,
sometimes you forgot to add certain files.

To add files to your last commit,
first stage the files you want to add,
and then amend the commit.

```bash
git add .
git commit --amend --no-edit
```

## Undo changes to previously commited files

You may start making changes to a committed file only to realize
you no longer want the changes. Luckily, rolling back to a known good state
is as simple as checking out the file.

```bash
git co file.extension
```

## Undo last local commit

You've been working on a significant feature only to realize
you just committed all your work to the **main** branch
when you meant to commit to a new feature branch.

To undo your commit locally, run the following command.

```bash
git reset --soft HEAD~1
```

The Git command will compare the current HEAD commit (your last commit)
to the commit before it (before you committed) and stage the files.
At this point, you have an opportunity to create a new branch
and save your changes correctly.

## Worktrees

Ever want to checkout multiple branches at the same time?
Or, you are busy on a branch a suddenly the is a bug that you need to look at...
Then you first have to stage you changes, create a new branch, fix the bug,
switch back to your branch, and unstage.

With `git worktrees` you don't have to. You can keep your current branch,
and create and checkout the branch to fix the bug on at the same time.

Suppose you have a repo `Test`. Every worktree you make is effectively a new
mini repository contained in a folder. So create a folder somewhere that will
contain all your worktrees.

I usually create a folder on the same level as the repo folder, with the same name
and plus a `.Worktrees`, so you'll have:

```
- Test
 - // code files
- Test.Worktrees
 - <worktree>
  - // code files
 - <worktree>
  - // code files
```

### Creating Worktrees

To create a worktree, from within you repo enter

```bash
git worktree add <path_to_worktrees_folder>/<worktree_name>
```

This will create a worktree in in your worktrees folder, and automatically
create and checkout a branch with the same name as your worktree.
So let's say I'm in my Test repo then I enter:

```bash
git worktree add ../Test.Worktree/UrgentBug
```

This will create the following structure:

```
- Test
 - // code files
- Test.Worktrees
 - UrgentBug
  - // code files
```

Remember UrgentBug will be like cloning the Test repo
and create a branch `UrgentBug` and then check it out.

When I am done with with my worktree I navigate back
to the Test repo and remove it with:

```bash
git worktree remove ../Test.Worktrees/UrgentBug
```

### Custom Worktree Branch Name

You can also create a worktree and specify a different name for the branch
by simply adding the name as an argument:

```bash
git worktree add <path_to_worktrees_folder>/<worktree_name> -b <branch_name>
```

### Checkout Existing Branch in Worktree

You can also create a worktree and checkout an existing branch.

```bash
git worktree add <path_to_worktrees_folder>/<worktree_name> <existing_branch>
```

## Git Bisect

Git bisect is a great way to identify a commit where something broke.
Let's say your on a branch and something you know should work isn't working
anymore. Now you need to identify what commit in history broke it.

To start the bisect process enter:

```bash
git bisect start
```

Then we need to mark the current commit as not working or bad:

```bash
git bisect bad
```

Now we need to tell git what commit in history was working;
let's say you know that 10 commits back this feature wasn't broken.
Get the hash of that commit and mark it as good

```bash
git bisect good <hash>
```

Bisect would've now starting traversing your history. You will continue the
traversing by marking commits as bad until you find a commit that's working
again, mark it as good, and then the bisecting is done.

Obviously the next commit after the one that was marked good is the culprit.

So let's say our history is:

```bash
123asd456bmn -- Commit 5, HEAD
234qwe567iop -- Commit 4
345iop678sdf -- Commit 3
456hkjl789ds -- Commit 2
567iop789dfg -- Commit 1
```

So let's say `Commit 5` is broken. We know the feature worked
in `Commit 1`. We start bisecting

```bash
git bisect start
```

Mark the commit we're at (`Commit 5`) as bad.

```bash
git bisect bad
```

Then specify the commit we know was good (`Commit 1`)

```bash
git bisect good 567iop789dfg 
```

Then start bisecting.

```bash
git bisect start
```

We will now be checked out at `Commit 4`. Let's say `Commit 4` is
also broken.

```bash
git bisect bad
```

We will now be checked out at `Commit 3`. Let's say `Commit 3` is
also broken.

```bash
git bisect bad
```

We will now be checked out at `Commit 2`. Let's say `Commit 2` is
working. We now mark this commit as good.

```bash
git bisect good
```

Bisect will output the first breaking commit for you
(in this case the hash of `Commit 3`).

Then to end the bisecting process enter:

```bash
git bisect reset
```

You will now be checked out at `HEAD` again.

You can now, for example enter

```bash
git show <broken_commit_hash>
```

This will show you all the changes in that commit.

## Interactive Rebasing

Rewrite and re-organize your history.Sometimes you make a lot of local
commits, some of these commits mights be duds, unncessary, not descriptive
enough, etc. Before you push to origin you want cleanup your history.

Run

```bash
git rebase -i <branch_to_rebase_on>
```

This will open an editor for you to edit your history in.
Each commit will have a keyword prefixed to it that allowes
you to modify/change that commit.

These keywords are:

- **p, pick**   `<commit>` = use commit
- **r, reword** `<commit>` = use commit, but edit the commit message
- **e, edit**   `<commit>` = use commit, but stop for amending
- **s, squash** `<commit>` = use commit, but meld into previous commit
- **f, fixup**  `<commit>` = like "squash", but discard this commit's log message
- **x, exec**   `<command>` = run command (the rest of the line) using shell
- **b, break** = stop here (continue rebase later with 'git rebase --continue')
- **d, drop**  `<commit>` = remove commit
- **l, label** `<label>` = label current HEAD with a name
- **t, reset** `<label>` = reset HEAD to a label
- **m, merge** [-C `<commit>` | -c `<commit>`] `<label>` [# `<oneline>`]
  - create a merge commit using the original merge commit's
  - message (or the oneline, if no original merge commit was
  - specified). Use -c `<commit>` to reword the commit message.
