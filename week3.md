# Git and GitHub for Project Management and Development Traceability

## Git
*Open-source industry standard version control*

> Initialization
Initialize an empty Git repository with `$ git init` inside the project root folder. This adds a `.git/` folder to the project root. `.git/` contains all metadata and history Git needs to track and manage the project.
- Always run Git commands from the project root directory containing the `.git/` folder.
- Avoid nested Git folders.

> Configurations
Add your name and email with the following commands, allowing you to be identified as the author of your commits [^1].
- `$ git config user.name "<your-name>"`
- `$ git config user.email "<your-email>"`
- Add `--global` for global config

> `.gitignore`
Specifies intentionally untracked files to ignore on `.gitignore`:
- For files you don't intend to share with other developers
- Contains items like local developments, logs, temporary files, virtual environments, and compiled code, os/language specific files, files containing sensitive data, dependencies, etc. 
- Create a file `.gitignore` if you don't have one.
- If a file is already in the repository's index, and later included in `.gitignore`, to completely remove it, run `$ git rm --cached <file-name>`.
- See [gitignore](https://git-scm.com/docs/gitignore) for patterns.

| Pattern | Description |
|-----|--------|
| `**/old*` | Matches any file or folder starting with "old" at any depth. |
| `**/e2e_testrunner/` | Matches all `e2e_testrunner/` folders. |
| `**/welcome.py` | Matches files named `welcome.py`  at any depth. |
| `local_stuffs/` | Matches only the `local_stuffs/` folder in the repo root. |
| `**/venv/` | Ignores virtual environment folders at any depth. Similarly, you should also ignore `**/node_modules/` once you have them. |
| `.vscode/` | Matches `.vscode/` in the repo root. |
| `**/__pycache__/`, `.pytest_cache/` | Python cache folders, test caches. |
| `*.env` | Environment files. |

>`README.md`
- A README file usually contains essential project information, including the project title, description, installation steps, usage instructions, technologies used, contribution guidelines, and license details.
- For assignments, include branch information, dependencies, and instructions to run the project. [^3]

### Key Concepts
> Branch

Branches exist both local and remote
- The main/master branch is usually used for stable or default code. 
- Feature branches are used for developing individual features.
- Switch branches using `$ git switch <branch-name>` (previously `$ git checkout <branch-name>`).
- A *tracking branch* is a local branch linked to a remote branch.
- `$ git cherry-pick` can pick particular commits from other branches.

> Commit

Commits are snapshots of your project.
- Identified by hashes, the first few digits of a hash are usually enough to identify a commit.
- Add a message directly to the commit with `$ git commit -m "a short description of the commit"`.
- Add a custom message by using `$ git commit` which opens a `Vim` editor. You can use the `type(scope): description` convention, e.g., "implement(login): add server-side validation for user login".
- View commit logs with `$ git log`
- Use `$ git log --oneline --graph --decorate -–all` for a graph view.
- `$ git show <commit-hash>` shows the commit message, author/date, line-by-line diffs.

> Areas
- Working directory: the files you are currently editing. One working tree per repository, based on the `HEAD`.
- Staging area (index): files added with `$ git add`.
- Local repository: contains history of all committed snapshots; commit changes from index to local repository with `$ git commit`.
- Check status with `$ git status`.

>`HEAD`
- `HEAD` is a pointer to your current working commit, i.e., the commit your working directory and index are based on. 
- In one Git repository, there is always exactly one `HEAD` at any point in time.
- `HEAD` is a local reference, meaning it exists only within your individual repository and is not affected by changes on a remote, shared repository until you push or pull changes. 

> Push
- `$ git push` is used to upload committed changes from your local Git repository to a remote repository.
- The remote is commonly called `origin`.
- The remote branch associated with your local branch is called the *upstream branch* (used by `$ git pull`, `$ git push`, `$ git status`).
- On the local branch that you want to push and track, you can set the upstream branch for tracking with: `$ git push -u origin <branch-name>`.

> Diverging Branches
- Your branch is *diverged* when your local branch has commits the remote branch does not have, and the remote branch has commits your local branch does not have.
- Happens when you committed locally while someone else pushed to the same branch in the remote, or when pull requests are merged.
- Resolve conflicts with *merge* (preserves history): 
    * When you run `$ git merge <branch-name>`, Git attempts to automatically combine changes.
    * If both branches modified the same lines of code, Git pauses the merge and marks the files with conflict markers.
    * You can manually edit the conflicted files by keeping the changes you want (make sure to delete the markers `<<<<<<<`, `=======`, and `>>>>>>>`).
    * Stage the changes with `$ git add`; then complete the merge with `$ git commit`

> Refer to the [Git Cheat Sheet](https://git-scm.com/cheat-sheet).

## GitHub
*Enterprise hosted remote for collaboration*

> Github authentication

Configure GitHub authentication by adding a SSH Key
1. Generate a key pair with `$ ssh-keygen -t ed25519 -C <label>`, where `ed25519` is a modern secure encryption algorithm.
2. Copy your public key to your GitHub Account.
3. Possible error "fatal: Could not read from remote repository." 
    - Add GitHub to `known_hosts` with `$ ssh-keyscan github.com >> ~/.ssh/known_hosts` (appends GitHub's public SSH key to `known_hosts`).

> Cloning a repository

If you want to start based on existing content, you do a `$ git clone <url> <folder-name>` where `<folder-name>` is the local folder name.
- Clones the entire repository, including all branches and the full commit history.
- Checks out only one branch (`main` or `master`).
- The other branches exist as remote tracking branches, but not as local branches by default.
- `$ git checkout feature/login` creates a local branch `feature/login` tracking `origin/feature/login`.

> Add a remote
If you start your project locally, you can later add a remote with `$ git remote add <name> <url>`, where `<name>` is the alias you want to use for the remote, usually `origin`.
- You don't specify the branch when adding the remote.
- Specify the branch when pushing to the remote (and set upstream branch) with `$ git push -u <remote> <local-branch>:<remote-branch>`, where `-u` is the shorthand for `--upstream`.

> Pull Request (PR)
- A proposal to merge code changes into a project.
- A feature of GitHub.
- The industry standard: 
    * Work should be done in a separate branch, not directly in `main` or `master`. 
    * Once changes are pushed to the remote feature branch, you can propose a PR to merge the feature branch into the main branch. 
    * GitHub typically enforces reviews before merging. 
    * If there are conflicts, the PR cannot be merged until the conflicts are resolved and the fixes are pushed to the same remote feature branch, updating the existing PR.
    * After Developer A (`feature/login`) merges a PR into `main` on the remote, Developer B (`feature/register`) should fetch commits from `origin/main` and merge into `feature/register`. 
- Making changes while a PR is open
    * A PR tracks the branch as it evolves.
    * New commits pushed to that branch are automatically included in the PR.

## GitHub Projects
*Project management | linking commits to issues for traceability*

> Create a Project
- Manage the development process.
- Add issues and assign them to team members
- View progress in Board (Kanban), Table, and Roadmap (Gantt Chart) views.
- Get all issues using GitHub REST API and perform custom analysis [^2]

> User Story
- Represented as issues.
- Include priority (H, M, L), estimate (Fibonacci), assignee, release (as a milestone), iteration, start/end dates, etc.
- Add custom labels for iteration, feature, etc.

> Acceptance Cretiria 
- Represented as sub-issues.
- Each sub-issue has its own issue number.

## Development traceablity
- A PR should ideally contain one complete user story, including implementation and testing.
- Reference issues/sub-issues in test commits to link tests to user stories.
- Use the keyword `Closes #40` in the PR description to automatically close `Issue #40` when the PR is merged, other recognized keywords are `Fixes` and `Resolves`.
- You can also close relevant sub-issues with the keyword `Closes` in a PR's description.

<!-- footnotes -->
[^1]: Include the full official name and UTS email to clearly identify the author in assignment marking.

[^2]: This will be used to understand individual contributions in assignment marking.

[^3]: markers should be able to just follow the instruction and run your code