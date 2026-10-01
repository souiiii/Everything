# Git-GitHub-Notes

# Git & GitHub — From Zero to Interview-Ready

*Written for someone who has typed `git push` a hundred times without being sure what it does. Nothing below assumes prior knowledge. Every command is explained by what it does to your files — because that’s the part nobody tells you, and it’s the part that makes git feel unpredictable.*

---

## Part 0 · The one idea that makes git make sense

Most tutorials teach commands. That’s why git feels arbitrary — you memorise `git add`, `git commit`, `git push` as a magic incantation and then panic the first time something unexpected happens.

Here is the actual model. Everything else follows from it.

**Git stores snapshots, not changes.** Every time you commit, git saves a complete photograph of your project at that moment. Not “line 42 changed” — the whole thing. (It compresses cleverly behind the scenes so this isn’t wasteful, but the mental model is photographs.)

**Those snapshots form a chain.** Each one points back to the one before it:

```
  A  ←──  B  ←──  C
first   second   third
photo    photo    photo
```

That chain is your project’s history. `C` knows its parent is `B`, `B` knows its parent is `A`.

**A branch is a sticky note pointing at one snapshot.** That’s genuinely all it is:

```
A  ←──  B  ←──  C
                ↑
              main
```

When you make a new commit, the sticky note slides forward:

```
A  ←──  B  ←──  C  ←──  D
                        ↑
                      main
```

**HEAD is a sticky note pointing at whichever branch you’re currently on.**

```
A  ←──  B  ←──  C
                ↑
              main
                ↑
              HEAD     ("you are here")
```

Creating a branch costs nothing — it’s writing one more sticky note on an existing commit. That’s why git people branch constantly and other version-control users find it alarming.

Hold those four facts. Snapshots, chained. Branch = pointer. HEAD = where you are.

---

## Part 1 · The three places your code lives

This is the concept that unlocks `add` vs `commit`, and almost nobody explains it properly.

Your project exists in **three separate areas** at once:

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│    WORKING     │    │    STAGING     │    │   REPOSITORY   │
│   DIRECTORY    │    │      AREA      │    │   (.git dir)   │
│                │    │                │    │                │
│  the actual    │    │  what you've   │    │  the permanent │
│  files you     │    │  marked for    │    │  chain of      │
│  edit          │    │  the next      │    │  snapshots     │
│                │    │  snapshot      │    │                │
└────────────────┘    └────────────────┘    └────────────────┘
        │                     │                     │
        │   git add           │   git commit        │
        └────────────────────►└────────────────────►│
```

**Working directory** — the folder on your disk. What your editor sees. Files here are just files; git isn’t protecting them yet.

**Staging area** (also called the “index”) — a waiting room. You put files here to say *“this is going in the next snapshot.”*

**Repository** — the `.git` folder. Once a snapshot is here, it’s essentially permanent and recoverable.

### Why does staging exist? Why not commit directly?

Because a commit should be **one logical change**, and your working directory usually contains several unrelated ones.

Concretely. You sat down to fix a login bug, and while you were there you also renamed a variable in an unrelated file and updated the README:

```
modified:  routes/auth.js      ← the actual bug fix
modified:  utils/format.js     ← unrelated tidying
modified:  README.md           ← unrelated
```

Committing all three together makes one messy snapshot. Six months later, when the login breaks again and you’re reading history to find what changed, you get “fixed login and also some other stuff.” Useless.

Staging lets you separate them:

```bash
git add routes/auth.js
git commit -m "Fix session cookie not clearing on logout"

git add utils/format.js README.md
git commit -m "Tidy formatting helper and update README"
```

Two clean snapshots. History becomes readable.

**The interview version:** *“The staging area lets me choose what goes into a commit, so each commit is one coherent change rather than a dump of everything I happened to have open.”*

---

## Part 2 · Your first repository, keystroke by keystroke

Let’s build one and watch each area change.

### Starting out

```bash
mkdir todo-app
cd todo-app
git init
```

**What just happened to your folder:** git created a hidden `.git` directory inside `todo-app`. That folder *is* the repository — every snapshot, every branch, every configuration lives inside it. Delete `.git` and you have an ordinary folder with no history. Nothing else changed; your files are untouched.

```bash
git status
```

Get used to running this constantly. It’s git telling you the state of all three areas, and it usually suggests the command you want next.

```
On branch main
No commits yet
nothing to commit (create/copy files and use "git add" to track)
```

### Adding a file

```bash
echo "console.log('hello');" > app.js
git status
```

```
Untracked files:
  app.js
