# matcha

This project was created with [Better-T-Stack](https://github.com/AmanVarshney01/create-better-t-stack), a modern TypeScript stack that combines React, TanStack Router, Express, TRPC, and more.

## Features

- **TypeScript** - For type safety and improved developer experience
- **TanStack Router** - File-based routing with full type safety
- **TailwindCSS** - Utility-first CSS for rapid UI development
- **shadcn/ui** - Reusable UI components
- **Express** - Fast, unopinionated web framework
- **tRPC** - End-to-end type-safe APIs
- **Bun** - Runtime environment
- **ORM** - TypeScript-first ORM
- **SQLite/Turso** - Database engine
- **Biome** - Linting and formatting

## Getting Started

First, install the dependencies:

```bash
pnpm install
```
## Database Setup

This project uses SQLite with none.

1. Start the local SQLite database:
```bash
cd apps/server && pnpm db:local
```


2. Update your `.env` file in the `apps/server` directory with the appropriate connection details if needed.

3. Apply the schema to your database:
```bash
pnpm db:push
```


Then, run the development server:

```bash
pnpm dev
```

Open [http://localhost:3001](http://localhost:3001) in your browser to see the web application.
The API is running at [http://localhost:3000](http://localhost:3000).



## Project Structure

```
matcha/
├── apps/
│   ├── web/         # Frontend application (React + TanStack Router)
│   └── server/      # Backend API (Express, TRPC)
```

## Available Scripts

- `pnpm dev`: Start all applications in development mode
- `pnpm build`: Build all applications
- `pnpm dev:web`: Start only the web application
- `pnpm dev:server`: Start only the server
- `pnpm check-types`: Check TypeScript types across all apps
- `pnpm db:push`: Push schema changes to database
- `pnpm db:studio`: Open database studio UI
- `pnpm check`: Run Biome formatting and linting
# matcha
