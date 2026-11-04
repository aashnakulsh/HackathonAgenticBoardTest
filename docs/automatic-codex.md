# Automatic Codex board: setup for all four teammates

Repository: https://github.com/aashnakulsh/HackathonAgenticBoardTest
Message board: https://github.com/aashnakulsh/HackathonAgenticBoardTest/issues/1

## What this does

Keep one terminal running. Every 30 seconds it checks for new board comments. A direct mention of your GitHub username by one of our four accounts starts a Codex CLI job. Codex reads AGENTS.md, handles the request, can edit/test/commit/push a task branch and open a PR, then the listener posts its final report. Aashna reviews and merges.

This runs separate CLI jobs, not your existing VS Code extension conversation. Repository files, AGENTS.md, PRs, local request reports and the board carry context between jobs. Your VS Code extension can still be used in another clone.

The four accounts are Aashna (aashnakulsh), Aditya (ADI-GHH), Chohan (yimchohan), and Orion (wthoreo). Each person uses their own account and computer. No shared token or API key is needed for the board; Codex needs its own working sign-in and usage allowance.

## A. Publish the listener once (Aditya and Aashna)

The prepared local changes are AGENTS.md, tools/board-auto.sh, and this guide. In Aditya's existing repository, run these commands ONE AT A TIME in Git Bash:

```bash
cd "/c/Users/ASUS PC/HackathonAgenticBoardTest"
git switch -c setup-codex-listener
git add AGENTS.md tools/board-auto.sh docs/automatic-codex.md
git commit -m "Add automatic Codex board listener and team guide"
git fetch origin
git rebase origin/main
git push -u origin setup-codex-listener
gh pr create --base main --head setup-codex-listener --title "Add automatic Codex board listener" --body "Adds a listener for trusted teammate mentions, automatic Codex jobs, and shared setup instructions. Keeps PR merging with Aashna."
```

Stop if any command fails. Aashna reviews and merges the PR on GitHub. Everyone follows section B AFTER it is merged. Do not start listeners from unpublished code in the working clone.

## B. Each teammate installs/checks prerequisites

Use Git Bash. Check:

```bash
git --version
gh --version
codex --version
```

If codex is missing, install current Node.js LTS from https://nodejs.org/, reopen Git Bash, then run:

```bash
npm install -g @openai/codex@latest
codex --version
```

If Codex is already installed but does not offer --approve-for-me in `codex exec --help`, update it through its original install method, or install/update the npm version above. Old examples using --full-auto are not current instructions.

Sign in separately to GitHub CLI and Codex:

```bash
gh auth status
gh api user --jq .login
codex login status
```

If needed:

```bash
gh auth login
codex login
```

Use your own accounts. GitHub's login output must match your username in the table above. Accept the repository invitation and ensure you have Write access. A public repository being readable does not establish push permission.

## C. Each teammate creates a dedicated automatic-work clone

Do not let the listener and a manually editing agent share a working folder. Run:

```bash
cd ~
gh repo clone aashnakulsh/HackathonAgenticBoardTest HackathonAgenticBoardAuto
cd HackathonAgenticBoardAuto
```

If this folder already exists, do not clone over it. Open that folder, finish any uncommitted work, and update main before continuing.

You can use `code .` to view this clone, but do not start another coding agent or manually edit it while the listener runs. Keep your ordinary editing in your original HackathonAgenticBoardTest clone.

Check your Git commit identity:

```bash
git var GIT_AUTHOR_IDENT
```

If it fails, set your own name and your GitHub-verified or GitHub no-reply email for this clone:

```bash
git config user.name "YOUR NAME"
git config user.email "YOUR GITHUB EMAIL"
```

Replace both placeholders; do not copy another teammate's identity.

## D. Check before starting

From HackathonAgenticBoardAuto, run:

```bash
bash tools/board-auto.sh --check
```

Expected: the correct teammate identity, the Message Board title, and checks passed. This does not run a model job or post a message. It verifies the tools, sign-ins, repository, and clean checkout. It does not establish that your account can use automatic approval review or push a PR; test those next.

If the script is missing, the setup PR is not merged or this clone needs updating. If it says there are uncommitted changes, stop and inspect them. Never discard work to make a check pass.

## E. Start the first rehearsal with a one-job limit

```bash
bash tools/board-auto.sh --max-jobs 1
```

Leave this terminal open. First launch deliberately skips all existing board comments. Wait for "Listening", THEN post a NEW test request. A teammate may post it in the board's GitHub browser page. You may also post a new request mentioning yourself; this listener permits own-account requests but ignores its own status reports.

Example for Aditya:

```text
[Aashna's Codex] TASK: @ADI-GHH Please add practice/ADI-GHH/hello.md with your name, GitHub username, frontend role and a greeting. Open a PR for Aashna. Do not merge it.
```

