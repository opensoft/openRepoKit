# openRepoKit

Tools for creating, structuring, and working across repositories.

This is a Triad: openRepoKit-spec owns requirements and component contracts;
openRepoKit-code owns installation and integration tests and mounts the three
existing component repositories as submodules. The components retain their
identities, histories and ownership. Tools' own nested Shape dependency can
differ from Kit's selected runtime Shape version.

```text
openRepoKit/
  spec/contracts/components.json       selected immutable revisions
  code/openRepoKit                     installer and verifier
  code/components/openRepoShape/       structure and validation
  code/components/openRepoTools/       park/resume, lanes and workspace tools
  code/components/openRepoProject/     project coordination and reporting
```

## Install the bundle

Requires Python 3.10+, Git, Bash and PyYAML. From this assembly checkout:

```sh
make validate
python3 code/openRepoKit verify
python3 -m unittest discover -s code/tests -v
python3 code/openRepoKit install --workbenches ../workBenches
export PATH="$HOME/.local/openRepoKit/bin:$PATH"
openRepoKit status
project --help
```

Use `--prefix /absolute/path` for another destination. Commands live in its
`bin/`; versioned source snapshots live in `releases/`. This separate prefix
allows workBenches to continue managing its existing commands. Kit places its
own release first on child processes' PATH. Project therefore uses bundled
Shape; Shape uses its bundled setup script. Inherited source/ref variables
cannot switch revisions. Upstream refusals and exit statuses are preserved.

Tools' own installer prepares Claude skills and hooks inside the prefix by
default. Add `--with-claude-integration` on a fresh-prefix installation to
activate them in user configuration through upstream's merge logic. Those
configuration writes are not transactional with command publication.
Installation never installs prerequisites, authenticates, creates a project
or initializes a workspace repository.

Project's `--workbenches` and `WORKBENCHES_ROOT` override the optionally saved
checkout path. Both openBenches and workBenches remain external; their
generators are not pinned by Kit. The saved release-manifest field, launcher
export, `.workbenches-path` marker, and installed artifacts keep their existing
`workbenches`/`WORKBENCHES_ROOT` names during the compatibility window.

Without an explicit, saved, or marked selection, launchers validate
`config/bench-config.json` and try `~/projects/openBenches`,
`~/Projects/openBenches`, `~/projects/workBenches`, then
`~/Projects/workBenches`. `PROJECTS_DIR` replaces the two home conventions and
tries `openBenches` before `workBenches` there. Invalid automatic candidates
are skipped; explicit and saved paths retain their existing validation and are
not silently replaced. Existing checkouts are not moved or merged.

Component self-installation refuses through Kit launchers. Install updates
from the assembly checkout. Repeating an unchanged installation is a no-op;
older releases remain available. Command links publish individually; do not
run concurrent installations into the same prefix.

Component gitlinks and `spec/contracts/components.json` must agree. Land both
legs before advancing assembly pins with openRepoShape's `bump-leg.py`.
See `spec/specs/001-kit-distribution/spec.md` for the acceptance contract.
Kit's suite checks composition; it does not replace or claim execution of
all upstream component suites.

## Public repositories and license

The assembly and both legs are public repositories under `opensoft`.
All three use Apache-2.0, matching the existing components. See [LICENSE](LICENSE).

The ASSEMBLY ROOT of the `openrepokit` project — the repository you clone.
It holds no product code of its own: it holds the manifest that says what this
project IS, the two legs as submodules, and the pins that say which commit of
each leg this project is.

