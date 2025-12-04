# E2B Template: fullstack-vue-fastapi-node22
# Pre-configured environment for Vue 3 + FastAPI full-stack development

FROM ubuntu:22.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install basic utilities and curl (needed for Node.js setup)
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    gnupg \
    git \
    jq \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 22.x (LTS)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm@latest

# Install Python build dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    sqlite3 \
    && rm -rf /var/lib/apt/lists/*

# Install uv (fast Python package manager)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:$PATH"

# Create project directories
RUN mkdir -p /home/user/server /home/user/client /home/user/server/static/media

# Set up backend with FastAPI dependencies pre-installed
WORKDIR /home/user/server
RUN uv init --app \
    && uv add fastapi "uvicorn[standard]" python-dotenv requests httpx aiofiles aiosqlite python-multipart pydantic-settings sqlalchemy pillow

# Set up frontend with Vite + Vue 3 + TypeScript + Pinia pre-installed
WORKDIR /home/user/client
RUN npm create vite@latest . -- --template vue-ts \
    && npm install \
    && npm install vite@5.4.11 @vitejs/plugin-vue@5.2.1 \
    && npm install axios pinia vue-router@4 @vueuse/core

# Pre-install Playwright for browser testing (optional but useful)
RUN npm install -g playwright \
    && npx playwright install chromium --with-deps

# Set default working directory
WORKDIR /home/user

# Verify installations
RUN node --version && npm --version && uv --version && python3 --version && sqlite3 --version
