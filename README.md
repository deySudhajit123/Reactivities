# Reactivities

A full-stack social activity planning application built with **.NET 10** and **React 19**. Users can create, join, and manage activities, upload photos, follow other users, and chat in real-time — all in a modern, responsive interface.


To access the app, register with a valid email address (confirmation required) or use **GitHub OAuth** login.

Test accounts available locally (see [Running Locally](#-running-locally)):
- **Email:** `bob@test.com` / `tom@test.com` / `jane@test.com`
- **Password:** `Pa$$w0rd`

---

## 🧰 Tech Stack

### Backend — .NET 10
| Layer | Technology |
|---|---|
| **API** | ASP.NET Core 10, minimal hosting model |
| **Application** | MediatR 14 (CQRS), AutoMapper 16, FluentValidation 12 |
| **Domain** | Plain C# entities — Activity, User, Comment, Photo, UserFollowing |
| **Persistence** | Entity Framework Core, SQL Server (docker or local) |
| **Infrastructure** | Cloudinary (photo storage), Resend (transactional email) |
| **Auth** | ASP.NET Core Identity, Cookie-based auth, GitHub OAuth |
| **Real-time** | SignalR (`CommentHub`) |

### Frontend — React 19
| Area | Technology |
|---|---|
| **Framework** | React 19 + TypeScript + Vite 7 |
| **UI** | Material UI (MUI) v7 + Emotion |
| **State** | MobX 6 + TanStack React Query v5 |
| **Routing** | React Router v7 |
| **Forms** | React Hook Form v7 + Zod v4 validation |
| **Maps** | Leaflet + React Leaflet |
| **Photos** | React Dropzone + React Cropper |
| **HTTP** | Axios |
| **Real-time** | @microsoft/signalr v10 |

---

## 🏗️ Architecture

The backend follows **Clean Architecture** with a strict dependency flow:

```
API  →  Application  →  Domain
         ↓                 ↑
    Infrastructure  →  Persistence
```

```
Reactivities/
├── API/                  # ASP.NET Core Web API (controllers, middleware, SignalR)
│   ├── Controllers/      # ActivitiesController, ProfilesController, AccountController
│   ├── SignalR/          # CommentHub (real-time comments)
│   ├── Middleware/       # Global exception handling
│   └── Program.cs        # App bootstrap, DI registration
├── Application/          # Business logic — CQRS handlers, DTOs, validators
│   ├── Activities/       # Queries, Commands, DTOs, Validators for activities
│   ├── Profiles/         # Queries, Commands, DTOs for user profiles
│   ├── Core/             # Shared: Result<T>, PagedList, MappingProfiles, ValidationBehavior
│   └── Interfaces/       # IUserAccessor, IPhotoService
├── Domain/               # Core entities (no dependencies)
│   ├── Activity.cs
│   ├── User.cs           # Extends IdentityUser
│   ├── Comment.cs
│   ├── Photo.cs
│   ├── ActivityAttendee.cs
│   └── UserFollowing.cs
├── Persistence/          # EF Core DbContext, migrations, seed data
│   ├── AppDbContext.cs
│   └── DbInitializer.cs
├── Infrastructure/       # External service implementations
│   ├── Photos/           # Cloudinary photo upload/delete
│   ├── Email/            # Resend transactional email
│   └── Security/         # IsHostRequirement, UserAccessor
├── client/               # React 19 + Vite frontend
│   └── src/
│       ├── app/          # Layout, router, shared components
│       └── features/     # activities, account, profiles, errors, home
├── Dockerfile
└── docker-compose.yml    # SQL Server 2022 container
```

---

## ✨ Features

- **Activity Management** — Create, edit, delete, and browse activities with title, description, date, category, city, venue, and geolocation.
- **Attendance** — Join or cancel attendance on any activity; host cannot leave their own activity.
- **User Profiles** — View/edit bio, display name, and profile photo. See activities hosted/attended.
- **Photo Upload** — Drag-and-drop upload with crop tool; stored on Cloudinary. Set a main photo.
- **Follow System** — Follow/unfollow users; view followers and following lists.
- **Real-time Comments** — Live chat on each activity via SignalR websockets.
- **Infinite Scroll** — Activity list uses cursor-based pagination with intersection observer.
- **Authentication** — Cookie-based sessions via ASP.NET Core Identity. Supports email/password registration (with email confirmation via Resend) and **GitHub OAuth**.
- **Role-based Authorization** — Only the activity host can edit or delete an activity (`IsActivityHost` policy).
- **Maps** — Each activity displays its location on a Leaflet map.
- **Filtering** — Filter activities by "All", "I'm going", or "I'm hosting".

---

## 🚀 Running Locally

### Prerequisites

| Tool | Version |
|---|---|
| .NET SDK | 10.x |
| Node.js | 18+ or 20+ |
| Docker Desktop | any recent version |
| Git | any |

### 1. Clone the repository

```bash
git clone https://github.com/TryCatchLearn/Reactivities.git
cd Reactivities
```

### 2. Start the SQL Server container

```bash
docker-compose up -d
```

This starts a **SQL Server 2022** container on port `1433` with:
- SA Password: `Password@1`
- Persistent volume: `sql-data`

### 3. Configure app secrets

Create `API/appsettings.json` (this file is git-ignored):

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=reactivities;User Id=SA;Password=Password@1;TrustServerCertificate=True"
  },
  "ClientAppUrl": "https://localhost:3000",
  "CloudinarySettings": {
    "CloudName": "REPLACEME",
    "ApiKey": "REPLACEME",
    "ApiSecret": "REPLACEME"
  },
  "Resend": {
    "ApiToken": "REPLACEME"
  },
  "Authentication": {
    "GitHub": {
      "ClientId": "REPLACEME",
      "ClientSecret": "REPLACEME"
    }
  },
  "AllowedHosts": "*"
}
```

> **Photo uploads** require a free [Cloudinary](https://cloudinary.com) account.  
> **Email confirmation** requires a free [Resend](https://resend.com) account.  
> **GitHub login** requires a GitHub OAuth App — create one at [github.com/settings/developers](https://github.com/settings/developers).

### 4. Restore backend dependencies and run the API

```bash
# From the solution root
dotnet restore

