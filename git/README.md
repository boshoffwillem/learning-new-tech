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

You can run the ```git remote -v``` command again to see that the URL has been updated.

## Clean repository of all non-tracked artifacts

Project repositories can quickly consume disk space with build artifacts,
third-party dependencies, and non-tracked files.
Folder bloat can especially be true for long-running projects
that have seen many iterations on your machine.
A trick you can use to get back to a "clean" state in your repository folder
is to use the ```git clean``` command.

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
