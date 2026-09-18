# Part 3 — Git + GitHub + CI/CD with GitHub Actions

> **Progress tracker**
> - [ ] 3.1 Setting up a new GitHub repo for DevOps
> - [ ] 3.2 Git workflow recap (what actually happens)
> - [ ] 3.3 "What happens when I git push?"
> - [ ] 3.4 CI vs CD — the concepts
> - [ ] 3.5 GitHub Actions — vocabulary
> - [ ] 3.6 Your first workflow — install and build
> - [ ] 3.7 Extend it — Docker build + push to Docker Hub
> - [ ] 3.8 Secrets in GitHub Actions
> - [ ] 3.9 Reading a failed workflow
> - [ ] 3.10 Interview checkpoint

---

## 3.1 — New GitHub Repo Setup

You said you already have this project on `smart-onboarding-system`. We are creating a **separate repo** for this DevOps learning project. This keeps your original repo clean.

### Step 1 — Create the repo on GitHub

1. Go to: https://github.com/new
2. Name it: `smart-onboarding-devops`
3. Keep it **public** (easier for CI/CD learning)
4. **Do NOT** tick "Add a README" — leave it empty
5. Click **Create repository**

GitHub will show you a page with setup instructions. Keep it open.

### Step 2 — Add a second remote to your local repo

Your local repo currently points to your original GitHub:

```powershell
git remote -v
# origin   https://github.com/KRITHIKSAI16/smart-onboarding-system.git
```

You still have one local copy of the code. We just add a second remote — a second destination to push to.

```powershell
git remote add devops https://github.com/KRITHIKSAI16/smart-onboarding-devops.git
```

**What this does:** Adds a named remote called `devops` pointing to your new repo. You now have two remotes:
- `origin` — original app repo
- `devops` — new DevOps learning repo

Verify:
```powershell
git remote -v
```

### Step 3 — Create and switch to a new branch

Don't work on `main` yet. Create a branch:

```powershell
git checkout -b devops/setup
```

**What this does:**
- Creates a new branch called `devops/setup`
- Switches you to it

### Step 4 — Stage your new DevOps files

```powershell
git status
```

You'll see `devops/` folder, `backend/Dockerfile`, `frontend/Dockerfile`, `frontend/nginx.conf`, `backend/.dockerignore`, `frontend/.dockerignore`, `docker-compose.yml`, `.gitignore` as untracked or modified.

Stage only the DevOps files (not any app code changes):

```powershell
git add devops/
git add backend/Dockerfile
git add backend/.dockerignore
git add frontend/Dockerfile
git add frontend/.dockerignore
git add frontend/nginx.conf
git add docker-compose.yml
git add .gitignore
```

### Step 5 — Commit

```powershell
git commit -m "devops: add Docker configuration and learning guides"
```

Good commit message format: `<type>: <what you did>`
Types: `feat`, `fix`, `devops`, `docs`, `chore`

### Step 6 — Push to the new remote

```powershell
git push devops devops/setup
```

Format: `git push <remote-name> <branch-name>`

Now go to `github.com/KRITHIKSAI16/smart-onboarding-devops` — your branch should be there.

### Step 7 — Create a Pull Request (PR)

On GitHub, click **"Compare & pull request"** for your `devops/setup` branch -> merge into `main`.

Create the PR, then merge it. Now `main` has your DevOps files.

---

## 3.2 — Git Workflow — What Actually Happens

Before CI/CD, understand the Git flow clearly.

```
Your Code (working directory)
      |
      |  git add
      v
Staging Area (index)
      |
      |  git commit
      v
Local Repository (.git folder)
      |
      |  git push
      v
Remote Repository (GitHub)
```

### The commands

```powershell
# See what's changed (working directory vs last commit)
git status

# See the actual diff
git diff

# Stage specific files
git add backend/Dockerfile

# Stage everything changed
git add .

# Commit staged changes with a message
git commit -m "your message"

# Push to remote
git push devops main

# Pull latest from remote (fetch + merge)
git pull devops main
```

### Branches

A branch is just a pointer to a commit. Main work happens on feature branches, then gets merged.

```powershell
git checkout -b feature/my-new-thing    # create and switch
git checkout main                        # switch back
git merge feature/my-new-thing           # merge into current branch
git branch -d feature/my-new-thing       # delete branch after merge
```

---

## 3.3 — What Happens When I `git push`?

This is the most important thing to understand for CI/CD.

```
You type: git push devops main
         |
         v
Git packages your commits as objects
         |
         v
Git opens HTTPS connection to github.com
         |
         v
GitHub authenticates you (token/SSH)
         |
         v
GitHub receives commits, updates the branch ref
         |
         v
GitHub fires a "push" event
         |
         v
GitHub Actions sees the event
         |
         v
GitHub spins up a runner (a fresh virtual machine)
         |
         v
Runner clones your repo
         |
         v
Runner executes your workflow steps
         |
         v
Results shown in GitHub -> Actions tab
```

