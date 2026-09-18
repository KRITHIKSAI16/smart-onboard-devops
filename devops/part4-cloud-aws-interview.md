# Part 4 — Cloud Fundamentals, AWS Overview & Interview Training

> **Progress tracker**
> - [ ] 4.1 What is cloud computing?
> - [ ] 4.2 IaaS / PaaS / SaaS
> - [ ] 4.3 AWS service overview
> - [ ] 4.4 How your MERN app maps to AWS
> - [ ] 4.5 Container registry — Docker Hub vs ECR
> - [ ] 4.6 Deployment architecture
> - [ ] 4.7 Final interview test

---

## 4.1 — What Is Cloud Computing?

### Before cloud

```
Your company buys servers
  -> Ships to your office (weeks)
  -> Install OS, configure networking (days)
  -> Deploy application
  -> Server is sitting idle at 3 AM (wasted money)
  -> Need more capacity? Buy more servers (weeks again)
```

### With cloud

```
You go to AWS console
  -> Click "launch EC2 instance" (30 seconds)
  -> Server is running
  -> Pay only for the hours you use
  -> Need more capacity? Click again. Done.
  -> Don't need it? Turn it off. Stop paying.
```

**Cloud computing = renting compute, storage, and networking from someone else's data center, on demand, pay-as-you-go.**

AWS owns the physical servers. You rent slices of them via the internet.

### Why companies use cloud

| Reason | What it means |
|---|---|
| **No upfront cost** | Don't buy servers. Pay for what you use. |
| **Scale instantly** | Handle 10x traffic tomorrow without buying hardware |
| **Global reach** | Deploy in 30+ regions worldwide in minutes |
| **Managed services** | AWS manages the database server — you just use it |
| **Reliability** | AWS guarantees 99.99% uptime. Hard to achieve yourself. |
| **Speed** | Go from idea to deployed in hours, not weeks |

---

## 4.2 — IaaS / PaaS / SaaS

This is a spectrum of "how much does the cloud provider manage?"

```
                YOU MANAGE                          PROVIDER MANAGES
<------------------------------------------------------------->
  IaaS              PaaS                    SaaS
  (EC2)        (Elastic Beanstalk)         (Gmail)
   |                  |                       |
   You manage:        You manage:             You just use it
   - OS               - Application code      - Nothing to manage
   - Runtime          - App config
   - App code
   - App config
   Provider manages:  Provider manages:
   - Virtualization   - OS, runtime, servers
   - Servers          - Scaling, patching
   - Storage
   - Networking
```

### Real-world examples

**IaaS — EC2**
Like renting a bare computer. You install Node.js, configure everything yourself. Full control. More work.

**PaaS — Heroku / Render / Elastic Beanstalk**
You give it your code. It handles the server, OS, scaling. Less control, but much faster. Your current Render deployment is PaaS.

**SaaS — MongoDB Atlas, Gmail, GitHub**
A fully managed product. You just use it. No server to manage.

---

## 4.3 — AWS Services — What You Need to Know

### EC2 — Elastic Compute Cloud

**What it is:** A virtual machine (server) in AWS.

**Problem it solves:** You need a server to run your application.

**Real example:**
- Your laptop runs Node.js locally
- An EC2 instance runs Node.js in AWS's data center

You SSH into an EC2 instance just like you'd log into a Linux machine. Install Node, clone your repo, `npm start`. It's just a server you rent.

**For your MERN app:** You could run your backend on an EC2 instance. You'd install Node, copy your code, run `node server.js`.

**Instance types:** `t2.micro` (free tier, 1 vCPU, 1GB RAM) is enough for learning. Production apps use larger instances.

---

### S3 — Simple Storage Service

**What it is:** Object storage — store any file (images, videos, backups, static websites) at massive scale.

**Problem it solves:** Where do you store files that your application uploads?

**Real example:**
- User uploads a profile photo in your app
- Your backend receives the file
- Backend uploads it to S3
- S3 gives you a URL: `https://s3.amazonaws.com/my-bucket/user123/photo.jpg`
- You store that URL in MongoDB

