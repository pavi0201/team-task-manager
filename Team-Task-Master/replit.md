# Team Task Manager

## Overview

Full-stack team task management app with React + Vite frontend, Express 5 backend, PostgreSQL, and JWT auth. Role-based access for admins and members.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **Frontend**: React + Vite + Wouter + TanStack Query + shadcn/ui + Tailwind
- **Backend**: Express 5 + Drizzle ORM + PostgreSQL + JWT (jsonwebtoken + bcryptjs)
- **API codegen**: Orval (contract-first OpenAPI → hooks + Zod schemas)
- **Build**: esbuild (API server)

## Architecture

```
artifacts/
  api-server/     → Express API at /api
  task-manager/   → React+Vite frontend at /
lib/
  api-spec/       → OpenAPI spec (source of truth)
  api-client-react/ → Orval-generated TanStack Query hooks
  api-zod/        → Orval-generated Zod validation schemas
  db/             → Drizzle schema + migrations
```

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks from OpenAPI spec (then manually fix `lib/api-zod/src/index.ts` to only export `export * from "./generated/api"`)
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Auth

- JWT stored in `localStorage` as `auth_token`
- `setAuthTokenGetter` from `@workspace/api-client-react` injects token into all API requests
- JWT secret from `SESSION_SECRET` env var
- Roles: `admin` (full CRUD) and `member` (update task status only)

## Seed Data

- Admin: alice@example.com / admin123
- Member: bob@example.com / member123
- Member: carol@example.com / member123
- 3 projects, 10 tasks (various statuses/priorities, some overdue)

## Database Schema

- `users` — id, name, email, password_hash, role, created_at
- `projects` — id, name, description, created_at, updated_at
- `project_members` — project_id, user_id (junction table)
- `tasks` — id, title, description, status, priority, due_date, project_id, assignee_id, created_at, updated_at

## Pages & Routes

- `/login` — Login
- `/register` — Register (with role selector)
- `/dashboard` — Stats summary + overdue tasks list
- `/projects` — Project list (admin: create/delete)
- `/projects/:id` — Project detail with members + tasks management
- `/tasks` — All tasks with status/priority filters

## Important Notes

- After running codegen, `lib/api-zod/src/index.ts` must only contain `export * from "./generated/api"` — orval overwrites it with stale exports
- Overdue = dueDate in the past AND status !== 'done', highlighted in red throughout the app
- Admin-only: create/delete project, create/edit/delete task, add/remove members
