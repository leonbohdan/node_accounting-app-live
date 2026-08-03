# Node Accounting App

REST API for financial accounting — managing users, expenses, and todos. Built with Node.js, Express, and PostgreSQL (Sequelize ORM).

## Tech Stack

| Technology | Purpose |
|------------|---------|
| **Node.js** | Runtime environment |
| **Express 5** | HTTP framework |
| **PostgreSQL** | Database |
| **Sequelize** | ORM for database interaction |
| **dotenv** | Environment variable management |
| **cors** | Cross-Origin request support |
| **nodemon** | Auto-restart in development mode |

## Project Structure

```
node_accounting-app-live/
├── src/
│   ├── api/
│   │   ├── expenses/       # Expenses module
│   │   ├── todos/          # Todos module
│   │   ├── users/          # Users module
│   │   └── routes.js       # Route registration
│   ├── createServer.js     # Express app factory
│   ├── db.js               # PostgreSQL connection
│   ├── index.js            # Entry point
│   ├── models.js           # Aggregated model exports
│   └── setupDb.js          # Database table initialization
├── .env.example            # Environment variables template
├── DEPLOY.md               # Deployment guide (Ukrainian)
├── DEPLOY.en.md            # Deployment guide (English)
└── package.json
```

## API Endpoints

### Users

| Method | URL | Description |
|--------|-----|-------------|
| `GET` | `/users` | Get all users |
| `GET` | `/users/:id` | Get user by ID |
| `POST` | `/users` | Create a new user |
| `PATCH` | `/users/:id` | Update a user |
| `DELETE` | `/users/:id` | Delete a user |

### Expenses

| Method | URL | Description |
|--------|-----|-------------|
| `GET` | `/expenses` | Get all expenses |
| `GET` | `/expenses/:id` | Get expense by ID |
| `POST` | `/expenses` | Create a new expense |
| `PATCH` | `/expenses/:id` | Update an expense |
| `DELETE` | `/expenses/:id` | Delete an expense |

### Todos

| Method | URL | Description |
|--------|-----|-------------|
| `GET` | `/todos` | Get all todos |
| `GET` | `/todos/:id` | Get todo by ID |
| `POST` | `/todos` | Create a new todo |
| `PATCH` | `/todos/:id` | Update a todo |
| `DELETE` | `/todos/:id` | Delete a todo |

## Local Development

### Prerequisites

- Node.js >= 20.6
- PostgreSQL >= 14
- pnpm (or npm)

### 1. Clone the repository

```bash
git clone <repository-url>
cd node_accounting-app-live
```

### 2. Install dependencies

```bash
pnpm install
```

### 3. Configure environment variables

Copy the template and fill in your values:

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
PORT=3000
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_DB=accounting
POSTGRES_DIALECT=postgres
```

### 4. Initialize the database

First create the database in PostgreSQL, then run:

```bash
pnpm setupDb
```

This will create all required tables (`todos`, `users`, `expenses`).

### 5. Start in development mode

```bash
pnpm dev
```

The server will start at [http://localhost:3000](http://localhost:3000) with auto-restart on file changes.

## Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start with nodemon (development) |
| `pnpm start` | Start without nodemon (production) |
| `pnpm startProd` | Start with `.env` (production simulation) |
| `pnpm setupDb` | Initialize database tables |
| `pnpm lint` | Run ESLint code check |
| `pnpm format` | Format code with Prettier |

## Deployment

For deployment to **Render.com**, see the step-by-step guide: [DEPLOY.en.md](./docs/DEPLOY.en.md)

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PORT` | Server port | `3000` |
| `POSTGRES_HOST` | PostgreSQL host | `localhost` |
| `POSTGRES_PORT` | PostgreSQL port | `5432` |
| `POSTGRES_USER` | Database user | `postgres` |
| `POSTGRES_PASSWORD` | Database password | `secret` |
| `POSTGRES_DB` | Database name | `accounting` |
| `POSTGRES_DIALECT` | Sequelize dialect | `postgres` |
