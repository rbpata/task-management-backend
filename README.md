# Task Management API

A modern RESTful API for managing tasks and users, built with FastAPI and PostgreSQL. This project demonstrates best practices in backend development including authentication, database migrations, CORS handling, and deployment to production.

## 📋 Table of Contents

- [Features](#features)
- [Project Architecture](#project-architecture)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Setup & Configuration](#setup--configuration)
- [Running the Application](#running-the-application)
- [API Documentation](#api-documentation)
- [Database Migrations](#database-migrations)
- [Testing](#testing)
- [Deployment](#deployment)
- [Contributing](#contributing)

## ✨ Features

- **User Management**: Create users with secure password hashing using bcrypt
- **Authentication**: JWT-based token authentication for secure API access
- **Task Management**: Create, read, update, and delete tasks
- **CORS Support**: Configured for cross-origin requests from frontend applications
- **Database Migrations**: Alembic-managed schema versioning
- **Docker Support**: Containerized deployment ready
- **API Documentation**: Interactive Swagger UI at `/docs`

## 🏗️ Project Architecture

### System Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                        Frontend (Vercel)                      │
│            task-management-frontend-seven-nu                  │
└────────────────────────┬─────────────────────────────────────┘
                         │
                    HTTP/HTTPS
                         │
        ┌────────────────▼────────────────┐
        │  FastAPI Application (Railway)  │
        │ task-management-backend         │
        │  - CORS Middleware              │
        │  - Authentication Layer         │
        │  - API Endpoints                │
        │  - Service Layer                │
        │  - Repository Layer             │
        └────────────────┬────────────────┘
                         │
                      TCP/IP
                         │
        ┌────────────────▼────────────────┐
        │   PostgreSQL Database           │
        │   (Railway Managed)             │
        │   - Users Table                 │
        │   - Tasks Table                 │
        └─────────────────────────────────┘
```

### Application Layers Diagram

```
┌─────────────────────────────────────────────────────────┐
│                   API Layer (Routes)                    │
│  - POST   /users           (Create User)                │
│  - POST   /token           (Login)                      │
│  - GET    /users/me        (Get Current User)           │
│  - POST   /tasks           (Create Task)                │
│  - GET    /tasks           (List Tasks)                 │
│  - GET    /tasks/{id}      (Get Task)                   │
│  - PUT    /tasks/{id}      (Update Task)                │
│  - DELETE /tasks/{id}      (Delete Task)                │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│              Service Layer                              │
│  - UserService (Business Logic)                         │
│  - TaskService (Business Logic)                         │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│             Repository Layer                            │
│  - UserRepository (Data Access)                         │
│  - TaskRepository (Data Access)                         │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│           Database Layer (SQLAlchemy ORM)               │
│  - User Model                                           │
│  - Task Model                                           │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│         PostgreSQL Database                             │
│  - Tables, Indexes, Constraints                         │
└─────────────────────────────────────────────────────────┘
```

### Authentication Flow Diagram

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       │ 1. POST /users (Register)
       ▼
┌──────────────┐     ┌─────────────────┐
│ UserService  │────▶│ UserRepository  │
└──────────────┘     └────────┬────────┘
                              │
                              ▼
                        ┌──────────────┐
                        │  PostgreSQL  │
                        │  users table │
                        └──────────────┘

┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       │ 2. POST /token (Login)
       │    username + password
       ▼
┌──────────────────────────────┐
│  Verify Credentials           │
│  - Check username exists      │
│  - Verify password (bcrypt)   │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Generate JWT Token           │
│ - Claims: sub, exp           │
│ - Signed with SECRET_KEY     │
└──────────┬───────────────────┘
           │
           ▼ Return token to client
┌──────────────────────────────┐
│  Client stores JWT           │
│  Uses in Authorization header│
└──────────┬───────────────────┘
           │
           │ 3. GET /tasks
           │    Authorization: Bearer {token}
           ▼
┌──────────────────────────────┐
│  Verify JWT Token            │
│  Extract user info           │
└──────────┬───────────────────┘
           │
           ▼ Access granted
    ┌─────────────────┐
    │ Execute request │
    └─────────────────┘
```

## 📁 Project Structure

```
backend/
├── alembic.ini                          # Alembic configuration for migrations
├── docker-compose.yml                   # Local PostgreSQL setup
├── Dockerfile                           # Production Docker image
├── .dockerignore                        # Docker build optimization
├── .env.example                         # Environment variables template
├── pyproject.toml                       # Python project configuration
├── main.py                              # Application entry point
│
├── migrations/                          # Database migrations
│   ├── env.py                          # Alembic environment config
│   ├── script.py.mako                  # Alembic template
│   └── versions/                       # Migration files
│       ├── 3b1e2c9d7f40_create_users_table.py
│       ├── 5654cab09dc4_add_email_field_to_users_table.py
│       ├── 9066f19beb6f_added_created_at_field.py
│       └── c589e5628da2_create_tasks_table.py
│
└── src/
    └── app/
        ├── app.py                      # FastAPI application setup
        ├── dependencies.py             # Dependency injection
        ├── security.py                 # Security utilities (JWT, passwords)
        │
        ├── auth/
        │   └── auth.py                # Authentication logic
        │
        ├── config/
        │   ├── database.py            # Database connection
        │   └── settings.py            # Environment settings
        │
        ├── db/
        │   ├── base.py                # SQLAlchemy base config
        │   └── session.py             # Database session management
        │
        ├── models/
        │   ├── user.py                # User model
        │   └── tasks.py               # Task model
        │
        ├── repositories/
        │   ├── user_repository.py     # User data access
        │   └── task_repository.py     # Task data access
        │
        ├── schemas/
        │   ├── user.py                # User request/response schemas
        │   └── task.py                # Task request/response schemas
        │
        └── services/
            ├── user_service.py        # User business logic
            └── task_service.py        # Task business logic

tests/
├── conftest.py                         # Pytest configuration
├── test_dependencies.py                # Dependency tests
├── test_task_repository.py             # Repository tests
├── test_task_service.py                # Service tests
└── integration/
    ├── test_tasks.py                  # Task endpoint tests
    └── test_users.py                  # User endpoint tests
```

## 🛠️ Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Framework** | FastAPI | 0.135.1+ |
| **Web Server** | Uvicorn | 0.41.0+ |
| **Database** | PostgreSQL | 16 |
| **ORM** | SQLAlchemy | 2.0.48+ |
| **Migrations** | Alembic | 1.18.4+ |
| **Authentication** | JWT / OAuth2 | - |
| **Password Hashing** | bcrypt | 4.3.0 |
| **Environment** | Python | 3.14+ |
| **Testing** | pytest | 8.4.2+ |

## 📦 Prerequisites

- **Python 3.14+**
- **PostgreSQL 16** (local or via Docker)
- **pip** or **uv** (Python package manager)
- **Docker & Docker Compose** (optional, for local PostgreSQL)

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/rbpata/task-management-backend.git
cd task-management-backend
```

### 2. Create Virtual Environment

```bash
# Using Python venv
python -m venv .venv

# Activate virtual environment
# On Windows:
.venv\Scripts\activate
# On macOS/Linux:
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -e .
```

To install development dependencies:

```bash
pip install -e ".[dev]"
```

## 🔧 Setup & Configuration

### 1. Configure Environment Variables

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```env
# Secret key for JWT tokens (generate with: openssl rand -hex 32)
SECRET_KEY=your-secret-key-here

# Database URL
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/task_db
```

### 2. Start PostgreSQL (Local Development)

Using Docker Compose:

```bash
docker compose up -d
```

This will start a PostgreSQL container with:
- **Username**: postgres
- **Password**: postgres
- **Database**: task_db
- **Port**: 5432

### 3. Run Database Migrations

```bash
alembic upgrade head
```

This creates all necessary tables:
- `users` - User accounts with authentication
- `tasks` - Tasks with completion status

## ▶️ Running the Application

### Development Mode

```bash
python main.py
```

The API will be available at: `http://localhost:8000`

### Production Mode (with Uvicorn directly)

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Access API Documentation

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## 📚 API Documentation

### Authentication

All protected endpoints require a JWT token in the `Authorization` header:

```
Authorization: Bearer <your_jwt_token>
```

### User Endpoints

#### Create User (Register)
```http
POST /users
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "secure_password"
}
```

**Response (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "john_doe",
  "email": "john@example.com"
}
```

#### Login (Get Token)
```http
POST /token
Content-Type: application/x-www-form-urlencoded

username=john_doe&password=secure_password
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

#### Get Current User
```http
GET /users/me
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "john_doe",
  "email": "john@example.com"
}
```

### Task Endpoints

#### Create Task
```http
POST /tasks
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Complete project",
  "description": "Finish the task management API",
  "completed": false
}
```

**Response (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "title": "Complete project",
  "description": "Finish the task management API",
  "completed": false,
  "created_at": "2026-03-23T10:30:00"
}
```

#### List All Tasks
```http
GET /tasks
Authorization: Bearer <token>
```

**Response (200):**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "title": "Complete project",
    "description": "Finish the task management API",
    "completed": false,
    "created_at": "2026-03-23T10:30:00"
  }
]
```

#### Get Single Task
```http
GET /tasks/{task_id}
Authorization: Bearer <token>
```

#### Update Task
```http
PUT /tasks/{task_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Updated title",
  "description": "Updated description",
  "completed": true
}
```

#### Delete Task
```http
DELETE /tasks/{task_id}
Authorization: Bearer <token>
```

**Response (204):** No content

## 🗄️ Database Migrations

### Understanding Migrations

Migrations are tracked version changes to the database schema. They're stored in `migrations/versions/`:

- **c589e5628da2**: Create tasks table
- **9066f19beb6f**: Add created_at field to tasks
- **3b1e2c9d7f40**: Create users table
- **5654cab09dc4**: Add email field to users table

### Create New Migration

After modifying a model, create a new migration:

```bash
alembic revision --autogenerate -m "Description of changes"
```

### Apply Migrations

```bash
# Apply all pending migrations
alembic upgrade head

# Rollback to previous version
alembic downgrade -1
```

## 🧪 Testing

### Run All Tests

```bash
pytest
```

### Run Tests with Coverage

```bash
pytest --cov=src --cov-report=html
```

### Run Specific Test File

```bash
pytest tests/test_dependencies.py -v
```

### Test Categories

- **Unit Tests**: `test_dependencies.py`, `test_task_service.py`
- **Repository Tests**: `test_task_repository.py`
- **Integration Tests**: `tests/integration/`

## 🚀 Deployment

### Deploy to Railway

#### Prerequisites
- GitHub account with repository pushed
- Railway account (free tier available)

#### Steps

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Deploy to Railway"
   git push origin master
   ```

2. **Create Railway Project**
   - Go to https://railway.app
   - Click "Create Project"
   - Select "Deploy from GitHub"
   - Authorize and select your repository

3. **Add PostgreSQL Database**
   - In Railway dashboard, click "+ Add"
   - Select "PostgreSQL"
   - Railway automatically creates database

4. **Configure Environment Variables**
   - In Railway Variables:
     - `SECRET_KEY`: Generate with `openssl rand -hex 32`
     - `DATABASE_URL`: Auto-injected by Railway PostgreSQL service

5. **Deploy**
   - Railway automatically deploys on push
   - View logs in dashboard
   - Your app is available at: `https://<app-name>.up.railway.app`

### Environment-Specific Configuration

**Development** (`.env`):
```env
SECRET_KEY=dev-key
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/task_db
```

**Production** (Railway):
```env
SECRET_KEY=<generated-secure-key>
DATABASE_URL=<auto-set-by-railway>
```

## 📋 CORS Configuration

The application allows requests from:
- Local development: `http://localhost:5173-5176`
- Production Frontend: `https://task-management-frontend-seven-nu.vercel.app`

To add more origins, edit `src/app/app.py`:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://your-frontend-url.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## 🐛 Troubleshooting

### Database Connection Error
```
sqlalchemy.exc.OperationalError: connection to server failed
```
**Solution**: Ensure PostgreSQL is running and `DATABASE_URL` is correct.

### CORS Error
```
Access to XMLHttpRequest blocked by CORS policy
```
**Solution**: Add your frontend URL to `allow_origins` in `app.py`.

### Migration Conflict
```
relation "tasks" already exists
```
**Solution**: Migrations are designed to be idempotent (safe to re-run). They check if tables exist before creating.

### JWT Token Expired
**Solution**: Get a new token by logging in again at `/token`.

## 📞 Support & Contribution

For issues or suggestions:
1. Check existing issues on GitHub
2. Create a new issue with detailed description
3. Submit pull requests for improvements

## 📄 License

This project is part of a training program. Use for learning purposes.

---

**Happy Coding! 🎉**

For questions or issues, visit: https://github.com/rbpata/task-management-backend