**Key insight:** The runner is a fresh VM every time. It knows nothing. You must install everything it needs in your workflow steps.

---

## 3.4 — CI vs CD

### CI — Continuous Integration

Every time you push code, automatically:
1. Pull the new code
2. Install dependencies
3. Run tests / lint / build

**Goal:** Catch broken code immediately. Never let broken code sit undetected.

"Continuous" = happens automatically on every push, not once a week.

### CD — Continuous Delivery / Continuous Deployment

After CI passes, automatically:
1. Build a Docker image
2. Push the image to a registry
3. (Optionally) Deploy it to a server

**Delivery** = the image is ready and pushed; a human triggers deployment.
**Deployment** = deployment also happens automatically.

### The full picture

```
Developer pushes code
      |
      v
CI: Install -> Build -> Test      <- if this fails, stop. fix the code.
      |
      v
CD: Docker build -> Push image   <- if CI passes, image is ready
      |
      v
Deploy (manual trigger or automatic)
```

---

## 3.5 — GitHub Actions Vocabulary

| Term | What it is | Real-world analogy |
|---|---|---|
| **Workflow** | The entire automation file | A recipe |
| **Trigger (on:)** | What starts the workflow | The "start cooking" signal |
| **Job** | A group of steps that run on one machine | A cook's shift |
| **Step** | One individual action or command | One cooking task |
| **Runner** | The VM that executes jobs | The kitchen |
| **Action** | A pre-built, reusable step | A kitchen appliance |
| **Secret** | Encrypted env variable stored in GitHub | A combination lock |
| **Artifact** | File output from a workflow (e.g., a build) | The finished dish |

Workflows live in: `.github/workflows/<name>.yml`

---

## 3.6 — First Workflow — Install & Build

Create: `.github/workflows/ci.yml`

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, "devops/*"]
  pull_request:
    branches: [main]

jobs:
  backend-check:
    name: Backend — Install & Check
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: backend/package-lock.json

      - name: Install backend dependencies
        run: npm install
        working-directory: ./backend

      - name: Verify server.js exists
        run: ls -la server.js
        working-directory: ./backend

  frontend-check:
    name: Frontend — Install & Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install frontend dependencies
        run: npm install
        working-directory: ./frontend

      - name: Build frontend
        run: npm run build
        working-directory: ./frontend
        env:
          VITE_API_URL: http://localhost:5000/api
          VITE_SOCKET_URL: http://localhost:5000
```

---

### YAML explained — line by line

```yaml
name: CI Pipeline
```
The display name shown in GitHub's Actions tab.

```yaml
on:
  push:
    branches: [main, "devops/*"]
  pull_request:
    branches: [main]
```
**Trigger.** This workflow runs when:
- You push to `main` or any branch starting with `devops/`
- You open/update a Pull Request targeting `main`

```yaml
jobs:
  backend-check:
```
Defines a job named `backend-check`. Multiple jobs run in parallel by default.

```yaml
    runs-on: ubuntu-latest
```
The type of runner. GitHub provides free Ubuntu, Windows, macOS runners. Ubuntu is standard.

```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
```
A step using a **pre-built Action** (`uses`). `actions/checkout` clones your repo onto the runner. Without this, the runner has no code. `@v4` = version 4 of this action.

```yaml
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: backend/package-lock.json
```
Installs Node.js 18. `cache: "npm"` caches `node_modules` between runs using `package-lock.json` as the cache key. If `package-lock.json` hasn't changed, npm install uses cache — fast.

```yaml
      - name: Install backend dependencies
        run: npm install
        working-directory: ./backend
```
A `run` step executes a shell command. `working-directory` sets where the command runs (equivalent to `cd ./backend && npm install`).

```yaml
      - name: Build frontend
        run: npm run build
        working-directory: ./frontend
        env:
          VITE_API_URL: http://localhost:5000/api
```
The `env:` key sets environment variables for just this step. Vite needs `VITE_*` vars at build time.

---

## 3.7 — Extended Workflow — Docker Build + Push to Docker Hub

### Pre-requisite — Create a Docker Hub account

1. Go to: https://hub.docker.com
2. Sign up (free)
3. Create repositories named `onboarding-backend` and `onboarding-frontend` (public)
4. Go to: Account Settings -> Security -> **New Access Token**
5. Name it `github-actions`, set Read/Write/Delete permissions
6. **Copy the token — you will not see it again**

### Step 1 — Add secrets to GitHub

Go to: Your repo on GitHub -> Settings -> Secrets and variables -> Actions -> **New repository secret**

Add:
- `DOCKERHUB_USERNAME` = your Docker Hub username
- `DOCKERHUB_TOKEN` = the access token you just created

**What are secrets?** Encrypted values stored in GitHub. They are never visible in logs. In your workflow, you reference them as `${{ secrets.DOCKERHUB_USERNAME }}`. GitHub injects them at runtime on the runner.

**Never put credentials in your YAML file directly.** Anyone who can see your repo can see the YAML. Secrets are encrypted and masked in logs.

### Step 2 — Add the Docker job to `ci.yml`

Add this job AFTER the existing two jobs in `ci.yml`:

```yaml
  docker-build-push:
    name: Docker Build and Push
    runs-on: ubuntu-latest
    needs: [backend-check, frontend-check]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          file: ./backend/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/onboarding-backend:latest

      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          file: ./frontend/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/onboarding-frontend:latest
