FROM node:22-bookworm-slim

# 1. Install all system dependencies in one go
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    libsqlite3-dev python3 python3-pip python3-venv build-essential g++ make && \
    yarn config set python /usr/bin/python3

# 2. Set up Python Virtual Env for TechDocs
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
RUN pip3 install mkdocs-techdocs-core==1.1.7

# 3. Prepare App Directory
WORKDIR /app
RUN chown node:node /app

# 4. Copy Yarn configuration
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml ./

# 5. Copy Skeleton and fix Yarn Berry permissions
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

USER root
RUN mkdir -p /home/node/.yarn/berry/index && chown -R node:node /home/node
USER node

# 6. Install dependencies
RUN --mount=type=cache,target=/home/node/.yarn/berry/cache,sharing=locked,uid=1000,gid=1000 \
    yarn workspaces focus --all --production

# 7. Copy Bundle and Configs
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# 8. THE SQLITE FIX: Ensure the driver is built for this specific OS
USER root
RUN yarn add better-sqlite3
RUN mkdir -p data && chown node:node data
USER node

# 9. THE CORRECT START COMMAND
# We point to the specific .js file and load BOTH config files
CMD ["node", "packages/backend/dist/index.cjs.js", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