For other people, replace both the mention and path:

| Person | Mention | Practice path |
|---|---|---|
| Aashna | @aashnakulsh | practice/aashnakulsh/hello.md |
| Aditya | @ADI-GHH | practice/ADI-GHH/hello.md |
| Chohan | @yimchohan | practice/yimchohan/hello.md |
| Orion | @wthoreo | practice/wthoreo/hello.md |

Within roughly 30 seconds, the listener should start a Codex run. Watch the terminal on this first run. A passing rehearsal means: a task branch is pushed, a PR is opened, and a RESULT report with the PR URL appears on the board. If it says BLOCKED, read the reason; a report alone is not proof of completion.

The one-job listener exits afterward. Aashna reviews the PR. Do not run all four copies against the exact same shared file.

## F. Normal automatic operation

After the rehearsal passes, restart:

```bash
bash tools/board-auto.sh
```

The default limit is 10 processed mentions per launch. This includes information-only mentions that require a model decision. Review results before restarting. To deliberately choose a different limit, use `--max-jobs N` with a positive integer.

Each person keeps exactly one listener running. It continues polling between jobs, while the terminal stays open, the computer is awake, the network works and account access remains available. Closing the terminal or sleeping/shutting down the computer interrupts service. Stop with Ctrl+C. This does not install a startup service or run in the cloud.

## G. How agents cooperate

- Use one direct recipient and a precise deliverable: `[Name's Codex] TASK: @login ...`.
- Normal teammate comments mentioning your login also launch a job; TASK is a convention, not a mandatory parser field.
- Links to a task issue, PR, API contract and the source request make handoffs understandable.
- Ask the existing owner for dependencies. Only Aashna reassigns ownership.
- Agents may post a necessary dependency TASK for another owner, then report BLOCKED instead of waiting forever. The recipient's RESULT can wake the original requester to continue the pending work.
- RESULT and BLOCKED replies are interpreted for useful followup. If nothing needs doing, the agent outputs NO_REPLY and the listener publishes nothing. ACK/INFO status comments are ignored. This is a best-effort model decision; the per-launch job cap limits accidental conversations, and Ctrl+C stops them.
- @all alone does not trigger jobs. To assign work, mention the individual owner.
- The listener does not automatically merge PRs. To use another person's code, their PR normally needs Aashna's merge first. Announce the merge with a new direct mention describing which pending task can continue.
- Do not automatically send the same task to several agents. Each listener handles one request at a time.

## H. Troubleshooting and recovery

- "Missing codex/gh": install it in the same terminal environment. Reopen the terminal afterward.
- Codex sign-in, usage-limit or automatic-review error: fix account access. Do not bypass approvals or turn off isolation to hide the failure. Automatic review can deny a request, so unattended operation is not unconditional.
- A Windows setup or browser login prompt can still require your presence on the first run. Finish it while supervising the rehearsal.
- Missing message: confirm the listener was already started, the author is one of the four accounts, and your exact username is mentioned. Old messages and edited comments are not replayed. Post a NEW comment to retry.
- Your listener ignores ACK/INFO and @all-only messages. These are intentionally informational.
- A request is saved BEFORE launching Codex to avoid automatic duplicate execution. A failed or interrupted request is not automatically retried. Read its logs, check for an existing branch/PR, fix unfinished work, and post a fresh mention explaining where to resume.
- Logs and incoming requests are in `.board/auto-<login>/request-<comment-id>/`. `.board/` is ignored by Git. Review `run.log`, `message.txt` and `final.txt`; these may contain project data, so do not publish them wholesale.
- If Codex leaves uncommitted edits, the listener stops. Review and commit intended changes before restarting; never discard work blindly.
- If a lock remains after a crash, first ensure no listener or Codex child is still running. In VS Code Explorer, delete ONLY the empty `.board/auto-<your-lowercase-login>/running.lock` folder, then restart. Do not delete the cursor to retry: that would change the replay boundary.
- Updating the listener: stop it, finish the current work, switch to main, run `git pull --ff-only`, then restart. Never update while a job is editing files.

## Limits and permissions

The script filters authors and direct mentions. Folder ownership and task limits are instructions to Codex, not a hardened per-folder permission system. Automatic approval review and Codex's workspace sandbox still apply. Use this with the team's test project, keep secrets out, and review PRs.

No live model job or GitHub write was performed while preparing this script. Offline tests verify dispatch behavior, not your account's end-to-end operation. The rehearsal is required before calling it working on your computer.

Official references:
- https://learn.chatgpt.com/docs/non-interactive-mode
- https://learn.chatgpt.com/docs/developer-commands?surface=cli
- https://learn.chatgpt.com/docs/codex/cli
