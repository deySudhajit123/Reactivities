# 🎯 Reactivities

> **A full-stack social activity planning app** — where people create events, join in, chat live, and connect with each other.

Built end-to-end with **.NET 10** on the backend and **React 19** on the frontend, Reactivities is a feature-complete reference application covering real-world concerns: clean architecture, CQRS, real-time communication, social features, OAuth, photo storage, and cloud deployment.

---

## 🌟 What Is This App About?

Reactivities lets users:

- **Post activities** — give it a title, date, location (with map pin), category, and description
- **Attend activities** — join any event or cancel your spot
- **Chat in real time** — every activity has a live comment thread powered by WebSockets
- **Follow people** — build a social graph; see who's hosting and who's going
- **Upload photos** — drag-and-drop with a crop tool; photos are stored on Cloudinary
- **Manage their profile** — bio, display name, main photo, and activity history

The **key theme** of this project is demonstrating how all the individual pieces of a modern web stack fit together into a coherent, production-grade application — not just a tutorial toy, but something with auth, real-time events, external services, CI/CD, and cloud hosting.

---

## 🧰 Tech Stack

### Backend — .NET 10

| Layer | Technology |
|---|---|
| **API** | ASP.NET Core 10, minimal hosting model |
| **Architecture** | Clean Architecture — API → Application → Domain ← Persistence / Infrastructure |
| **CQRS** | MediatR 14 with a pipeline `ValidationBehavior` |
| **Mapping** | AutoMapper 16 |
| **Validation** | FluentValidation 12 |
| **Database** | Entity Framework Core + SQL Server 2022 |
| **Auth** | ASP.NET Core Identity, cookie-based sessions, GitHub OAuth |
| **Email** | Resend (transactional — email confirmation, password reset) |
| **Photo storage** | Cloudinary |
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

The backend follows **Clean Architecture** with a strict, inward-only dependency flow:

```
┌────────────────────────────────────────────────────┐
│                      API Layer                     │
│  Controllers · Middleware · SignalR · Program.cs   │
└───────────────────┬────────────────────────────────┘
                    │ depends on
┌───────────────────▼────────────────────────────────┐
│               Application Layer                    │
│  CQRS Handlers · DTOs · Validators · Interfaces   │
└───────────┬───────────────────────────┬────────────┘
            │ depends on                │ depends on
┌───────────▼──────────┐   ┌───────────▼────────────┐
│    Domain Layer      │   │  Persistence Layer     │
│  Activity · User     │   │  EF Core · DbContext   │
│  Comment · Photo     │   │  Migrations · Seeder   │
│  UserFollowing       │   └────────────────────────┘
└──────────────────────┘
                            ┌────────────────────────┐
                            │  Infrastructure Layer  │
                            │  Cloudinary · Resend   │
                            │  UserAccessor · Auth   │
                            └────────────────────────┘
```

### Project Layout

```
Reactivities/
├── API/                    # ASP.NET Core host
│   ├── Controllers/        # ActivitiesController, ProfilesController, AccountController
│   ├── SignalR/            # CommentHub — live comments over WebSocket
│   ├── Middleware/         # Global exception handling with ProblemDetails
│   └── Program.cs          # DI wiring, middleware pipeline, DB migration on startup
│
├── Application/            # Pure business logic — no HTTP, no DB drivers
│   ├── Activities/         # Queries, Commands, DTOs, Validators (CQRS via MediatR)
│   ├── Profiles/           # Follow, photo, profile queries & commands
│   ├── Core/               # Result<T>, PagedList, MappingProfiles, ValidationBehavior
│   └── Interfaces/         # IUserAccessor, IPhotoService (implemented in Infrastructure)
│
├── Domain/                 # Plain C# entities — zero dependencies
│   ├── Activity.cs         # Title, Date, Category, City, Venue, Lat/Lon, Attendees, Comments
│   ├── User.cs             # Extends IdentityUser; has Bio, DisplayName, Photos, Followings
│   ├── Comment.cs
│   ├── Photo.cs
│   ├── ActivityAttendee.cs # Join table — User ↔ Activity (isHost flag)
│   └── UserFollowing.cs    # Self-referencing follow relationship
│
├── Persistence/            # EF Core
│   ├── AppDbContext.cs
│   └── DbInitializer.cs    # Seeds bob / tom / jane with sample activities
│
├── Infrastructure/         # External-service adapters
│   ├── Photos/             # Cloudinary upload & delete
│   ├── Email/              # Resend transactional email (confirm, reset)
│   └── Security/           # IsHostRequirement policy, UserAccessor (reads ClaimsPrincipal)
│
├── client/                 # React 19 + Vite SPA
│   └── src/
│       ├── app/            # Layout, router, shared components (maps, photo upload, inputs)
│       └── features/       # activities · account · profiles · errors · home
│
├── Dockerfile              # Multi-stage: sdk:10.0 build → aspnet:10.0 runtime (port 8080)
├── docker-compose.yml      # SQL Server 2022 container for local dev
└── .github/workflows/      # Azure Web App CI/CD (manual trigger)
```

