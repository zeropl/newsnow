# CLAUDE.md - NewsNow Project Guide for AI Assistants

> This file provides comprehensive guidance for AI assistants (like Claude Code) working on the NewsNow project.

## Project Overview

**NewsNow** is an elegant, real-time news aggregation platform that collects and displays trending news from 40+ sources in a clean, organized interface. It's a full-stack TypeScript application with React frontend and Nitro backend.

- **Version**: 0.0.36
- **License**: MIT
- **Author**: ourongxing (https://github.com/ourongxing/)
- **Repository**: https://github.com/ourongxing/newsnow
- **Primary Language**: Chinese (with plans for internationalization)

### Key Features
- Real-time news aggregation from 40+ sources
- GitHub OAuth authentication with cloud sync
- Intelligent caching system (30-minute default TTL)
- Drag-and-drop column customization
- Dark mode support
- PWA (Progressive Web App) support
- MCP (Model Context Protocol) server integration

## Technology Stack

### Frontend
- **React 19** - Latest React with concurrent features
- **TypeScript 5.8** - Strict type checking
- **Vite 6.2** - Build tool and dev server
- **TanStack Router 1.112** - Type-safe file-based routing
- **TanStack Query 5.66** - Server state management
- **Jotai 2.12** - Atomic client state management
- **UnoCSS 66.0** - Atomic CSS framework
- **Framer Motion 12.4** - Animations

### Backend
- **Nitro** - Universal server framework
- **H3 1.15** - HTTP server
- **db0 0.3** - Universal database layer
- **Cheerio 1.0** - HTML parsing for scraping
- **fast-xml-parser 5.0** - RSS/Atom feed parsing
- **Jose 6.0** - JWT authentication

### Build & Dev Tools
- **pnpm 10.14** - Package manager (required)
- **ESLint 9.21** - Linting
- **Vitest 3.0** - Testing
- **tsx 4.19** - TypeScript execution

## Project Structure

```
/home/user/newsnow/
├── src/                          # Frontend React application
│   ├── components/              # React components
│   │   ├── column/             # News column components (Card, Dnd, etc.)
│   │   ├── header/             # Header and navigation
│   │   └── common/             # Reusable UI components
│   ├── routes/                 # TanStack Router routes (file-based)
│   ├── atoms/                  # Jotai state atoms
│   ├── hooks/                  # Custom React hooks
│   ├── utils/                  # Frontend utilities
│   └── styles/                 # Global styles
│
├── server/                      # Backend Nitro server
│   ├── api/                    # API route handlers
│   │   ├── s/                 # Source endpoints (/api/s)
│   │   ├── oauth/             # GitHub OAuth (/api/oauth/github)
│   │   └── me/                # User endpoints (/api/me)
│   ├── sources/                # Data source scrapers (40+ files)
│   ├── database/               # Database models and utilities
│   ├── middleware/             # HTTP middleware (auth, etc.)
│   ├── utils/                  # Server utilities
│   └── mcp/                    # MCP server implementation
│
├── shared/                      # Shared code between client/server
│   ├── types.ts                # TypeScript type definitions
│   ├── pre-sources.ts          # Source configurations
│   ├── sources.json            # Generated source metadata
│   ├── metadata.ts             # Column and category metadata
│   ├── consts.ts               # Shared constants
│   └── dir.ts                  # Path utilities
│
├── scripts/                     # Build scripts
│   ├── source.ts               # Generate sources.json and pinyin.json
│   └── favicon.ts              # Fetch source favicons
│
├── tools/                       # Custom build tools
│   └── rollup-glob.ts          # Rollup plugin for glob imports
│
├── public/                      # Static assets
│
└── Configuration files:
    ├── vite.config.ts           # Vite configuration
    ├── nitro.config.ts          # Nitro server configuration
    ├── uno.config.ts            # UnoCSS styling configuration
    ├── pwa.config.ts            # PWA configuration
    ├── tsconfig.*.json          # TypeScript configurations
    ├── eslint.config.mjs        # ESLint configuration
    ├── vitest.config.ts         # Test configuration
    ├── wrangler.toml            # Cloudflare deployment config
    ├── Dockerfile               # Docker container configuration
    └── docker-compose.yml       # Docker Compose setup
```

## Key Concepts & Patterns

### 1. Source System

**Sources** are the core concept - each represents a news feed from a platform (e.g., Weibo, Zhihu, GitHub).

#### Source Registration (3-step process):

**Step 1**: Register in `shared/pre-sources.ts`
```typescript
"newsource": {
  name: "Source Name",      // Display name
  color: "blue",            // Theme color
  home: "https://...",      // Homepage URL
  column: "tech",           // Category: china/world/tech/finance
  type: "hottest",          // Type: hottest/realtime
  interval: Time.Medium,    // Refresh interval
  sub: {                    // Optional sub-sources
    "subsource": {
      title: "Sub Title",
      column: "china",
      type: "realtime"
    }
  }
}
```

**Step 2**: Implement fetcher in `server/sources/newsource.ts`
```typescript
export default defineSource({
  "newsource": async () => {
    const data = await myFetch(url)
    return data.map(item => ({
      id: item.id,              // Unique identifier
      title: item.title,        // News title
      url: item.url,            // Link to content
      mobileUrl: item.mobile,   // Optional mobile URL
      pubDate: item.date,       // Publication date
      extra: {
        info: "metadata",       // Additional info (views, author, etc.)
        hover: "description",   // Hover tooltip text
        icon: "icon_url",       // Icon URL
        diff: 3                 // Ranking change (for hottest)
      }
    }))
  }
})
```

**Step 3**: Run `npm run presource` to regenerate metadata files

#### Helper Functions:

- `defineSource(fn)` - Type-safe source definition
- `defineRSSSource(url, options?)` - Quick RSS source creation
- `defineRSSHubSource(route, options?)` - RSSHub integration
- `proxySource(proxyUrl, source)` - Cloudflare Pages proxy pattern

#### Source Types:

1. **Direct API**: Fetch structured JSON from official APIs
2. **HTML Scraping**: Use Cheerio to parse HTML pages
3. **RSS/Atom**: Parse XML feeds with fast-xml-parser
4. **RSSHub**: Use RSSHub aggregator
5. **GraphQL**: Query GraphQL APIs

### 2. Caching Strategy

**Location**: `server/database/cache.ts`

- **Default TTL**: 30 minutes
- **Refresh Intervals**: Source-specific (2-60 minutes based on update frequency)
- **Smart Caching**: Returns stale cache on fetch failure
- **Force Refresh**: Authenticated users can bypass cache

**Cache Flow**:
```
Request → Check Cache → If Valid: Return Cache
                     → If Invalid: Fetch → Update Cache → Return Fresh Data
                     → If Fetch Fails: Return Stale Cache (if exists)
```

### 3. State Management

#### Client State (Jotai):
- `focusSourcesAtom` - User's favorited sources
- `currentColumnIDAtom` - Currently selected column
- `currentSourcesAtom` - Sources in current column
- `primitiveMetadataAtom` - Core metadata with sync

#### Server State (TanStack Query):
- News data fetching with automatic refetching
- Optimistic updates
- Stale-while-revalidate pattern

### 4. Authentication Flow

1. User clicks "Login with GitHub"
2. Redirects to GitHub OAuth
3. GitHub redirects to `/api/oauth/github` with code
4. Server exchanges code for access token
5. Server creates/updates user in database
6. Server generates JWT token
7. JWT stored in cookie for session management

**Files**: `server/api/oauth/github.post.ts`, `server/middleware/auth.ts`

### 5. Type Safety

All types are centralized in `shared/types.ts`:

- `SourceID` - Union type of all valid source IDs (auto-generated from pre-sources)
- `NewsItem` - Structure for news items
- `SourceResponse` - API response structure
- `Column`, `Metadata` - Column and metadata types

TypeScript is configured strictly (`tsconfig.*.json`) for maximum type safety.

## Development Workflow

### Setup (First Time)

```bash
# Enable pnpm
corepack enable

# Install dependencies
pnpm install

# Start development server
pnpm dev
```

### Common Commands

```bash
# Development
pnpm dev                 # Start dev server (runs presource first)
pnpm presource           # Generate source metadata & fetch favicons

# Build
pnpm build              # Production build
pnpm typecheck          # Type checking without build

# Testing & Linting
pnpm test               # Run tests with Vitest
pnpm lint               # Lint code with ESLint

# Deployment
pnpm preview            # Preview production build (Cloudflare Pages mode)
pnpm deploy             # Deploy to Cloudflare Pages
pnpm start              # Start production server (Node.js)

# Version Management
pnpm release            # Bump version and create git tag
```

### Development Server

- **Frontend**: `http://localhost:5173` (Vite)
- **Backend**: Integrated with Vite via vite-plugin-with-nitro
- **Hot Module Replacement**: Enabled for frontend
- **Auto-restart**: Backend restarts on file changes

### Environment Variables

Create `.env.server` (see `example.env.server`):

```env
# Required for authentication & sync
G_CLIENT_ID=                # GitHub OAuth Client ID
G_CLIENT_SECRET=            # GitHub OAuth Client Secret
JWT_SECRET=                 # JWT signing secret

# Database
INIT_TABLE=true             # Initialize DB on first run
ENABLE_CACHE=true           # Enable caching

# Optional
PRODUCTHUNT_API_TOKEN=      # Product Hunt API access
```

## Common Tasks

### Adding a New Source

**See `CONTRIBUTING.md` for detailed guide**

1. **Register source** in `shared/pre-sources.ts`
2. **Implement fetcher** in `server/sources/newsource.ts`
3. **Regenerate metadata**: `pnpm presource`
4. **Test**: `pnpm dev` and verify in browser
5. **Commit**: `git commit -m "feat(source): add newsource"`

### Modifying Existing Source

1. Update implementation in `server/sources/[source].ts`
2. If changing metadata, update `shared/pre-sources.ts`
3. Run `pnpm presource` if metadata changed
4. Test changes locally

### Adding API Endpoint

1. Create file in `server/api/` (e.g., `server/api/test.get.ts`)
2. Export default event handler:
```typescript
export default defineEventHandler(async (event) => {
  return { message: "Hello" }
})
```
3. Access at `/api/test`

### Styling Components

Uses **UnoCSS** (atomic CSS):

```tsx
// Example
<div className="flex items-center gap-2 p-4 bg-blue-500 text-white rounded">
  Content
</div>
```

**Configuration**: `uno.config.ts`

### State Management

**Client State (Jotai)**:
```tsx
// Define atom
export const myAtom = atom(initialValue)

// Use in component
const [value, setValue] = useAtom(myAtom)
const value = useAtomValue(myAtom)
const setValue = useSetAtom(myAtom)
```

**Server State (TanStack Query)**:
```tsx
const { data, isLoading, refetch } = useQuery({
  queryKey: ['news', sourceId],
  queryFn: () => fetchNews(sourceId)
})
```

## Important Files Reference

### Configuration Files

| File | Purpose |
|------|---------|
| `vite.config.ts` | Vite build configuration, plugins, aliases |
| `nitro.config.ts` | Backend server configuration, API routes |
| `uno.config.ts` | UnoCSS styling configuration, theme, shortcuts |
| `pwa.config.ts` | PWA manifest, service worker, offline support |
| `tsconfig.*.json` | TypeScript compiler options |
| `eslint.config.mjs` | Linting rules and configuration |
| `wrangler.toml` | Cloudflare Pages deployment settings |

### Core Application Files

| File | Purpose |
|------|---------|
| `shared/types.ts` | All TypeScript type definitions |
| `shared/pre-sources.ts` | Source configuration registry |
| `shared/metadata.ts` | Column and category definitions |
| `shared/consts.ts` | Shared constants (Time intervals, etc.) |
| `server/utils/source.ts` | Source helper functions (defineSource, etc.) |
| `server/database/index.ts` | Database initialization and models |
| `server/middleware/auth.ts` | JWT authentication middleware |
| `src/atoms/index.ts` | Jotai state atoms |
| `src/components/column/card.tsx` | Main news card component |

### Generated Files (Do Not Edit Manually)

| File | Generated By | Purpose |
|------|-------------|---------|
| `shared/sources.json` | `scripts/source.ts` | Source metadata (IDs, names, colors) |
| `shared/pinyin.json` | `scripts/source.ts` | Pinyin search index for Chinese sources |
| `public/sources/[id].png` | `scripts/favicon.ts` | Source favicon images |
| `imports.app.d.ts` | `unimport` | Auto-import type definitions |
| `src/routeTree.gen.ts` | TanStack Router | Generated route tree |

## Coding Conventions

### Naming Conventions

- **Source IDs**: lowercase-kebab-case (`zhihu`, `github-trending-today`)
- **Components**: PascalCase (`NewsCard`, `ColumnHeader`)
- **Hooks**: camelCase with `use` prefix (`useFetch`, `useAuth`)
- **Atoms**: camelCase with `Atom` suffix (`focusSourcesAtom`)
- **Types/Interfaces**: PascalCase (`NewsItem`, `SourceResponse`)
- **Constants**: UPPER_SNAKE_CASE or camelCase based on context
- **Files**: kebab-case for most, PascalCase for React components

### Code Style

- **2 spaces** indentation
- **No semicolons** (enforced by ESLint)
- **Double quotes** for strings
- **Arrow functions** preferred
- **Destructuring** when possible
- **Type annotations** for function parameters and returns

### Import Order

1. Node.js built-ins
2. External packages
3. Internal modules (aliases: `~`, `@shared`, `#`)
4. Relative imports
5. Types (if separated)

Example:
```typescript
import { join } from "node:path"
import React from "react"
import { defineSource } from "#/utils/source"
import type { NewsItem } from "@shared/types"
import { formatDate } from "./utils"
```

### Type Safety Best Practices

- Use `satisfies` operator for type checking with inference
- Prefer `interface` for object shapes, `type` for unions/intersections
- Use `as const` for literal types
- Avoid `any` - use `unknown` if type is truly unknown
- Use discriminated unions for variant types

### Error Handling

- Use `try-catch` for async operations
- Return graceful fallbacks (e.g., stale cache on fetch failure)
- Log errors with `consola` library
- Display user-friendly error messages in UI

## Testing & Debugging

### Testing

```bash
pnpm test          # Run all tests
pnpm test --watch  # Watch mode
```

**Test files**: Co-located with source files (`.test.ts`, `.spec.ts`)

### Debugging

**Frontend**:
- React DevTools
- TanStack Router DevTools (included in dev mode)
- TanStack Query DevTools (included in dev mode)

**Backend**:
- Console logs with `consola`
- Node.js debugger: `node --inspect`

**Network**:
- Browser DevTools Network tab
- Check `/api/s` endpoint responses

### Common Issues

**Sources not loading**:
1. Check cache in database (`.data/newsnow.db` or Cloudflare D1)
2. Verify source implementation in `server/sources/`
3. Check network requests in browser DevTools
4. Look for errors in server console

**Build failures**:
1. Run `pnpm typecheck` to identify type errors
2. Check ESLint errors: `pnpm lint`
3. Verify all dependencies installed: `pnpm install`
4. Clear cache: `rm -rf node_modules/.vite`

**Authentication not working**:
1. Verify environment variables in `.env.server`
2. Check GitHub OAuth app configuration (callback URL)
3. Inspect JWT token in browser cookies
4. Check middleware logs for auth errors

## Deployment

### Supported Platforms

1. **Cloudflare Pages** (Recommended)
2. **Vercel**
3. **Node.js Server**
4. **Docker**
5. **Bun**

### Cloudflare Pages Deployment

**Prerequisites**:
- Cloudflare account
- D1 database created
- GitHub OAuth app configured

**Setup**:
1. Configure `wrangler.toml`:
```toml
name = "newsnow"
compatibility_date = "2024-12-01"
pages_build_output_dir = "dist/output/public"

[[d1_databases]]
binding = "NEWSNOW_DB"
database_name = "your-db-name"
database_id = "your-db-id"

[vars]
INIT_TABLE = "true"
ENABLE_CACHE = "true"
```

2. Add secrets to Cloudflare:
```bash
wrangler pages secret put G_CLIENT_ID
wrangler pages secret put G_CLIENT_SECRET
wrangler pages secret put JWT_SECRET
```

3. Deploy:
```bash
pnpm deploy
```

### Docker Deployment

```bash
# Build and run with docker-compose
docker compose up -d

# Or build manually
docker build -t newsnow .
docker run -p 4444:4444 \
  -e G_CLIENT_ID=... \
  -e G_CLIENT_SECRET=... \
  -e JWT_SECRET=... \
  newsnow
```

Data persisted in `./data` volume.

### Environment-Specific Builds

The build process adapts based on environment:
- **Cloudflare Pages**: Detected via `CF_PAGES` env var
- **Vercel**: Auto-detected
- **Node.js**: Default build

## Database Schema

### Cache Table
```sql
CREATE TABLE cache (
  id TEXT PRIMARY KEY,        -- Source ID
  data TEXT NOT NULL,         -- JSON serialized NewsItem[]
  updated INTEGER NOT NULL    -- Timestamp
);
```

### User Table
```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,        -- GitHub user ID
  email TEXT,                 -- User email
  type TEXT,                  -- User type (github)
  data TEXT,                  -- JSON serialized user data
  created INTEGER NOT NULL,   -- Creation timestamp
  updated INTEGER NOT NULL    -- Update timestamp
);
```

## API Reference

### GET `/api/s`

Fetch news items for a source.

**Query Parameters**:
- `id` (required): Source ID
- `cache` (optional): Set to `"0"` to bypass cache (requires auth)

**Response**:
```typescript
{
  status: "success" | "cache",
  id: string,
  updatedTime: number,
  items: NewsItem[]
}
```

### POST `/api/oauth/github`

GitHub OAuth callback endpoint.

**Body**:
```typescript
{
  code: string,     // OAuth authorization code
  state?: string    // Optional state parameter
}
```

### GET `/api/me`

Get current user information (requires auth).

**Response**:
```typescript
{
  id: string,
  email: string,
  data: object
}
```

### POST `/api/me/sync`

Sync user preferences (requires auth).

**Body**:
```typescript
{
  data: {
    updatedTime: number,
    data: Record<ColumnID, SourceID[]>,
    action: "init" | "manual" | "sync"
  }
}
```

### GET `/api/proxy/img.png`

Image proxy for CORS issues.

**Query Parameters**:
- `url`: Image URL to proxy

## Performance Optimization

### Frontend
- Code splitting via TanStack Router
- Lazy loading components with `React.lazy()`
- Memoization with `useMemo`, `useCallback`, `memo()`
- Virtual scrolling for long lists
- Service worker caching (PWA)

### Backend
- Multi-level caching (database, CDN)
- Intelligent cache TTL based on source update frequency
- Parallel requests with `Promise.all()`
- Retry logic with exponential backoff
- Efficient database queries

### Build
- Tree shaking with Rollup
- Asset optimization
- CSS atomic classes (minimal CSS bundle)
- Pre-compression (gzip, brotli)

## MCP Server Integration

NewsNow provides an MCP (Model Context Protocol) server for AI assistants.

**Configuration** (in Claude Desktop config):
```json
{
  "mcpServers": {
    "newsnow": {
      "command": "npx",
      "args": ["-y", "newsnow-mcp-server"],
      "env": {
        "BASE_URL": "https://newsnow.example.com"
      }
    }
  }
}
```

**Capabilities**:
- List available news sources
- Fetch news from specific sources
- Query trending topics

**Implementation**: `server/mcp/index.ts`

## Troubleshooting

### "Cannot find module" errors
```bash
pnpm install
pnpm presource
```

### TypeScript errors after pulling changes
```bash
pnpm typecheck
# Fix reported errors
```

### Sources showing old data
```bash
# Check cache table in database
# For SQLite: sqlite3 .data/newsnow.db "SELECT * FROM cache;"
# Try force refresh in UI (requires login)
```

### Build fails on deployment platform
```bash
# Verify Node.js version (requires >= 20)
# Check platform-specific build settings
# Review build logs for specific errors
```

### Database not initializing
```bash
# Ensure INIT_TABLE=true in environment variables
# Check database permissions
# Verify database configuration in wrangler.toml (for Cloudflare)
```

## Resources

- **Project Repository**: https://github.com/ourongxing/newsnow
- **Contributing Guide**: `CONTRIBUTING.md`
- **Deployment Guide**: `README.md` (Deployment section)
- **UnoCSS Docs**: https://unocss.dev/
- **TanStack Router**: https://tanstack.com/router
- **Nitro Docs**: https://nitro.unjs.io/
- **db0 Docs**: https://db0.unjs.io/

## Notes for AI Assistants

### When Adding Features
1. Always run `pnpm typecheck` before committing
2. Update relevant documentation if public APIs change
3. Follow existing patterns and conventions
4. Test locally with `pnpm dev` before pushing

### When Fixing Bugs
1. Identify the affected area (frontend/backend/shared)
2. Check for similar patterns in the codebase
3. Add tests if applicable
4. Verify fix doesn't break existing functionality

### When Refactoring
1. Maintain backward compatibility
2. Update type definitions if signatures change
3. Run full test suite
4. Update documentation

### Code Generation Guidelines
- Use existing helpers and utilities
- Follow the established architectural patterns
- Prioritize type safety
- Write self-documenting code with clear names
- Add comments for complex logic only

### Best Practices
- Check `shared/types.ts` for existing types before creating new ones
- Use `defineSource()` helper for source implementations
- Leverage auto-imports (configured in `vite.config.ts`)
- Prefer composition over inheritance
- Keep components small and focused

---

**Last Updated**: 2025-11-14 (v0.0.36)

For questions or clarifications about this guide, refer to the project repository or existing code examples.