# Run the API (auto-migrates DB and seeds test data on first start)
cd API
dotnet run
```

The API will be available at `https://localhost:5001` (or `http://localhost:5000`).  
On startup it automatically:
- Applies all EF Core migrations
- Seeds three test users (`bob`, `tom`, `jane`) with sample activities

### 5. Install frontend dependencies and run the dev server

```bash
# From the solution root, open a second terminal
cd client
npm install
npm run dev
```

The React app starts at **`https://localhost:3000`** (HTTPS via `vite-plugin-mkcert`).

### 6. Log in with seeded test accounts

| Email | Password |
|---|---|
| `bob@test.com` | `Pa$$w0rd` |
| `tom@test.com` | `Pa$$w0rd` |
| `jane@test.com` | `Pa$$w0rd` |

---

## 🐳 Docker (Production Build)

The `Dockerfile` uses a **multi-stage build**:

1. **Build stage** — uses `mcr.microsoft.com/dotnet/sdk:10.0` to restore and publish the API.
2. **Runtime stage** — uses `mcr.microsoft.com/dotnet/aspnet:10.0`; serves on port `8080`.

The Vite client must be built separately and output to `API/wwwroot` before building the Docker image:

```bash
cd client
npm run build      # outputs to ../API/wwwroot

cd ..
docker build -t reactivities .
docker run -p 8080:8080 reactivities
```

---

## 🔑 Key Configuration Reference

| Key | Description |
|---|---|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string |
| `ClientAppUrl` | Frontend origin (e.g. `https://localhost:3000`) |
| `CloudinarySettings:CloudName/ApiKey/ApiSecret` | Cloudinary credentials |
| `Resend:ApiToken` | Resend API token for transactional email |
| `Authentication:GitHub:ClientId/ClientSecret` | GitHub OAuth App credentials |
| `Licences:MediatR` | MediatR commercial licence key (required in v14+) |

---

## 📡 API Endpoints (Summary)

| Method | Route | Description |
|---|---|---|
| `GET` | `/api/activities` | Paginated activity list (cursor-based) |
| `GET` | `/api/activities/{id}` | Single activity detail |
| `POST` | `/api/activities` | Create activity |
| `PUT` | `/api/activities/{id}` | Edit activity (host only) |
| `DELETE` | `/api/activities/{id}` | Delete activity (host only) |
| `POST` | `/api/activities/{id}/attend` | Toggle attendance |
| `GET` | `/api/profiles/{userId}` | Get user profile |
| `PUT` | `/api/profiles` | Update profile |
| `POST` | `/api/profiles/add-photo` | Upload photo to Cloudinary |
| `DELETE` | `/api/profiles/{photoId}/photos` | Delete photo |
| `PUT` | `/api/profiles/{photoId}/setMain` | Set main photo |
| `POST` | `/api/profiles/{userId}/follow` | Toggle follow |
| `GET` | `/api/profiles/{userId}/follow-list` | Followers/following list |
| `GET` | `/api/profiles/{userId}/activities` | User activity history |
| `POST` | `/api/account/register` | Register with email |
| `POST` | `/api/account/github-login` | GitHub OAuth login |
| `GET` | `/api/account/user-info` | Get current user info |
| `POST` | `/api/account/logout` | Sign out |
| `POST` | `/api/account/change-password` | Change password |
| `WS` | `/comments` | SignalR hub for real-time comments |