**For your MERN app:** You currently use Cloudinary for image storage. S3 is the AWS equivalent. Cheaper at scale.

**Key concept:** S3 stores **objects** (files), not a filesystem. Each file has a unique URL. You can make files public or private.

**Also used for:** Hosting static websites (your React `dist/` folder could be served from S3 + CloudFront).

---

### RDS — Relational Database Service

**What it is:** Managed relational databases (PostgreSQL, MySQL, etc.) in AWS.

**Problem it solves:** Running your own database server is hard — backups, patching, replication.

**Real example:**
- Instead of running PostgreSQL on an EC2 instance and managing it yourself
- You use RDS — AWS handles backups, failover, patches
- You just connect with a connection string

**For your MERN app:** Your app uses MongoDB (not a relational DB), so you'd use **MongoDB Atlas** (their managed cloud service) or **DocumentDB** (AWS's MongoDB-compatible service) instead of RDS. But the concept is the same — managed database.

---

### Lambda — Serverless Functions

**What it is:** Run a function without a server. You upload code, AWS runs it on demand.

**Problem it solves:** You need to run a small task (resize an image, send an email, process a webhook) but don't want to pay for a server sitting idle.

**Real example:**
- User uploads image -> S3 triggers Lambda function -> Lambda resizes image -> saves thumbnail back to S3
- That function runs in milliseconds, then stops. You pay for those milliseconds only.

**Key concept: serverless does not mean no server.** It means you don't manage the server. AWS handles everything — scaling, OS, runtime.

**For your MERN app:** If you had a background job (like sending a weekly summary email), Lambda is perfect.

**Difference from EC2:**
| | EC2 | Lambda |
|---|---|---|
| Server | Always running | Starts on demand |
| Cost | Per hour | Per request (milliseconds) |
| Scaling | You configure | Automatic |
| Use for | Long-running apps | Short event-driven tasks |

---

### ECS — Elastic Container Service

**What it is:** AWS's service for running Docker containers.

**Problem it solves:** "I have a Docker image. How do I run it in AWS at scale?"

**Real example:**
- You build `onboarding-backend:latest` image
- Push it to ECR (the registry)
- Create an ECS Task Definition (describe the container: image, port, memory, env vars)
- ECS runs the container on AWS infrastructure
- ECS can run multiple copies and replace crashed containers automatically

**For your MERN app:** Instead of running your backend on an EC2 instance directly, you'd containerize it (Part 2 of this guide) and run it with ECS.

---

### Fargate — Serverless Containers

**What it is:** A way to run ECS containers without managing the underlying EC2 instances.

**ECS gives you a choice:**
- **ECS + EC2 launch type** — you pick and manage EC2 instances that run your containers
- **ECS + Fargate launch type** — AWS manages the machines. You just define: "run this container with 0.5 CPU and 1GB RAM"

**Fargate = containers without servers.**

For learning and small apps, Fargate is simpler. You don't manage EC2 instances.

**Analogy:**
- EC2 = renting a truck and driving it yourself
- Fargate = calling Uber. You just say where you want to go.

---

### ECR — Elastic Container Registry

**What it is:** AWS's container image registry. Like Docker Hub but private and integrated with AWS.

**Problem it solves:** Where do you store your Docker images when you're deploying to AWS?

**The flow:**
```
GitHub Actions builds image
      |
      v
Push image to ECR
      |
      v
ECS pulls image from ECR
      |
      v
Container runs in AWS
```

