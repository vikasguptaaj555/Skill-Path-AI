# ⚙️ SkillPath AI — Backend

> **AI-Powered Personalized Learning & Project Guidance Platform**
> Backend built with **Node.js + Express.js + MongoDB + Groq AI** — part of a full-stack MERN + AI project.

[![Node.js](https://img.shields.io/badge/Node.js-%E2%89%A518-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com/atlas)
[![Groq](https://img.shields.io/badge/AI-Groq%20Cloud-orange)](https://groq.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#-license)

---

## 📖 Table of Contents

1. [About the Project](#-about-the-project)
2. [How the Backend Fits In](#-how-the-backend-fits-in)
3. [Tech Stack](#-tech-stack)
4. [Folder Structure](#-folder-structure)
5. [Request Lifecycle](#-request-lifecycle)
6. [Database Schema](#-database-schema)
7. [AI Integration](#-ai-integration)
8. [API Endpoints Reference](#-api-endpoints-reference)
9. [Authentication & Security](#-authentication--security)
10. [Getting Started](#-getting-started)
11. [Environment Variables](#-environment-variables)
12. [Available Scripts](#-available-scripts)
13. [Deployment](#-deployment)
14. [Contributing](#-contributing)
15. [License](#-license)

---

## 🧠 About the Project

**SkillPath AI** gives every learner a personal AI mentor: a system that builds a custom learning roadmap, answers questions in real time, recommends projects, and tracks progress. This repository is the **backend** — the Node.js/Express API server that powers all of that.

Think of the backend as the **"brain and memory"** of the platform:

- 🧠 **Brain** — talks to the Groq AI to generate roadmaps, answer chat questions, and recommend projects
- 💾 **Memory** — stores users, profiles, roadmaps, progress, and chat history in MongoDB
- 🔐 **Gatekeeper** — verifies who's logged in and what they're allowed to do

> A companion **frontend README** describes the React application that consumes this API.

---

## 🔗 How the Backend Fits In

The backend sits in the middle of a **3-tier architecture**: it receives requests from the React frontend, talks to MongoDB for storage, and talks to Groq's AI API for intelligence.

```
┌─────────────┐        HTTPS / JSON        ┌─────────────────────┐
│             │ ──────────────────────────▶│                     │
│   React     │        (Axios calls)       │  THIS REPO:         │
│  Frontend   │                             │  Node.js + Express  │
│  (Vercel)   │◀────────────────────────── │  Backend API        │
└─────────────┘                             └─────────┬───────────┘
                                                       │
                                     ┌─────────────────┴─────────────────┐
                                     │                                   │
                                     ▼                                   ▼
                          ┌────────────────────┐              ┌────────────────────┐
                          │   MongoDB Atlas     │              │   Groq Cloud API    │
                          │   (Mongoose ODM)    │              │   llama-3.3-70b     │
                          │   7 Collections      │              │   Roadmap/Chat/Quiz │
                          └────────────────────┘              └────────────────────┘
```

**In plain terms:** every time a student generates a roadmap, sends a chat message, or checks off a step, the frontend sends a request here → this backend validates the user, talks to MongoDB and/or Groq → sends back a clean JSON response.

---

## 🧰 Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | ≥18 LTS | JavaScript runtime |
| **Express.js** | v4 | Web framework & routing |
| **Mongoose** | v9 | MongoDB ODM (schemas, validation) |
| **bcryptjs** | v3 | Password hashing |
| **jsonwebtoken (JWT)** | v9 | Auth token generation/verification |
| **Groq SDK** | v1.1 | AI API integration (`llama-3.3-70b-versatile`) |
| **Helmet** | v8 | Security HTTP headers |
| **cors** | v2 | Cross-Origin Resource Sharing |
| **express-rate-limit** | v8 | Rate limiting on sensitive routes |
| **morgan** | v1 | HTTP request logging (dev) |
| **cookie-parser** | v1 | Parses HttpOnly auth cookies |
| **dotenv** | v17 | Environment variable management |

**Database:** MongoDB Atlas (cloud-hosted NoSQL)
**AI Provider:** Groq Cloud (`llama-3.3-70b-versatile`)
**Deployment:** Render / Railway

---

## 📁 Folder Structure

```
backend/
├── server.js                    # App entry point — bootstraps Express
├── package.json                 # Backend dependencies
├── .env                         # Secret environment variables (never commit)
├── .env.example                 # Template for required env vars
│
├── config/
│   └── db.js                    # MongoDB connection setup (Mongoose)
│
├── middleware/
│   ├── authMiddleware.js        # JWT verification / route protection
│   ├── errorMiddleware.js       # Centralized error handler
│   └── rateLimiter.js           # API rate-limiting rules
│
├── models/                      # Mongoose schemas
│   ├── User.js
│   ├── Profile.js
│   ├── Roadmap.js
│   ├── Progress.js
│   ├── ChatHistory.js
│   ├── SavedProject.js
│   └── Resource.js
│
├── controllers/                 # Request handlers (business logic)
│   ├── authController.js
│   ├── userController.js
│   ├── profileController.js
│   ├── roadmapController.js
│   ├── progressController.js
│   ├── chatController.js
│   ├── projectController.js
│   ├── resourceController.js
│   ├── learningController.js
│   └── adminController.js
│
├── routes/                      # Express route definitions
│   ├── authRoutes.js
│   ├── userRoutes.js
│   ├── profileRoutes.js
│   ├── roadmapRoutes.js
│   ├── progressRoutes.js
│   ├── chatRoutes.js
│   ├── projectRoutes.js
│   ├── resourceRoutes.js
│   ├── learningRoutes.js
│   └── adminRoutes.js
│
├── services/
│   ├── aiService.js              # All Groq API communication logic
│   └── fallbackService.js        # Rule-based offline fallback responses
│
├── utils/                        # Shared helper functions
└── data/                         # Seed data / static content
```

---

## 🔄 Request Lifecycle

Every request that hits this API travels through the same pipeline before a response is sent back. Think of it as an **airport security line** — each checkpoint has one job, and a request can't skip ahead.

```
Client Request (from React frontend)
      │
      ▼
┌──────────────────────────────────────────────────┐
│  ① Global Middleware (applied to every route)     │
│     helmet()        → adds security headers       │
│     cors()          → allows only the frontend URL│
│     express.json()  → parses JSON request body     │
│     cookieParser()  → reads the JWT cookie         │
│     morgan()        → logs the request (dev mode) │
└──────────────────┬───────────────────────────────┘
                   ▼
┌──────────────────────────────────────────────────┐
│  ② Route Matching  (server.js)                    │
│     /api/auth      → authRoutes.js                │
│     /api/roadmaps  → roadmapRoutes.js              │
│     /api/chat      → chatRoutes.js                 │
│     ...            → ...                            │
└──────────────────┬───────────────────────────────┘
                   ▼
┌──────────────────────────────────────────────────┐
│  ③ Route-Level Middleware                          │
│     authMiddleware.js → verifies JWT, sets req.user│
│     rateLimiter.js    → blocks excessive requests  │
└──────────────────┬───────────────────────────────┘
                   ▼
┌──────────────────────────────────────────────────┐
│  ④ Controller (business logic)                     │
│     Reads req.body / req.params / req.user         │
│     Calls the database and/or the AI service        │
│     Sends back res.json(...)                        │
└──────────────────┬───────────────────────────────┘
          ┌────────┴────────┐
          ▼                 ▼
   MongoDB Atlas       Groq AI API
   (via Mongoose)      (via groq-sdk)
          └────────┬────────┘
                   ▼
             JSON Response → Client
```

**Example — a single request walked through the pipeline:**
`PUT /api/progress/me` (student marks a roadmap step complete)
1. `helmet`, `cors`, `express.json()` run automatically.
2. Express matches the path to `progressRoutes.js`.
3. `authMiddleware.js` checks the JWT cookie — if invalid, the request stops here with `401`.
4. `progressController.js` updates the `Progress` document in MongoDB.
5. The updated progress is returned as JSON, and the dashboard chart updates.

---

## 🗄️ Database Schema

The backend uses **MongoDB Atlas** with **7 collections**, managed through **Mongoose** schemas.

### Entity Relationship Diagram

```
                     ┌────────────┐
                     │    User    │
                     └─────┬──────┘
     ┌───────────┬─────────┼─────────┬─────────────┐
     ▼           ▼         ▼         ▼             ▼
 ┌────────┐ ┌─────────┐ ┌────────┐ ┌────────────┐ ┌──────────────┐
 │Profile │ │ Roadmap │ │Progress│ │ChatHistory │ │ SavedProject │
 │ (1:1)  │ │(1:many) │ │(1:many)│ │  (1:1)     │ │  (1:many)    │
 └────────┘ └─────────┘ └────────┘ └────────────┘ └──────────────┘

 Admin User ──► Resource (1 : many)
```

### Collections at a Glance

| Collection | Key Fields | Purpose |
|------------|-----------|---------|
| **users** | `name`, `email`, `password` (hashed), `isAdmin`, `onboarded` | Core account record |
| **profiles** | `user`, `goal`, `level`, `weeklyHours`, `bio`, `avatar` | Learner's personalization data |
| **roadmaps** | `user`, `goal`, `level`, `steps[]`, `aiGenerated` | AI-generated learning path |
| **progresses** | `user`, `roadmap`, `completedSteps[]` | Tracks which steps are done |
| **chathistories** | `user`, `messages[]` (`role`, `content`, `timestamp`) | Stores AI chat conversations |
| **savedprojects** | `user`, `title`, `difficulty`, `techStack[]` | Projects a student has bookmarked |
| **resources** | `title`, `url`, `category`, `addedBy` | Admin-curated learning links |

<details>
<summary><strong>📄 Full schema definitions (click to expand)</strong></summary>

```js
// User
{
  name:      String  (required),
  email:     String  (unique, required),
  password:  String  (hashed with bcryptjs),
  isAdmin:   Boolean (default: false),
  onboarded: Boolean (default: false),
  createdAt: Date
}

// Profile
{
  user:        ObjectId → ref: User (unique, required),
  goal:        String,   // e.g. "Learn React"
  level:       String,   // beginner | intermediate | advanced
  weeklyHours: Number,
  bio:         String,
  avatar:      String,
  updatedAt:   Date
}

// Roadmap
{
  user:              ObjectId → ref: User,
  goal:              String,
  level:             String,
  weeklyHours:       Number,
  estimatedDuration: String,
  aiGenerated:       Boolean,
  steps: [{
    stepNumber:  Number,
    title:       String,
    description: String,
    duration:    String,
    resources:   [String]
  }],
  createdAt: Date
}

// Progress
{
  user:           ObjectId → ref: User,
  roadmap:        ObjectId → ref: Roadmap,
  completedSteps: [Number],
  updatedAt:      Date
}

// ChatHistory
{
  user: ObjectId → ref: User,
  messages: [{
    role:      String ("user" | "assistant"),
    content:   String,
    timestamp: Date
  }],
  updatedAt: Date
}

// SavedProject
{
  user:          ObjectId → ref: User,
  title:         String,
  description:   String,
  difficulty:    String,
  techStack:     [String],
  estimatedTime: String,
  savedAt:       Date
}

// Resource
{
  title:       String (required),
  description: String,
  url:         String,
  category:    String,
  addedBy:     ObjectId → ref: User (Admin),
  createdAt:   Date
}
```
</details>

---

## 🤖 AI Integration

The AI layer is powered by **Groq Cloud**, using the `llama-3.3-70b-versatile` model via the `groq-sdk` package. All AI logic is centralized in `services/aiService.js`.

```
                    ┌──────────────────────────────────┐
                    │         aiService.js               │
                    │                                    │
   Roadmap Gen ────►│  generateRoadmapWithAI()          │────► Groq API
   Project Gen ────►│  generateProjectRecommendations() │────► Groq API
   AI Chat     ────►│  chatWithAI()                     │────► Groq API
   Lesson Gen  ────►│  generateStepLesson()             │────► Groq API
   Quiz Gen    ────►│  generateStepQuiz()               │────► Groq API
                    │                                    │
                    │  If GROQ_API_KEY is missing         │
                    │  or the call fails ⤵                │
                    │  fallbackService.js takes over      │
                    └──────────────────────────────────┘
```

### Why a Fallback Service?

If a student's roadmap request hit a dead Groq API with no backup plan, the whole feature — and possibly the app — would break. `fallbackService.js` guarantees the platform **never crashes**, even with no API key configured:

| Feature | If AI Fails... |
|---------|-----------------|
| **Roadmap** | Returns a pre-built, rule-based roadmap matched by goal keywords |
| **Projects** | Returns a hardcoded list of projects grouped by difficulty |
| **Chat** | Returns a friendly "AI temporarily unavailable" message |
| **Quiz** | Returns `null` (frontend hides the quiz gracefully) |

### Prompting Strategy

| Feature | Temperature | Output Format | Reasoning |
|---------|-------------|----------------|-----------|
| Roadmap Generation | `0.3` (low) | JSON object | Low temperature = consistent, valid JSON |
| Project Recommendations | `0.5` | JSON array | Balanced creativity and structure |
| AI Chat | `0.7` (higher) | Markdown text | More natural, conversational tone |
| Step Lesson | `0.5` | Markdown text | Clear but slightly varied explanations |
| Step Quiz | `0.3` (low) | JSON array | Precise, parseable question format |

> 💡 **Analogy:** *temperature* is like a creativity dial. Low temperature (0.3) is a strict teacher who gives the same structured answer every time — good for JSON. High temperature (0.7) is a friendly tutor who explains things more freely — good for chat.

---

## 📡 API Endpoints Reference

All endpoints are prefixed with `/api`. 🔒 = requires login · 🔑 = requires admin role.

### Auth (`/api/auth`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/register` | Public | Create a new account |
| `POST` | `/login` | Public | Log in and set JWT cookie |
| `POST` | `/logout` | Public | Clear the auth cookie |
| `GET` | `/me` | 🔒 | Get the current logged-in user |

### Users (`/api/users`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/` | 🔑 | List all users |
| `DELETE` | `/:id` | 🔑 | Delete a user |

### Profiles (`/api/profiles`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/me` | 🔒 | Get the user's profile |
| `PUT` | `/me` | 🔒 | Update the user's profile |

### Roadmaps (`/api/roadmaps`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/generate` | 🔒 | Generate a new AI roadmap |
| `GET` | `/me` | 🔒 | Get the user's saved roadmap |

### Progress (`/api/progress`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/me` | 🔒 | Get progress on the current roadmap |
| `PUT` | `/me` | 🔒 | Mark a step complete/incomplete |

### Chat (`/api/chat`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/` | 🔒 | Send a message, get an AI reply |
| `GET` | `/history` | 🔒 | Fetch saved chat history |

### Projects (`/api/projects`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/suggestions` | 🔒 | Get AI-recommended project ideas |
| `POST` | `/save` | 🔒 | Save a project |
| `GET` | `/saved` | 🔒 | List saved projects |

### Resources (`/api/resources`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/` | 🔒 | List all resources |
| `POST` | `/` | 🔑 | Add a new resource |
| `DELETE` | `/:id` | 🔑 | Delete a resource |

### Learning (`/api/learning`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/lesson` | 🔒 | Generate an AI mini-lesson for a step |
| `POST` | `/quiz` | 🔒 | Generate an AI quiz for a step |

### Admin (`/api/admin`)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/stats` | 🔑 | Platform-wide analytics |
| `GET` | `/users` | 🔑 | All users with details |

### Health Check
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/health` | Public | Confirms the API is running |

---

## 🔐 Authentication & Security

Authentication uses **JWT tokens stored in HttpOnly cookies** — this means the token can't be accessed via JavaScript in the browser, which protects it from XSS attacks.

### Registration Flow

```
User submits Register form
        │
        ▼
POST /api/auth/register
        │
        ▼
Validate input (unique email, password length)
        │
        ▼
Hash password with bcryptjs (10 salt rounds)
        │
        ▼
Save new User document to MongoDB
        │
        ▼
Generate JWT (signed with JWT_SECRET)
        │
        ▼
Set JWT as HttpOnly Cookie (30-day expiry)
        │
        ▼
Respond with { user: { id, name, email, isAdmin } }
```

### Login Flow

```
User submits Login form
        │
        ▼
POST /api/auth/login
        │
        ▼
Find user by email
        │
        ├── Not found ─────► 401 Unauthorized
        ▼
Compare password (bcryptjs.compare)
        │
        ├── Mismatch ──────► 401 Unauthorized
        ▼
Generate new JWT → set HttpOnly Cookie
        │
        ▼
Respond with user data
```

### Protecting a Route

```
Incoming request to a protected endpoint
        │
        ▼
authMiddleware.js reads the JWT from the cookie
        │
        ├── Missing / invalid / expired ──► 401 Unauthorized
        ▼
Token verified → req.user is set
        │
        ▼
Controller runs with access to req.user
```

### Security Measures Summary

| Layer | Measure | Implementation |
|-------|---------|-----------------|
| **Transport** | HTTPS enforced | Handled by Render/Vercel |
| **Headers** | Security headers | `helmet()` |
| **Auth Token** | JWT in HttpOnly cookie | Not readable by JS → protects against XSS |
| **Passwords** | Hashed, never stored plain | `bcryptjs` |
| **CORS** | Only the frontend origin allowed | `cors()` with whitelist |
| **Rate Limiting** | Throttles abuse on sensitive/AI routes | `express-rate-limit` |
| **Role Guard** | Admin-only routes checked server-side | `isAdmin` check in middleware |

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** v18+ (LTS recommended)
- **npm** v9+
- A **MongoDB Atlas** cluster (free tier is fine)
- A **Groq API key** ([console.groq.com](https://console.groq.com))

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/skillpath-ai-backend.git
cd skillpath-ai-backend

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env
# then fill in the values (see below)

# 4. Start the development server
npm run dev
```

The API will be available at **http://localhost:5000**. Test it's alive with:

```bash
curl http://localhost:5000/api/health
```

---

## 🔑 Environment Variables

Create a `.env` file in the project root:

```env
NODE_ENV=development
PORT=5000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/skillpath
JWT_SECRET=your_super_secret_random_string
GROQ_API_KEY=your_groq_api_key
CLIENT_URL=http://localhost:5173
```

| Variable | Description |
|-----------|-------------|
| `NODE_ENV` | `development` or `production` |
| `PORT` | Port the server listens on |
| `MONGO_URI` | MongoDB Atlas connection string |
| `JWT_SECRET` | Secret used to sign/verify JWT tokens |
| `GROQ_API_KEY` | API key for Groq Cloud (AI features) |
| `CLIENT_URL` | Frontend URL, used for CORS whitelisting |

> ⚠️ Never commit your real `.env` file. Only `.env.example` should be tracked in Git.
> ⚠️ If `GROQ_API_KEY` is left blank, AI features automatically fall back to rule-based responses instead of failing.

---

## 📜 Available Scripts

| Command | What It Does |
|---------|----------------|
| `npm run dev` | Starts the server with hot-reload (nodemon) |
| `npm start` | Starts the server in production mode |
| `npm run lint` | Runs ESLint to check code quality |

---

## 📦 Deployment

### Production Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  INTERNET (Users worldwide)                                  │
└───────────────┬─────────────────────────────────────────────┘
                │
    ┌───────────┴───────────┐
    ▼                       ▼
┌────────────┐       ┌──────────────┐
│  Vercel    │       │  Render /    │
│  (Frontend)│──────▶│  Railway     │
│  React app │       │  THIS BACKEND│
└────────────┘       │  Port 5000   │
                      └──────┬───────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
    ┌─────────────────┐        ┌──────────────────────┐
    │  MongoDB Atlas  │        │  Groq Cloud API      │
    └─────────────────┘        └──────────────────────┘
```

### Deployment Checklist

| Step | Platform | Action |
|------|----------|--------|
| 1 | MongoDB Atlas | Create a cluster and copy the `MONGO_URI` |
| 2 | Groq Console | Generate a `GROQ_API_KEY` |
| 3 | Render / Railway | Connect the GitHub repo, set all env vars, deploy the `backend/` folder |
| 4 | Backend | Set `CLIENT_URL` to the deployed frontend's URL (for CORS) |
| 5 | Frontend | Point `VITE_API_URL` to this backend's deployed URL |
| 6 | Test | Register a user end-to-end and confirm roadmap/chat features work |

---

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m "Add: your feature"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the `LICENSE` file for details.

---

<p align="center">Built with ❤️ to make quality tech education accessible to everyone — in line with <strong>UN SDG 4</strong>.</p>
