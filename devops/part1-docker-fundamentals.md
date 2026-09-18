# Part 1 — Docker Fundamentals

> **Progress tracker**
> - [ ] 1.1 What is Docker?
> - [ ] 1.2 Container vs VM
> - [ ] 1.3 Image vs Container
> - [ ] 1.4 Docker Hub & registries
> - [ ] 1.5 Your first container (`docker run`)
> - [ ] 1.6 Core commands
> - [ ] 1.7 Port mapping
> - [ ] 1.8 Environment variables
> - [ ] 1.9 Volumes
> - [ ] 1.10 Docker networks
> - [ ] 1.11 Interview checkpoint

---

## 1.1 — What is Docker?

### Concept

Right now, your MERN application runs because:
- You have **Node.js** installed on your laptop
- You have the right **npm packages** installed
- You have a **.env file** with credentials
- MongoDB is either local or on Atlas

This works on **your machine**. But what happens when:
- You give your project to a classmate who has a different Node version?
- You want to run it on a server?
- You want to run tests in a CI pipeline?

It breaks. Because the **environment is different**.

**Docker solves this.**

Docker packages your application together with everything it needs to run:
- The correct Node.js version
- All dependencies
- Configuration

That package is called an **image**. When you run it, it becomes a **container** — an isolated, consistent environment that behaves the same everywhere.

### Real-world analogy

> Think of it like a shipping container.  
> Before containers, shipping was chaotic — different boxes, different sizes.  
> After containers, you pack everything into a standard box and it ships the same way everywhere.  
> Docker containers are the same idea for software.

---

## 1.2 — Container vs VM

### Virtual Machine (VM)

```
Your Laptop Hardware
  └── Host OS (Windows)
        └── Hypervisor (VMware / VirtualBox)
              ├── Guest OS (full Ubuntu = 1-2 GB)
              │     └── Your App
              └── Guest OS (full Windows Server)
                    └── Another App
```

A VM runs a **full separate operating system**. Heavy. Slow to start. Uses a lot of memory.

### Docker Container

```
Your Laptop Hardware
  └── Host OS (Windows)
        └── Docker Engine
              ├── Container A (just your app + dependencies, ~100MB)
              └── Container B (another app, ~80MB)
```

A container **shares the host OS kernel**. It only packages your app and its dependencies. Lightweight. Starts in seconds.

### Key differences

| | VM | Container |
|---|---|---|
| Startup | Minutes | Seconds |
| Size | GBs | MBs |
| OS | Full guest OS | Shares host kernel |
| Isolation | Strong (full OS) | Process-level |
| Use case | Run different OSes | Run apps consistently |

> **Important**: On Windows, Docker uses a lightweight Linux VM (WSL2) internally. Your containers still feel fast and small because they share this single VM — not one VM per container.

---

## 1.3 — Image vs Container

This is one of the most important distinctions.

| | Image | Container |
|---|---|---|
| What | Blueprint / snapshot | Running instance |
| Analogy | Class definition | Object instance |
| Analogy 2 | Cake recipe | Actual cake |
| Modifiable | No (read-only layers) | Yes (has a writable layer) |
| Lives | On disk | In memory (while running) |

**One image → many containers**

You can run 5 containers from the same image simultaneously. Like baking 5 cakes from one recipe.

```
node:18 image
  ├── container-1 (running your backend on port 5000)
  ├── container-2 (running backend on port 5001)
  └── container-3 (running backend on port 5002)
```

---

## 1.4 — Docker Hub & Container Registry

A **registry** is where images are stored and shared — like GitHub, but for Docker images.

**Docker Hub** (`hub.docker.com`) is the default public registry.

When you run:
```
docker pull node:18
```
Docker goes to Docker Hub, finds the official `node` image with the `18` tag, and downloads it to your machine.

Image naming format:
```
[registry]/[username]/[image-name]:[tag]

Examples:
  node:18                        → official Node.js image, version 18
  mongo:7                        → official MongoDB image, version 7
  nginx:alpine                   → official Nginx, based on Alpine Linux (tiny)
  yourname/my-app:latest         → your own image pushed to Docker Hub
```

