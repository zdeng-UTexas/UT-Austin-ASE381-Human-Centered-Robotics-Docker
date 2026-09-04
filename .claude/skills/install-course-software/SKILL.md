---
name: install-course-software
description: Install and verify the ASE 381P9 Human Centered Robotics Docker environment on a student's MacBook (Apple Silicon or Intel). Use when the student asks to set up, install, build, or run the course software, the hw1 conda environment, the simulator, or MeshCat, or when they hit build/run errors with the container.
---

# Installing the Human Centered Robotics software on a MacBook

You are helping a student — assume they are new to Docker and may be new to the
terminal. Your job is to get them from "fresh MacBook" to "simulator running and
MeshCat visible in the browser."

UT students get Claude and several other AI assistants free through their
university account — see <https://tech.utexas.edu/services-tools/ai>. Sign in at
<https://claude.ai/> with `[your EID]@eid.utexas.edu` (not a personal address),
then use the same account when the `claude` CLI prompts for sign-in. If the
student mentions being logged into a personal account, out of usage, or unable to
sign in, point them at that page.

## How to work through this

- Run each phase in order. Each phase ends with a **Checkpoint** — run it and
  confirm the expected output before moving on. Do not skip ahead.
- Run the commands yourself via the terminal. Only hand a command to the student
  when it needs a GUI, a password, or a login (those are marked
  **student action**).
- Tell the student what you are doing in one plain sentence before each phase.
  No jargon dumps.
- **If a command fails, check the Troubleshooting table before retrying.** Do not
  run the same failing command more than twice. If it is not in the table and two
  attempts fail, stop and report: the command, the full error, and what you think
  is wrong. Guessing at Docker fixes wastes a lot of the student's time.
- Never delete or overwrite anything under `workspace/` — that is the student's
  homework. Confirm with them first.

## What is being installed

The course software targets old, pinned versions (Python 3.9.1, Pinocchio 2.5.5,
PyBullet 3.0.8) that no longer install cleanly on modern macOS. So everything
runs inside an **x86_64 Ubuntu 18.04 Docker container** instead.

| Piece | Where it lives |
| --- | --- |
| `Dockerfile` | Recipe for the container image |
| `environment.yml` | The `hw1` conda environment (installed into the image) |
| `run-container.sh` | Starts the container with the workspace mounted |
| `workspace/` | Shared folder — visible on the Mac *and* at `/root/workspace` in the container |

Homework code goes in `workspace/`. Edit it in a normal Mac editor; run it in the
container. Files written to `workspace/` survive the container being deleted.
**Anything written outside `/root/workspace` in the container is lost when the
container is removed.**

---

## Phase 1 — Check the machine

```bash
sw_vers -productVersion          # macOS version
uname -m                         # arm64 = Apple Silicon, x86_64 = Intel
df -h / | tail -1                # need ~15 GB free
```

Record which chip it is; several later steps branch on it.

**Apple Silicon note (say this to the student):** the container is x86_64, so it
runs under emulation. The image build takes roughly 20–45 minutes and the
simulator runs slower than native. That is expected, not a bug.

**Checkpoint:** you know the macOS version, the chip, and that there is ≥15 GB free.

---

## Phase 2 — Docker Desktop

Check whether it is already installed:

```bash
docker --version && docker info --format '{{.ServerVersion}}'
```

If `docker` is missing or `docker info` says it cannot connect to the daemon:

**Student action** — install and launch Docker Desktop:
- Download from https://www.docker.com/products/docker-desktop/ (choose the
  **Apple Silicon** or **Intel** build to match Phase 1), or run
  `brew install --cask docker` if they have Homebrew.
- Open Docker Desktop from Applications, accept the terms, and wait for the whale
  icon in the menu bar to stop animating.

Docker Desktop must be **running** for every later step. If the student reboots,
they must reopen it.

### Give Docker enough memory

The conda solve was OOM-killed at the default memory limit on some machines, which
is why the image uses Miniforge/mamba. Confirm there is headroom:

```bash
docker info --format 'Memory: {{.MemTotal}} bytes / CPUs: {{.NCPU}}'
```

If memory is under ~8 GiB (8000000000 bytes), **student action**: Docker Desktop →
Settings → Resources → raise **Memory** to 8 GB, click **Apply & restart**.

### Apple Silicon only — Rosetta

```bash
/usr/bin/pgrep -q oahd && echo "Rosetta installed" || echo "Rosetta MISSING"
```

If missing: `softwareupdate --install-rosetta --agree-to-license`

Then **student action**: Docker Desktop → Settings → General → enable
**"Use Rosetta for x86_64/amd64 emulation on Apple Silicon"** → Apply & restart.
This makes the build several times faster. (If the build later dies with an
illegal-instruction or bus error, turn this back off — see Troubleshooting.)

**Checkpoint:**

```bash
docker run --rm --platform linux/amd64 ubuntu:18.04 uname -m
```

Must print `x86_64`. If it prints `aarch64` or errors, emulation is not working —
stop and report.

---

## Phase 3 — Get the repository

If the student is not already inside the repo:

```bash
git clone <course repo URL> UT-Austin-ASE381-Human-Centered-Robotics-Docker
cd UT-Austin-ASE381-Human-Centered-Robotics-Docker
git branch -a
```

Check out the branch for the **current semester** (e.g. `Fall2026`), not `main`
and not a past semester:

```bash
git checkout Fall2026
```

Confirm the four setup files are present:

```bash
ls Dockerfile environment.yml run-container.sh workspace
```

**Checkpoint:** on the correct semester branch, all four exist.

---

## Phase 4 — Build the image

