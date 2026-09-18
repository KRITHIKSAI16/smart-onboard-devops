# Part 2 — Dockerizing the MERN Application

> **Progress tracker**
> - [ ] 2.1 What we are building
> - [ ] 2.2 Backend Dockerfile — line by line
> - [ ] 2.3 Backend .dockerignore
> - [ ] 2.4 Build & run the backend image
> - [ ] 2.5 Frontend Dockerfile — line by line
> - [ ] 2.6 Frontend .dockerignore
> - [ ] 2.7 Build & run the frontend image
> - [ ] 2.8 Docker Compose — bring it all together
> - [ ] 2.9 Debugging scenarios
> - [ ] 2.10 Interview checkpoint
> - [x] 2.11 Real errors encountered (done — read this after completing the section)

---

## 2.1 — What We Are Building

Your app today (without Docker):
`
Your machine
  ├── node (globally installed)
  ├── backend/  → npm start  → localhost:5000
  ├── frontend/ → npm run dev → localhost:5173
  └── MongoDB Atlas (cloud)
`

After this part:
`
Docker
  ├── backend container  → port 5000
  ├── frontend container → port 80 (served by Nginx)
  └── mongo container    → port 27017 (local, for dev)

All on the same Docker network. Docker Compose starts all three with one command.
`

**Rule:** We are only ADDING files to the project (Dockerfile, .dockerignore, docker-compose.yml). Zero application code changes.

---

## 2.2 — Backend Dockerfile

Create this file at: ackend/Dockerfile

`dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
`

Now let's understand **every single line**.

---

### FROM node:18-alpine

`
FROM <image>:<tag>
`

**What it does:** Sets the base image — the starting point for your image.


ode:18-alpine means:
- 
ode = official Node.js image from Docker Hub
- 18 = Node.js version 18 (LTS)
- lpine = built on Alpine Linux — a tiny ~5MB Linux distro

**Why alpine?**
A full 
ode:18 image is ~900MB. 
ode:18-alpine is ~50MB. Same Node.js, much smaller image. Smaller images = faster downloads, faster CI, less attack surface.

**Always pin a version.** FROM node:latest is dangerous — it changes without warning and can break your build.

---

### WORKDIR /app

`
WORKDIR <path>
`

**What it does:** Sets the working directory inside the container. All following commands run from this path.

Think of it as cd /app — but it also creates /app if it doesn't exist.

Without WORKDIR, everything would run from the root / of the container. That's messy and risky.

---

### COPY package*.json ./

`
COPY <source-on-your-machine> <destination-in-container>
`

**What it does:** Copies package.json and package-lock.json from your machine into /app/ inside the container.

package*.json is a glob — it matches both package.json and package-lock.json.

**Why copy these BEFORE the rest of the code?** This is the layer caching trick.

`
Layer 1: FROM node:18-alpine         ← almost never changes
Layer 2: WORKDIR /app                ← never changes
Layer 3: COPY package*.json          ← only changes when you add/remove packages
Layer 4: RUN npm install             ← only re-runs if layer 3 changed
Layer 5: COPY . .                    ← changes every time you edit code
Layer 6: CMD                         ← never changes
`

If you change a .js file, Docker rebuilds from Layer 5 only. Layers 1-4 come from cache. npm install doesn't re-run. **Fast.**

If you wrote COPY . . first and then RUN npm install, every code change would re-run npm install. **Slow.**

---

### RUN npm install --omit=dev

`
RUN <command>
`

**What it does:** Runs a command during the image build. The result becomes a new layer.

--omit=dev = skip devDependencies (like nodemon). We don't need them in production — keeps the image smaller.

Note: RUN is for BUILD time. CMD is for RUN time.

---

### COPY . .

Copies everything else from your local ackend/ folder into /app/ in the container.

**This is why .dockerignore exists** — we don't want to copy 
ode_modules (huge), .env (secrets), etc. We'll create that next.

---

### EXPOSE 5000

`
EXPOSE <port>
`

**What it does:** Documents that the container listens on port 5000.

**Important:** This does NOT actually open the port. It's documentation — telling Docker (and humans) what port the app uses. The actual port mapping happens at docker run -p.

Think of it as a label: "this container expects to use port 5000."

---

### CMD ["node", "server.js"]

`
CMD ["executable", "arg1", "arg2"]
`

**What it does:** The default command that runs when the container starts.

