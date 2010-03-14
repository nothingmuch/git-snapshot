# git snapshot

This command is provides a snapshotting mechanism for refs/workdirs.

## Synopsis

Routinely run git snapshot to save undo points:

	% vim quxx.txt
	% git snapshot
	# ...
	% git add . && git commit -am "new commit"
	% git snapshot

Later, if you need to recall any recorded information:

	% git log refs/snapshots/HEAD
	% git log refs/snapshots/refs/heads/master
	% git checkout refs/snapshots/refs/heads/master:quxx.txt

## Rationale

Often times I find myself sketching out projects without wanting to actually
record a revision history yet.

Even though the history has no organizational value I could run:

	git add . && git commit -m "$(date)"

every once in a while in order to record an edit history, sort of like an
on-disk undo log.

`git snapshot` provides something less messy than this ad-hoc approach, and
safer than simply not recording anything at all.

## Behavior

When invoked `git snapshot` will run `git snapshot-ref HEAD`, and if `HEAD` is
a symbolic ref, also `git snapshot-ref refs/heads/master` or whatever HEAD is
referring to.

`git-snapshot-ref` starts by running `git snapshot-tree`, which reads the
current working tree into a temporary Git index file, uses `git write-tree` to
record that data in the object database, and prints the sha1 of the recorded
tree.

`git-snapshot-ref` then checks if the previous snapshot (recorded in
`refs/snapshots/$source_ref`, i.e. `refs/snapshots/HEAD`,
`refs/snapshots/refs/heads/master`, ...) is up to date. If the tree has changed
or new commits have been made on the ref, then a new commit is recorded into
the snapshots ref.

If you're working on a large patch, you can also snapshot an explicit ref:

	git snapshot-ref topic_branch

Once you've created your commits, and then squashed and merged them you can:
delete the snapshot log for that branch:

	git update-ref -d refs/snapshots/topic_branch

## Usage

By routinely running `git snapshot` (for instance from within a crontab) you
get a fully versioned history that is somewhat similar in usefulness to `git
reflog`; if you made a mistake in recording your changes, or if you
accidentally modified a file before comitting it, you can recall a previous
version from the snapshot history:

	git log refs/snapshots/...
	git checkout refs/snapshots/...:the/file/i/want.txt

## With Spotlight

On MacOS X you can use the Finder's _Get Info_ dialogue to add a spotlight
comment or a label to project directories.

Then you can use one of the following commands in your crontab to automatically
snapshot it:

	# snapshot every directory with 'git-snapshot' in its comment
	mdfind -0 "kMDItemFinderComment == '*git-snapshot*'" | xargs -0 -n 1 git-snapshot

	# snapshot every directory under ~/code that has the red label
	mdfind -0 -onlyin ~/code "kMDItemFSLabel == 6" | xargs -0 -n 1 git-snapshot

## See Also

- `git help rev-parse` for specifying time based revisions (e.g.
  `refs/snapshots/HEAD@{yesterday}`)
- `git commit --amend` in conjuction with `git reflog` can provide a similar
  safety net.