---

## ✨ Feature Highlights

| Feature | How it works |
|---|---|
| **Activity CRUD** | Create/edit/delete with title, description, date, category, city, venue, and geo-coordinates |
| **Attendance** | Toggle join/cancel; host cannot leave their own activity; cancelled activities shown differently |
| **Real-time comments** | SignalR `CommentHub` broadcasts new comments to all connected clients instantly |
| **Photo upload** | Drag-and-drop → crop → upload to Cloudinary; set any photo as your main avatar |
| **Follow system** | Follow/unfollow users; dedicated followers/following tabs on every profile page |
| **Infinite scroll** | Cursor-based pagination with `IntersectionObserver` — loads more activities as you scroll |
| **Activity filters** | "All", "I'm going", "I'm hosting" — filtered on the server |
| **Maps** | Every activity shows a Leaflet map pinned to the venue coordinates |
| **Email auth** | Register with email → confirmation email via Resend → log in |
| **GitHub OAuth** | One-click sign-in with GitHub; merges into the same Identity system |
| **Password flows** | Forgot password → reset link by email; change password when logged in |
| **Role-based policy** | `IsActivityHost` authorization policy prevents other users from editing/deleting activities |
| **Global error handling** | Middleware returns `ProblemDetails`; React shows 404/500 pages gracefully |
| **CI/CD** | GitHub Actions workflow builds the React app and deploys to **Azure App Service** |

---

## 🚀 Running Locally

### Prerequisites

| Tool | Version |
|---|---|
| .NET SDK | 10.x |
| Node.js | 18+ or 20+ |
| Docker Desktop | any recent version |
| Git | any |

---

### 1. Clone the repo

```bash
git clone https://github.com/TryCatchLearn/Reactivities.git
cd Reactivities
```

### 2. Start SQL Server (Docker)

```bash
docker-compose up -d
```

Starts **SQL Server 2022** on `localhost:1433` with SA password `Password@1` and a persistent `sql-data` volume.

### 3. Configure secrets

Create `API/appsettings.json` (git-ignored):

```json
{
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
  "Licences": {
    "MediatR": "REPLACEME"
  },
  "AllowedHosts": "*"
}
```

