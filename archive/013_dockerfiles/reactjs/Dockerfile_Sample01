# Install dependencies only when needed
FROM --platform=linux/amd64 node:19-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR ./
COPY package.json package-lock.json ./
RUN npm install

# Rebuild the source code only when needed
FROM --platform=linux/amd64 node:19-alpine AS builder

WORKDIR /

COPY --from=deps ./node_modules ./node_modules

COPY . .

RUN npm run build

# Production image, copy all the files and run next
FROM --platform=linux/amd64 node:19-alpine AS runner
WORKDIR /

ENV NODE_ENV production

RUN addgroup --system --gid 1001 bloggroup
RUN adduser --system --uid 1001 bloguser

COPY --from=builder ./public ./public
COPY --from=builder ./package.json ./package.json
COPY --from=builder ./node_modules ./node_modules

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=bloguser:bloggroup ./.next/standalone ./
COPY --from=builder --chown=bloguser:bloggroup ./.next/static ./.next/static

USER bloguser

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
#CMD ["npm", "start"]
