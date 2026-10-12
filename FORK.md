# arosl/brenner_bot: the fork

This fork of
[Dicklesworthstone/brenner_bot](https://github.com/Dicklesworthstone/brenner_bot)
runs brenner's research cockpit on [herdr](https://herdr.dev) through
[nhm](https://github.com/arosl/nhm), in place of tmux and ntm. It is part of
the [ACFS fork](https://github.com/arosl/agentic_coding_flywheel_setup), which
replaces ntm with nhm (ACFS bead acfs-9hkh.28).

The fork follows upstream closely. It carries only the changes below, keeps
them small, and merges upstream regularly. It adds no features of its own.

## The delta

| File | Change |
|---|---|
| `brenner.ts`, `cockpit start` | runs `nhm spawn <thread> …` and `nhm send <thread> --all <msg>` instead of the same `ntm` commands; the dry-run plan and the two error messages say `nhm` |
| `brenner.test.ts` | one test: a dry run plans `nhm spawn RS-… --cc=1 --cod=1 --agy=1` and `nhm send RS-… --all <msg>` |
| `install.sh` | `REPO_OWNER="arosl"`: it installs the fork's release binary, which has the change above |
| `.github/workflows/notify-acfs.yml` | the job runs only in upstream's repository; the fork has no `ACFS_NOTIFY_TOKEN`, and the ACFS fork pins brenner by tag |
| `FORK.md` | this file |

## Releases

The fork tags `v<upstream version>-nhm.<n>` (for example `v0.4.1-nhm.1`) on
its `main`. Upstream's unchanged `release.yml` builds the binaries and their
`.sha256` files for the tag. The ACFS fork pins the tagged `install.sh` in
`checksums.yaml` and runs it with `--skip-ntm`, since ACFS installs nhm
itself.

The change is a rename. nhm takes ntm's grammar for these two calls, so
brenner's arguments stay as they are: `--ntm-args` (default
`--cc=1 --cod=1 --agy=1`) still passes straight to `spawn`. nhm records these
forms in its `compat/ntm-contract.md`.

These names stay, so brenner's interface does not change: the flags
`--skip-ntm` and `--ntm-args`, and the `ntm` key in `cockpit start --json`.

## Transport note

- **The session is a herdr workspace.** `nhm spawn <thread>` makes a herdr
  workspace labelled with the thread id, and one tab per agent. Each agent
  gets its Agent Mail identity first; its herdr name is that name in lower
  case. `nhm send <thread> --all` prompts every bound agent of that workspace
  except the caller.
- **nhm must own the herdr server.** Before ACFS's takeover
  (`nhm admin takeover`), ACFS's Bash owns the server, and both `nhm spawn`
  and `nhm send` refuse. Run the cockpit after the takeover. Before it, start
  the agents with `acfs agents spawn` and run the cockpit with `--skip-ntm
  --skip-broadcast`; the kickoffs still go through Agent Mail.
- **Kickoff.** Without `--prompt` in `--ntm-args`, `nhm spawn` sends ACFS's
  `default_new_agent` prompt, as ntm sends its configured default prompts.
  brenner's role kickoffs follow through Agent Mail as before.
- **Admission.** `nhm spawn` checks quota, capacity and the dcg guard before
  it starts an agent. Exit 3 means admission refused; `brenner` reports the
  failure with nhm's message.

## Not changed yet (known gaps)

- `brenner doctor` checks `ntm version`. On an ACFS host without ntm, use
  `brenner doctor --skip-ntm`.
- `install.sh` and `specs/toolchain.manifest.json` install ntm. ACFS installs
  nhm itself (`stack.nhm`), so ACFS runs brenner's installer with `--skip-ntm`.
- The docs and the web tutorial still name ntm and tmux.

Each gap gets a bead before it changes here, so the delta stays listed.

## Syncing with upstream

```bash
git fetch upstream
git merge upstream/main     # keep this file and the delta above; resolve by hand
bun test brenner.test.ts
```
