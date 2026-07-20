# 1. Prisma Config Environment Variable Exception

**Status:** Accepted

## Context
Our architectural rules strictly mandate centralized environment variable parsing via `src/config/env.js` (using Zod for validation). However, `prisma.config.ts` is executed solely by the Prisma CLI layer, completely outside the standard application module lifecycle.

## Decision
We accept that `prisma.config.ts` reads `process.env.DATABASE_URL` directly as an isolated CLI-layer exception. It will not be routed through `src/config/env.js`.

## Consequences
- The Prisma CLI won't run into potential issues importing or executing app-level initialization code.
- This is a documented, intentional exception so it will not be flagged as a violation in future codebase audits.