The `latest` tag means the most recent version (default if you don't specify a tag).

---

## ✅ EXERCISE 1 — Your First Container

### Pre-check
Make sure Docker Desktop is running. Open a PowerShell terminal.

Verify Docker is installed:
```powershell
docker --version
```
Expected output: `Docker version 27.x.x, build ...`

---

### Step 1 — Pull an image

```powershell
docker pull hello-world
```

**What this does:** Downloads the `hello-world` image from Docker Hub to your local machine.

**Verify it downloaded:**
```powershell
docker images
```

You'll see something like:
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   3 months ago   13.3kB
```

---

### Step 2 — Run a container

```powershell
docker run hello-world
```

**What this does:** Creates and starts a container from the `hello-world` image. It prints a message and exits.

Read the output carefully. It tells you exactly what Docker did step by step.

---

### Step 3 — See running containers

```powershell
docker ps
```

`docker ps` shows **currently running** containers. Since `hello-world` already exited, you'll see nothing.

```powershell
docker ps -a
```

`-a` = all containers (including stopped ones). You'll see the hello-world container in `Exited` status.

```
CONTAINER ID   IMAGE         COMMAND    STATUS                     NAMES
abc123def456   hello-world   "/hello"   Exited (0) 5 seconds ago   funny_name
```

---

### Step 4 — Run something interactive

```powershell
docker run -it ubuntu bash
```

**Flags explained:**
- `-i` = interactive (keep stdin open)
- `-t` = allocate a pseudo-terminal (TTY) — gives you a terminal prompt
- `ubuntu` = image name
- `bash` = the command to run inside the container

You're now **inside** the container. Try:
```bash
ls
pwd
cat /etc/os-release
```

You're running Ubuntu commands on your Windows machine — inside a container.

Type `exit` to leave.

---

### Step 5 — Inspect stopped containers

```powershell
docker ps -a
```

You'll see the ubuntu container in `Exited` state.

**Remove a stopped container:**
```powershell
docker rm <container-id-or-name>
```

You can use just the first 3-4 characters of the container ID.

**Remove all stopped containers at once:**
```powershell
docker container prune
```

---

## 1.5 — Core Commands Reference

Run these yourself and observe the output.

### `docker pull`
```powershell
docker pull node:18-alpine
```
Downloads an image. `alpine` = very small Linux variant (~5MB base).

### `docker images`
```powershell
docker images
```
Lists all images on your machine. Note the SIZE column.

### `docker run`
```powershell
docker run node:18-alpine node --version
```
Runs `node --version` inside a fresh container and exits.

### `docker run --name`
```powershell
docker run --name my-node-test node:18-alpine node --version
```
Gives the container a memorable name instead of a random one.

### `docker ps` and `docker ps -a`
```powershell
docker ps        # running only
docker ps -a     # all (running + stopped)
```

### `docker stop`
```powershell
docker stop <container-name-or-id>
```
Sends a graceful shutdown signal (SIGTERM) to the container.

### `docker start`
```powershell
docker start <container-name-or-id>
```
Restarts a stopped container.

### `docker rm`
```powershell
docker rm <container-name-or-id>
```
Deletes a stopped container. Add `-f` to force-remove a running one.

### `docker logs`
```powershell
docker logs <container-name-or-id>
docker logs -f <container-name-or-id>   # -f = follow (stream live)
```
Shows stdout/stderr output from the container. **Essential for debugging.**

### `docker exec`
```powershell
docker exec -it <container-name-or-id> bash
```
Opens a shell inside a **running** container. Like SSH-ing into it.

### `docker inspect`
```powershell
docker inspect <container-name-or-id>
```
Returns a JSON blob with all details about a container: IP address, mounts, environment variables, etc.

---

## ✅ EXERCISE 2 — Run a Long-Running Container

Run an Nginx web server (like running your own tiny web server locally):

```powershell
docker run -d -p 8080:80 --name my-nginx nginx
```

**Flags explained:**
- `-d` = detached mode — runs in the background
- `-p 8080:80` = **port mapping** (explained below)
- `--name my-nginx` = give it a name

Open your browser: **http://localhost:8080**

You should see the Nginx welcome page.

**What just happened?**
- Docker pulled the `nginx` image
- Started a container that runs Nginx
- Made it accessible on your local port 8080

**Verify it's running:**
```powershell
docker ps
```

**See the logs:**
```powershell
docker logs my-nginx
docker logs -f my-nginx   # refresh your browser and watch new log lines appear
```

**Stop it:**
```powershell
docker stop my-nginx
```

**Remove it:**
```powershell
docker rm my-nginx
```

---

## 1.6 — Port Mapping

Containers run in their own isolated network. By default, you cannot reach a container from your browser.

**Port mapping** punches a hole: `HOST_PORT:CONTAINER_PORT`

```
-p 8080:80
     │    └── Port INSIDE the container (Nginx listens on 80)
     └──────── Port on YOUR machine (you visit localhost:8080)
```

```
Your Browser
  → localhost:8080
      → Docker routing
          → Container port 80
              → Nginx
```

**Common patterns:**
```powershell
-p 3000:3000    # React dev server
-p 5000:5000    # Node/Express backend
-p 27017:27017  # MongoDB
-p 8080:80      # Nginx serving on 80, mapped to 8080
```

You can map any host port to any container port. The host port just has to be free.

---

## 1.7 — Environment Variables

Applications need configuration — database URLs, API keys, secrets. You pass these as environment variables.

```powershell
docker run -e MY_VAR=hello -e ANOTHER=world node:18-alpine node -e "console.log(process.env.MY_VAR)"
```

**`-e`** = set an environment variable inside the container.

For multiple variables, use an env file:
```powershell
docker run --env-file ./backend/.env node:18-alpine node --version
```

**Why this matters for your app:**
Your backend reads `process.env.MONGO_URI`, `process.env.JWT_SECRET`, etc. When containerized, you pass these via `-e` flags or `--env-file`. The `.env` file never gets baked into the image (that would be a security risk).

---

## 1.8 — Volumes

Containers are **ephemeral** — when you remove a container, all data inside it is gone.

**Problem:** Your MongoDB container stores data inside itself. Remove the container → lose all data.

**Solution: Volumes**

A volume is a persistent storage location managed by Docker (or a folder on your host machine) that survives container removal.

```powershell
# Named volume (Docker manages the location)
docker run -d -v my-data:/data/db --name test-mongo mongo:7

# Bind mount (maps a host folder into the container)
docker run -d -v C:\Users\krish\mydata:/data/db --name test-mongo mongo:7
```

**Syntax:** `-v HOST_PATH_OR_VOLUME_NAME:CONTAINER_PATH`

**When to use each:**
| | Named Volume | Bind Mount |
|---|---|---|
| Use for | Databases, persistent app data | Dev: sync code into container |
| Location | Docker manages it | You specify the host folder |
| Survives `docker rm`? | Yes | Yes (it's your folder) |

**Verify volumes:**
```powershell
docker volume ls
docker volume inspect my-data
```

---

## 1.9 — Docker Networks

By default, each container is isolated. Containers can't talk to each other unless you connect them.

**Docker creates a default bridge network.** Containers on the same network can reach each other by **container name**.

Create a network:
```powershell
docker network create my-network
```

Run containers on the same network:
```powershell
docker run -d --name backend --network my-network node:18-alpine node -e "require('http').createServer((req,res)=>res.end('hi')).listen(5000)"
docker run -d --name mongo   --network my-network mongo:7
```

Now `backend` can reach `mongo` using `mongodb://mongo:27017` — the container name acts as a hostname.

This is exactly what your MERN app needs:
```
Frontend container  →  http://backend:5000/api  →  Backend container
                                                         ↓
                                               mongodb://mongo:27017
                                                         ↓
                                                  MongoDB container
```

**List networks:**
```powershell
docker network ls
docker network inspect my-network
```

---

## 1.10 — Docker Image Layers (Basic Understanding)

Every Docker image is made of **layers** — like a stack of transparent slides.

```
Layer 4: Your app code           (changes frequently)
Layer 3: npm packages installed  (changes occasionally)
Layer 2: Node.js installed       (rarely changes)
Layer 1: Base OS (alpine)        (almost never changes)
```

**Why this matters:**
- Docker **caches** each layer
- If you change your app code (Layer 4), Docker only rebuilds Layer 4
- Layers 1-3 are reused from cache → fast builds
- This is why the ORDER of instructions in a Dockerfile matters

---

## 📝 Interview Checkpoint — Part 1

Answer these without looking at notes. Write your answer, then check.

1. **What is Docker?** Explain it to a non-technical person in 2 sentences.

2. **Container vs VM** — what's the key difference?

3. **Image vs container** — which one runs? Which one is the blueprint?

4. **You run `docker run -p 3000:3000 my-app`.** What does `-p 3000:3000` do?

5. **Your container runs a database and you delete it. Is the data gone?** What would prevent that?

6. **Two containers need to talk to each other. What do you use?**

7. **A teammate says "the app works on my machine but not in Docker."** What's your first instinct to check? (Hint: `docker logs <container>`)

8. **What command shows you all containers, including stopped ones?**

---

## 🚦 Status

> Complete this section, run every exercise, and answer the interview checkpoint before moving to Part 2.
