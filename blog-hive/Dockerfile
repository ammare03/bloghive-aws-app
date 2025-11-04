# ---------- Stage 1: Build the Next.js application ----------
FROM public.ecr.aws/docker/library/node:20-alpine AS build
WORKDIR /app

# Copy dependency manifests and install dependencies
COPY package*.json ./
RUN npm ci --omit=dev

# Copy the rest of the app source
COPY . .

# --- ✅ Build argument and environment variable setup ---
# Accept build-time argument (injected by CodeBuild)
ARG NEXT_PUBLIC_API_URL
# Export it so Next.js sees it at build time
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

# Log to verify it's being picked up (optional)
RUN echo "Building with NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}"

# Build the Next.js app (this will embed NEXT_PUBLIC_API_URL)
RUN npm run build

# Prune dev dependencies to keep the image light
RUN npm prune --production


# ---------- Stage 2: Create the final production image ----------
FROM public.ecr.aws/docker/library/node:20-alpine AS runner
WORKDIR /app

# Create non-root user for security
RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs

# Copy production build output
COPY --from=build --chown=nextjs:nodejs /app/public ./public
COPY --from=build --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/.next/static ./.next/static

# Expose port
EXPOSE 3000

# Environment defaults for runtime
ENV HOSTNAME=0.0.0.0
ENV PORT=3000

# Switch to non-root user
USER nextjs

# Start the Next.js standalone server
CMD ["node", "server.js"]
