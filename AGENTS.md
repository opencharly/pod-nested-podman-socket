# AGENTS.md — pod-nested-podman-socket

Standalone candy repo for the `nested-podman-socket` candy — it serves a pod's
OWN rootless podman API socket at uid 1000, making the container's podman store
reachable as an API endpoint. The entire candy lives in `charly.yml` at the repo
root.

Canonical files:

- `charly.yml` — the `nested-podman-socket:` candy entity (description, `candy`,
  `volume`, `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-Calver history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:nested-podman-socket` — the owning skill: the socket, the
  load-bearing runtime-dir named volume, and why it is separate from
  `container-nesting`. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-distros:container-nesting` — the nested rootless podman recipe whose
  socket this candy serves.
- `/charly-build:load` — `charly box load`, the delivery verb this socket
  enables.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, services).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The candy's own `check:` steps are the R10 witness: they assert the seeded
  runtime dir is uid-1000-owned as a WHOLE, the `podman` binary is present, the
  socket exists at `/run/user/1000/podman/podman.sock`, and the socket reports a
  graph root inside the container's OWN rootless store. The end-to-end witness is
  the `check-boxload-pod` bed.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `nested-podman-socket:` candy entity in `charly.yml`; the `skill:`
  entity in the same file is the owning skill's source — a candy change and its
  skill change land together.
- The seed step MUST chown the WHOLE `/run/user/1000` directory, not just its
  `podman/` subdir: the nested podman writes its `libpod` runtime dir alongside,
  and seeding only the subdir leaves the service unable to start.
- The `runtime-dir` named volume at `/run/user/1000` is the mechanism that yields
  a uid-1000-owned runtime dir on a pod venue; do not replace it with a tmpfs
  (podman's `--tmpfs` rejects `uid=`/`gid=`).
- The `skill:` entity is the source for
  `/charly-distros:nested-podman-socket`; never edit the generated `SKILL.md` —
  regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