**Why ECR instead of Docker Hub?**
- Private by default (your image isn't public)
- ECR is inside AWS's network — pulling images from ECR to ECS is faster and free
- Integrated with AWS IAM for access control

**Quick ECR concept:**
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Tag your image for ECR
docker tag onboarding-backend:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/onboarding-backend:latest

# Push to ECR
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/onboarding-backend:latest
```

You don't need to do this now — just understand the concept.

---

### CloudWatch — Monitoring & Logs

**What it is:** AWS's monitoring service. Collects logs, metrics, and alerts.

**Problem it solves:** Your app is running on AWS. Something breaks at 3 AM. How do you know?

**What it does:**
- Collects logs from EC2, ECS, Lambda
- Monitors metrics (CPU %, memory, requests per second)
- Sends alerts (email, SMS) when thresholds are crossed
- Like `docker logs` but for your entire AWS infrastructure

**For your MERN app:** When your backend crashes in AWS, CloudWatch has the logs. Same as `docker logs backend` but cloud-scale.

---

### IAM — Identity and Access Management

**What it is:** AWS's permission system. Controls who (users, services) can do what in AWS.

**Problem it solves:** You have 5 developers. One should only see S3. One should manage EC2. Your application itself needs to access S3. How do you control this?

**Core concepts:**
- **User** = a person (developer, admin)
- **Role** = permissions assigned to a service or application (e.g., "EC2 instance can read from S3")
- **Policy** = a JSON document that says "allow" or "deny" specific actions

**For your MERN app:**
- Your backend on EC2 needs to write to S3 (for file uploads)
- You create an IAM Role with `S3:PutObject` permission
- You attach that role to the EC2 instance
- The backend can now upload to S3 without hardcoding credentials

**Never hardcode AWS credentials in code.** Use IAM Roles.

---

### Load Balancer (ALB — Application Load Balancer)

**What it is:** Distributes incoming traffic across multiple instances or containers.

**Problem it solves:**
```
Without load balancer:
  1000 requests/sec -> one backend instance -> overload -> crash

With load balancer:
  1000 requests/sec -> load balancer -> splits across 5 instances -> no overload
```

**Also does:**
- Health checks — if one instance crashes, stop sending traffic to it
- SSL termination — handles HTTPS so your app doesn't have to
- Routes `/api/*` to backend, `/*` to frontend

**For your MERN app:** When you scale to multiple backend containers in ECS, an ALB sits in front and distributes requests.

---

### Route 53 — DNS Service

**What it is:** AWS's DNS (Domain Name System) service.

**What DNS does:**
```
User types: www.myapp.com
  |
  v
DNS lookup -> "www.myapp.com" = 54.23.11.45
  |
  v
Browser connects to 54.23.11.45 (your load balancer or server)
```

Route 53 lets you:
- Register domains
- Point domain names to your AWS resources (EC2, load balancer, S3, etc.)
- Route traffic based on geography, latency, health

**For your MERN app:** After deploying to AWS, you'd buy a domain, add it to Route 53, and point it at your load balancer or EC2 instance.

---

## 4.4 — How Your MERN App Maps to AWS

```
User's Browser
      |
      v
Route 53 (DNS — resolves myapp.com)
      |
      v
Load Balancer (ALB)
      |         |
      v         v
  Frontend   Backend
  (ECS/      (ECS/
  Fargate)   Fargate)
               |
               v
         MongoDB Atlas
         or DocumentDB
               |
               v
           S3 (images/files)
               |
               v
         CloudWatch (logs)

All secured by IAM roles.
Images stored in ECR.
Code deployed via GitHub Actions.
```

---

## 4.5 — Container Registry: Docker Hub vs ECR

| | Docker Hub | ECR |
|---|---|---|
| Provider | Docker | AWS |
| Privacy | Public by default | Private by default |
| Integration | Universal | Deep AWS integration |
| Cost | Free (public) | $0.10/GB/month |
| Use case | Learning, open source | Production AWS deployments |
| Auth | Docker Hub account | AWS IAM |

**For learning:** Docker Hub is fine.
**For real AWS deployments:** ECR is the standard.

---

## 4.6 — Deployment Architecture

The full picture of what you've learned:

```
YOUR LAPTOP
  |
  | git push
  v
GITHUB (source code)
  |
  | GitHub Actions triggers
  v
RUNNER (ubuntu VM, fresh)
  |-- checkout code
  |-- npm install
  |-- npm run build
  |-- docker build backend image
  |-- docker build frontend image
  |-- docker push -> ECR or Docker Hub
  v
CONTAINER REGISTRY (ECR / Docker Hub)
  |
  | ECS pulls image (or manual pull)
  v
AWS ECS / FARGATE
  |-- backend container (port 5000)
  |-- frontend container (port 80, behind Nginx)
  |
  |-- connects to: MongoDB Atlas
  |-- connects to: S3 (file uploads)
  |-- logs to: CloudWatch
  |
  v
LOAD BALANCER (ALB)
  |
  v
ROUTE 53 (DNS)
  |
  v
USER'S BROWSER -> myapp.com
```

**You can explain every step of this diagram. That is the goal.**

---

## 4.7 — Final Interview Test

Work through these. Answer in your own words. Do not copy from the guide.

---

### Round 1 — Docker

**Q1.** What is Docker? Explain to a non-technical person.

**Q2.** What is the difference between a container and a VM?

**Q3.** You have a Docker image. Is it running? What would make it run?

**Q4.** What does this command do?
```bash
docker run -d -p 5000:5000 --env-file ./backend/.env --name api onboarding-backend:latest
```
Explain every flag.

**Q5.** A container is running but you can't access it on `localhost:5000`. What are the possible causes?

**Q6.** Why do we `COPY package*.json ./` before `COPY . .` in the Dockerfile?

**Q7.** What is a multi-stage build? Why did we use one for the frontend?

**Q8.** Your backend connects to MongoDB. In Docker Compose, the `MONGO_URI` is `mongodb://mongo:27017`. Why `mongo` and not `localhost`?

**Q9.** What command lets you run commands inside a running container?

**Q10.** How do you follow live logs from the backend container?

---

### Round 2 — Git / CI/CD

**Q11.** What happens step by step when you run `git push`?

**Q12.** What is CI? What is CD? What is the difference?

**Q13.** What is a GitHub Actions runner? What does it know when it starts?

**Q14.** Why does every job start with `actions/checkout@v4`?

**Q15.** Your CI workflow passes but the Docker push fails with "denied: access forbidden". What is the cause and how do you fix it?

**Q16.** What does `needs: [backend-check, frontend-check]` do in a workflow?

**Q17.** A developer pushes code with a bug that breaks `npm run build`. What happens in the CI pipeline?

**Q18.** What is a GitHub Secret? Why can't you just put passwords in the YAML file?

**Q19.** Explain this sentence completely:
> "My code is on GitHub. I push a change. GitHub Actions runs tests and builds the application, creates a Docker image and pushes that image to a registry. That image can then be deployed to cloud infrastructure."

**Q20.** What is a container registry? Name two examples.

---

### Round 3 — Cloud / AWS

**Q21.** What is cloud computing? Why do companies use it instead of buying servers?

**Q22.** What is IaaS? Give an example.

**Q23.** What is the difference between EC2 and Lambda?

**Q24.** What is S3? Where would you store user-uploaded profile photos?

**Q25.** What is ECS? What is Fargate? How are they different?

**Q26.** What is ECR? How is it different from Docker Hub?

**Q27.** What is CloudWatch used for?

**Q28.** What is IAM? Why should you never hardcode AWS credentials in your application code?

**Q29.** What does a load balancer do?

**Q30.** What is Route 53?

---

### Round 4 — Troubleshooting Scenarios

Work through each scenario. State what you would check and how.

**Scenario A:**
> "My container is running (`docker ps` shows it) but I can't access `localhost:5000`."

**Scenario B:**
> "My frontend loads but all API calls fail with a network error."

**Scenario C:**
> "GitHub Actions workflow failed. The red X is on the `Build frontend` step."

**Scenario D:**
> "Docker build works perfectly on my machine but fails in CI."

**Scenario E:**
> "The Docker image builds and pushes successfully. ECS starts the container but the application crashes immediately."

**Scenario F:**
> "After `git push`, the GitHub Actions workflow doesn't trigger at all."

---

### Round 5 — Architecture Explanation

Without looking at notes, draw or describe verbally:

1. The path from a developer typing `git push` to a user seeing the updated application in their browser.

2. Which AWS services would you use for each part of your MERN application? Why?

3. What is the difference between running your Node.js app on an EC2 instance vs running it as an ECS Fargate container?

---

## Status

> This is the complete DevOps learning path.
>
> When you work through the interview test, go question by question and write your answers.
> The goal is not to quote this document. The goal is to explain it like you own it.
