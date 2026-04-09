# GEMINI.md

## Project Overview
UIGen is an AI-powered React component generator with live preview. It allows users to describe UI components in a natural language chat interface, which are then generated and rendered in real-time.

### Core Technologies
- **Framework**: Next.js 15 (App Router)
- **UI Library**: React 19
- **Styling**: Tailwind CSS v4
- **AI Integration**: Vercel AI SDK with Anthropic Claude (claude-haiku-4-5)
- **Database**: SQLite with Prisma
- **Runtime Polyfill**: `node-compat.cjs` for Windows/Node compatibility issues in certain environments.
- **Editor**: Monaco Editor (`@monaco-editor/react`)
- **Transformation**: Babel Standalone for client-side JSX/TSX transformation.

### Key Architectural Concepts
- **Virtual File System (VFS)**: Located in `src/lib/file-system.ts`. Components are not written to the actual disk during generation. Instead, they live in an in-memory tree that is serialized/deserialized between the client and server.
- **Live Preview**: `src/lib/transform/jsx-transformer.ts` transforms the VFS files into a self-contained HTML document with an import map (pointing to esm.sh), which is rendered in an iframe (`PreviewFrame`).
- **Tool-Based Generation**: The AI uses two primary tools:
    - `str_replace_editor`: For creating and modifying files using precise string replacement or full file writes.
    - `file_manager`: For listing, deleting, or renaming files in the VFS.
- **Mock Provider**: If `ANTHROPIC_API_KEY` is missing, the app falls back to a `MockLanguageModel` (`src/lib/provider.ts`) that returns static components (Counter, Form, Card) to demonstrate functionality.

## Building and Running

### Prerequisites
- Node.js 18+
- npm

### Setup
```bash
npm run setup
```
This installs dependencies, generates the Prisma client, and runs migrations.

### Development
```bash
npm run dev
```
Starts the Next.js development server with Turbopack on `http://localhost:3000`.

### Testing
```bash
npm run test
```
Runs the test suite using Vitest.

### Linting
```bash
npm run lint
```
Runs ESLint.

### Database Management
- `npx prisma studio`: Open Prisma Studio to view/edit data.
- `npm run db:reset`: Reset the SQLite database.

## Development Conventions

### File Structure
- `src/actions`: Server Actions for project and auth management.
- `src/app`: Next.js App Router pages and API routes.
- `src/components`: React components organized by feature (auth, chat, editor, preview, ui).
- `src/hooks`: Custom React hooks (e.g., `use-auth`).
- `src/lib`: Core logic, contexts, tools, and utility functions.
- `src/lib/contexts`: React Contexts for global state (FileSystem, Chat).

### State Management
- **FileSystemProvider**: Manages the VFS state and handles tool call execution.
- **ChatProvider**: Manages the chat history and AI streaming state using Vercel AI SDK.

### Testing Practices
- Use Vitest for unit and integration tests.
- Place tests in `__tests__` directories adjacent to the implementation.
- Focus on testing the VFS logic and JSX transformation, as these are the project's core "engine".

### AI Development
- System prompts are located in `src/lib/prompts/`.
- Tool definitions are in `src/lib/tools/`.
- When modifying the generation flow, ensure that tool calls are handled both on the server (to update the VFS for the next turn) and on the client (to update the UI and editor).

## Key Files
- `src/lib/file-system.ts`: The heart of the application's data structure.
- `src/app/api/chat/route.ts`: The primary interface with the AI.
- `src/lib/transform/jsx-transformer.ts`: Responsible for the "Live" part of the preview.
- `node-compat.cjs`: Important polyfill for standard Node.js globals in certain environments.
