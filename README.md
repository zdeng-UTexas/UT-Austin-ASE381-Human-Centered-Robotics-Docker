# Human Centered Robotics — Docker Environment

Course material and Docker development environment for **ASE 381P9 — Human Centered Robotics**, The University of Texas at Austin.

## Fall 2026

- **Instructor:** TBD
- **Student Helpers:** TBD

## Branches

Each offering of the course has its own branch:

- `Fall2025` — previous semester
- `Fall2026` — current semester

## AI tools for students

UT Austin provides several AI assistants free to students, signed in with your
university account. See the full list and terms on UT's page:
<https://tech.utexas.edu/services-tools/ai>

For the coding-agent workflow described below, use **Claude**:

1. Go to <https://claude.ai/> and sign in with your UT EID email —
   `[your EID]@eid.utexas.edu` — not a personal address.
2. Install the Claude Code CLI (see <https://claude.com/product/claude-code>).
3. Run `claude` in the repo root and complete the browser sign-in prompt with the
   same UT account.

UT also provides ChatGPT, Google Gemini, Microsoft 365 Copilot Chat, and UT Sage
(an in-house AI tutor that integrates with Canvas); login details for each are on
the page above. Follow the course policy on AI use for graded work.

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
