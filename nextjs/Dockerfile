ARG NODE_VERSION=20.13.1
################################################################################
# Node image for base image for all stages.
FROM node:${NODE_VERSION}-alpine AS base

# Set working directory for all build stages.
WORKDIR /app

# Install pnpm.
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable pnpm

################################################################################
# Stage for declaring enviroment variables.
FROM base AS shared-env

#ARG DATABASE_URI
#ENV DATABASE_URI=${DATABASE_URI}

ENV NEXT_TELEMETRY_DISABLED=1

################################################################################
# Stage for installing production dependecies.
FROM shared-env AS deps

RUN apk add --no-cache libc6-compat

RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=pnpm-lock.yaml \
    --mount=type=cache,target=/root/.local/share/pnpm/store \
    pnpm install --prod --frozen-lockfile

################################################################################
# Stage for building the application.
FROM deps AS build

RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=pnpm-lock.yaml \
    --mount=type=cache,target=/root/.local/share/pnpm/store \
    pnpm install --frozen-lockfile

COPY . .

RUN pnpm run build

################################################################################
# Create a new stage to run the application with minimal runtime dependencies
# where the necessary files are copied from the build stage.
FROM shared-env AS final

ENV NODE_ENV production

# Needed for Coolify healthcheck
RUN apk add curl

# Run the application as a non-root user.
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=build /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Copy package.json so that package manager commands can be used.
COPY package.json .

# Copy the production dependencies from the deps stage and also
# the built application from the build stage into the image.
COPY --from=build --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/.next/static ./.next/static

# Switch to nextjs user
USER nextjs

# Expose the port that the application listens on.
EXPOSE 3000

# Run the application.
CMD HOSTNAME="0.0.0.0" node server.js
