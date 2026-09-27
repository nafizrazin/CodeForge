# CodeForge 3.0 — SvelteKit + Laravel 11 + MySQL

This is the framework migration of the working CodeForge plain-PHP project. The feature set and database history are preserved, while the application is split into a SvelteKit frontend and Laravel 11 API backend.

## Stack

- **Frontend:** SvelteKit + Svelte 5 + TypeScript
- **Backend:** Laravel 11 + Laravel Sanctum
- **Database:** MySQL / XAMPP-compatible MySQL/MariaDB
- **Authentication:** first-party SPA session cookies + CSRF protection
- **UI:** Space Grotesk + JetBrains Mono, aggressive dark CodeForge design

## Feature parity

The migration contains the working features from the current project:

- Dedicated animated public landing page
- Registration, login and logout
- Protected dashboard
- Problem library and practice sessions
- Safe deterministic prototype programming judge
- Contest creation (admin), participation, scoreboards and quick duels
- Rivalry / head-to-head comparison
- University leaderboard and university comparison
- Global search
- User profiles
- Performance Profile analytics and radar chart
- Ghost Race historical timeline replay
- SQL Battle practice, battles and leaderboard
- Gamification / XP / levels / badges
- Authenticated database view
- Admin user management
- Admin read-only SQL Lab
- Technical “How it works” page

## Important compatibility decision

The migration **does not replace the existing CodeForge application tables**. Laravel's adoption migration checks for the current tables and creates only missing tables. A separate optimization migration adds missing indexes without deleting data.

That means your existing MySQL `project` database — including the high-level accounts and their history — can be reused.

## Folder structure

```text
codeforge/
├── backend/                 Laravel 11 API
│   ├── app/
│   │   ├── Http/Controllers
│   │   ├── Http/Middleware
│   │   ├── Models
│   │   └── Services
│   ├── database/
│   │   ├── migrations
│   │   ├── seeders
│   │   └── legacy_schema_and_seed.sql
│   ├── routes/
│   └── tools/contract_test.php
├── frontend/                SvelteKit app
│   ├── src/lib
│   ├── src/routes
│   └── tools/contract-test.mjs
├── setup-codeforge.ps1
└── start-codeforge.ps1
```


## Migrating directly from the current plain-PHP CodeForge

If you are using the accompanying `PATCH-009-Framework-Migration.ps1`, put that patch in the current `D:\xampp\htdocs\codeforge` folder and run:

```powershell
cd D:\xampp\htdocs\codeforge
powershell -ExecutionPolicy Bypass -File .\PATCH-009-Framework-Migration.ps1
```

The patch archives the current plain-PHP source under `legacy-plain-php-*`, installs this SvelteKit/Laravel codebase, preserves the existing MySQL `project` data, runs source regression checks, and then runs `setup-codeforge.ps1`. Use `-SkipSetup` only if you want to install Composer/npm dependencies later.

The root `index.php` is only a convenience bridge that redirects `http://localhost/codeforge/` to the SvelteKit frontend at `http://localhost:5173/`. All application logic is in SvelteKit/Laravel after migration.

## One-time setup on Windows/XAMPP

### Requirements

Install/enable:

- XAMPP with PHP **8.2+** and MySQL running
- Composer 2
- Node.js 20+ and npm

### Automatic setup

Open PowerShell in the project root and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\setup-codeforge.ps1
```

The script will:

1. prepare `backend/.env` and `frontend/.env`;
2. install Composer packages;
3. generate the Laravel app key;
4. run the non-destructive migrations and index optimization;
5. run the safe demo seeder (it **skips automatically if users already exist**);
6. run backend contract tests;
7. install frontend packages;
8. run SvelteKit contract checks and `svelte-check`;
9. build the SvelteKit production bundle.

### Database port

The backend example is set to the XAMPP port used during the previous CodeForge setup:

```env
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=project
DB_USERNAME=root
DB_PASSWORD=
```

If your MySQL is on `3306`, change only `DB_PORT` in `backend/.env`.

## Verification

Source-only regression verification (does not require installed npm/Composer dependencies):

```powershell
powershell -ExecutionPolicy Bypass -File .\verify-codeforge.ps1
```

After setup, run the full framework/compiler verification:

```powershell
powershell -ExecutionPolicy Bypass -File .\verify-codeforge.ps1 -FullBuild
```

## Start the application

For development, run:

```powershell
powershell -ExecutionPolicy Bypass -File .\start-codeforge.ps1
```

To serve the compiled SvelteKit adapter-node build instead, run:

```powershell
powershell -ExecutionPolicy Bypass -File .\start-codeforge-production.ps1
```

Then open:

- Frontend: `http://localhost:5173`
- Backend health check: `http://localhost:8000/up`

The start script launches Laravel and SvelteKit in separate PowerShell windows.

## Manual setup commands

Backend:

```powershell
cd backend
copy .env.example .env
composer install
D:\xampp\php\php.exe artisan key:generate
D:\xampp\php\php.exe artisan migrate --force
D:\xampp\php\php.exe artisan db:seed --force
D:\xampp\php\php.exe tools\contract_test.php
```

Frontend:

```powershell
cd frontend
copy .env.example .env
npm install
npm run test:contract
npm run check
npm run build
```

## Demo accounts for a clean database

The Laravel seeder creates these only when the `users` table is empty:

- `Ismail` / `123456`
- `Admin` / `admin123`

When migrating your existing database, the seeder does nothing, so your current accounts remain unchanged.

## Authentication architecture

```text
SvelteKit browser
    ↓ GET /sanctum/csrf-cookie
Laravel CSRF cookie
    ↓ POST /login or /register
Laravel session
    ↓ credentialed /api/* requests
Sanctum auth:sanctum middleware
```

No authentication token is placed in `localStorage`.

## Code judge note

The migration intentionally preserves the current **safe prototype judge**. Submitted C++/Python/Java source is not executed on the host machine. The service produces deterministic verdict/performance metadata for the software-lab workflow.

A containerized real judge can later replace `PrototypeJudgeService` without changing the SvelteKit API contract or the submission/session database model.

## SQL safety

SQL Battle:

- accepts one SELECT/WITH statement;
- blocks destructive SQL and multiple statements;
- blocks production application tables;
- allows only `arena_*` tables;
- limits result rows;
- supports MariaDB `max_statement_time` and MySQL `MAX_EXECUTION_TIME` when available.

Admin SQL Lab is separately protected by the `admin` middleware and is read-only.

## Tests included

Backend source checks:

```powershell
D:\xampp\php\php.exe backend\tools\contract_test.php
```

Frontend route/API/UI contract checks:

```powershell
cd frontend
npm run test:contract
```

After dependencies are installed:

```powershell
npm run check
npm run build
```

## Migration audit

See `MIGRATION_AUDIT.md` for the old-page → new-route mapping and the regression/optimization work completed during the migration.

## Feature Demo

A complete visual walkthrough of CodeForge is available here:

[View the CodeForge Feature Demo Guide](docs/DEMO_GUIDE.md)
