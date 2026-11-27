# API Documentation – Culture & Arts e-Services Portal

This document lists every backend endpoint. Frontend and backend
must both follow it. **Change this file before changing an endpoint.**

## Basics

| Item | Value |
|---|---|
| Base URL (local) | `http://localhost:5000/api` |
| Data format | JSON |
| Authentication | JWT token in the header: `Authorization: Bearer <token>` |

## Standard response format

Every response uses the same shape.

**Success**
```json
{
  "success": true,
  "data": { },
  "message": "Optional short message"
}
```

**Error**
```json
{
  "success": false,
  "data": null,
  "message": "What went wrong, in plain English"
}
```

## Status codes we use

| Code | Meaning | When |
|---|---|---|
| 200 | OK | Request worked |
| 201 | Created | Something new was saved |
| 400 | Bad Request | Missing or invalid data |
| 401 | Unauthorized | Not logged in / bad token |
| 403 | Forbidden | Logged in, but not allowed (wrong role) |
| 404 | Not Found | Item or route doesn't exist |
| 409 | Conflict | Already exists (e.g. email already registered) |
| 500 | Server Error | Bug on the backend |

## User roles

| Role | Can do |
|---|---|
| `CITIZEN` | Browse services, apply, track own applications |
| `OFFICER` | Review applications, approve or reject |
| `ADMIN` | Create and manage services and users |

## Endpoint summary

| Method | Endpoint | Who | Owner | Status |
|---|---|---|---|---|
| GET | `/health` | Public | Person 2 | Built |
| POST | `/auth/register` | Public | Person 2 | Built |
| POST | `/auth/login` | Public | Person 2 | Built |
| GET | `/auth/me` | Logged in | Person 2 | Built |
| GET | `/services` | Public | Person 3 | To do |
| GET | `/services/:slug` | Public | Person 3 | To do |
| POST | `/services` | Admin | Person 3 | To do |
| PUT | `/services/:id` | Admin | Person 3 | To do |
| POST | `/applications` | Citizen | Person 3 | To do |
| GET | `/applications/mine` | Citizen | Person 3 | To do |
| PATCH | `/applications/:id/status` | Officer | Person 3 | To do |

Status: **To do** → **Draft** (written here) → **Agreed** (team approved) → **Built**

---

## Endpoint details

Copy this format for every endpoint.

### GET /health

Checks that the backend is running.

- **Auth:** None
- **Request body:** None

**Response 200**
```json
{
  "success": true,
  "data": { "status": "ok" }
}
```

---

### Auth endpoints

**Public user.** Every auth answer returns the user in this shape. The password hash is never returned.
```json
{ "id": 1, "name": "Test Citizen", "email": "citizen@mca.test", "phone": "0970000001", "role": "CITIZEN" }
```

**Token.** A JWT that lasts 1 hour. Send it on every protected request:
`Authorization: Bearer <token>`. It holds only the user id and role, never personal data.
When it expires, protected requests answer 401 and the user logs in again.

**Protecting a route (for backend developers).** Two middlewares in `mca-backend/src/middleware/`:

- `requireAuth` checks the token. If it is valid, it sets `req.auth = { userId, role }`
  (`userId` is a number). A missing, forged or expired token answers 401 `Please log in`.
- `requireRole(roles, message)` goes **after** `requireAuth`. A user whose role is not in `roles`
  gets 403 (default message `You do not have permission to do this`).

```js
router.post('/', requireAuth, requireRole('CITIZEN'), controller.submit);
router.patch('/:id/status', requireAuth, requireRole(['OFFICER', 'ADMIN']), controller.updateStatus);
```

