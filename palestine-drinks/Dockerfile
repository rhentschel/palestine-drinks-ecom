# Medusa v2 Backend (API + Admin Dashboard)
FROM node:20-alpine AS base
RUN apk add --no-cache libc6-compat python3 make g++

# --- deps stage: ALLE Dependencies (inkl. devDeps) fuer den Build ---
FROM base AS deps
WORKDIR /app
ENV NODE_ENV=development
COPY package.json package-lock.json* ./
RUN npm ci --include=dev

# --- builder stage: Medusa kompilieren ---
FROM base AS builder
WORKDIR /app
ENV NODE_ENV=development
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx medusa build

# --- runner stage: Production ---
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

# Medusa v2 Build-Output liegt in .medusa/server/ mit eigener package.json
COPY --from=builder /app/.medusa/server ./
RUN npm ci --omit=dev --omit=optional --ignore-scripts

EXPOSE 9000

CMD ["sh", "-c", "npx medusa db:migrate && npx medusa start"]
