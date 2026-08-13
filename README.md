# Recommendation System for Events

This repository contains a Laravel-based web platform that recommends events based on popularity, location, category, promoter activity, and each user’s historical interactions.

It is designed as a practical, full-stack project that showcases backend architecture, data modeling, recommendation logic, and containerized delivery.

## Why this project matters

- Demonstrates a complete product flow: event discovery, user authentication, and transactional interactions.
- Implements recommendation-oriented ranking logic directly in SQL/Eloquent queries.
- Includes personalization by updating user preference signals after each interaction.
- Uses production-like tooling: Docker, Nginx, MySQL, CI checks, and automated formatting/tests.

## Core product features

- Public event catalog with search.
- Event details page with upcoming performances.
- Popular events feed based on interaction scoring.
- Personalized home feed for logged-in users.
- Recommendations by:
  - City / location
  - Event category
  - Promoter
  - Hall
- Interaction tracking through:
  - `tickets` (strong positive signal)
  - `deleted_ticket` (weaker signal, e.g. cart/abandoned action)

## Technology stack

### Backend
- PHP 8.2
- Laravel 11
- Eloquent ORM
- Blade templating
- Laravel Auth (`laravel/ui`)

### Data & infrastructure
- MySQL
- Docker + Docker Compose
- Nginx (reverse proxy / web server)
- phpMyAdmin for DB inspection

### Frontend tooling
- Vite
- Bootstrap 5
- Sass
- Tailwind CSS (available in tooling)
- Axios

### Quality & CI
- Prettier + `@prettier/plugin-php`
- PHPUnit via `php artisan test`
- GitHub Actions workflow for formatting and tests

## High-level architecture

The application follows Laravel’s MVC structure inside `/src`:

- **Controllers**: request orchestration and view composition
  - `EventController` (catalog, search, details)
  - `HomeController` (personalized dashboard)
  - `TransactionController` (user interactions that influence recommendations)
- **Models**: domain and recommendation query logic
  - `Event`, `Performance`, `Hall`, `Promoter`, `User`, `Ticket`, `DeletedTicket`, `Transaction`, `Category`, `City`
- **Views**: Blade templates for listing/searching/recommending events
- **Routes**: web routes in `src/routes/web.php`
- **Persistence**: MySQL schema via Laravel migrations

## Recommendation logic (business overview)

The ranking logic is implemented mainly in:

- `src/app/Models/Event.php`
- `src/app/Models/Hall.php`
- `src/app/Models/Promoter.php`
- `src/app/Http/Controllers/HomeController.php`

### Scoring model

Popularity scores are computed with weighted signals:

- `tickets` contribution: **2 points**
- `deleted_ticket` contribution: **1 point**

This creates a simple but effective ranking formula:

- Events with real purchases/confirmed interest rise faster.
- Soft intent still contributes to trend detection.

### Personalization model

After a user interaction (`buy` or `cart`), the system:

1. stores the interaction record
2. recalculates user preference anchors:
   - `city_id_e`
   - `category_id_e`
   - `promoter_id_e`
   - `hall_id_e`
3. clears user-specific caches
4. builds personalized feed sections from those anchors

This hybrid strategy combines:
- global popularity
- contextual popularity (city/category)
- user-specific affinity signals

## Data model snapshot

Main entities:

- `events` (title, description, image, category)
- `performance` (event schedule + hall + promoter)
- `hall` (venue and city)
- `promoters`
- `category`
- `cities`
- `transaction`
- `tickets`
- `deleted_ticket`
- `users` (+ personalized preference columns)

Relationships are represented through Eloquent model methods and used for feed composition and scoring joins.

## Caching strategy

Laravel cache is used to reduce repeated heavy ranking queries:

- Popular events by city/category
- Popular halls and promoters
- Performance lists per hall/promoter clusters
- User-specific recommendation segments

Caches are invalidated for user-personalized keys after new user interactions.

## Project structure

```text
recommendation_system/
├─ docker-compose.yml
├─ Dockerfile
├─ Makefile
├─ nginx/
│  └─ default.conf
└─ src/
   ├─ app/
   ├─ config/
   ├─ database/
   ├─ resources/views/
   ├─ routes/
   └─ tests/
```

## Local setup

### Prerequisites

- Docker + Docker Compose
- GNU Make (optional, but recommended)

### Quick start (recommended)

From repository root:

```bash
make run-app-with-setup-db
```

This will:
- copy `.env`
- build/start containers
- install Composer and npm dependencies
- generate app key
- run fresh migrations + seeding

### Useful commands

- `make run-app` – start containers
- `make kill-app` – stop containers
- `make enter-php-container` – shell into PHP container
- `make flush-db` – fresh migration
- `make flush-db-with-seeding` – fresh migration with seeding
- `make code-format-check` – run Prettier check
- `make code-test` – run Laravel test suite

### Default local endpoints

- App: `http://localhost:8001`
- phpMyAdmin: `http://localhost:8082`
- MySQL host port: `4306` (configurable via root `.env`)

## Testing and CI

CI workflow (`.github/workflows/code-check.yml`) runs:

1. JavaScript/PHP formatting check via Prettier
2. Laravel tests with SQLite in CI

This ensures baseline code quality for each push.

## Recruiter-focused summary

This project highlights practical engineering competencies:

- Designing domain models for recommendation scenarios
- Translating business intent into SQL-backed scoring strategies
- Building personalized experiences with incremental user profiling
- Delivering containerized, reproducible environments
- Applying CI-driven quality checks in a team-friendly workflow

It is a solid showcase project for backend/full-stack roles using PHP/Laravel in data-driven product contexts.