Scaffolded from [opensoft/openRepoShape](https://github.com/opensoft/openRepoShape)
at `54aa69a370f319b86e1b5f304b5bb373b8b53747`. Elected by Brett Heap on 2026-10-11, against
`openxFactory docs/project-repo-schema.md`.

## Get started

```sh
git clone --recurse-submodules https://github.com/opensoft/openRepoKit.git
cd openRepoKit
make bootstrap
```

`make bootstrap` puts each leg on the `main` branch **at its
pinned commit** (so you are not staring at a detached HEAD), runs the three
neutral validators, and prints whatever review authority a wallet register
names for this project — or says plainly that authority is not wallet-carried
here, and continues.

### Parking work for another workstation

Unfinished work moves between machines as a RECORD, not as a folder:

```sh
make park                  # before you leave this workstation
# on the next one:
git clone --recurse-submodules https://github.com/opensoft/openRepoKit.git
cd openRepoKit && make bootstrap && make resume
```

`make park` commits and pushes every open feature worktree and records what
it parked; `make resume` recreates those worktrees here and un-commits the
work, leaving it in your tree as you left it. `ARGS=--dry-run` rehearses
either one and writes nothing. The mechanics are the Speckit git extension's
— `setup-openspeckit` installs it, and both targets refuse by name when it is
missing — and `resume` REFUSES a feature whose branch has moved since the
park rather than resetting over somebody else's commit. Nothing under
`worktrees/` is ever committed, pushed or synced, on either side.

## The three legs

| role | repository | path | holds |
|---|---|---|---|
| assembly | `opensoft/openRepoKit` | `.` | this manifest, the pins, the gate |
| spec | `opensoft/openRepoKit-spec` | `spec/` | requirements, decisions, acceptance |
| code | `opensoft/openRepoKit-code` | `code/` | the implementation and its tests |

All three carry the GitHub topic `xf-project-openrepokit`, so the organisation's own search
surfaces the group without a checkout.

## Reading private legs in CI: a GitHub App first, `SHAPE_LEGS_TOKEN` as fallback

If `opensoft/openRepoKit-spec` or `opensoft/openRepoKit-code` is **private or internal**,
the `validate` workflow's default `GITHUB_TOKEN` cannot clone it as a
submodule.

Preferred: a dedicated GitHub App — permissions **Contents: Read-only** and
**Metadata: Read**, installed on this organisation with access to the legs —
mints a short-lived token at run time:

```sh
gh secret set SHAPE_LEGS_APP_ID --org <your-org> --body '<app id>'
gh secret set SHAPE_LEGS_APP_PRIVATE_KEY --org <your-org> < app-private-key.pem
```

Fallback: a fine-grained **`SHAPE_LEGS_TOKEN`** PAT, `contents:read` on the
LEGS ONLY:

```sh
gh secret set SHAPE_LEGS_TOKEN --org <your-org> --body '<token>'
```

**On the GitHub Free plan, set these as REPOSITORY secrets.** GitHub delivers
an ORGANISATION secret only to PUBLIC repositories on Free, so on a private
repository `secrets.SHAPE_LEGS_APP_ID` is the empty string — silently — the
App steps skip, and `validate` goes GREEN with the lockstep pin check degraded
away rather than red. Use `--repo <org>/<Repo>` in place of `--org <your-org>`
above, or upgrade the organisation to Team. (Measured on InkRouter,
2026-09-04.)

The root repository itself is always readable by the workflow's own default
token, so `actions/checkout` never carries a `token:` override — putting a
legs-scoped credential there instead is what broke the ROOT checkout with a
403 the first time a PAT was tried for real. Whichever credential resolves is
read only inside the guarded "fetch the legs (submodules)" step, scoped to
that step's `env:`, and used through a `git -c url.<...>.insteadOf=<...>`
rewrite covering both HTTPS and SSH leg URLs — it never touches the root
checkout.

`validate` tries the App first — a `mint a leg-reader token from the GitHub
App` step, scoped by `repositories:` to the legs this organisation itself
owns (a leg under a different owner is excluded with a warning, since an
installation token is per-owner) — and falls back to `SHAPE_LEGS_TOKEN` when
the App is not configured. A configured App that fails to mint fails the job
outright, naming both secrets and the required installation, rather than
degrading.

Without either credential the workflow does not go red on that account: it
checks out the root without submodules, tries `git submodule update --init
--recursive` best-effort, and — if that fails — still runs the naming and
manifest checks, skips `validate-pins.py` with a warning explaining why, and
only fails outright if a credential (App or PAT) **is** configured and the
fetch still failed — naming which source it used. That presence check reads
job-level `env: SHAPE_LEGS_APP_SET` / `SHAPE_LEGS_TOKEN_SET` booleans rather
than the `secrets` context directly in the step's `if:` — the `secrets`
context is not allowed in a step-level `if:` expression, and using it there
makes GitHub reject the whole workflow file instead of just that step.

## The lockstep invariant

For each leg, THREE things name the same commit and they move in ONE commit:

1. the **gitlink** — the `160000` entry recorded at the leg's path
2. **`commit:`** in `contracts/<role>-pin.yaml`
3. every **`.github/workflows/*.yml` `@<sha>`** reference naming that leg

`python3 scripts/validate-pins.py` (also `make pins`, also the `validate` check
on every pull request) refuses if they disagree, and recomputes the leg's tree
digest on top. Advancing a pin is therefore one commit that touches the
submodule, the pin file, and any workflow ref — never a bare `git submodule
update` followed by a commit.

This is written down because the family learned it the expensive way: seven
consecutive pin-syncs in the xFactory aggregation moved the gitlink alone and
left `validate` red on every pull request for a day, unnoticed because the
check runs on pull requests only.

## What the election confers

Nothing. Electing this shape changes no gate, no floor, no grant and no
clearance eligibility; a one-repository project is reviewed identically,
because the authority travels in the grants rather than in the layout. The
`role:` fields in `project.yaml` are navigation. A tool that reads `role: spec`
as "spec authority lives here" has quietly turned a layout into a governance
boundary, and is defective.

## Layout

```
AGENTS-shape.md                  the RULES OF THE SHAPE, for an agent (copied)
AGENTS.md                        this project's own instructions (yours)
CLAUDE.md                        one line, pointing at AGENTS.md
project.yaml                     the manifest — the SOURCE of this group
contracts/repository-naming.yaml the six naming families (copied from the shape)
contracts/spec-pin.yaml          the spec leg's commit + tree digest
contracts/code-pin.yaml          the code leg's commit + tree digest
contracts/shape-pin.yaml         the openRepoShape revision + per-file digests
scripts/bootstrap.py             the one command after a recursive clone
scripts/validate-manifest.py     project.yaml, and the legs' names
scripts/validate-pins.py         THE LOCKSTEP VALIDATOR
scripts/validate-repository-naming.py
scripts/repo_shape.py            shared helpers, standard library only
.github/workflows/validate.yml   the neutral gate, on pull_request
.gitattributes                   LF in every worktree, whatever core.autocrlf says
```

Everything under `scripts/`, plus `contracts/repository-naming.yaml` and
`AGENTS-shape.md`, is a COPY from `opensoft/openRepoShape`, digest-pinned in
`contracts/shape-pin.yaml`. Edit them upstream, not here — a local edit is
reported as drift. `AGENTS.md` and `CLAUDE.md` have no row and are this
project's own.