```

**Untracked** means git can see the file but isn’t managing it. It’s in your working directory only.

```bash
git add app.js
git status
```

```
Changes to be committed:
  new file:   app.js
```

**What `git add` did to your files:** *nothing.* `app.js` is unchanged on disk. All that happened is git recorded a copy of its current contents in the staging area. This is worth internalising — **`git add` never modifies your working directory.** It’s a note-taking operation.

```bash
git commit -m "Add initial app file"
```

```
[main (root-commit) a1b2c3d] Add initial app file
 1 file changed, 1 insertion(+)
```

**What `git commit` did to your files:** also nothing. Your working directory is identical. What changed is that a permanent snapshot now exists in the repository, and the `main` pointer moved to it.

```
a1b2c3d
   ↑
 main, HEAD
```

That `a1b2c3d` is the commit hash — a unique fingerprint. You’ll use short forms of these constantly.

### The daily loop

```bash
# edit app.js in your editor
git status          # what changed?
git diff            # show me exactly what changed
git add app.js      # stage it
git commit -m "Add task list rendering"
```

**`git diff` with no arguments shows working directory vs staging area** — that is, changes you have *not* staged yet. This trips people up:

```bash
git diff              # unstaged changes
git diff --staged     # staged changes (what's actually going into the commit)
```

Run `git diff --staged` before every commit. It’s the code-review equivalent of hand-tracing before you run.

---

## Part 3 · Undoing things — the section that saves you

This is where people panic, and where the working-directory question matters most. **Read the table carefully — some of these destroy work permanently.**

| Command | Working directory | Staging | History | Recoverable? |
| --- | --- | --- | --- | --- |
| `git restore --staged <file>` | untouched | unstages | untouched | n/a |
| `git restore <file>` | **overwritten** | untouched | untouched | **NO** |
| `git commit --amend` | untouched | consumed | replaces last commit | yes (reflog) |
| `git reset --soft <commit>` | untouched | keeps changes | moves branch back | yes |
| `git reset --mixed <commit>` | untouched | cleared | moves branch back | yes |
| `git reset --hard <commit>` | **wiped** | wiped | moves branch back | commits yes, uncommitted **NO** |
| `git revert <commit>` | new changes | — | **adds** a new commit | yes |

### Walking through each one

**“I staged a file by mistake.”**

```bash
git restore --staged app.js
```

Removes it from the waiting room. **Your file on disk is untouched** — your edits are all still there, just no longer marked for the next commit.

**“I edited a file and want my changes gone.”**

```bash
git restore app.js
```

**This overwrites your file with the last committed version. Your edits are gone permanently.** Git never had them — they were only ever on disk. There is no undo. Think before running this one.

**“I committed but the message is wrong / I forgot a file.”**

```bash
git add forgotten-file.js
git commit --amend -m "Better message"
```

Replaces the last commit rather than adding a new one. Safe on your own machine. **Dangerous if you’ve already pushed** — see Part 8.

**“I want to undo my last commit but keep the work.”**

```bash
git reset --soft HEAD~1
```

`HEAD~1` means “one commit before where I am.” The branch pointer slides back one, and everything from that commit is sitting in your staging area, ready to be re-committed differently. Your files never changed.

```
before:   A ← B ← C          after:   A ← B          (C's changes now staged)
                  ↑                        ↑
                main                     main
```

**“I want to throw away the last commit entirely.”**

```bash
git reset --hard HEAD~1
```

**Branch pointer moves back, staging is cleared, and your working directory is overwritten to match.** Any uncommitted work is destroyed with no recovery.

The commit itself is technically still findable via `git reflog` for a few weeks, so *committed* work is recoverable. Uncommitted work is not.

**“I need to undo a commit that’s already pushed.”**

```bash
git revert a1b2c3d
```

This is the safe one. It doesn’t delete anything — it creates a **new commit** that reverses the changes:

```
A ← B ← C ← C'      where C' undoes C
```

**Rule of thumb: `reset` rewrites history, `revert` adds to it. Use `revert` for anything already shared.**

### The safety net nobody tells beginners about

```bash
git reflog
```

Git logs every position HEAD has occupied — including commits you “deleted” with `reset --hard`. If you destroy a commit and panic:

```bash
git reflog                    # find the hash of the lost commit
git reset --hard a1b2c3d      # go back to it
```

This works for about 30 days. **It only saves committed work.** Anything that was never committed is genuinely gone — which is the real argument for committing often.

---

## Part 4 · Branches — what they actually are

You now know a branch is a sticky note on a commit. Let’s watch one move.

You’re on `main` with three commits:

```
A ← B ← C
        ↑
      main, HEAD
```

```bash
git branch feature-login
```

**What changed on disk: nothing.** You wrote a second sticky note on the same commit:

```
A ← B ← C
        ↑
   main, feature-login
        ↑
      HEAD          (still on main)
```

```bash
git switch feature-login
```

**Now something does happen to your working directory.** HEAD moves, and git makes your files match that branch’s snapshot. Here both branches point at `C`, so nothing visibly changes — but that’s the mechanism:

```
A ← B ← C
        ↑
   main, feature-login
             ↑
           HEAD
```

Commit twice on the feature branch:

```
A ← B ← C ← D ← E
        ↑       ↑
      main   feature-login, HEAD
```

`main` hasn’t moved. Now switch back:

```bash
git switch main
```

**Your files change.** `D` and `E` vanish from your working directory — not deleted, just not part of `main`’s snapshot. Switch back to `feature-login` and they return. This is the moment git clicks for most people: **the files on disk are a rendering of whichever commit HEAD points at.**

```bash
git switch -c feature-signup    # create AND switch, the common shortcut
```

*(You’ll see `git checkout` in older material. `checkout` does too many unrelated things — switching branches, restoring files, detaching HEAD — which is why git split it into `switch` and `restore`. Use the new ones; recognise the old one.)*

### The one branch trap

If you have uncommitted changes and try to switch, git refuses when those changes would be overwritten. Either commit them, or stash them:

```bash
git stash              # put changes aside, clean the working directory
git switch main
# ... do something ...
git switch feature-login
git stash pop          # bring them back
```

**Stash is a temporary shelf.** `git stash` takes your uncommitted work off the working directory and stores it; `git stash pop` reapplies it and removes it from the shelf. Useful when a colleague needs you to look at `main` right now and you’re mid-thought.

---

## Part 5 · Merging — bringing branches back together

```
A ← B ← C ← F              main
          ↖ D ← E          feature-login
```

You want `feature-login`’s work in `main`. **You merge *into* the branch you’re standing on**, so switch there first:

```bash
git switch main
git merge feature-login
```

### Two things can happen

**Fast-forward.** If `main` hasn’t moved since the branch was created, there’s nothing to combine — git just slides the pointer:

```
before:  A ← B ← C ← D ← E        after:  A ← B ← C ← D ← E
                 ↑       ↑                                ↑
               main   feature                        main, feature
```

**Three-way merge.** If both branches have new commits, git creates a **merge commit** with two parents:

```
A ← B ← C ← F ← M          M has two parents: F and E
          ↖ D ← E ↗
```

### Merge conflicts, demystified

A conflict happens when **both branches changed the same lines of the same file.** Git can combine changes to different parts of a file automatically; it cannot decide which of two competing versions of line 12 you meant.

Your file gets rewritten with markers:

```jsx
function greet(name) {
<<<<<<< HEAD
  return `Hello,${name}!`;
=======
  return `Hi there,${name}`;
>>>>>>> feature-login
}
```

Reading it:
- Between `<<<<<<< HEAD` and `=======` — **what’s on your current branch**
- Between `=======` and `>>>>>>>` — **what’s on the branch you’re merging in**

**Resolving is manual and it’s just editing.** Delete the markers, leave the code you want — which might be either version, or a combination you write yourself:

```jsx
function greet(name) {
  return `Hello,${name}!`;
}
```

Then:

```bash
git add index.js       # "I've resolved this file"
git commit             # completes the merge
```

If you get lost mid-conflict: `git merge --abort` puts everything back the way it was.

**Interview answer:** *“A conflict means both branches edited the same lines, so git can’t decide automatically. It marks the file with both versions, I edit it into what I actually want, then `git add` to mark it resolved and commit to finish the merge.”*

---

## Part 6 · Rebase vs merge — the classic question

Both combine branches. They produce **different histories**, and that difference is the whole question.

Starting position:

```
A ← B ← C ← F            main
          ↖ D ← E        feature
```

**Merge** preserves what actually happened:

```
A ← B ← C ← F ← M        main
          ↖ D ← E ↗
```

**Rebase** rewrites your commits as if you’d started from the latest `main`:

```bash
git switch feature
git rebase main
```

```
A ← B ← C ← F ← D' ← E'      feature
```

Note `D'` and `E'` — those are **new commits with new hashes**. The originals are discarded. Git replayed your changes on top of `F`.

|  | Merge | Rebase |
| --- | --- | --- |
| History | Truthful, shows the branching | Linear, easy to read |
| Commit hashes | Preserved | **Rewritten** |
| Safe on shared branches | **Yes** | **No** |
| Extra commit | Yes, the merge commit | No |
| Conflicts | Resolved once | Possibly once *per commit* |

### The golden rule

> **Never rebase commits that other people have pulled.**
> 

Rebasing creates new commits and abandons the old ones. If a colleague’s machine has the originals, their history and yours now disagree, and merging them produces duplicated commits and confusion.

**Safe:** rebasing your own local feature branch onto the latest `main` before opening a pull request — it makes the branch apply cleanly and keeps history tidy.

**Unsafe:** rebasing `main`, or any branch someone else is working on.

**The one-line answer:** *“Merge preserves history and is safe on shared branches. Rebase rewrites history to keep it linear, which is great for cleaning up my own branch before a PR, but never on anything others have pulled.”*

### Cherry-pick — taking one commit, not a whole branch

Sometimes a branch contains several commits, but you only need **one specific change** on the branch you are currently working on. Merging would bring the whole branch with it; cherry-pick lets you take the change introduced by one commit and replay it here.

Suppose history looks like this:

```
A ← B ← C                 main
     ↖ D ← E              feature
```

Commit `E` contains a bug fix that `main` needs immediately, while `D` is unfinished feature work. From `main`:

```bash
git switch main
git cherry-pick <hash-of-E>
```

Git applies the change introduced by `E` to `main` and creates a **new commit**:

```
A ← B ← C ← E'            main
     ↖ D ← E              feature
```

`E'` has the same logical change as `E`, but it is a different commit with a different hash because it now has a different parent. The original feature branch is untouched.

Cherry-pick can conflict for the same reason a merge or rebase can: the change may no longer apply cleanly to the code at your current `HEAD`. Resolve the file, stage it, then continue:

```bash
git add <resolved-file>
git cherry-pick --continue
```

If you decide the operation was a mistake, `git cherry-pick --abort` restores the state from before the cherry-pick began.

**When to use it:** a hotfix, a small isolated change, or one commit that belongs on another branch. **When not to use it:** as a substitute for merging an entire feature branch; copying many related commits individually makes history harder to reason about.

**The interview version:** *“Cherry-pick applies the change from a specific commit onto my current branch and records it as a new commit. I use it when I need one isolated change without merging the whole source branch.”*

---

## Part 7 · GitHub — remotes, push, pull

Everything so far was local — no internet involved. GitHub is a **copy of your repository on someone else’s server**, which enables backup and collaboration.

**“origin” is just a nickname for a URL.** There’s nothing magic about the word:

```bash
git remote add origin https://github.com/you/todo-app.git
git remote -v          # show configured remotes
```

### Pushing

```bash
git push -u origin main
```

Uploads your commits and points GitHub’s `main` at the same place yours points. **What it does to your working directory: nothing.** Push is upload-only.

### What `-u` actually sets — upstream tracking

The first push of a new branch often includes `-u` (short for `--set-upstream`) because git needs to know which remote branch your local branch should normally compare and synchronize with:

```bash
git push -u origin feature-search
```

This does two related things: it pushes `feature-search` to the remote called `origin`, and it records that your local `feature-search` branch **tracks** `origin/feature-search`. After that relationship exists, plain `git push` and `git pull` know which remote branch you mean.

Keep two ideas separate:

- `origin/feature-search` is a **remote-tracking branch** — a local pointer representing where git last saw that branch on the remote.
- The **upstream** is the relationship saying “my current local branch normally corresponds to that remote-tracking branch.”

`git fetch origin` can move `origin/feature-search` when new remote commits exist, but it does **not** move your local `feature-search` branch or rewrite your working files. That separation is why fetching is safe: it updates your knowledge before you decide whether to merge or rebase.

Useful inspection commands:

```bash
git branch -vv        # show local branches and their upstreams
git status            # tells you whether you are ahead/behind the upstream
```

If you forgot `-u` on the first push, you can establish the relationship later with:

```bash
git push --set-upstream origin feature-search
```

**The interview version:** *“An upstream tells git which remote-tracking branch my local branch corresponds to. `git push -u origin branch` creates the remote branch and records that tracking relationship, so later `push` and `pull` don’t need the remote and branch names every time.”*

### Fetch vs pull — a real distinction

```bash
git fetch origin       # download, change nothing locally
git pull origin main   # download AND merge into your branch
```

**`git fetch` is safe and non-destructive.** It updates your knowledge of what’s on the server, stored as `origin/main` — a read-only pointer showing where the remote branch is. Your files don’t move.

**`git pull` is `fetch` + `merge`.** It downloads *and* immediately combines, which changes your working directory and can produce conflicts.

The cautious pattern, worth knowing because it makes you look like you’ve been burned before:

```bash
git fetch origin
git log HEAD..origin/main      # what's coming?
git merge origin/main          # now merge, knowing what's in it
```

**A pull config worth setting once:**

```bash
git config --global pull.rebase false    # pull = fetch + merge (default, safe)
```

The alternative, `pull.rebase true`, replays your local commits on top of the remote ones. Cleaner history, but it rewrites your commits — fine solo, surprising in a team.

### Clone

```bash
git clone https://github.com/someone/project.git
```

Downloads the entire repository — **all history, all branches**, not just the current files — sets up `origin` automatically, and checks out the default branch. One command, done.

---

## Part 8 · Force push, and why it’s feared

```bash
git push --force
```

Normally git rejects a push that would erase commits on the server. `--force` says “overwrite it with mine.”

**If a colleague had commits you didn’t, they’re now gone from the server.** This is how teams lose work.

If you genuinely need it — you rebased or amended a commit you’d already pushed on your own branch — use the safer variant:

```bash
git push --force-with-lease
```

This refuses if the remote has changed since you last fetched. It protects against exactly the case where someone else pushed while you weren’t looking.

**Never force-push a shared branch.** On your own feature branch, before anyone else has touched it, it’s routine.

---

## Part 9 · The pull request workflow

This is how actual teams work, and it’s what a “do you know GitHub” question is really asking.

```
1.  git switch -c feature/add-search        create a branch
2.  ... work, commit, commit ...
3.  git push -u origin feature/add-search   push the branch
4.  On GitHub: open a Pull Request           propose merging into main
5.  Teammates review, leave comments
6.  ... more commits addressing feedback ...
7.  Merge the PR on GitHub
8.  git switch main && git pull              bring the merge down locally
9.  git branch -d feature/add-search         delete the finished branch
```

**Why branch at all?** So `main` always stays deployable. Unfinished work lives on its own branch and cannot break anything.

**Why a PR rather than merging directly?** It’s a review checkpoint — a place for discussion, automated tests, and a record of *why* a change was made, not just what changed.

**Three merge buttons on GitHub, and the difference is asked:**

- **Merge commit** — preserves every commit plus adds a merge commit. Truthful, noisier.
- **Squash and merge** — collapses the whole branch into one commit on `main`. Clean history, loses intermediate steps. The most common choice for feature branches.
- **Rebase and merge** — replays commits onto `main` with no merge commit. Linear, and rewrites hashes.

---

## Part 10 · Files you should know about

### .gitignore

A list of things git should pretend not to see:

```
node_modules/
.env
dist/
*.log
.DS_Store
```

**Why each matters:** `node_modules` is enormous and reproducible from `package.json`. `.env` holds secrets and **must never be committed**. `dist/` is build output — regenerable. Logs are noise.

**The trap:** `.gitignore` only affects **untracked** files. If you already committed `.env`, adding it to `.gitignore` changes nothing — git is already tracking it. You need:

```bash
git rm --cached .env      # stop tracking, keep the file on disk
```

And be aware: **it’s still in history.** Anyone can read it from an old commit. If real credentials were pushed, the only correct response is to rotate them.

### README.md

The first thing anyone sees. For your projects it should carry what it does, the stack, how to run it locally, and environment variables needed. An interviewer who looks at your GitHub reads this first.

---

## Part 11 · Commands you’ll actually use

```bash
# looking around
git status                          # state of all three areas — run constantly
git log --oneline --graph --all     # visual history of every branch
git diff                            # unstaged changes
git diff --staged                   # what's about to be committed
git show a1b2c3d                    # what did this commit change?

# the daily loop
git add <file>                      # stage one file
git add .                           # stage everything changed
git commit -m "message"
git push

# branching
git branch                          # list branches
git switch -c new-branch            # create and switch
git switch main
git branch -d old-branch            # delete a merged branch

# combining
git merge <branch>
git rebase main
git merge --abort                   # bail out of a conflicted merge

# undoing (see Part 3 for what each destroys)
git restore --staged <file>
git restore <file>                  # DESTROYS your edits
git commit --amend
git reset --soft HEAD~1
git reset --hard HEAD~1             # DESTROYS uncommitted work
git revert <commit>                 # safe undo for pushed commits

# remotes
git clone <url>
git fetch origin
git pull
git push -u origin <branch>

# rescue
git reflog                          # every position HEAD has held
git stash / git stash pop
```

---

## Part 12 · Interview questions, with answers

**“Git vs GitHub?”**
Git is the version control tool that runs on your machine. GitHub is a hosting service for git repositories that adds collaboration on top — pull requests, issues, reviews, CI. Git works with no internet and no GitHub; GitHub without git is just file storage. GitLab and Bitbucket are alternatives.

**“What’s the staging area for?”**
It lets me choose what goes into a commit. My working directory usually has several unrelated changes; staging means each commit is one coherent change, which keeps history readable.

**“Merge vs rebase?”**
Merge preserves history and creates a merge commit — safe on shared branches. Rebase rewrites my commits onto a new base for linear history — good for tidying my own branch before a PR, never on anything others have pulled.

**“How do you resolve a conflict?”**
It means both branches changed the same lines. Git marks the file with both versions; I edit it into what I actually want, `git add` to mark it resolved, then commit to complete the merge. `git merge --abort` if I want out.

**“reset vs revert?”**`reset` moves the branch pointer backwards and rewrites history — fine locally, dangerous once pushed. `revert` creates a new commit that undoes the old one, so history is preserved. Anything already shared gets `revert`.

**“What does `git reset --hard` do?”**
Moves the branch pointer, clears staging, and overwrites the working directory. Uncommitted work is destroyed with no recovery. Committed work can still be found via `git reflog` for around 30 days.

**“fetch vs pull?”**`fetch` downloads and changes nothing locally. `pull` is fetch plus merge, so it modifies my working directory and can conflict. I fetch first when I want to see what’s coming before letting it in.

**“What’s `origin`?”**
A nickname for a remote repository’s URL — conventionally the one you cloned from. Nothing special about the name; you can have several remotes with any names.

**“Why is force push dangerous?”**
It overwrites the remote branch with mine. If someone else pushed commits I don’t have, those are gone. `--force-with-lease` is the safer form — it refuses if the remote changed since I last fetched.

**“How do you undo the last commit?”**
Depends. Keep the changes staged: `reset --soft HEAD~1`. Throw them away entirely: `reset --hard HEAD~1`. Already pushed: `revert`.

**“What is HEAD?”**
A pointer to where you currently are — normally the branch you have checked out. `HEAD~1` means one commit back.

**“Committed a secret. What now?”**
Remove it from tracking with `git rm --cached`, add it to `.gitignore`, and **rotate the credential** — it’s still in history and anyone with the repo can read it. Scrubbing history is possible with `filter-repo` or BFG, but rotating is the part that actually matters.

---

## Part 13 · Practising this properly

Reading git doesn’t work. Ninety minutes, in this order:

**1 · [Learn Git Branching](https://learngitbranching.js.org/)** — interactive, visual, and the single fastest way to internalise branches, merge and rebase. Do the Main and Remote sequences. Roughly 60 minutes and worth more than any article.

**2 · Break something on purpose.** In a throwaway repo: make a commit, `reset --hard` it, then recover it with `reflog`. Manufacture a conflict by editing the same line on two branches and resolve it. Doing each once removes the fear permanently.

**3 · Use it on your own projects.** Next feature you build, put it on a branch, push it, open a PR against your own `main`, and merge it. That’s the workflow, and having done it makes the interview answer real rather than recited.

Reference when stuck: [git-scm.com/docs](https://git-scm.com/docs) · [GitHub Docs](https://docs.github.com/)

---

## The five that matter most

If you retain nothing else:

1. **Three areas** — working directory, staging, repository — and that `add` and `commit` never change your files
2. **A branch is a pointer**, and switching branches re-renders your working directory
3. **`reset --hard` and `restore <file>` destroy uncommitted work.** Everything else is recoverable, often via `reflog`
4. **`revert` for anything pushed, `reset` only for local**
5. **Rebase for your own branch, merge for shared ones** — because rebase rewrites hashes