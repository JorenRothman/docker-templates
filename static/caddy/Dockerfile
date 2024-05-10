FROM node:20-alpine AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
COPY . /app
WORKDIR /app

FROM base as builder
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm run build

FROM caddy:2.7.6-alpine as runner
COPY ./Caddyfile /etc/caddy/Caddyfile
COPY --from=builder /app/dist /usr/share/caddy

EXPOSE 80
