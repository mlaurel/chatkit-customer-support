# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Quick Start
```bash
# Install dependencies and start both backend and frontend
npm start
```

### Backend Development
```bash
cd backend
uv sync                                         # Install Python dependencies  
export OPENAI_API_KEY="sk-proj-..."           # Required for agent functionality
uv run uvicorn app.main:app --reload --port 8001  # Start FastAPI server
```

### Frontend Development  
```bash
cd frontend
npm install                                     # Install dependencies
npm run dev                                     # Start Vite dev server (port 5171)
```

### Testing & Linting
```bash
cd backend
uv run ruff check .                            # Lint Python code
uv run mypy .                                  # Type check Python code

cd frontend  
npm run lint                                   # Lint TypeScript/React code
```

### Production Build
```bash
cd frontend
npm run build                                  # Build for production
npm run preview                                # Preview production build
```

## Environment Variables

### Required
- `OPENAI_API_KEY`: OpenAI API key for agent functionality
- `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY`: ChatKit domain key (use any non-empty value for local dev)

### Optional
- `BACKEND_URL`: Backend URL for frontend proxy (defaults to `http://127.0.0.1:8001`)

## Architecture Overview

### Application Structure
This is a **full-stack airline customer support demo** built with:
- **Frontend**: React + TypeScript + Vite + TailwindCSS + ChatKit React components
- **Backend**: FastAPI + OpenAI Agents + ChatKit server integration
- **State Management**: In-memory airline state management with customer profiles

### Key Components

#### Backend (`backend/app/`)
- **`main.py`**: FastAPI application with ChatKit server integration and CORS setup
- **`support_agent.py`**: OpenAI agent with airline-specific tools (seat changes, cancellations, etc.)
- **`airline_state.py`**: In-memory customer profile and flight data management
- **`memory_store.py`**: Thread-based conversation memory storage

#### Frontend (`frontend/src/`)
- **`App.tsx`**: Root component with theme management
- **`components/Home.tsx`**: Main layout with two-panel design (chat + customer context)
- **`components/ChatKitPanel.tsx`**: ChatKit React integration
- **`components/CustomerContextPanel.tsx`**: Real-time customer profile display
- **`hooks/useCustomerContext.ts`**: Customer data fetching and state management

### Agent Architecture
The support agent is built using OpenAI's Agents framework with:
- **Model**: GPT-4.1-mini with 0.4 temperature
- **Tools**: `change_seat`, `cancel_trip`, `add_checked_bag`, `set_meal_preference`, `request_assistance`
- **Context**: Customer profile data injected into each conversation turn
- **State**: Thread-based customer state management with timeline tracking

### API Endpoints
- `POST /support/chatkit`: ChatKit streaming endpoint for agent responses
- `GET /support/customer?thread_id=<id>`: Customer profile data endpoint  
- `GET /support/health`: Health check endpoint

### Data Flow
1. User sends message via ChatKit React component
2. Frontend proxies request to FastAPI backend `/support/chatkit`
3. Backend enriches message with customer context from `AirlineStateManager`
4. OpenAI agent processes request and may invoke tools to modify state
5. Agent streams response back through ChatKit
6. Frontend refreshes customer context panel after response completion

### Development Patterns

#### Adding New Agent Tools
1. Add tool function to `support_agent.py` with `@function_tool` decorator
2. Implement state change logic in `AirlineStateManager`
3. Add tool to agent's tools list
4. Update agent instructions if needed

#### Frontend State Updates
- Customer context automatically refreshes after agent responses
- Use `useCustomerContext` hook for customer data access
- Thread ID management handled by `ChatKitPanel`

#### Error Handling
- Agent tool errors are surfaced to user through ChatKit
- Network errors handled by React error boundaries
- Backend validation errors propagated as tool exceptions

## Development Notes

### Local Development Setup
1. The frontend dev server runs on port 5171 with proxy to backend on 8001
2. Use any non-empty string for `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY` locally
3. For production, register domain on OpenAI's allowlist and update `vite.config.ts`

### Agent Behavior
- Agent maintains conversation context across turns via thread-based memory
- Customer profile data is injected as context on each request
- Tools modify shared state that persists across the session

### Threading Model
- Default thread ID used when none provided: `"demo_default_thread"`
- Thread IDs map to customer profiles in `AirlineStateManager`
- Each thread maintains independent customer state

### Proxy Configuration
- Frontend proxies `/support/*` requests to backend
- Configured in `vite.config.ts` with `BACKEND_URL` override support
- Production deployments need domain allowlisting in `server.allowedHosts`