# ------------------------------------
#! 1) Builder Stage
#! Builds the entire monorepo context
# ------------------------------------
FROM node:20-slim AS builder

# Set the working directory for the entire monorepo
WORKDIR /app

# Copy lockfile and package configuration files first (for caching)
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY turbo.json ./

# Copy packages and app code to the workspace root
# This ensures pnpm and turbo have the full context
COPY apps apps
COPY packages packages

# Install pnpm globally and dependencies using the lockfile
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Disable telemetry during build
ENV TURBO_TELEMETRY_DISABLED=1
ENV NEXT_TELEMETRY_DISABLED=1

# Build the specific 'web' application using the 'build' script
# The output for 'web' will be in /app/apps/web/.next
RUN pnpm --filter=web build

# ------------------------------------
#! 2) Production Runtime Image (Runner)
#! Uses a minimal base image for security and size
# ------------------------------------
FROM node:20-slim AS runner

# Use Next.js standalone output if you prefer a minimal image
# FROM node:20-alpine AS runner
# Note: For simplicity and broader compatibility, using node:20-slim here

# Set production environment variables
ENV NODE_ENV=production
ENV PORT=3001
WORKDIR /app

# 1. Copy the necessary Next.js runtime dependencies (for the app itself)
COPY --from=builder /app/node_modules /app/node_modules

# 2. Copy the Next.js artifacts from the built app path
COPY --from=builder /app/apps/web/.next /app/.next

# 3. Copy public assets
COPY --from=builder /app/apps/web/public /app/public

# 4. Copy the package.json and start script from the app folder
# We need package.json to run 'pnpm start' which requires dependencies 
COPY --from=builder /app/apps/web/package.json /app/package.json

# If you use the root package.json for start:
# COPY --from=builder /app/package.json /app/package.json
# RUN pnpm install --prod --frozen-lockfile

# Expose the port
EXPOSE 3001

# Run the application
# Assumes 'web' package.json has a 'start' script that runs 'next start'
CMD ["pnpm", "start"]