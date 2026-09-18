# ───────────────────────────────────────────────
# Backend Dockerfile — Smart Onboarding System
# ───────────────────────────────────────────────

# FROM  — Base image: Official Node.js 18 on Alpine Linux (tiny ~5MB OS)
#         Alpine keeps the image small. 18 = LTS version matching your dev env.
FROM node:18-alpine

# WORKDIR — All subsequent commands run from /app inside the container.
#            Also creates the directory if it doesn't exist.
WORKDIR /app

# COPY package*.json first — BEFORE copying the rest of the source.
#      WHY: Docker builds in layers. If package.json hasn't changed,
#      Docker reuses the cached 'npm install' layer → much faster rebuilds.
COPY package*.json ./

# RUN npm install — Installs dependencies inside the container.
#     --omit=dev  — Skips devDependencies (nodemon etc) in production.
RUN npm install --omit=dev

# COPY . . — Now copy the rest of the source code.
#            This layer changes every time you edit code, but that's OK
#            because npm install (the slow step) is cached above.
COPY . .

# EXPOSE — Documents which port this container listens on.
#          Does NOT actually open the port — that's done with -p at runtime.
#          Think of it as a label / documentation for other developers.
EXPOSE 5000

# CMD — The default command to run when the container starts.
#       Uses array form (exec form) — more reliable than shell form.
#       node server.js starts your Express app.
CMD ["node", "server.js"]
#test