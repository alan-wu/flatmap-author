# Use `bun` to build the viewer
FROM oven/bun:canary-alpine AS viewer

# Flatmap viewer to install
ENV VIEWER_VERSION=v4.7.1

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

# Flatmap server to install
ENV SERVER_VERSION=v1.10.4

# Install curl, build and font tools, and mapmaker's dependencies
RUN apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    curl \
    build-essential \
    fontconfig \
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
COPY --from=viewer /build/viewer/package.json /server/viewer/

# Copy mapmaker app
COPY ./maker/runmaker.py /server

# Fix `fatal: detected dubious ownership in repository` errors from git when
# accessing host repositories from inside container
RUN git config --global --add safe.directory '*'

# Initialise the `flatmaps` volume with a pre-built anatomical term hierarchy
WORKDIR /flatmaps
COPY ./flatmaps/* /flatmaps
RUN gunzip sparc-distances.npy.gz

# Install fonts that are used in SVGs exported from Powerpoint
RUN mkdir -p /usr/share/fonts/truetype/calibri/
COPY ./fonts/calibri*.ttf /usr/share/fonts/truetype/calibri/
RUN mkdir -p /usr/share/fonts/truetype/custom
COPY ./fonts/cambria.ttc /usr/share/fonts/truetype/custom/
RUN chmod 644 /usr/share/fonts/truetype/custom/*; \
    fc-cache -fv