```

---

### Line-by-line: the Docker job

```yaml
    needs: [backend-check, frontend-check]
```
This job only runs if BOTH previous jobs passed. This is **job dependency**. Don't build a Docker image if the code is broken.

```yaml
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
```
Buildx is Docker's extended build toolkit. Supports multi-platform builds and build cache.

```yaml
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```
`${{ secrets.NAME }}` — GitHub Actions expression syntax. Injects the secret value at runtime. This logs the Docker CLI into Docker Hub on the runner.

```yaml
      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/onboarding-backend:latest
```
Builds the image from `./backend/Dockerfile` and pushes it to Docker Hub with the tag `yourusername/onboarding-backend:latest`.

---

## ✅ EXERCISE 6 — Push and Watch Your First CI Run

### Step 1 — Create the workflow file

```powershell
mkdir -Force .github\workflows
```

Create `.github/workflows/ci.yml` with the first workflow content from section 3.6.

### Step 2 — Commit and push

```powershell
git add .github/
git commit -m "ci: add CI pipeline for backend and frontend"
git push devops main
```

### Step 3 — Watch it run

Go to: `github.com/KRITHIKSAI16/smart-onboarding-devops/actions`

Click into the run. Watch each step execute in real time.

### Step 4 — Break it intentionally

Edit `ci.yml` — change the build command to:
```yaml
run: npm run buiild
```

Push again. Watch it fail. Read the error. Fix it. Push again. Watch it go green.

**This is what CI does — it catches problems automatically.**

---

## ✅ EXERCISE 7 — Full Pipeline: Docker Build + Push

### Steps

1. Create Docker Hub account and access token (see 3.7 above)
2. Add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` as GitHub secrets
3. Update `.github/workflows/ci.yml` with the `docker-build-push` job
4. Push to GitHub

```powershell
git add .github/
git commit -m "ci: add Docker build and push to Docker Hub"
git push devops main
```

5. Watch the Actions tab — three jobs: two in parallel, one waiting

### Step 6 — Verify on Docker Hub

Go to: `hub.docker.com/r/yourusername/onboarding-backend`

Your image should be there with the `latest` tag.

### Step 7 — Pull and run from Docker Hub

```powershell
docker pull yourusername/onboarding-backend:latest
docker run -d -p 5000:5000 --env-file ./backend/.env yourusername/onboarding-backend:latest
```

You just pulled an image built by GitHub's servers and ran it locally. This is the CI/CD loop.

---

## 3.8 — Reading a Failed Workflow

When CI fails:

1. Go to Actions tab
2. Click the failed run (red X)
3. Click the failed job
4. Expand the failed step
5. Read the error message

**Common failures:**

| Error | Cause |
|---|---|
| `npm ERR! missing script: build` | `package.json` has no `build` script |
| `Error: Cannot find module` | Missing dependency or wrong `working-directory` |
| `denied: access forbidden` | Wrong Docker Hub credentials in secrets |
| `COPY failed: file not found` | `.dockerignore` is excluding a needed file |
| `exit code 137` | Container ran out of memory on the runner |

**Always read the logs before asking for help.** The error message almost always tells you exactly what is wrong.

---

## 📝 Interview Checkpoint — Part 3

Answer without notes:

1. **What is CI?** Give a one-sentence definition.

2. **What is the difference between CI and CD?**

3. **What is a GitHub Actions runner?**

4. **What does `needs: [backend-check]` do in a workflow?**

5. **Why do we store Docker Hub credentials as GitHub Secrets instead of putting them in the YAML?**

6. **After `git push`, what is the sequence of events that leads to your Docker image being on Docker Hub?**

7. **The CI pipeline is green but the Docker push fails with "access forbidden". What do you check first?**

8. **A teammate asks: "Why do we need to `actions/checkout` in every job?" What do you say?**

---

## Status

> Complete all exercises, push to GitHub, watch your pipeline run green, verify the image on Docker Hub.
> Answer the checkpoint. Then move to Part 4 — AWS and Cloud Fundamentals.