This is the long step. Tell the student the time estimate from Phase 1 and that
they should keep the laptop plugged in and awake.

```bash
docker build --platform linux/amd64 -t ubuntu18-conda-x11 .
```

Notes:
- Run it from the repo root (the directory holding the `Dockerfile`).
- The build is cached. Re-running after a successful build finishes in seconds.
  If only `environment.yml` changed, only the conda step re-runs.
- The slow part is `mamba env create`. Long silence there is normal.
- Run it in the foreground so the student sees progress; do not background it and
  poll.

**Checkpoint:**

```bash
docker images ubuntu18-conda-x11
```

The image should be listed, roughly 4–6 GB.

---

## Phase 5 — Put the homework in the workspace

Homework archives are distributed through Canvas, not this repo.

**Student action:** download the homework zip (e.g. `hw1_software.zip`) and move
it into the `workspace/` folder of the repo.

```bash
ls workspace/
unzip -n workspace/hw1_software.zip -d workspace/   # -n: never overwrite existing work
ls workspace/hw1_software
```

If `workspace/hw1_software` already exists with the student's edits in it, **do
not** unzip over it. Ask first.

**Checkpoint:** `workspace/hw1_software/simulator/pybullet/manipulator_main.py` exists.

---

## Phase 6 — Start the container

```bash
./run-container.sh
```

This mounts `workspace/` at `/root/workspace`, publishes MeshCat on host port
**7001**, and drops the student into a bash shell inside the container with the
`hw1` conda environment already active.

The script also calls `xhost` to set up X11 forwarding. **X11 is not required** —
the simulator runs headless (`p.DIRECT`) and visualizes through MeshCat in a
browser. If `xhost: command not found` appears, it is harmless; either ignore it
or install XQuartz (`brew install --cask xquartz`, then log out and back in).

If the script fails outright, use this equivalent without X11:

```bash
docker run -it --platform linux/amd64 \
  -v "$(pwd)/workspace":/root/workspace \
  -p 7001:7000 \
  --name ubuntu18-container \
  ubuntu18-conda-x11
```

### Reconnecting later

`run-container.sh` creates a container named `ubuntu18-container`. It only works
the first time. After that:

```bash
docker start -ai ubuntu18-container          # resume the same container
docker exec -it ubuntu18-container bash      # extra shell, while it is running
```

To throw it away and start clean (safe — `workspace/` is on the Mac, not in the
container):

```bash
docker rm -f ubuntu18-container
```

**Checkpoint** — inside the container:

```bash
echo $CONDA_DEFAULT_ENV                 # -> hw1
python -c "import pybullet, pinocchio, meshcat, scipy; print('imports ok')"
ls /root/workspace                      # -> shows hw1_software
```

---

## Phase 7 — Run the simulator

Inside the container:

```bash
cd /root/workspace/hw1_software
python simulator/pybullet/manipulator_main.py
```

Headless run — it prints simulation output and no window opens. That is correct.

Now the visualized version:

```bash
python simulator/pybullet/manipulator_main_with_meshcat.py
```

It prints a URL like `http://127.0.0.1:7000/static/`. **That URL is wrong for the
student's Mac** — port 7000 is inside the container. Tell them to open:

```
http://localhost:7001/static/
```

**Checkpoint:** the three-link manipulator is visible and moving in the browser.
Setup is done.

Close the sim with `Ctrl-C`, and leave the container with `exit`.

---

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `Cannot connect to the Docker daemon` | Docker Desktop not running | Student opens Docker Desktop, waits for the whale to settle |
| `docker: command not found` | Not installed, or PATH not picked up | Reinstall Docker Desktop; open a new terminal window |
| Build killed during `mamba env create`, or exit code 137 | Docker out of memory | Raise Docker Desktop memory to 8 GB (Phase 2) and rebuild |
| Build fails with illegal instruction / bus error / `qemu: uncaught signal` | Rosetta emulation edge case | Docker Desktop → Settings → General → **disable** "Use Rosetta…" → Apply & restart → rebuild (slower but more reliable) |
| `no matching manifest for linux/arm64` | Missing platform flag | Always pass `--platform linux/amd64` |
| Build very slow on Apple Silicon | Expected — x86 emulation | Confirm Rosetta is enabled; otherwise just wait |
| `The container name "/ubuntu18-container" is already in use` | Container already exists | `docker start -ai ubuntu18-container`, or `docker rm -f ubuntu18-container` to start clean |
| `xhost: command not found` | XQuartz not installed | Harmless — ignore, or use the no-X11 `docker run` in Phase 6 |
| `ifconfig: interface en0 does not exist` / blank IP in `run-container.sh` | Wi-Fi off, or Ethernet-only | Turn Wi-Fi on, or use the no-X11 `docker run` in Phase 6 |
| Browser shows nothing at `localhost:7001` | Wrong port, or sim not running | The sim must be actively running; use port **7001** on the Mac (not 7000), and the `/static/` path |
| `/root/workspace` is empty in the container | Started from the wrong directory | `run-container.sh` must be run from the repo root |
| Edits inside the container vanished | Written outside `/root/workspace` | Only `/root/workspace` persists |
| `ModuleNotFoundError` for pybullet/pinocchio | Wrong conda env | `conda activate hw1`; verify with `echo $CONDA_DEFAULT_ENV` |
| No space left on device | Disk full | `docker system prune -a` (removes unused images; the course image will need a rebuild) |

## Reporting back

When done, give the student a short summary: chip type, image name, container
name, the command to get back in (`docker start -ai ubuntu18-container`), and the
MeshCat URL (`http://localhost:7001/static/`). If anything is unresolved, say
exactly what and what you already tried.
