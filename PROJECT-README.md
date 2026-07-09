# 🎯 SkillPath AI

> **AI-Powered Personalized Learning & Project Guidance Platform**
> A full-stack MERN application aligned with **UN SDG 4 — Quality Education**.

[![React](https://img.shields.io/badge/Frontend-React-61DAFB?logo=react&logoColor=black)](./frontend)
[![Node.js](https://img.shields.io/badge/Backend-Node.js-339933?logo=node.js&logoColor=white)](./backend)
[![MongoDB](https://img.shields.io/badge/Database-MongoDB-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com/atlas)
[![Groq](https://img.shields.io/badge/AI-Groq%20Cloud-orange)](https://groq.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#license)

---

## 💡 What It Does

Most self-learners quit because they don't know **what to learn next**. SkillPath AI fixes that by acting as a **personal AI mentor**:

> *Example:* A student sets the goal "Become a Web Developer." SkillPath AI asks a few quick questions (skill level, hours/week available), then generates a custom step-by-step roadmap — like "Week 1: HTML/CSS basics → Week 2: JavaScript fundamentals → Week 3: Build a to-do app." The student can chat with the AI anytime they're stuck, and check off steps as they progress.

## 🧩 Core Features

| Feature | What It Does |
|---------|----------------|
| 🗺️ **AI Roadmap Generator** | Builds a personalized, step-by-step learning path |
| 💬 **AI Doubt Chat** | 24/7 conversational help when a student is stuck |
| 💡 **Project Recommender** | Suggests real projects matched to skill level |
| 📊 **Progress Tracker** | Visual dashboard showing % complete |
| 📚 **Resource Library** | Admin-curated learning links and videos |
| 🛠️ **Admin Panel** | Manage users and resources platform-wide |

## 🏗️ How It's Built

```
┌────────────┐   REST API (Axios)   ┌────────────┐   Mongoose   ┌──────────────┐
│  Frontend  │ ───────────────────▶ │  Backend   │ ────────────▶│  MongoDB     │
│  (React)   │ ◀─────────────────── │  (Node/    │              │  Atlas       │
│            │       JSON            │  Express)  │──── Groq SDK ─▶│ Groq AI API │
└────────────┘                       └────────────┘              └──────────────┘
```

| Layer | Tech | Deployed On |
|-------|------|-------------|
| **Frontend** | React 19 + Vite | Vercel |
| **Backend** | Node.js + Express | Render / Railway |
| **Database** | MongoDB Atlas | MongoDB Cloud |
| **AI** | Groq (`llama-3.3-70b-versatile`) | Groq Cloud |

## 📂 Project Structure

```
skillpath-ai/
├── frontend/    → React app (see frontend/README.md)
├── backend/     → Node.js/Express API (see backend/README.md)
└── docs/        → Architecture & teaching documentation
```

## 🚀 Quick Start

```bash
# Backend
cd backend && npm install && npm run dev     # http://localhost:5000

# Frontend (in a new terminal)
cd frontend && npm install && npm run dev    # http://localhost:5173
```

Each folder has its own `.env.example` — copy it to `.env` and fill in your MongoDB URI, JWT secret, and Groq API key before running.

📖 **For full setup, API docs, and architecture details**, see:
- [`frontend/README.md`](./frontend/README.md)
- [`backend/README.md`](./backend/README.md)

## 🌍 SDG Alignment

**SDG 4 — Quality Education**, Targets **4.4** (technical skills for employment) and **4.b** (expanding learning opportunities) — by giving every learner free, personalized, AI-guided access to structured tech education.

## 📄 License

MIT License — see the `LICENSE` file for details.

---

<p align="center">Built with ❤️ to make quality tech education accessible to everyone.</p>
