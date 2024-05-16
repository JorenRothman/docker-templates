FROM node:20-alpine AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

COPY . /app
WORKDIR /app

FROM base AS prod-deps

RUN apk add --no-cache libc6-compat

RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --prod --frozen-lockfile

FROM base AS builder

# Disable NextJS Telemetry
ENV NEXT_TELEMETRY_DISABLED 1

RUN apk add --no-cache libc6-compat

RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm run build

# If you need an post build script(like a migration script) uncomment and update next line
# RUN pnpm run migration:run

FROM base AS runner

# Disable NextJS Telemetry
ENV NEXT_TELEMETRY_DISABLED 1

# Used for Coolify healthcheck
RUN apk add curl

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=prod-deps /app/node_modules /app/node_modules

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD HOSTNAME="0.0.0.0" node server.js

