`git-rv`
========

`git-rv` is a command line tool for syncing local git clients with code
reviews hosted on a [Rietveld][rietveld] code review server such as
[codereview.appspot.com][codereview].

## `git-rv` Basics

Using `git-rv`, you can create micro-commits locally, export them for
review, and `git-rv` will create a single commit once your patch has
been approved by your reviewers.

When you begin your review, `git-rv` will track a specific
`git` [remote][git-remote] and a branch in that remote. Once a remote branch
has been tracked for a review, all [diffs][git-diff] sent for review will be
relative to that remote branch. If that branch is updated by another commit
during your review, you can bring your review branch up to date by running
`git rv sync`.

**NOTE**: If there are multiple remotes associated with your local `git`
repository and/or multiple branches in the selected remote, the tool will
prompt you to make a choice.

To commit reviewed code, you'll never need to run `git push`;
`git rv submit` will handle that for you. In this process, the tool combines
your micro-commits into a single commit and makes sure it pushes your local
code to the correct branch in the correct `git` remote.

## Installation

To install `git-rv`, first clone this repository:

    $ git clone https://github.com/GoogleCloudPlatform/git-rv.git

and then either copy the `git-rv` binary into a directory on your `$PATH`,
create a symlink, or add the `git-rv` directory to your `$PATH`:

    $ cp git-rv/git-rv /some/directory/on/your/path
    $ # OR
    $ ln -s git-rv/git-rv /some/directory/on/your/path/git-rv
    $ # OR
    $ PATH=${PATH}:/absolute/path/to/git-rv-directory

**NOTE:** No matter your choice, it is possible it will require `sudo` to copy
or create a symlink for directories on your path.

Once you've installed, executing `git rv {$COMMAND}` from within a `git`
repository will call `git-rv`.

## Basic Workflow

`git-rv` supports many commands (run `git-rv --help`) so see them all, but
the main ones needed for code reviews are `export`, `submit` and `sync`.
For editing and committing your code, creating branches and doing work
locally, you can use `git` as you usually would. It is only when interacting
with your code review that you'll need to use `git-rv`.

A typical review may look like the following:

1.  **Start a feature branch:**

        $ git branch
        * master
        $ git checkout -b {$BRANCH}
        $ git branch
        * {$BRANCH}
          master

1.  **Make an initial commit:**

        $ emacs ... # Do some work
        $ git add .
        $ git commit -m "Adding super cool feature X."
        ...
        $ git rv export -r reviewer@example.com
        Upload server: codereview.appspot.com (change with -s/--server)
        Loaded authentication cookies from {$HOME}/.codereview_upload_cookies
        Issue created. URL: http://codereview.appspot.com/{$ISSUE}
        Uploading base file for {$FILENAME1}
        Uploading base file for {$FILENAME2}
        ...
        Metadata update from code server succeeded.

1.  **Address comments from your code review:**

        $ emacs ... # Do some more work
        $ git add .
        $ git commit -m "Fixing the blerg typo per reviewer request."
        ...
        $ git rv export
        Upload server: codereview.appspot.com (change with -s/--server)
        Loaded authentication cookies from {$HOME}/.codereview_upload_cookies
        Issue updated. URL: http://codereview.appspot.com/{$ISSUE}
        Metadata update from code server succeeded.

1.  **Find out someone else committed to master, sync their changes in:**

        $ git rv sync
        ...
        Auto-merging {$FILENAME1}
        Auto-merging {$FILENAME2}
        ...
        Squash commit -- not updating HEAD
        [{$BRANCH} {$SYNC_COMMIT}] Syncing review {$BRANCH} at {$SYNC_COMMIT}
        2 files changed, ...
        ...
        Exporting synced changes.
        Upload server: codereview.appspot.com (change with -s/--server)
        Loaded authentication cookies from {$HOME}/.codereview_upload_cookies
        Issue updated. URL: http://codereview.appspot.com/{$ISSUE}
        Uploading base file for {$FILENAME1}
        Uploading base file for {$FILENAME2}
        ...
        Metadata update from code server succeeded.

    **NOTE:** This assumes the merge initiated by the `sync` was all peachy. If
    not, `git-rv` will do it's best to make sure you resolve the merge
    conflicts and get the review back on track.

1.  **Time to submit:**

    Trying to submit before one of your reviewers gives an LGTM (short
    for "looks good to me") will result in a failure:

        $ git rv submit
        This review has not been approved.

    Once the review has been LGTM'ed by one of your reviewers, you can submit
    your changes:

        $ git rv submit
        Checking out origin/review-{$ISSUE} at {$SYNC_COMMIT}
        Adding reviewed commits.

        Adding commit:
        Adding super cool feature X.

        Reviewed in https://codereview.appspot.com/{$ISSUE}

        Replacing review branch '{$BRANCH}' with newly committed content.

        Loaded authentication cookies from {$HOME}/.codereview_upload_cookies

        Message:

        Added in
        https://code.google.com/p/dhermes-projects/source/detail?r=7fb62f85b1209fb62d8181097bfb2529cc2fc875

        posted to code review issue.

        Issue {$ISSUE} has been closed.

## Power Users and Committers

For more details on the other commands, simply execute `git-rv --help` or
`git rv {$COMMAND} --help`.

Feel free to file new issues and feature request, comment on existing ones
and fork this repository to your heart's content.

If you are working on changes to `git-rv`, you will need to be able to run
`make_executable.py` to create a new `git-rv` binary based on your working
copy of the repository.

In order to do this you'll need to initialize the Rietveld `git` submodule.
Since Rietveld is a [Mercurial][mercurial] project, we use
[`git-remote-hg`][git-remote-hg] to include it as a `git` submodule. To
pull it, run

    $ # If you don't have git-remote-hg installed
    $ sudo pip install --upgrade git-remote-hg
    $ git submodule update --init

## Using `git-rv` for Mercurial repositories

If you'd like to use `git-rv` to do code reviews for your [`hg`][mercurial]
repositories, this is fully supported. In the same fashion we use
[`git-remote-hg`][git-remote-hg] to support the Rietveld submodule, it can
be used to work on an `hg` repository.

For example, to work on [`google-api-python-client`][google-api-python-client],
we check out the repository with `git` by appending a dummy protocol `hg::`

```
git clone hg::https://code.google.com/p/google-api-python-client/
```

(This looks similar to the protocols we're used
to: `http:`, `https:` and `git:`.)

After doing this, you can jump in right away and interact with the repository
as if it were a `git` repository:

```
cd google-api-python-client
emacs ... # Do some work
git rv export -r reviewer@example.com
```

After the review is complete, you can submit via `git rv submit`. For Google
Code Hosting reviews (and other Mercurial hosting sites to follow), after
submitting, a link to the newly pushed commit will be added to the Rietveld
review.

[rietveld]: https://code.google.com/p/rietveld/
[codereview]: https://codereview.appspot.com
[mercurial]: http://mercurial.selenic.com/
[git-remote-hg]: https://github.com/rfk/git-remote-hg
[google-api-python-client]: https://code.google.com/p/google-api-python-client/
[git-remote]: http://git-scm.com/book/en/Git-Branching-Remote-Branches
[git-diff]: http://git-scm.com/docs/git-diff
