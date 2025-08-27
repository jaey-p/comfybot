# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development
```bash
# Development server with hot reload
pnpm dev

# Production build and start (recommended for testing)
pnpm build:local && pnpm start

# Code quality and testing
pnpm check              # Run all checks (lint, types, tests)
pnpm lint:fix           # Fix linting issues
pnpm check-types        # TypeScript type checking
pnpm test               # Run unit tests
pnpm test:watch         # Unit tests in watch mode
```

### Database Management
```bash
pnpm docker:pg          # Start local PostgreSQL container
pnpm db:generate        # Generate Drizzle migrations
pnpm db:push           # Push schema changes to DB
pnpm db:studio         # Open Drizzle Studio
pnpm db:migrate        # Run migrations
```

### E2E Testing
```bash
pnpm playwright:install # Install Playwright browsers (once)
pnpm test:e2e          # Run all E2E tests
pnpm test:e2e -- tests/agents/  # Run specific test suite
pnpm test:e2e -- tests/agents/agent-visibility.spec.ts --headed  # Debug specific test
```

### Docker & Deployment
```bash
pnpm docker-compose:up     # Start all services with Docker
pnpm docker-compose:down   # Stop Docker services
```

## Architecture Overview

### Core Technology Stack
- **Framework**: Next.js 15 with App Router and TypeScript
- **Database**: PostgreSQL with Drizzle ORM
- **AI Integration**: Vercel AI SDK v2 with multiple providers (OpenAI, Anthropic, Google, xAI, Ollama, OpenRouter)
- **Authentication**: Better Auth with OAuth support (GitHub, Google, Microsoft)
- **State Management**: Zustand with persistence
- **Styling**: Tailwind CSS with Radix UI components
- **Testing**: Vitest (unit), Playwright (E2E)
- **Code Quality**: Biome for linting/formatting

### Key Application Concepts

**MCP (Model Context Protocol) Integration**
- `src/lib/ai/mcp/` - Complete MCP client management system
- Supports both stdio and remote MCP servers with OAuth authentication
- Auto-disconnect after 30 minutes of inactivity
- Tools are dynamically loaded and made available to LLMs
- Custom per-user tool/server instructions via database customization tables

**Multi-AI Provider Architecture** 
- `src/lib/ai/models.ts` - Centralized model provider management
- Static and dynamic (OpenAI-compatible) provider support
- Tool call capability detection per model
- Fallback model handling

**Visual Workflow System**
- `src/lib/ai/workflow/` - Complete workflow engine using ts-edge for execution graphs
- Visual node-based editor with React Flow
- Node types: Input, LLM, Tool, HTTP, Template, Condition, Output
- Workflows become callable tools in chat via `@workflow_name` syntax
- Supports conditional branching with branch synchronization

**Custom Agents**
- Database-stored agents with custom instructions and tool access
- Callable via `@agent_name` mentions in chat
- Visibility controls (public, private, readonly)

**Built-in Tools**
- Web search via Exa AI
- Code execution (JavaScript/Python) in isolated environments  
- Data visualization (charts, tables)
- HTTP client for API requests

### Database Schema Structure

**Core Tables** (`src/lib/db/pg/schema.pg.ts`):
- `user` - User accounts with preferences
- `chat_thread` + `chat_message` - Conversation storage
- `agent` - Custom AI agents with instructions
- `workflow` + `workflow_node` + `workflow_edge` - Visual workflow definitions
- `mcp_server` - MCP server configurations
- `mcp_server_tool_custom_instructions` - Per-user tool customizations
- `archive` + `archive_item` - Chat archiving system
- `bookmark` - Bookmarked agents/workflows

### Component Architecture

**Layout Structure**:
- `src/app/(chat)/layout.tsx` - Main chat layout with sidebar
- `src/components/layouts/` - Reusable layout components
- `src/components/ui/` - Shadcn UI component library

**Chat System**:
- `src/components/chat-bot.tsx` - Main chat interface with AI SDK useChat hook
- Real-time streaming with tool execution support
- Manual tool confirmation mode available
- Voice chat integration via OpenAI Realtime API

**State Management**:
- `src/app/store/index.ts` - Zustand store with persistence
- Handles chat model selection, tool preferences, MCP server allowlists
- Thread-specific mention tracking

### Path Aliases
```typescript
"ui/*": ["./src/components/ui/*"]
"auth/*": ["./src/lib/auth/*"] 
"app-types/*": ["./src/types/*"]
"lib/*": ["./src/lib/*"]
"@/*": ["./src/*"]
```

### Development Guidelines

**Code Quality Requirements**:
- All new logic requires comprehensive unit tests
- UI changes require E2E test updates  
- Run `pnpm check` before commits
- PR titles must follow Conventional Commits format

**Testing Requirements**:
- PostgreSQL database (use `pnpm docker:pg`)
- At least one LLM provider API key
- `BETTER_AUTH_SECRET` environment variable

**MCP Development**:
- New MCP tools are auto-discovered and made available to LLMs
- Test MCP servers using the `/mcp/test` page
- MCP clients auto-disconnect after 30 minutes of inactivity
- OAuth flow supported for authenticated MCP servers

**Workflow Development**:
- New node types require: interface definition, executor implementation, validation, and UI config component
- Node executors are in `src/lib/ai/workflow/executor/node-executor.ts`
- Workflow examples in `src/lib/ai/workflow/examples/`

### Environment Setup
The `.env` file is auto-created on `pnpm i`. Minimum required:
- One LLM provider API key (OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.)
- `POSTGRES_URL` for database connection
- `BETTER_AUTH_SECRET` for authentication

Optional but recommended:
- `EXA_API_KEY` for enhanced web search capabilities