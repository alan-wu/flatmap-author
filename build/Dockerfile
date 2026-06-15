# Pulls an image containing Python 3.12, Node.js (LTS), and npm
FROM nikolaik/python-nodejs:python3.12-nodejs24

# Flatmap package versions to install
ENV SERVER_VERSION=v1.10.3
ENV VIEWER_VERSION=v4.6.4

# Set sume Python environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install Git and `mapmaker` dependecies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    git \
    libfontconfig1 libegl1 libgl1 \
    && rm -rf /var/lib/apt/lists/*

# Install `bun` and `vite`
RUN npm install -g bun vite

# Install `uv`
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Verify installations
RUN cat /etc/os-release
RUN git --version && node --version && bun --version && python3 --version && uv --version

# Install Flatmap packages here
WORKDIR /flatmaps

# Install map server
RUN git clone --depth 1 --branch ${SERVER_VERSION} \
        https://github.com/AnatomicMaps/flatmap-server.git server && \
    cd server && \
    uv sync

# Install the map viewer as part of the server
WORKDIR /flatmaps/server
RUN git clone  --depth 1 --branch ${VIEWER_VERSION} --recurse-submodules \
        https://github.com/AnatomicMaps/flatmap-viewer.git viewer && \
    cd viewer && \
    bun install