> **Photo uploads** → free [Cloudinary](https://cloudinary.com) account  
> **Email confirmation / password reset** → free [Resend](https://resend.com) account  
> **GitHub login** → OAuth App at [github.com/settings/developers](https://github.com/settings/developers)  
> **MediatR** → commercial licence required for v14+ (free tier available)

### 4. Run the API

```bash
dotnet restore
cd API
dotnet run
```

The API starts at `https://localhost:5001`. On first boot it:
- Applies all EF Core migrations automatically
- Seeds three test users with sample activities

### 5. Run the React dev server

```bash
# new terminal from the solution root
cd client
npm install
npm run dev
```

The SPA starts at **`https://localhost:3000`** (HTTPS via `vite-plugin-mkcert`).

### 6. Log in with seeded test accounts

| Email | Password |
|---|---|
| `bob@test.com` | `Pa$$w0rd` |
| `tom@test.com` | `Pa$$w0rd` |
| `jane@test.com` | `Pa$$w0rd` |

---

## 🐳 Docker (Production Build)

The `Dockerfile` uses a **multi-stage build** — compile with the full SDK, run on the lightweight runtime image.

```bash
# 1. Build the React app first (outputs to API/wwwroot)
cd client && npm install && npm run build && cd ..

# 2. Build and run the Docker image
docker build -t reactivities .
docker run -p 8080:8080 \
  -e "ConnectionStrings__DefaultConnection=<your-connection-string>" \
  reactivities
```

The container listens on port `8080` and serves both the API and the pre-built React SPA as static files.

---

## ☁️ Cloud Deployment

The project includes a **GitHub Actions workflow** ([`.github/workflows/main_reactivities-2026.yml`](.github/workflows/main_reactivities-2026.yml)) that:

1. Builds the React client with Node 22
2. Restores, builds, and publishes the .NET 10 API
3. Deploys to **Azure App Service** (`reactivities-2026`) using OIDC federated credentials

The workflow is triggered manually (`workflow_dispatch`) — push to Azure whenever you're ready.

---

## 📡 API Reference

| Method | Route | Auth | Description |
|---|---|---|---|
| `GET` | `/api/activities` | ✅ | Paginated, filterable activity list |
| `GET` | `/api/activities/{id}` | ✅ | Single activity with attendees & comments |
| `POST` | `/api/activities` | ✅ | Create a new activity |
| `PUT` | `/api/activities/{id}` | Host only | Edit activity |
| `DELETE` | `/api/activities/{id}` | Host only | Delete activity |
| `POST` | `/api/activities/{id}/attend` | ✅ | Toggle attendance |
| `GET` | `/api/profiles/{userId}` | ✅ | Get user profile |
| `PUT` | `/api/profiles` | ✅ | Edit own profile |
| `POST` | `/api/profiles/add-photo` | ✅ | Upload photo to Cloudinary |
| `PUT` | `/api/profiles/{photoId}/setMain` | ✅ | Set main photo |
| `DELETE` | `/api/profiles/{photoId}/photos` | ✅ | Delete photo |
| `POST` | `/api/profiles/{userId}/follow` | ✅ | Toggle follow |
| `GET` | `/api/profiles/{userId}/follow-list` | ✅ | Followers / following list |
| `GET` | `/api/profiles/{userId}/activities` | ✅ | User activity history |
| `POST` | `/api/account/register` | ❌ | Register with email |
| `POST` | `/api/account/github-login` | ❌ | GitHub OAuth sign-in |
| `GET` | `/api/account/user-info` | ✅ | Current user info |
| `POST` | `/api/account/logout` | ✅ | Sign out |
| `POST` | `/api/account/change-password` | ✅ | Change password |
| `WS` | `/comments` | ✅ | SignalR hub — real-time comments |

---

## 🔑 Configuration Reference

| Key | Description |
|---|---|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string |
| `ClientAppUrl` | Frontend origin for CORS (e.g. `https://localhost:3000`) |
| `CloudinarySettings:CloudName/ApiKey/ApiSecret` | Cloudinary media storage credentials |
| `Resend:ApiToken` | Resend API token for transactional email |
| `Authentication:GitHub:ClientId/ClientSecret` | GitHub OAuth App credentials |
| `Licences:MediatR` | MediatR v14 commercial licence key |

---

## 📚 Learning Context

This project was built as part of the **Udemy course "Build a Complete App with React and .NET"** by Neil Cummings. It is a hands-on, full-stack reference that demonstrates:

- Clean Architecture separation of concerns
- CQRS with the MediatR pipeline (commands, queries, validation behaviors)
- EF Core with SQL Server — migrations, seeding, relationships
- ASP.NET Core Identity with cookie auth and social login (GitHub OAuth)
- Real-time features via SignalR
- React state management patterns with MobX and React Query side-by-side
- Form handling with React Hook Form and Zod schema validation
- Cloud-native deployment to Azure via GitHub Actions CI/CD

---

*Sudhajit Dey · 2026*
