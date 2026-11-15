# repos

One bash file that keeps the git repos of a folder in step with GitHub: what changed, what is ahead, what is
behind, pull, push, and a place for the cases that need a human.

    ./install            # repos -> ~/.local/bin (PREFIX, DESTDIR respected)
    repos                # status of every repo in ~/src (REPOS_ROOT changes the folder)
    repos fetch          # git fetch --all everywhere, read only
    repos pull frost     # one repo, or several names, or none = all; -i picks with fzf
    repos push -y        # shows the plan, asks once, pushes what is ahead
    repos needmerge      # what is waiting in needmerge/* branches
    repos merge frost    # merge needmerge/<branch> into the current branch

## The rule behind `needmerge`

Local and remote each have commits the other lacks (diverged). `pull` does not merge and does not guess: it puts the
remote's version on a new branch `needmerge/<branch>` (e.g. `needmerge/main`) and leaves the working branch, the
files and HEAD exactly as they were. `push` refuses such a repo and does the same. When you have time:
`repos needmerge` shows how far apart the two are, `repos merge NAME` shows what would come in, asks, merges; on a
conflict it aborts and nothing changes (finish by hand with `git merge needmerge/main`). If the remote moves again,
`needmerge/<branch>` is moved forward; if its history was rewritten, a second branch `needmerge/<branch>-<hash>` is added.

## What it never does

No force push, no `reset --hard`, no deleted branch or file, no touching a working tree with changed tracked files
(a behind repo with local edits is skipped and says so), no push without asking (`-y` or a terminal answer).
`-n` shows what would happen; `fetch` still runs then, it only reads.

## Good to know

- A repo that was pushed by URL (`git push git@github.com:you/x.git HEAD:main`) has no upstream; it shows
  `no remote branch` until `git push -u origin main` once.
- ahead/behind in `repos status` is as of the last fetch; `pull` and `push` fetch first.
- A fetch or push that hangs is cut after 25 s and retried over `ssh.github.com:443` (closed port 22).
- Exit status: 0 fine, 1 something failed, 2 bad usage.

Tests: `./test` (shellcheck + 16 bats tests on throw-away repos with local bare remotes; nothing touches GitHub).
