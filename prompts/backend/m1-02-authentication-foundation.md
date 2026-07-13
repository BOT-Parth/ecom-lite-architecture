# M1.2 — Authentication Foundation

## Objective

Implement the authentication foundation for E-Com Lite.

This milestone establishes:

- User registration
- User login
- JWT authentication
- User profile retrieval

Follow the approved architecture.

---

# Scope

## 1. Register User

Endpoint:

POST /auth/register

Requirements:

- Accept:
  - email
  - username
  - password

- Validate input.
- Prevent duplicate emails.
- Hash passwords using bcrypt.
- Create user record.

---

## 2. Login User

Endpoint:

POST /auth/login

Requirements:

- Accept:
  - email
  - password

- Verify credentials.
- Compare password using bcrypt.
- Generate JWT token.

JWT payload:

{
userId
}

Do not add unnecessary information.

---

## 3. Get Profile

Endpoint:

GET /auth/profile

Requirements:

- Require JWT authentication.
- Return authenticated user information.
- Never return password.

---

# Architecture Rules

Follow:

Routes
↓
Controllers
↓
Services
↓
Repositories
↓
Database

Responsibilities:

Routes:

- Define API endpoints only.

Controllers:

- Handle request and response.
- No business logic.

Services:

- Authentication business logic.

Repositories:

- Prisma database operations.

Validators:

- Request validation.

Middleware:

- JWT authentication.

---

# Required Technical Rules

Use:

- Node.js
- Express.js
- PostgreSQL
- Prisma ORM
- bcrypt
- JWT
- Existing project structure

Use:

- async/await only.
- Environment variables for secrets.
- Consistent API responses.
- Proper HTTP status codes.

---

# JWT Configuration

Add required environment variables:

JWT_SECRET

JWT_EXPIRES_IN

Never hardcode secrets.

---

# Out Of Scope

Do not implement:

- Role authorization
- Permissions
- Store authorization
- Store creation
- Membership
- Store context selection
- Email verification
- Password reset
- Frontend authentication pages

---

# Verification Requirements

After implementation verify:

Registration:

- New user can register.
- Duplicate email rejected.
- Password stored hashed.

Login:

- Correct credentials return JWT.
- Incorrect credentials rejected.

Profile:

- Valid token returns user.
- Invalid token rejected.
- Password never returned.

---

# Reporting

After completion provide:

1. Files changed.
2. Purpose of each change.
3. API endpoints created.
4. Database impact.
5. Verification performed.
6. Results.
7. Known limitations.
8. Code walkthrough explaining request flow.

After verification, stop and request approval before starting the next milestone.
