# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
EventHub is a full-stack event ticket booking platform built for QA training. Users can browse events, book tickets, manage bookings, and create events. Each user operates in an isolated sandbox.

## Tech Stack
- **Frontend**: Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, React Query v5
- **Backend**: Express.js, Prisma ORM, MySQL 8+
- **Auth**: JWT (7-day expiry), bcryptjs
- **Testing**: Playwright E2E (Chromium only)

## Commands

```bash
# First-time setup
npm run setup        # Install deps in both /backend and /frontend

# Database
npm run db:push      # Push Prisma schema to DB (non-interactive)
npm run migrate      # Run prisma migrate dev (interactive, creates migration files)
npm run seed         # Insert 10 sample events

# Development
npm run dev          # Start frontend (port 3000) + backend (port 3001) concurrently
npm run lint         # Lint frontend TypeScript/TSX

# Testing
npm run test                                                    # Run all Playwright tests
npm run test:ui                                                 # Playwright with interactive UI
npm run test:report                                             # Open last HTML report
npx playwright test tests/<file>.spec.js --reporter=line       # Run a single test file
```

## Architecture Pattern
Backend follows layered architecture: Routes → Controllers → Services → Repositories → Database

```
eventhub/
├── frontend/          # Next.js 14 app (port 3000)
│   ├── app/           # Pages: /, /events, /events/[id], /bookings, /bookings/[id], /login, /register, /admin/...
│   ├── components/    # React components (ui/, events/, bookings/, layout/)
│   ├── lib/           # Axios client, React Query hooks, providers
│   └── types/         # TypeScript interfaces
├── backend/           # Express API (port 3001)
│   ├── app.js         # Express setup: CORS, routes, Swagger UI at /api/docs
│   ├── server.js      # HTTP server, DB connect, graceful shutdown
│   └── src/
│       ├── routes/        # authRoutes, eventRoutes, bookingRoutes
│       ├── controllers/   # Thin HTTP layer — calls services
│       ├── services/      # Business logic, validation, transactions
│       ├── repositories/  # Pure Prisma data access
│       ├── validators/    # express-validator middleware
│       ├── middleware/    # authMiddleware (JWT), errorHandler, requestLogger
│       └── utils/errors.js  # NotFoundError, InsufficientSeatsError, ValidationError
│   └── prisma/            # schema.prisma + seed.js
├── tests/             # Playwright E2E tests (<feature>.spec.js)
├── docs/              # Test scenario and strategy documents
└── playwright.config.ts
```

## API Endpoints
Base URL: `http://localhost:3001`  
All `/api/events` and `/api/bookings` routes require `Authorization: Bearer <token>`.

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/register` | No | Register new user |
| POST | `/api/auth/login` | No | Login, returns JWT |
| GET | `/api/events` | Yes | List events (paginated; params: category, city, search, page, limit) |
| POST | `/api/events` | Yes | Create event |
| PUT | `/api/events/:id` | Yes | Update event |
| DELETE | `/api/events/:id` | Yes | Delete event (cascades bookings) |
| GET | `/api/bookings` | Yes | List user's bookings |
| POST | `/api/bookings` | Yes | Create booking (atomically decrements seats) |
| DELETE | `/api/bookings/:id` | Yes | Cancel booking (restores seats) |

## Testing Conventions
- **Tests run against the hosted app**: `https://eventhub.rahulshettyacademy.com` (see `playwright.config.ts` `baseURL`) — not localhost
- Test files go in `tests/` as `<feature-name>.spec.js`
- Locator priority: `data-testid` > role > label/placeholder > ID > CSS class
- No `page.waitForTimeout()` — use `expect().toBeVisible()`
- Tests must be self-contained (login → action → assert)
- Use test account: `rahulshetty1@gmail.com` / `Magiclife1!`

### Key `data-testid` attributes
`event-card`, `book-now-btn`, `quantity-input`, `customer-name`, `customer-email`, `customer-phone`, `confirm-booking-btn`, `booking-ref`, `booking-card`, `cancel-booking-btn`, `confirm-dialog-yes`, `admin-event-form`, `event-title-input`, `add-event-btn`, `event-table-row`, `edit-event-btn`, `delete-event-btn`, `nav-events`, `nav-bookings`

## Key Business Rules
- Max 6 user-created events (FIFO pruning on overflow)
- Max 9 bookings per user (FIFO pruning on overflow)
- Booking ref first character = event title first character (uppercase)
- Seat count reduces on booking, restores on cancellation
- Refund eligibility: 1 ticket = eligible, >1 tickets = not eligible (client-side only)
- Cross-user booking access returns "Access Denied"
- Static seeded events are immutable

## Custom Slash Commands (Agents)
- `/generate-tests <feature>` — AI Test Automation Engineer: generates Playwright tests
- `/review-tests <file>` — AI Code Reviewer: reviews test code quality
- `/create-scenarios <area>` — AI Functional Tester: creates test scenario documents
- `/test-strategy <scenarios>` — AI Test Architect: assigns tests to optimal pyramid layers

## Reference Docs
- `docs/test-scenarios.md` — 53 test scenarios (TC-001 to TC-510) for booking management
- `docs/test-strategy.md` — Test pyramid layer assignments (Unit 5, API 22, Component 14, E2E 12)

## Code Style
- Backend: JavaScript with JSDoc, Express patterns
- Frontend: TypeScript, React hooks, Tailwind utility classes
- Tests: JavaScript with Playwright test runner
- Add step comments in tests (`// Step: ...`) for readability
