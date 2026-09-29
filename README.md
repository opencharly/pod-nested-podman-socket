# pod-nested-podman-socket

The `nested-podman-socket` candy of the OpenCharly candy library, as a standalone
repo (kind-prefixed naming). It serves a pod's OWN rootless podman API socket at
uid 1000, so the container's podman store is reachable as an API endpoint rather
than only as a CLI.

## What it provides

`container-nesting` supplies nested rootless podman; this candy supplies its
SOCKET. The two are separate because a nesting box that only builds and runs
containers needs no API endpoint, while anything that must be driven from outside
the container — the `charly box load` delivery verb, or an AgentTeams controller
spawning Manager/Worker containers into the pod's own store — needs exactly that
endpoint. Composing this candy is what makes a pod a venue `charly box load` can
deliver into. The socket's store IS the container's rootless store, which is the
whole point: containers spawned through it stay inside the candybox instead of
landing in the host store.

| Property | Value |
|---|---|
| Service | `podman-socket` (`podman system service --time=0 unix:///run/user/1000/podman/podman.sock`, `user: "1000"`) |
| Candy | `layer-container-nesting` |
| Volume | `runtime-dir` at `/run/user/1000` (named volume, copy-up seeded) |
| Env | `XDG_RUNTIME_DIR=/run/user/1000` |

The runtime-dir named volume is load-bearing: `/run` is a fresh root-owned tmpfs
at container start, so a build-time `mkdir /run/user/1000` does not survive, and
the uid-1000 service cannot create its own runtime dir under a root-owned
`/run/user`. The named volume's copy-up populates the empty volume from the
image's content, ownership preserved. A tmpfs is NOT an alternative — podman's
`--tmpfs` rejects `uid=`/`gid=`, so a tmpfs there can only ever be root-owned.

## How to use it

```bash
charly box validate
```

The candy's own `check:` steps assert the seeded runtime dir is uid-1000-owned as
a WHOLE (not just its `podman/` subdir), the `podman` binary is present, the
socket exists at the uid-1000 path, and — the discriminating assertion — the
socket reports a graph root inside the container's OWN rootless store
(`/home/user/.local/share/containers/storage`).

## Layout

- `charly.yml` — the `nested-podman-socket:` candy entity (description, `candy`,
  `volume`, `service`, `plan`) plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-Calver history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:nested-podman-socket` — the socket, the
  runtime-dir named volume, and why it is separate from `container-nesting`.
- `/charly-distros:container-nesting` — the nested rootless podman recipe this
  candy's socket serves.
- `/charly-build:load` — `charly box load`, the delivery verb this socket enables.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
