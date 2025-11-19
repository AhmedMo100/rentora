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

# Disable telemetry during build
ENV TURBO_TELEMETRY_DISABLED=1
ENV NEXT_TELEMETRY_DISABLED=1

# Build the specific 'docs' application
RUN pnpm --filter=docs build

# ------------------------------------
#! 2) Production Runtime Image (Runner)
# ------------------------------------
FROM node:20-slim AS runner

ENV NODE_ENV=production
ENV PORT=3003
WORKDIR /app

# 1. Copy the necessary Next.js runtime dependencies
COPY --from=builder /app/node_modules /app/node_modules

# 2. Copy the Next.js artifacts from the built app path
COPY --from=builder /app/apps/docs/.next /app/.next

# 3. Copy public assets
COPY --from=builder /app/apps/docs/public /app/public

# 4. Copy the package.json and start script from the app folder
COPY --from=builder /app/apps/docs/package.json /app/package.json

# Expose the port
EXPOSE 3003

# Run the application
CMD ["pnpm", "start"]