# ---------- Stage 1: Build the Next.js application ----------
FROM public.ecr.aws/docker/library/node:20-alpine AS build
WORKDIR /app

# Copy dependency manifests and install all dependencies (including dev)
COPY package*.json ./
RUN npm install

# Copy the rest of the app source
COPY . .

# --- ✅ Build argument and environment variable setup ---
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

RUN echo "Building with NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}"

# Build the Next.js app (this will embed NEXT_PUBLIC_API_URL)
RUN npm run build

# ✅ Remove dev dependencies *after* build
RUN npm prune --production

# ---------- Stage 2: Create the final production image ----------
FROM public.ecr.aws/docker/library/node:20-alpine AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs

COPY --from=build --chown=nextjs:nodejs /app/public ./public
COPY --from=build --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/.next/static ./.next/static

EXPOSE 3000
ENV HOSTNAME=0.0.0.0
ENV PORT=3000

USER nextjs
CMD ["node", "server.js"]
