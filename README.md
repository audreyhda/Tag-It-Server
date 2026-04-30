# 🎨 Tag It — Server

> **The Vapor REST API backend for [Tag It](https://github.com/audreyhda/Tag-It) — a full-stack iOS app for discovering and documenting street art.**

---

## 📌 Overview

This repository contains the server-side component of Tag It, built with [Vapor](https://vapor.codes/) (server-side Swift). It exposes a JSON REST API consumed by the iOS SwiftUI client, handles user authentication via JWT, and persists street art works, users, and favourites to a relational database through Fluent ORM.

Built during **Mini Challenge 3 (Zeus)** of the Apple Foundation Program Advanced in Paris (September–October 2024). The challenge required designing a coherent data flow, building a REST API with its own database, and applying Swift Concurrency practices throughout.

---

## 🛠️ Tech Stack

### 🌐 Languages
- Swift 5.9+

### 📦 Frameworks & Libraries
- [Vapor 4](https://vapor.codes/) — Server-side Swift HTTP framework
- [Fluent](https://docs.vapor.codes/fluent/overview/) — Swift ORM for database modelling and migrations
- [JWT Kit](https://github.com/vapor/jwt-kit) — JSON Web Token signing and verification
- SwiftNIO — Non-blocking, event-driven networking (underlying Vapor)

### 🗄️ Databases & Storage
- SQLite (development) / PostgreSQL (production) — via Fluent drivers

### ☁️ Infrastructure & DevOps
- Swift Package Manager — dependency management
- Docker — optional containerised deployment

---

## ✨ What It Does

- Serves all street art **works** data (list, detail, create, delete)
- Manages **user accounts** with hashed passwords (registration + login)
- Issues and validates **JWT tokens** for authenticated routes
- Stores and retrieves per-user **favourites**
- Handles all requests **concurrently** via Swift's `async`/`await` and SwiftNIO's event loop

---

## 🏗️ Architecture & Technical Details

### System Design

The server is a stateless REST API. Each request carries a JWT bearer token (on protected routes) which is verified by Vapor's authentication middleware — no server-side sessions. Fluent models map directly to database tables and are queried using Swift's type-safe API.

### Data Flow

```
iOS Client
    │
    │  HTTP + JSON (Bearer JWT on protected routes)
    ▼
Vapor Router → Middleware (JWT auth) → Controller
    │
    ▼
Fluent ORM → SQLite / PostgreSQL
```

### Database Schema

```
User ──< Favourite >── Work
  id            id         id
  username      user_id    title
  passwordHash  work_id    artist
                           description
                           imageURL
                           latitude
                           longitude
                           createdAt
```

### Key Technical Decisions

- **Vapor over Node/Python**: The challenge required server-side Swift. Vapor gives full Swift parity between client and server — shared mental models for `async`/`await`, `Codable`, and type safety across the stack.
- **Fluent ORM**: Type-safe queries and automatic migration management. SQLite for local dev; swapping to PostgreSQL for production requires only a driver change.
- **JWT (stateless auth)**: No session storage needed on the server. The iOS client stores the token and includes it in every protected request. Tokens are signed with a shared secret and verified per-request by Vapor middleware.
- **`async`/`await` throughout**: The challenge explicitly required Swift Concurrency for smooth, non-blocking request handling. All controller functions are `async throws`, keeping the code readable alongside Fluent's async query API.

---

## 📁 Project Structure

```
Tag-It-Server/
├── Sources/
│   └── App/
│       ├── Controllers/
│       │   ├── WorkController.swift       # CRUD for street art works
│       │   ├── UserController.swift       # Registration and login
│       │   └── FavouriteController.swift  # Favourites per user
│       ├── Models/
│       │   ├── Work.swift                 # Fluent model + DTO
│       │   ├── User.swift                 # Fluent model, password hashing
│       │   └── Favourite.swift            # Pivot model (User ↔ Work)
│       ├── Migrations/
│       │   ├── CreateWork.swift
│       │   ├── CreateUser.swift
│       │   └── CreateFavourite.swift
│       ├── routes.swift                   # Route registration
│       └── configure.swift                # App config, middleware, DB setup
├── Tests/
├── Package.swift                          # Dependencies
└── docker-compose.yml                     # Optional local Postgres setup
```

---

## 🌐 API Reference

All endpoints return and accept `application/json`. Protected routes require a `Bearer <token>` header obtained from `/api/login`.

### Authentication

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/register` | Public | Create a new user account |
| POST | `/api/login` | Public | Authenticate and receive a JWT |

**Register request body:**
```json
{
  "username": "audrey",
  "password": "s3cr3t"
}
```

**Login response:**
```json
{
  "token": "<jwt>",
  "user": { "id": "...", "username": "audrey" }
}
```

---

### Works

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/works` | Public | List all street art works |
| GET | `/api/works/:id` | Public | Get a single work by ID |
| POST | `/api/works` | JWT | Add a new work |
| DELETE | `/api/works/:id` | JWT | Delete a work |

**POST `/api/works` request body:**
```json
{
  "title": "Fresque Arc-en-ciel",
  "artist": "Unknown",
  "description": "Colorful mural on Rue de la Roquette",
  "imageURL": "https://example.com/image.jpg",
  "latitude": 48.8566,
  "longitude": 2.3522
}
```

---

### Favourites

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/favourites` | JWT | Get the authenticated user's favourites |
| POST | `/api/favourites/:workId` | JWT | Add a work to favourites |
| DELETE | `/api/favourites/:workId` | JWT | Remove a work from favourites |

---

## ⚙️ Getting Started

### Prerequisites

- Swift 5.9+
- [Vapor Toolbox](https://docs.vapor.codes/install/macos/): `brew install vapor`
- Xcode 15+ (macOS) or Swift on Linux

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/audreyhda/Tag-It-Server.git
cd Tag-It-Server

# 2. Resolve dependencies
swift package resolve

# 3. Run database migrations
swift run App migrate

# 4. Start the server (http://localhost:8080)
swift run
```

Or open in Xcode:
```bash
open Package.swift
```
Then select the `App` scheme and press **⌘R**.

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string (production) | ✅ prod only |
| `JWT_SECRET` | Secret used to sign and verify JWT tokens | ✅ |
| `PORT` | Server port (default: `8080`) | ⬜ optional |

For local development with SQLite, no `DATABASE_URL` is needed — Fluent will create a local `db.sqlite` file automatically.

### Docker (optional)

```bash
# Start a local PostgreSQL instance
docker-compose up -d

# Then run migrations and start the server as above
```

---

## 🧪 Running Tests

```bash
swift test
```

---

## 🗺️ Roadmap

- [x] Works CRUD endpoints
- [x] User registration and login
- [x] JWT authentication middleware
- [x] Favourites pivot (User ↔ Work)
- [x] Fluent migrations
- [ ] Image upload endpoint (multipart/form-data)
- [ ] Pagination for `/api/works`
- [ ] Rate limiting middleware
- [ ] Production PostgreSQL deployment

---

## 🔗 Related

- **iOS Client** → [github.com/audreyhda/Tag-It](https://github.com/audreyhda/Tag-It)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Audrey**
- GitHub: [@audreyhda](https://github.com/audreyhda)

---

<div align="center">
<br>
<a href="https://www.balystick.fr/Github/Tag%20It.mp4">
    <img src="https://www.balystick.fr/Github/Tag%20It%20logo.png" alt="Tag It App" style="width:340px">
</a>
<br><br>
<em>▶ Click the logo to watch the demo video</em>
</div>


*Apple Foundation Program Advanced · Mini Challenge 3: Zeus · Built with ❤️ and SwiftUI in Paris*
