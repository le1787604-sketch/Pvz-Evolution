# PvZ Fusion Hub

## Overview

PvZ Fusion Hub is a full-stack web application for managing and browsing Plants vs Zombies fusion mods. It serves as a content hub where users can explore mod releases, browse a "FusionDex" of fusion recipes, watch curated YouTube videos, and where admins can manage all content through an admin panel. The app supports Vietnamese and English languages.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend

- **Framework**: React 18 with TypeScript, built with Vite
- **Routing**: Wouter (lightweight client-side router)
- **State/Data Fetching**: TanStack React Query for server state management
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives
- **Styling**: Tailwind CSS with a custom dark gaming theme using CSS variables. Custom fonts: Fredoka (display), Rubik (body), Press Start 2P (gaming accents)
- **Forms**: React Hook Form with Zod resolvers for validation
- **Internationalization**: Custom Zustand-based language store supporting Vietnamese (`vi`) and English (`en`)
- **Path aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`

**Key pages**: Home, Mods, FusionDex, Videos, Admin, Login, 404

### Backend

- **Framework**: Express.js running on Node with TypeScript (via tsx)
- **Authentication**: Passport.js with Local Strategy (email/password). Sessions stored in PostgreSQL via `connect-pg-simple`. Passwords hashed with bcryptjs. First user auto-gets ADMIN role.
- **File uploads**: Multer for multipart/form-data handling. Upload endpoint at `POST /api/upload`
- **API structure**: RESTful JSON API under `/api/` prefix. Route contracts defined in `shared/routes.ts` with Zod schemas for input validation and response typing
- **Dev server**: Vite dev server runs as middleware in development; static files served from `dist/public` in production

### Shared Layer (`shared/`)

- **`schema.ts`**: Drizzle ORM table definitions and Zod insert schemas. Tables: `users`, `mods`, `fusions`, `videos`
- **`routes.ts`**: API contract definitions with method, path, input/output Zod schemas for type-safe client-server communication

### Database

- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Database**: PostgreSQL (required, connection via `DATABASE_URL` env var)
- **Session store**: PostgreSQL-backed sessions via `connect-pg-simple`
- **Schema push**: `npm run db:push` uses drizzle-kit to push schema changes
- **Migrations**: Output to `./migrations` directory

### Database Schema

| Table | Key Columns |
|-------|------------|
| `users` | id, email, password, role (ADMIN/USER), createdAt |
| `mods` | id, title, version, description, changelog, imageUrl, fileUrl, createdAt |
| `fusions` | id, name, type (Plant/Zombie), recipe, ability, imageUrl, fileUrl, videoUrl, createdAt |
| `videos` | id, title, thumbnailUrl, youtubeUrl, description, createdAt |

### Build System

- **Client**: Vite builds to `dist/public`
- **Server**: esbuild bundles server code to `dist/index.cjs`, with strategic dependency bundling for faster cold starts
- **Dev**: `npm run dev` runs both Vite HMR and Express via tsx
- **Production**: `npm run build` then `npm start`

### Key Design Decisions

1. **Shared route contracts**: API contracts in `shared/routes.ts` ensure client and server stay in sync with typed paths, methods, and Zod schemas. Client hooks validate responses against these schemas.
2. **Auto-registration of first user as admin**: The first user to sign up automatically gets the ADMIN role, simplifying initial setup.
3. **Database seeding**: The server seeds sample mods and fusions on startup if the database is empty.
4. **Dark-only theme**: No light mode toggle; the app uses a fixed dark gaming-inspired color palette.

## External Dependencies

- **PostgreSQL**: Required. Connection string via `DATABASE_URL` environment variable
- **Session Secret**: `SESSION_SECRET` env var (falls back to a default in dev)
- **Google Fonts**: Fredoka, Rubik, Press Start 2P, DM Sans, Fira Code, Geist Mono loaded via CDN
- **YouTube**: Videos page links out to YouTube URLs (no embed API integration, just URL references)
- **Replit Plugins**: `@replit/vite-plugin-runtime-error-modal`, `@replit/vite-plugin-cartographer`, `@replit/vite-plugin-dev-banner` used in development on Replit