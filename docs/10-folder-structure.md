# Folder Structure

## Philosophy

The project follows the principle:

**One Concern → One Home**

Every responsibility has exactly one official location.

Avoid duplicate responsibilities.

Folder structure should remain stable throughout the lifetime of the project.

---

# Backend

src/

## routes/

Responsibilities

- Register API endpoints
- Apply middleware
- Forward requests to controllers

Never:

- Business logic
- Database queries

---

## controllers/

Responsibilities

- Receive HTTP request
- Parse request
- Call services
- Return HTTP response

Never:

- Business logic
- Prisma queries
- Validation logic

---

## services/

Responsibilities

Own all business logic.

Examples

- Authentication
- Store approval
- Product management
- Membership workflows

---

## repositories/

Responsibilities

Database access only.

Examples

- Prisma queries
- Transactions
- Query helpers

Repositories never contain business rules.

---

## validators/

Responsibilities

Input validation.

Shared validation should live here.

Avoid duplicate validation.

---

## middleware/

Responsibilities

Authentication

Authorization

Request preprocessing

Never place business logic here.

---

## permissions/

Responsibilities

Permission definitions.

Permission helpers.

Centralized authorization rules.

---

## constants/

Responsibilities

Shared constants.

Examples

- Role names
- Permission names
- Status values

Avoid magic strings.

---

## config/

Responsibilities

Application configuration.

Environment configuration.

External service configuration.

---

## utils/

Responsibilities

Generic helper functions.

Should not contain business logic.

---

## prisma/

Responsibilities

Prisma schema

Migrations

Seed files

---

# Frontend

src/

## pages/

Route-level pages.

---

## components/

Reusable UI.

---

## layouts/

Shared layouts.

---

## hooks/

Reusable React hooks.

---

## services/

API communication.

Axios configuration.

Never business rules.

---

## context/

React Context providers.

---

## constants/

Shared constants, such as API endpoints.

---

## utils/

Shared helper functions.

---

## assets/

Images

Icons

Fonts

---

## styles/

Global styling.

Tailwind configuration.

---

# Shared Rule

When adding a new file ask:

"Which concern does this belong to?"

That answer determines its folder.

Never place files based on convenience.


---
*Last verified against code on 2026-07-23: Verified architectural principles against current codebase.*