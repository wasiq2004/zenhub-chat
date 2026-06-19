FROM node:20-bookworm-slim

# ffmpeg for transcoding WhatsApp voice notes (Ogg/Opus → MP3) so Safari can play them
RUN apt-get update \
 && apt-get install -y --no-install-recommends ffmpeg \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Build context is the REPO ROOT (see docker-compose.yml: context: . / dockerfile:
# backend/Dockerfile) so we can also bake in the SQL migrations that live at
# <repo>/db/migrations, outside the backend/ folder.
COPY backend/package.json backend/package-lock.json ./
RUN npm ci --omit=dev

COPY backend/src/ ./src/
COPY backend/scripts/ ./scripts/

# Bake the DB migrations into the image at /app/db/migrations — exactly where the
# boot-time runner looks (src/db/migrate.js: __dirname/../../db/migrations). This
# guarantees migrations are always present (no bind-mount dependency) AND makes
# every new migration change the image, so the backend container is recreated and
# the migration actually runs on the next deploy.
COPY db/migrations/ ./db/migrations/

ENV NODE_ENV=production
EXPOSE 3011

CMD ["node", "src/index.js"]
