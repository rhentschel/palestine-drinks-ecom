# Next.js Storefront (Palestine Drinks)
FROM node:20-alpine AS base
RUN apk add --no-cache libc6-compat

# --- deps: ALLE Deps inkl. devDeps (fuer next build) ---
FROM base AS deps
WORKDIR /app
ENV NODE_ENV=development
COPY package.json package-lock.json* ./
RUN npm ci --include=dev

# --- builder: Next.js kompilieren, NEXT_PUBLIC_* muessen als ARG/ENV gesetzt sein ---
FROM base AS builder
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Build-Args die Coolify/docker build --build-arg liefern muss
ARG MEDUSA_BACKEND_URL
ARG NEXT_PUBLIC_MEDUSA_BACKEND_URL
ARG NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY
ARG NEXT_PUBLIC_BASE_URL
ARG NEXT_PUBLIC_DEFAULT_REGION
ARG NEXT_PUBLIC_STRIPE_KEY
ARG REVALIDATE_SECRET

# ARGs in ENV uebernehmen, damit next build sie sieht
ENV MEDUSA_BACKEND_URL=${MEDUSA_BACKEND_URL}
ENV NEXT_PUBLIC_MEDUSA_BACKEND_URL=${NEXT_PUBLIC_MEDUSA_BACKEND_URL}
ENV NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=${NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY}
ENV NEXT_PUBLIC_BASE_URL=${NEXT_PUBLIC_BASE_URL}
ENV NEXT_PUBLIC_DEFAULT_REGION=${NEXT_PUBLIC_DEFAULT_REGION}
ENV NEXT_PUBLIC_STRIPE_KEY=${NEXT_PUBLIC_STRIPE_KEY}
ENV REVALIDATE_SECRET=${REVALIDATE_SECRET}

COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# --- runner: Production ---
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs && adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/next.config.js ./next.config.js
COPY --from=builder /app/next-sitemap.js ./next-sitemap.js

USER nextjs
EXPOSE 8000

CMD ["npx", "next", "start", "-p", "8000"]
