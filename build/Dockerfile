# Use `bun` to build the viewer
FROM oven/bun:canary-alpine AS viewer

# Flatmap package versions to install
ENV SERVER_VERSION=v1.10.3
ENV VIEWER_VERSION=v4.6.4

# First build the viewer distribution
WORKDIR /build

# Get the viewer's sources and build it
RUN apk add --no-cache bash git; \
    git clone  --depth 1 --branch ${VIEWER_VERSION} --recurse-submodules \
        https://github.com/AnatomicMaps/flatmap-viewer.git viewer; \
    cd viewer; \
    bun install --ignore-scripts --no-progress; \
    bash ./scripts/postinstall.sh; \
    bun run build

# Python with `uv` for the server
FROM astral/uv:python3.12-trixie-slim

# Install curl, build tools, and mapmaker's dependencies
RUN apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    curl \
    build-essential \
    git \
    libfontconfig1 libegl1 libgl1; \
    rm -rf /var/lib/apt/lists/*

# Create directories
WORKDIR /maker
WORKDIR /server/viewer/app/dist

# Install the map server
WORKDIR /
RUN curl -L https://github.com/AnatomicMaps/flatmap-server/archive/refs/tags/${SERVER_VERSION}.tar.gz | \
    tar xz -C server --strip-components=1; \
    cd server; \
    uv sync

# Copy the built viewer into the server tree
COPY --from=viewer /build/viewer/app/dist /server/viewer/app/dist

# Copy mapmaker app
COPY ./maker/runmaker.py /server

# Fix `fatal: detected dubious ownership in repository` errors from git when
# accessing host repositories from inside container
RUN git config --global --add safe.directory '*'

# Initialise the `flatmaps` volume with a pre-built anatomical term hierarchy
WORKDIR /flatmaps
COPY ./flatmaps/* /flatmaps
