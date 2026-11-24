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
  {
  "success": false,
  "data": null,
  "message": "What went wrong, in plain English"
}
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
| GET | `/health` | Public | Person 2 | Draft |
| POST | `/auth/register` | Public | Person 2 | To do |
| POST | `/auth/login` | Public | Person 2 | To do |
| GET | `/auth/me` | Logged in | Person 2 | To do |
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
_To be added by Person 2 (see mca-backend issue: "Draft auth endpoints in API document")._

### Service endpoints
_To be added by Person 3._

### Application endpoints
_To be added by Person 3._
