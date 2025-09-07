# syntax=docker/dockerfile:1.7

#########################
# Builder stage
#########################
FROM node:20-slim AS builder

# Faster apt with no recommends
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
# Cache npm modules smartly
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# Copy source
COPY . .

# Optional private registry auth or tokens at build time
# echo "MY_TOKEN=..." | docker build --secret id=npm_token,src=/dev/stdin .
# Then use it here without leaking into layers:
RUN --mount=type=secret,id=npm_token \
    --mount=type=cache,target=/root/.npm \
    npm run build

#########################
# Runtime stage
#########################
FROM node:20-slim AS runtime
WORKDIR /app

# Copy only needed artifacts
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

# Drop root. Yes, be a responsible adult.
USER node

ENV NODE_ENV=production \
    PORT=8080

EXPOSE 8080
CMD ["node", "dist/index.js"]
