# Smart Pack AI — Backend

> [!IMPORTANT]
> **Demo Access Credentials — Available on Request**
> Full demo credentials (Admin, Individual, and Business accounts) for the live staging environment are available upon request on imranraza2016a@gmail.com or 6200966346

A production-ready **Node.js / TypeScript** REST API that powers the Smart Pack AI platform — an intelligent packing and moving assistant. The backend handles two user types (Individual & Business), AI-driven item recognition, cloud backup integrations, subscriptions, notifications, and a full Admin control panel.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Core Features](#core-features)
- [API Route Groups](#api-route-groups)
- [Data Models](#data-models)
- [Authentication & Authorization](#authentication--authorization)
- [AI Integration](#ai-integration)
- [Cloud Backup Integrations](#cloud-backup-integrations)
- [File Storage](#file-storage)
- [Subscription Plans](#subscription-plans)
- [Background Jobs](#background-jobs)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Scripts](#scripts)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js (ESM via TypeScript) |
| Framework | Express 5 |
| Language | TypeScript |
| Database | MongoDB (Mongoose) |
| Cache / Session | Redis |
| AI — Vision | Azure AI Vision (GPT model) |
| AI — Vision | Google Gemini (gemini-1.5-flash) |
| Push Notifications | Firebase Cloud Messaging (FCM) |
| File Storage | Azure Blob Storage, AWS S3 |
| OAuth / Cloud Backup | Google Drive, Dropbox, OneDrive |
| Email | Nodemailer |
| Auth | JWT (jsonwebtoken), bcrypt |
| Security | Helmet, CORS, express-useragent |
| Logging | Morgan |
| Scheduler | node-cron |
| Package Manager | pnpm (workspaces) |

---

## Architecture Overview

```
Client (Mobile / Web)
        │
        ▼
 Express API  (/api/*)
        │
   ┌────┴─────────────────┐
   │  Auth Middleware      │  JWT verification, role guards
   │  Validation Middleware│  Input sanitization
   │  Upload Middleware    │  Multer (memory storage)
   └────┬─────────────────┘
        │
   ┌────▼────────────────────────────────┐
   │  Route Groups                       │
   │  /individual  /business  /admin     │
   │  /subscription /transaction /ai ... │
   └────┬────────────────────────────────┘
        │
   ┌────▼─────────────────────────────────────────┐
   │  Controllers  →  Services  →  Models          │
   │  (thin HTTP layer)  (business logic)  (ODM)   │
   └────┬──────────────────────────────────────────┘
        │
   ┌────▼───────────────────────┐
   │  MongoDB  │  Redis Cache   │
   └────────────────────────────┘
```

---

## Project Structure

```
src/
├── app.ts                   # Express app setup (CORS, Helmet, middleware)
├── server.ts                # HTTP server entry point
├── server.check.ts          # Startup health / env checks
│
├── config/
│   ├── db.config.ts         # MongoDB connection
│   └── redis.config.ts      # Redis client setup
│
├── constants/               # Shared enums, HTTP codes, error messages
│
├── controller/              # Route handler functions grouped by domain
│   ├── individual/          # Auth, profile, referral, security
│   ├── bussiness/           # Auth, profile, staff, customers
│   ├── admin/               # Dashboard, users, business, reports, prompts…
│   ├── ai/                  # AI item recognition endpoint
│   ├── backup/              # Backup CRUD + provider OAuth callbacks
│   ├── subscription/        # Plan management
│   ├── transaction/         # Payment records
│   ├── ticket/              # Support tickets
│   ├── feedback/            # User feedback
│   ├── notification/        # Push notification management
│   ├── upload/              # Generic file upload
│   └── errors/              # Global error handler
│
├── middleware/
│   ├── auth.middleware.ts       # JWT + role-based access control
│   ├── upload.middleware.ts     # Multer configuration
│   └── validation.middleware.ts # Request body validation
│
├── model/                   # Mongoose schemas (see Data Models section)
│
├── routes/                  # Express routers, one file per domain
│
├── services/                # Reusable business-logic / third-party wrappers
│   ├── azure.service.ts         # Azure AI Vision
│   ├── azureBlobStorage.service.ts
│   ├── gemini.service.ts        # Google Gemini (singleton)
│   ├── backup.service.ts        # Backup orchestration
│   ├── backupSchedule.service.ts
│   ├── firebase.service.ts      # FCM push notifications
│   ├── googleOAuth.service.ts
│   ├── dropboxOAuth.service.ts
│   ├── onedriveOAuth.service.ts
│   ├── s3Service.ts
│   ├── announcement.service.ts
│   ├── admin.service.ts
│   ├── notification.service.ts
│   └── cronjob.service.ts
│
├── types/                   # Global TypeScript type augmentations
└── utils/                   # Pure helpers (JWT, OTP, crypto, email, rate limiter…)
```

---

## Core Features

### User Management
- **Individual users** — register/login via email+password or Google/Apple OAuth, email OTP verification, profile management, referral system, account deletion flow.
- **Business users** — business account registration with approval workflow, staff sub-accounts with granular permissions, customer management.
- **Admin** — separate admin authentication, full CRUD over users & businesses, platform-wide dashboard and stats.

### AI Item Recognition
Upload a photo of items and the backend identifies them using either:
- **Azure AI Vision** (OpenAI/GPT model)
- **Google Gemini** (gemini-1.5-flash)

The active AI model is configurable at runtime via the `Prompt` collection in the database (no redeployment needed).

### Cloud Backup
Users can connect their preferred cloud storage provider and back up their packing database file (SQLite):

| Provider | OAuth Flow |
|---|---|
| Google Drive | Google OAuth 2.0 |
| Dropbox | Dropbox OAuth |
| Microsoft OneDrive | Microsoft OAuth |

Backup files are **AES-encrypted** before upload using a user-specific derived key. Schedules support manual, daily, weekly, and monthly frequencies with configurable retention limits.

### Subscriptions & Transactions
- Plans: **Free**, **Pro**, **Business**
- Features gated per plan: item limits, AI recognition quotas, label printing, backup, custom logo, multi-user access, customer creation limits.
- Transaction history stored per user.

### Notifications
- Firebase Cloud Messaging (FCM) for push notifications.
- In-app notification records stored in MongoDB.

### Admin Panel
- Dashboard with aggregate stats.
- User & business management (view, lock, approve, reject, delete).
- Announcement broadcasts (instant + scheduled).
- Report generation & history.
- Support ticket management.
- Feedback review.
- AI prompt / model configuration.
- Subscription plan management.

---

## API Route Groups

All routes are prefixed with `/api`.

| Prefix | Description |
|---|---|
| `/individual/auth` | Register, login, OTP verify, password reset |
| `/individual/profile` | Get/update profile, profile picture |
| `/individual/security` | Change password, update email |
| `/individual/referral` | Referral code generation & stats |
| `/business/auth` | Business register, login, OTP verify |
| `/business/profile` | Business profile management |
| `/business/security` | Password/email changes |
| `/business/customer` | Customer CRUD |
| `/business/staff` | Staff CRUD & permissions |
| `/subscription` | View & activate plans |
| `/transaction` | Transaction history |
| `/notification` | List, mark-read notifications |
| `/upload` | Generic file upload (Azure/S3) |
| `/ai` | AI item image analysis |
| `/backup/*` | Backup CRUD, OAuth callbacks, restore |
| `/feedback` | Submit feedback |
| `/ticket` | Support ticket CRUD |
| `/admin/auth` | Admin login |
| `/admin/users` | User management |
| `/admin/business` | Business management |
| `/admin/subscription` | Plan management |
| `/admin/dashboard` | Platform statistics |
| `/admin/reports` | Report generation |
| `/admin/announcements` | Announcements |
| `/admin/action` | Bulk user actions |
| `/admin/feedback` | Feedback review |
| `/admin/ticket` | Ticket management |
| `/admin/prompt` | AI model / prompt config |
| `/admin/profile` | Admin profile |
| `/admin/security` | Admin password/email |
| `/app/stats` | App package statistics |
| `/verify-email-otp` | Email change OTP verification |
| `/delete-account` | Account deletion flow |

**Health check**: `GET /health` — returns `{ status: "ok" }` with no auth required.

---

## Data Models

| Model | Purpose |
|---|---|
| `User` | Individual user accounts |
| `Business` | Business admin & staff accounts |
| `Admin` | Platform administrators |
| `Subscription` | Plan definitions (Free / Pro / Business) |
| `Transaction` | Payment records |
| `Backup` | Backup file metadata |
| `Notification` | In-app notifications |
| `Announcement` | Platform-wide announcements |
| `ScheduleAnnouncement` | Scheduled announcement jobs |
| `Feedback` | User feedback entries |
| `Ticket` | Support tickets |
| `Referral` | Referral tracking |
| `Prompt` | Active AI model + prompt config |
| `ReportHistory` | Generated report records |
| `AppPackStats` | App usage statistics |

---

## Authentication & Authorization

- **JWT Bearer tokens** — issued on login, stored as `accessToken` on the user document (single-session enforcement).
- **Role-based guards** — `authMiddleware(roles[])` restricts endpoints to `individual`, `business-admin`, `business-staff`, or `admin`.
- **Locked / deleted account checks** — enforced on every authenticated request.
- **OTP flows** — email-based OTP for registration, password reset, and email change (TTL-based expiry).
- **Password hashing** — bcrypt with auto-hash on save via Mongoose pre-save hook.

---

## AI Integration

### Azure AI Vision
File: `src/services/azure.service.ts`

Accepts an image buffer and a list of item names, calls the Azure Vision API, and returns structured recognition results.

### Google Gemini
File: `src/services/gemini.service.ts`

Singleton service (lazy-initialized). Accepts an image buffer, auto-detects MIME type (PNG/JPEG/GIF/WebP), encodes to base64, and calls the Gemini API with a custom object-detection prompt.

### Switching Models
Update the `aiModel` field in the `Prompt` collection to `"gpt"` (Azure) or `"gemini"` — no restart required.

---

## Cloud Backup Integrations

| Provider | Service File |
|---|---|
| Google Drive | `src/services/googleOAuth.service.ts` |
| Dropbox | `src/services/dropboxOAuth.service.ts` |
| OneDrive | `src/services/onedriveOAuth.service.ts` |

**Encryption**: `src/utils/crypto.util.ts` — AES encryption with per-user derived keys. Files are validated as SQLite before upload.

Orchestration logic lives in `src/services/backup.service.ts` and schedule management in `src/services/backupSchedule.service.ts`.

---

## File Storage

| Provider | Usage |
|---|---|
| Azure Blob Storage | Profile pictures, business logos, general uploads |
| AWS S3 | Alternative/additional file storage |

Upload middleware uses Multer with in-memory storage; the controller streams the buffer to the chosen provider.

---

## Subscription Plans

| Feature | Free | Pro | Business |
|---|---|---|---|
| Item limit | Limited | Higher | Unlimited |
| AI image recognition | Limited | Higher | Higher |
| Label printing | ✗ | ✓ | ✓ |
| Cloud backup | ✗ | ✓ | ✓ |
| Custom logo | ✗ | ✗ | ✓ |
| Multi-user access | ✗ | ✗ | ✓ |
| Customer creation | ✗ | ✗ | ✓ |

---

## Background Jobs

File: `src/services/cronjob.service.ts`

Uses `node-cron` for:
- Scheduled announcements delivery
- Automated backup triggers (daily / weekly / monthly per user settings)
- Subscription expiry checks

---

## Environment Variables

Create a `.env` file in the project root. Required variables:

```env
# Server
NODE_ENV=development
PORT=5000

# Database
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/<db>

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=7d

# Google OAuth / Gemini
GOOGLE_API_KEY=your_google_api_key
GOOGLE_GEMINI_MODEL=gemini-1.5-flash
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Azure
AZURE_VISION_ENDPOINT=
AZURE_VISION_KEY=
AZURE_STORAGE_CONNECTION_STRING=
AZURE_STORAGE_CONTAINER_NAME=

# AWS S3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
AWS_S3_BUCKET_NAME=

# Dropbox
DROPBOX_APP_KEY=
DROPBOX_APP_SECRET=

# OneDrive / Microsoft
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
MICROSOFT_REDIRECT_URI=

# Firebase
# (place firebase-service-account.json in project root)

# Email (Nodemailer)
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=
EMAIL_FROM=
```

---

## Getting Started

### Prerequisites

- **Node.js** >= 18
- **pnpm** >= 8
- **MongoDB** instance (local or Atlas)
- **Redis** instance

### Installation

```bash
# Install dependencies
pnpm install
```

### Development

```bash
# Windows
pnpm dw

# Linux / macOS
pnpm dl
```

The server auto-restarts on file changes via `ts-node-dev`.

### Production Build

```bash
pnpm build   # compiles TypeScript to ./build
pnpm start   # runs ./build/server.js
```

---

## Scripts

| Script | Description |
|---|---|
| `pnpm dev` | Start dev server with ts-node-dev |
| `pnpm dw` | Dev server with increased heap (Windows) |
| `pnpm dl` | Dev server with increased heap (Linux) |
| `pnpm build` | Compile TypeScript → `./build` |
| `pnpm start` | Run compiled production server |
| `pnpm lint` | Run ESLint with auto-fix |
| `pnpm format` | Format all files with Prettier |
| `pnpm clean` | Remove build artifacts and node_modules |

---

