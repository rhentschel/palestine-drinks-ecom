# Medusa v2 Backend (API + Admin Dashboard)
FROM node:20-alpine AS base

RUN apk add --no-cache libc6-compat python3 make g++

FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/.medusa/server ./
COPY --from=builder /app/package-lock.json ./package-lock.json
RUN npm ci --omit=dev --ignore-scripts

EXPOSE 9000

CMD ["sh", "-c", "npx medusa db:migrate && node node_modules/.bin/medusa start"]