Use the **array (exec) form** ["node", "server.js"], not the string form "node server.js". The exec form runs the process directly. The string form runs via a shell, which causes signal handling issues (your app won't receive CTRL+C / stop signals cleanly).

CMD can be overridden at docker run time. ENTRYPOINT cannot. For a simple app, CMD is fine.

---

## 2.3 — Backend .dockerignore

Create this file at: ackend/.dockerignore

`
node_modules
.env
.env.*
npm-debug.log
*.log
`

**What it does:** Like .gitignore but for Docker. Tells COPY . . what to skip.

**Why each line:**
- 
ode_modules — huge folder. We run 
pm install inside the container anyway. Never copy from host.
- .env / .env.* — NEVER bake secrets into an image. Anyone who pulls your image could read them. Pass env vars at runtime.
- *.log — no need for log files in the image.

---

## ✅ EXERCISE 3 — Build & Run the Backend Image

### Step 1 — Create the files

Create ackend/Dockerfile and ackend/.dockerignore with the content above.

### Step 2 — Build the image

Run this from the project root:

`powershell
docker build -t onboarding-backend:dev ./backend
`

**Flags:**
- -t onboarding-backend:dev — tag (name) the image. Format: 
ame:tag
- ./backend — the build context. Docker sends this folder to the Docker engine.

Watch the output. You'll see each step (FROM, WORKDIR, COPY, RUN, COPY, EXPOSE, CMD) execute and be assigned a layer hash.

### Step 3 — Verify

`powershell
docker images
`

You should see onboarding-backend with tag dev.

### Step 4 — Run it

`powershell
docker run -d 
  --name backend-test 
  -p 5000:5000 
  --env-file ./backend/.env 
  onboarding-backend:dev
`

**Backtick `  ` in PowerShell = line continuation (same as \ in bash).**

### Step 5 — Verify it's running

`powershell
docker ps
docker logs backend-test
`

You should see: Server running on port 5000

Open: http://localhost:5000 — you should see Smart Onboarding API Running...

### Step 6 — Stop and remove

`powershell
docker stop backend-test
docker rm backend-test
`

### Step 7 — Rebuild speed test

Make no changes. Run docker build again:

`powershell
docker build -t onboarding-backend:dev ./backend
`

Watch the output — every step should say CACHED. That's layer caching working.

Now change one character in server.js (like a comment), rebuild, and watch which layers rebuild and which are cached.

---

## 2.4 — Frontend Dockerfile

The frontend is a **React + Vite** app. It compiles to static HTML/CSS/JS in the dist/ folder.

For Docker, we use a **multi-stage build**:
- Stage 1: Node.js builds the app (
pm run build → generates dist/)
- Stage 2: Nginx serves the built static files

Create at: rontend/Dockerfile

`dockerfile
# ─── Stage 1: Build ───────────────────────────────────────────
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

# ─── Stage 2: Serve ───────────────────────────────────────────
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
`

---

### Understanding multi-stage builds

`
Stage 1 (builder) — Node.js image (~50MB)
  COPY source code
  RUN npm install       ← installs all 200MB of node_modules
  RUN npm run build     ← produces dist/ folder (~2MB of optimized JS/CSS)

Stage 2 — Nginx image (~8MB)
  COPY --from=builder /app/dist  ← only copy the tiny built output
  (node_modules are LEFT BEHIND — they never enter the final image)

Final image size: ~12MB instead of 250MB+
`

**COPY --from=builder** — copies files FROM a previous build stage. This is the core of multi-stage builds.

**CMD ["nginx", "-g", "daemon off;"]** — starts Nginx in the foreground. Docker requires the process to stay in the foreground. daemon off prevents Nginx from going to background (which would make Docker think it crashed).

---

### Nginx config for React Router

React Router uses client-side routing. When you visit /dashboard directly, the server looks for a file at /dashboard — which doesn't exist. Nginx returns 404.

Create: rontend/nginx.conf

`
ginx
server {
    listen 80;

    root /usr/share/nginx/html;
    index index.html;

    # All routes → index.html (let React Router handle it)
    location / {
        try_files  / /index.html;
    }
}
`

	ry_files  / /index.html — try the file, then a directory, then fall back to index.html. React Router takes over from there.

---

## 2.5 — Frontend .dockerignore

Create: rontend/.dockerignore

`
node_modules
dist
.env
.env.*
npm-debug.log
*.log
`

- 
ode_modules — don't copy, rebuild inside container
- dist — Stage 1 rebuilds this fresh; don't copy old build artefacts
- .env — secrets, never in image

---

## ✅ EXERCISE 4 — Build & Run the Frontend Image

### Step 1 — Create the files

Create rontend/Dockerfile, rontend/.dockerignore, and rontend/nginx.conf with the content above.

### Step 2 — Build the image

**Important:** The frontend talks to the backend. During build (
pm run build), Vite embeds the API URL from .env. For local Docker testing, the backend will be at http://localhost:5000.

Create a temporary env file for Docker local builds: rontend/.env.docker

`
VITE_API_URL=http://localhost:5000/api
VITE_SOCKET_URL=http://localhost:5000
`

Build:

`powershell
docker build -t onboarding-frontend:dev ./frontend
`

Watch Stage 1 (Node build) and Stage 2 (Nginx) execute separately.

### Step 3 — Verify image size

`powershell
docker images
`

Compare onboarding-frontend vs onboarding-backend sizes. Note how small the frontend image is — only Nginx + built static files.

### Step 4 — Run it

`powershell
docker run -d -p 3000:80 --name frontend-test onboarding-frontend:dev
`

Note: -p 3000:80 — Nginx serves on port 80 inside the container, we access it on 3000 from the browser.

Open: http://localhost:3000

You should see your React app. (API calls may fail since the backend isn't running — that's fine for now.)

### Step 5 — Clean up

`powershell
docker stop frontend-test && docker rm frontend-test
`

---

## 2.6 — Docker Compose

Running containers individually with docker run flags is tedious. **Docker Compose** lets you define your entire application stack in one YAML file and start everything with one command.

Create at the project root: docker-compose.yml

`yaml
version: "3.9"

services:

  mongo:
    image: mongo:7
    container_name: mongo
    restart: unless-stopped
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - onboarding-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend
    restart: unless-stopped
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    environment:
      - MONGO_URI=mongodb://mongo:27017/onboardingDB
    depends_on:
      - mongo
    networks:
      - onboarding-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend
    restart: unless-stopped
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - onboarding-network

volumes:
  mongo-data:

networks:
  onboarding-network:
    driver: bridge
`

---

### Understanding every section

#### ersion: "3.9"
The Docker Compose file format version. 3.9 is modern and stable.

#### services:
Each service = one container. You define backend, frontend, mongo here.

---

#### mongo: service
`yaml
image: mongo:7
`
Uses the official MongoDB 7 image directly from Docker Hub. No custom Dockerfile needed.

`yaml
restart: unless-stopped
`
If the container crashes, Docker restarts it. Stops only when you manually stop it.

`yaml
volumes:
  - mongo-data:/data/db
`
Mounts the named volume mongo-data to /data/db inside the container (where MongoDB stores its data). Your data survives container restarts.

---

#### ackend: service
`yaml
build:
  context: ./backend
  dockerfile: Dockerfile
`
Instead of a pre-built image, build one from your ackend/Dockerfile. context tells Docker where to find the files.

`yaml
env_file:
  - ./backend/.env
`
Loads environment variables from your .env file into the container. These are runtime variables (not baked into the image).

`yaml
environment:
  - MONGO_URI=mongodb://mongo:27017/onboardingDB
`
This **overrides** the MONGO_URI from your .env file. Inside Docker Compose, the MongoDB hostname is mongo (the service name) — not localhost and not an Atlas URL. Containers reach each other by service name.

`yaml
depends_on:
  - mongo
`
Start mongo before ackend. Note: this only waits for the container to START — not for MongoDB to be fully ready. For that you'd need a healthcheck, but that's advanced.

---

#### rontend: service
`yaml
ports:
  - "3000:80"
`
Nginx inside the container serves on port 80. We map it to port 3000 on your machine.

---

#### olumes: (top-level)
`yaml
volumes:
  mongo-data:
`
Declares the named volume. Docker creates and manages this.

#### 
etworks: (top-level)
`yaml
networks:
  onboarding-network:
    driver: bridge
`
Creates a custom bridge network. All three services are on this network, so they can reach each other by service name.

---

## ✅ EXERCISE 5 — Run the Full Stack with Docker Compose

### Step 1 — Create docker-compose.yml

Create it in the project root with the content above.

### Step 2 — Start everything

From the project root:

`powershell
docker compose up --build
`

**Flags:**
- --build — rebuild images before starting (always use this after changing Dockerfiles)
- No -d first time — watch the logs. See all three services starting.

Docker will:
1. Build onboarding-backend image from ./backend/Dockerfile
2. Build onboarding-frontend image from ./frontend/Dockerfile
3. Pull mongo:7 from Docker Hub
4. Create the onboarding-network
5. Create the mongo-data volume
6. Start all three containers

### Step 3 — Verify

In a NEW terminal:

`powershell
docker compose ps
`

All three should show unning.

Open:
- http://localhost:3000 — your React frontend
- http://localhost:5000 — your API

### Step 4 — Watch logs

`powershell
# All services
docker compose logs -f

# Just the backend
docker compose logs -f backend

# Just mongo
docker compose logs -f mongo
`

### Step 5 — Stop everything

`powershell
docker compose down
`

This stops and removes all containers. The mongo-data volume survives.

To also remove the volume (wipe database):
`powershell
docker compose down -v
`

### Step 6 — Restart without rebuilding

`powershell
docker compose up -d
`

-d = detached (background). Images are reused.

---

## 2.7 — Debugging Scenarios

Work through these. Don't just read them.

---

### Scenario A — "Container exits immediately after starting"

`powershell
docker compose up
# backend exits with code 1
`

**Diagnosis steps:**
`powershell
docker compose logs backend
`

Look for: Error: Cannot find module, ECONNREFUSED, MongoServerError

**Common causes:**
- Missing env variable (MONGO_URI not set)
- Wrong file path in COPY
- Syntax error in code (but we're not changing code, so unlikely)

---

### Scenario B — "Frontend loads but API calls fail (CORS / network error)"

Your React app loads at localhost:3000 but API requests fail.

**Check:**
`powershell
docker compose logs frontend
docker compose logs backend
`

Open browser DevTools → Network tab. What URL is the frontend calling?

**Common cause:** The frontend was built with VITE_API_URL=http://localhost:5000/api — this is the browser's perspective (your machine), which is correct for dev. Inside Docker Compose, the frontend Nginx just serves static files; the browser still calls the API directly.

---

### Scenario C — "I changed my backend code but the container is running old code"

`powershell
docker compose up -d
# edits don't appear
`

**Fix:**
`powershell
docker compose up --build -d
`

You must rebuild the image after code changes. The container runs the image — it doesn't hot-reload from your files.

---

### Scenario D — "exec into a running container to debug"

`powershell
docker compose exec backend sh
`

(Alpine containers use sh, not ash.)

Now you're inside the backend container. Try:
`sh
ls
cat server.js
env | grep MONGO
`

Check what environment variables are actually set. This is how you verify your .env is being passed correctly.

---

### Scenario E — "I want to see the Nginx config inside the frontend container"

`powershell
docker compose exec frontend sh
cat /etc/nginx/conf.d/default.conf
`

---

## 📝 Interview Checkpoint — Part 2

Answer without notes:

1. **What does FROM node:18-alpine AS builder mean?** Why lpine? Why AS builder?

2. **Why do we COPY package*.json BEFORE COPY . . in the Dockerfile?**

3. **What does EXPOSE 5000 actually do?** Does it open the port?

4. **What is a multi-stage build?** Why did we use one for the frontend?

5. **In docker-compose.yml, the backend has MONGO_URI=mongodb://mongo:27017.** Why mongo and not localhost?

6. **depends_on: - mongo — does this guarantee MongoDB is fully ready?**

7. **You edited a backend route. What exact command re-deploys it?**

8. **A container is running but crashing every 30 seconds. How do you investigate?**

---

## 🚦 Status

> Complete all 5 exercises, go through each debugging scenario, answer the checkpoint.
> Come back with your answers. Then we move to Part 3 — Git + GitHub + CI/CD.

---

## 2.11 — Real Errors Encountered (Actual Session)

These are real errors that came up while Dockerizing this exact project. This section is more valuable than any tutorial because these are things you will hit again.

---

### Error 1 — MongoDB Atlas: `querySrv ENOTFOUND`

**Full error:**
```
MongoDB connection failed: querySrv ENOTFOUND _mongodb._tcp.mycluster.fvbse2s.mongodb.net
```

**What happened:**
The backend container started fine but couldn't reach MongoDB Atlas. The same connection string works fine when running the app normally on the machine.

**Root cause:**
Docker containers have their own network. When the container makes an outbound request to Atlas, it goes out from a **different IP** than your laptop. MongoDB Atlas has an IP whitelist — it was only allowing the laptop's IP, not the Docker container's IP.

**Fix:**
Go to MongoDB Atlas → **Network Access** → **Add IP Address** → **Allow Access from Anywhere** (`0.0.0.0/0`).

**What you learned:**
- "Works on my machine" failures in Docker are often networking or environment differences
- Containers have their own network identity — their outbound IP differs from the host machine
- `docker logs <container>` is your first debugging tool — it showed the exact error immediately

---

### Error 2 — MongoDB Atlas: `Authentication Failed`

**Full error:**
```
MongoServerError: Authentication failed
```

**What happened:**
After fixing the IP whitelist (Error 1), the container could now reach Atlas but got rejected with wrong credentials.

**Root cause:**
The Atlas database user `OnBoarding_Admin` was created with access scoped to **only the `onboardingDB` database**. We were now connecting to `onboardingDB-docker` (a new database for clean separation between Vercel and Docker). The user had no permission for this new database.

**Fix:**
MongoDB Atlas → **Database Access** → Edit `OnBoarding_Admin` → change role to **"Read and Write to Any Database"** (built-in role). This grants the user access to all databases in the cluster.

**What you learned:**
- MongoDB Atlas user roles are database-level, not just cluster-level
- A new database is created automatically on first connect + write — you never need to manually create it in Atlas
- Authentication errors and authorization errors look similar but are different:
  - Authentication = wrong username/password
  - Authorization = right credentials but no permission for that resource

---

### Error 3 — Tailwind CSS v4: `@tailwindcss/oxide` native binding fails on Alpine

**Full error:**
```
ERROR: failed to build: failed to solve: process "/bin/sh -c npm run build" did not complete successfully: exit code 1
```

**Underlying cause:**
`@tailwindcss/oxide` (Tailwind v4's native Rust-based engine) requires a binary compatible with the system's C library. Alpine Linux uses **musl libc**. The version of `@tailwindcss/oxide` that ships with Node 18 only had **glibc binaries** — incompatible with Alpine.

**Fix:**
Changed the builder stage from `node:18-alpine` to `node:20-alpine`.

```dockerfile
# Before (broken)
FROM node:18-alpine AS builder

# After (works)
FROM node:20-alpine AS builder
```

**Why node:20-alpine works:**
Newer versions of `@tailwindcss/oxide` ship pre-compiled binaries for both glibc AND musl (Alpine). Node 20 pulls a newer enough version of the package that includes the musl binary.

**What you learned:**
- Alpine Linux uses musl libc, not glibc — native binaries (Rust, C extensions) compiled for glibc will not run on Alpine
- This is a very common Docker pain point with packages that have native bindings (bcrypt, sharp, canvas, etc.)
- Always pin a specific Node version. `node:latest` or `node:18` can behave differently on Alpine vs Debian
- The multi-stage build still wins here: `node:20` (Debian) could also fix this but results in a larger image. `node:20-alpine` keeps the image small AND fixes the binary compatibility

---

### Error 4 — nginx.conf: BOM (Byte Order Mark) causes parse error

**Full error:**
```
nginx: [emerg] BOM not permitted in nginx.conf
```

**What happened:**
The `nginx.conf` file was written using PowerShell's `Out-File -Encoding utf8`. PowerShell's default utf8 encoding adds a **BOM** (Byte Order Mark — three invisible bytes `EF BB BF` at the start of the file). Nginx does not accept a BOM in its config files.

**Fix:**
Re-save `nginx.conf` without BOM. Options:
- In VS Code: bottom-right corner shows encoding → click it → "Save with Encoding" → select **UTF-8** (not UTF-8 with BOM)
- Or use `Out-File -Encoding utf8NoBOM` in PowerShell (PowerShell 6+)

**What you learned:**
- BOM is an invisible character — it causes silent, hard-to-diagnose failures
- PowerShell's default `utf8` encoding includes BOM. Use `utf8NoBOM` or save via VS Code
- When a config file suddenly "doesn't work" in a container, check the encoding and line endings (`CRLF` vs `LF`) — both cause issues in Linux containers

---

### Error 5 — Duplicate `MONGO_URI` in `.env`

**Observed:**
The original `backend/.env` had `MONGO_URI` defined twice — once as a placeholder and once as the real Atlas URI:

```
MONGO_URI=your_mongodb_connection_string   ← line 3
...
MONGO_URI=mongodb+srv://...onboardingDB    ← line 9
```

**What this taught:**
- `dotenv` typically reads the **first occurrence** of a variable. Having duplicates is a bug waiting to happen.
- We solved this cleanly by creating `backend/.env.docker` — a separate env file for Docker with a single, clean `MONGO_URI` pointing to `onboardingDB-docker`
- `.env.docker` is added to `.gitignore` (`*.env.*` pattern) so credentials never go to GitHub

---

### The separation we ended up with

| Environment | Config file | Database |
|---|---|---|
| Local dev (no Docker) | `backend/.env` | `onboardingDB` |
| Vercel (production) | Vercel dashboard env vars | `onboardingDB` |
| Docker local | `backend/.env.docker` | `onboardingDB-docker` |
| GitHub Actions CI | No DB connection | Image build only |

Clean separation. Zero interference. One Atlas cluster, multiple databases, all free.

---

> These errors are the real curriculum. The commands in the earlier sections tell you what to do.
> These errors tell you what Docker actually feels like.

