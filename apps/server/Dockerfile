# ------------------------------------
#! 1) Builder Stage
#! Builds the entire monorepo context
# ------------------------------------
FROM node:20-slim AS builder

WORKDIR /app

# Copy lockfile and package configuration files first (for caching)
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY turbo.json ./

# Copy packages and app code to the workspace root
COPY apps apps
COPY packages packages

# Install dependencies
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Build the NestJS 'server' application
# Output will be in /app/apps/server/dist
RUN pnpm --filter=server build

# ------------------------------------
#! 2) Production Runtime Image (Runner)
# ------------------------------------
FROM node:20-slim AS runner

# Use non-root user for better security (optional but recommended)
# RUN addgroup --system appgroup && adduser --system appuser --ingroup appgroup
# USER appuser

ENV NODE_ENV=production
ENV PORT=8080
WORKDIR /app

# 1. Copy the necessary runtime dependencies
COPY --from=builder /app/node_modules /app/node_modules

# 2. Copy the build output (dist)
COPY --from=builder /app/apps/server/dist /app/dist

# 3. Copy the package.json from the app (needed if using pnpm start)
COPY --from=builder /app/apps/server/package.json /app/package.json

# If you prefer to use the root package.json to install only production dependencies:
# COPY --from=builder /app/package.json /app/package.json
# RUN pnpm install --prod --frozen-lockfile

# Expose the port
EXPOSE 8080

# Start the NestJS application (main.js is the entry point in dist)
CMD ["node", "dist/main.js"]