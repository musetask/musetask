# Tasks for Muse

This repo is an async task channel between dilfish and Muse.
dilfish drops task files here; Muse polls the repo on a schedule, executes the tasks,
and writes result files back. No need to open the Muse app.

## Layout

- One directory per month: `2026-10`, `2026-11`, ...
- Task files: `NNtask-<slug>`, e.g. `2026-10/01task-fix-the-repo`
  - `NN` is a two-digit sequence number within the month (`01`, `02`, ...)
  - `<slug>` is a short lowercase hyphenated name
  - the file content is the task description, in any language
- Result files: `NNfinished-<slug>` in the same directory,
  e.g. `2026-10/01finished-fix-the-repo`
  - written by Muse only, after the task is done (or blocked)
  - records: when the task was picked up, what was done step by step,
    the result, and any files/commits produced

## Rules

1. Task files are write-once: dilfish creates them, Muse only reads them.
   Muse never edits or deletes a task file.
2. Result files are write-once: Muse creates `NNfinished-<slug>` exactly once
   per task. dilfish never edits result files; follow-ups go in new task files.
3. A task counts as "new" when no matching `NNfinished-<slug>` exists yet
   in the same directory.
4. Write task files atomically — create the file complete in one go, don't
   append to it later. Muse may start reading at any poll.
5. One task per file. Independent tasks go in separate files so they can be
   picked up and run in parallel.
6. If a task is unclear, or it needs dilfish's interactive approval (sending
   messages, purchases, logins, anything irreversible), Muse does NOT guess.
   It records the task as blocked in the result file, and dilfish follows up with
   a new task file.
7. No secrets in this repo. Credentials stay in Muse's secure storage;
   only public keys and non-sensitive config live here.
8. Polling: Muse pulls this repo on a schedule (cron job `musetask-poll`,
   every N minutes). Each run does `git pull`, processes new task files,
   commits the result files, and pushes.

## SSH access

- Muse connects as the `musetask` machine user via SSH through
  `ssh.github.com:443` (direct `github.com:22` is blocked from Muse's network).
- The public key named `musetask-bot` is registered on the musetask GitHub account.
- The private key lives in Muse's persistent home directory and is backed up
  (see `2026-10/02finished-store-the-sshkey` for locations).
