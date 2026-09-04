# Human Centered Robotics — Docker Environment

Course material and Docker development environment for **ASE 381P9 — Human Centered Robotics**, The University of Texas at Austin.

## Fall 2026

- **Instructor:** TBD
- **Student Helpers:** TBD

## Branches

Each offering of the course has its own branch:

- `Fall2025` — previous semester
- `Fall2026` — current semester

## Setup (students)

Everything runs in an x86_64 Ubuntu 18.04 container, so the pinned course
dependencies (Python 3.9.1, Pinocchio 2.5.5, PyBullet 3.0.8) work on both Apple
Silicon and Intel MacBooks.

The full step-by-step install guide lives at
[`.claude/skills/install-course-software/SKILL.md`](.claude/skills/install-course-software/SKILL.md).
Read it directly, or — if you use Claude Code — clone this repo, run `claude` in
the repo root, and ask:

> Install the course software on my MacBook.

The `install-course-software` skill will be picked up automatically and will walk
your coding agent through prerequisites, the image build, the shared `workspace/`
folder, and verifying the simulator and MeshCat.

Quick reference once you are set up:

```bash
docker build --platform linux/amd64 -t ubuntu18-conda-x11 .   # once
./run-container.sh                                            # first start
docker start -ai ubuntu18-container                           # later starts
```

MeshCat is published on the host at <http://localhost:7001/static/>.