Always take the caller from `req.auth`, never from the request body (a user could send someone
else's id). `req.auth` comes from the token, so it can be up to 1 hour old: when you need the
user's current details, read them from the database, as `/auth/me` does.

---

### POST /auth/register

Creates a citizen account and logs the user in.

- **Auth:** None
- **Request body:**

| Field | Required | Rules |
|---|---|---|
| `name` | Yes | 2–100 characters |
| `email` | Yes | Valid email; stored in lowercase; must not already be registered |
| `password` | Yes | 8 to 72 characters (bcrypt ignores anything longer; emoji and accents count as more than one) |
| `phone` | No | `0` or `+260`, then 9 digits (e.g. `0970000001` or `+260970000001`); always stored as `0970000001` |

Any other field (for example `role`, `nrc` or `fullName`) is ignored. Every new account is a `CITIZEN`.
The NRC is not collected at registration: a service that needs it asks for it on its application form.

```json
{ "name": "Mwila Banda", "email": "mwila@example.com", "password": "Password123!", "phone": "0970000001" }
```

**Response 201**
```json
{
  "success": true,
  "data": {
    "token": "<jwt>",
    "user": { "id": 4, "name": "Mwila Banda", "email": "mwila@example.com", "phone": "0970000001", "role": "CITIZEN" }
  },
  "message": "Account created"
}
```

**Errors**

| Status | Message | When |
|---|---|---|
| 400 | `Name is required` (or the first rule broken) | A field is missing or breaks a rule |
| 409 | `An account with this email already exists` | Email already registered |

---

### POST /auth/login

- **Auth:** None
- **Request body:** `email`, `password` (both required)

```json
{ "email": "citizen@mca.test", "password": "Password123!" }
```

**Response 200**: same `data` shape as register (`token` and `user`), message `Logged in`.

**Errors**

| Status | Message | When |
|---|---|---|
| 400 | `Email and password are required` | Either is missing |
| 401 | `Email or password is incorrect` | Wrong email **or** wrong password (same message on purpose, so nobody can find out which emails are registered) |

---

### GET /auth/me

Returns the logged-in user, read fresh from the database.

- **Auth:** Logged in (`Authorization: Bearer <token>`)
- **Request body:** None

**Response 200**
```json
{
  "success": true,
  "data": { "user": { "id": 1, "name": "Test Citizen", "email": "citizen@mca.test", "phone": "0970000001", "role": "CITIZEN" } },
  "message": "OK"
}
```

**Errors**

| Status | Message | When |
|---|---|---|
| 401 | `Please log in` | No token, a bad token, an expired token, or the user no longer exists |

### Service endpoints
_To be added by Person 3._

### Application endpoints
### POST /applications

Submits a completed form for a service.

- **Auth:** Logged in, role `CITIZEN`
- **Request body:**

| Field | Required | Rules |
|---|---|---|
| `serviceSlug` | Yes | A published service, e.g. `event-permit` |
| `formData` | Yes | The answers, keyed by field `name` from the service's form_schema |

The backend checks `formData` against the service's form_schema v1: required fields, `validation` rules, and `showIf` (a hidden field is not required, and its answer is dropped). Unknown field names are rejected.

**Response 201**
```json
{
  "success": true,
  "data": { "id": 12, "referenceNo": "MCA-2026-00012", "status": "SUBMITTED", "currentStep": 1, "fee": "500" },
  "message": "Application submitted"
}
```

**Errors**

| Status | Message | When |
|---|---|---|
| 400 | The first problem found, e.g. `Venue name is required` | `formData` breaks a rule |
| 401 | `Please log in` | No or bad token |
| 403 | `Only citizens can apply` | Officer or admin token |
| 403 | `Grants are only for registered artists...` | The `requires` rule isn't met (arts grant) |
| 404 | `Service not found` | Unknown or unpublished slug |

---

### GET /applications/mine

The logged-in citizen's applications, newest first.

- **Auth:** Logged in, role `CITIZEN`

**Response 200**
```json
{
  "success": true,
  "data": [
    { "id": 12, "referenceNo": "MCA-2026-00012", "status": "SUBMITTED", "currentStep": 1,
      "createdAt": "2026-11-26T09:14:02.000Z",
      "service": { "name": "Event / Performance Permit", "slug": "event-permit" } }
  ],
  "message": "Applications retrieved"
}
```

---

### GET /applications/:referenceNo

One application with its answers and history.

- **Auth:** The citizen who owns it, or an `OFFICER` / `ADMIN`
- **404** `Application not found` if it doesn't exist **or** belongs to someone else (so nobody can find out which reference numbers exist)

---

### Coming in Week 6
`GET /applications/queue` (officer/admin), `PATCH /applications/:id/status` with `{ "action": "APPROVE" | "REJECT" | "RETURN", "comment" }`, and `GET /applications/:id/history`.

### Open questions (for the DB session)
- **Fees:** stored in kwacha or ngwee? The event permit fee comes from `feeRules` by attendance.
- **File fields:** the upload endpoint comes later. Until then, should file fields be skipped by validation?
