# ──────────────────────────────────────────────────────────────
# Frontend Dockerfile — Smart Onboarding System
# MULTI-STAGE BUILD: Build with Node → Serve with Nginx
# ──────────────────────────────────────────────────────────────
# WHY MULTI-STAGE?
# Stage 1 (builder): Uses Node.js to run 'npm run build'
#                    Creates optimised static files in /app/dist
# Stage 2 (production): Uses tiny Nginx image to serve those files
#
# Final image only contains Nginx + the built files (~25MB).
# NOT Node.js, NOT node_modules, NOT source code → small & secure.
# ──────────────────────────────────────────────────────────────

# ── STAGE 1: Build ──────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files first → cache npm install layer
COPY package*.json ./

# Install ALL dependencies (including devDeps needed for build)
RUN npm install

# Copy the rest of the source
COPY . .

# Build-time variables — Vite bakes these into the JS bundle at build time
# Passed from docker-compose build args (never from .env which is gitignored)
ARG VITE_API_URL=http://localhost:5000/api
ARG VITE_SOCKET_URL=http://localhost:5000
ENV VITE_API_URL=$VITE_API_URL
ENV VITE_SOCKET_URL=$VITE_SOCKET_URL

# Run Vite build → outputs optimised files to /app/dist
RUN npm run build

# ── STAGE 2: Production (Nginx) ──────────────────────────────
FROM nginx:alpine AS production

# Copy the built React files from Stage 1 into Nginx's serve directory
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy our custom Nginx config (handles /api proxy + React Router)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Nginx listens on port 80
EXPOSE 80

# Start Nginx in the foreground (daemon off = required for Docker)
CMD ["nginx", "-g", "daemon off;"]
