# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, Claude generates the code via tool calls, and a live preview renders the result in-browser using a virtual file system (no files written to disk).

## Commands

- `npm run setup` — install deps, generate Prisma client, run migrations
- `npm run dev` — start dev server (Next.js + Turbopack) on localhost:3000
- `npm run test` — run all tests with Vitest
- `npx vitest run src/path/to/test.ts` — run a single test file
- `npm run lint` — ESLint
- `npm run db:reset` — reset SQLite database

**Note:** Scripts use `cross-env` for Windows compatibility. The `NODE_OPTIONS` flag loads `node-compat.cjs` as a polyfill.

## Architecture

### AI Chat Flow
1. User sends message via `ChatProvider` (`src/lib/contexts/chat-context.tsx`) which uses Vercel AI SDK's `useChat`
2. POST to `/api/chat` route (`src/app/api/chat/route.ts`) which calls `streamText` with Claude (haiku-4-5) and two tools: `str_replace_editor` and `file_manager`
3. Tool calls manipulate a server-side `VirtualFileSystem` instance; the same tool calls are replayed client-side via `handleToolCall` in `FileSystemProvider`
4. Files are serialized as JSON and sent with each request (no server-side persistence of file state between requests for anonymous users)

### Virtual File System
`src/lib/file-system.ts` — in-memory tree of `FileNode` objects. Supports create/read/update/delete/rename and str_replace/insert operations. Serializes to plain objects for transport. This is the core abstraction — generated components live here, not on disk.

### Live Preview
`src/lib/transform/jsx-transformer.ts` — transforms JSX/TSX files using Babel standalone, creates blob URLs, builds an import map pointing `@/` aliases and package imports to esm.sh. `createPreviewHTML` produces a self-contained HTML document loaded in an iframe (`PreviewFrame` component).

### Provider Fallback
`src/lib/provider.ts` — if `ANTHROPIC_API_KEY` is empty, a `MockLanguageModel` generates static counter/form/card components instead of calling the API. The mock implements the full `LanguageModelV1` interface.

### Auth
JWT-based sessions stored in httpOnly cookies (`src/lib/auth.ts`). Users can sign up/in or use the app anonymously. Anonymous work is tracked client-side via `anon-work-tracker.ts`.

### Data Model
SQLite via Prisma. Two models: `User` and `Project`. Projects store `messages` (JSON array) and `data` (serialized file system) as strings. Projects optionally belong to a user.

### Routing
- `/` — anonymous landing or redirect to most recent project for authenticated users
- `/[projectId]` — project workspace (chat + editor + preview)

### Key Contexts
- `FileSystemProvider` — owns the VirtualFileSystem instance, handles tool call side effects
- `ChatProvider` — wraps Vercel AI SDK's useChat, passes serialized files with each request

## Testing

Tests use Vitest + jsdom + React Testing Library. Test files live in `__tests__` directories adjacent to the code they test.
