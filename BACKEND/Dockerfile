# syntax=docker/dockerfile:1
########################
# Base image
########################
# Pinned to an exact patch release, not a floating `node:22-alpine`: 22.23.4 is the first
# 22.x carrying the fix for CVE-2026-111497 ("Node2Shell"), an out-of-bounds write in the
# Brotli decoder bundled into the node binary (node:zlib). It is reached by anything that
# decodes a `br` body - axios, fetch() and node-fetch all decode `br` responses. A floating
# tag can resolve to a stale cached image on the build host and silently ship the
# vulnerable runtime. Node 20 is EOL and has no fix at all. Both stages MUST use the same tag.
# Advisory: https://nodejs.org/en/blog/vulnerability/october-2026-security-releases

FROM node:22.23.4-alpine AS deps
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --only=production

FROM node:22.23.4-alpine
WORKDIR /usr/src/app

# Give the base image's built-in `node` user ownership of WORKDIR itself. This
# chown is deliberately NON-recursive: it costs ~0 bytes and covers the one
# directory COPY --chown cannot, since WORKDIR is created root-owned.
RUN chown node:node /usr/src/app

# Ownership is set by COPY --chown, NOT by a later `RUN chown -R`. A recursive
# chown rewrites every file's metadata, and because overlayfs is copy-on-write at
# file granularity that duplicates the ENTIRE tree into an extra layer - measured
# at 235MB (31% of the image) on the largest service in the estate.
COPY --from=deps --chown=node:node /usr/src/app/node_modules ./node_modules
COPY --chown=node:node . .

# Secure environment by running under non-root node user
USER node

# Expose API and Health server ports
EXPOSE 80
EXPOSE 801

CMD ["node", "server.js"]
