# E2B Template: fullstack-vue-fastapi-node22

A pre-configured E2B sandbox template for full-stack web development with Vue 3 frontend and FastAPI backend.

## What's Included

### Frontend (`/home/user/client/`)
- **Node.js 22.x** (LTS)
- **Vite 5.4.11** - Fast build tool
- **Vue 3** with TypeScript
- **Pinia** - State management
- **Vue Router 4** - Routing
- **Axios** - HTTP client

### Backend (`/home/user/server/`)
- **Python 3.11+**
- **uv** - Fast Python package manager
- **FastAPI** - Modern web framework
- **uvicorn** - ASGI server
- **aiosqlite** - Async SQLite support
- **python-dotenv** - Environment variables
- **requests** - HTTP client
- **aiofiles** - Async file operations

### Tools
- **SQLite 3** - Database
- **Playwright + Chromium** - Browser testing

## Usage

### 1. Build the Template

```bash
# Install E2B CLI if not already installed
npm install -g @e2b/cli

# Login to E2B
e2b auth login

# Build the template (from this directory)
cd e2b-templates/fullstack-vue-fastapi-node22
e2b template build
```

### 2. Use the Template

```bash
# With the sandbox CLI
uv run sbx init --template fullstack-vue-fastapi-node22 --timeout 3600

# Or via E2B SDK
from e2b import Sandbox
sandbox = Sandbox(template="fullstack-vue-fastapi-node22")
```

### 3. Start Development

The template comes with pre-configured projects. Just add your code:

**Backend:**
```bash
# Files are in /home/user/server/
# Start the server:
cd /home/user/server && uv run uvicorn main:app --host 0.0.0.0 --port 8000
```

**Frontend:**
```bash
# Files are in /home/user/client/
# Start dev server:
cd /home/user/client && npm run dev -- --host 0.0.0.0 --port 5173
```

## Directory Structure

```
/home/user/
├── server/                 # FastAPI backend
│   ├── pyproject.toml     # Python dependencies (pre-configured)
│   ├── .venv/             # Virtual environment
│   └── static/media/      # Media storage directory
│
└── client/                # Vue 3 frontend
    ├── package.json       # Node dependencies (pre-configured)
    ├── node_modules/      # Dependencies installed
    ├── vite.config.ts     # Vite configuration
    └── src/               # Source files
```

## Customization

To modify this template:

1. Edit the `Dockerfile` to add/remove dependencies
2. Run `e2b template build` to rebuild
3. The new template version will be available immediately

## Version Info

- Node.js: 22.x
- npm: latest
- Python: 3.11+
- Vite: 5.4.11
- Vue: 3.x
- FastAPI: latest
- uv: latest
