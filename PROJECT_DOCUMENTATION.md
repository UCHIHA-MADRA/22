# Poetry Hub - Project Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Core Features](#core-features)
6. [Database Schema](#database-schema)
7. [API Endpoints](#api-endpoints)
8. [Authentication & Authorization](#authentication--authorization)
9. [Deployment](#deployment)
10. [Local Development Setup](#local-development-setup)
11. [Configuration & Environment Variables](#configuration--environment-variables)
12. [Runtime & Operations](#runtime--operations)
13. [Production Readiness Assessment](#production-readiness-assessment)
14. [Recommendations for Production](#recommendations-for-production)
15. [Known Issues](#known-issues)

## Project Overview

Poetry Hub is a web application designed for poetry enthusiasts to discover, share, and translate poetry in multiple languages. The platform currently supports English and Hindi, with a foundation for expanding to additional languages. Users can create accounts, publish poems, schedule publications, and view poems in their preferred language.

### Key Features

- User registration and authentication
- Multilingual poetry content (English and Hindi support)
- Poetry creation, editing, and publishing
- Scheduled post publishing via background tasks
- User profiles and dashboards
- API endpoints for programmatic access
- Responsive web design

## Architecture

Poetry Hub follows a modular Flask application architecture with the following components:

- **Web Application**: Flask-based web server handling HTTP requests
- **Database**: SQL database (configured via `DATABASE_URL`; SQLite by default in development)
- **Task Queue**: Celery with Redis for background task processing
- **Web Server**: Nginx for production deployment
- **Containerization**: Docker and Docker Compose for consistent deployment
- **Serverless Deployment**: Vercel configuration for serverless deployment option

### Architecture Diagram

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Web Browser    │────▶│  Nginx          │────▶│  Flask App      │
│                 │     │  (Web Server)   │     │  (Web App)      │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Celery Worker  │◀───▶│  Redis          │◀───▶│  MySQL          │
│  (Task Queue)   │     │  (Message Broker)│     │  (Database)     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Technology Stack

### Backend
- **Framework**: Flask 2.3.3
- **Database ORM**: SQLAlchemy 2.0.21
- **Authentication**: Flask-Login 0.6.3 (session-based)
- **Password Hashing**: Flask-Bcrypt 1.0.1
- **Database Migrations**: Flask-Migrate 4.0.5 (library present; migrations not initialized)
- **Email Handling**: Flask-Mail 0.9.1
- **Form Handling**: Flask-WTF 1.1.1
- **Task Queue**: Celery 5.3.1 (optional; guarded if not configured)
- **Message Broker**: Redis 4.6.0

### Frontend
- **Template Engine**: Jinja2 3.1.2
- **CSS Framework**: Bootstrap 5.3.0
- **Icons**: Font Awesome 6.4.0
- **JavaScript Libraries**: Moment.js, AOS Animation

### DevOps & Deployment
- **Containerization**: Docker
- **Container Orchestration**: Docker Compose
- **Web Server**: Nginx
- **WSGI Server**: Gunicorn 21.2.0
- **Serverless Platform**: Vercel

## Project Structure

The project follows a modular structure with blueprints for different functional areas:

```
poetry_hub/
├── api/                    # Vercel serverless functions
├── app/                    # Main application package
│   ├── blueprints/         # Route modules
│   │   ├── api/            # API endpoints
│   │   ├── auth/           # Authentication routes
│   │   └── main/           # Main site routes
│   ├── models/             # Database models
│   ├── static/             # Static assets (CSS, JS)
│   ├── templates/          # HTML templates
│   └── utils/              # Utility functions
├── instance/               # Instance-specific files (SQLite DB present by default)
├── migrations/             # Database migrations (not yet created)
├── tests/                  # Test suite (not implemented yet)
├── .env                    # Environment variables (local, not committed)
├── config.py               # Application configuration
├── celery_worker.py        # Celery worker configuration
├── docker-compose.yml      # Docker services configuration
├── Dockerfile              # Docker container configuration
├── requirements.txt        # Python dependencies
└── run.py                  # Application entry point
```

## Core Features

### User Management

The application provides user registration, authentication, and profile management through the `User` model and the `auth` blueprint. Key features include:

- User registration with email verification
- Secure password hashing with bcrypt
- User login and session management
- User profiles with activity tracking

### Poetry Content Management

Poetry content is managed through the `Post` and `PostTranslation` models, allowing for multilingual content:

- Create, edit, and delete poems
- Support for multiple language translations
- Scheduled publishing via Celery tasks
- Content categorization and tagging

### Multilingual Support

The application supports multiple languages through the `Language` model and translation system:

- Language selection for user interface
- Content translation for poems
- Language-specific content discovery

## Database Schema

The database schema includes the following key tables:

### Users Table
- `id`: Primary key
- `username`: Unique username
- `email`: Unique email address
- `password_hash`: Hashed password
- `active`: User account status
- `is_admin`: Administrator flag
- `created_at`: Account creation timestamp
- `updated_at`: Last update timestamp

### Posts Table
- `id`: Primary key
- `title`: Post title
- `author_id`: Foreign key to users table
- `is_published`: Publication status
- `is_scheduled`: Scheduling flag
- `scheduled_for`: Scheduled publication time
- `published_at`: Actual publication timestamp
- `created_at`: Creation timestamp
- `updated_at`: Last update timestamp
- `task_id`: Celery task ID for scheduled posts

### Post Translations Table
- `id`: Primary key
- `post_id`: Foreign key to posts table
- `language_id`: Foreign key to languages table
- `title`: Translated title
- `content`: Translated content
- `created_at`: Creation timestamp
- `updated_at`: Last update timestamp

### Languages Table
- `id`: Primary key
- `code`: Language code (e.g., 'en', 'hi')
- `name`: Language name
- `is_active`: Language availability flag

## API Endpoints

The application exposes endpoints via two blueprints:

- `api` (mounted at `/api`): content endpoints
- `auth` (mounted at `/auth`): web auth pages and JSON auth endpoints

### Posts API (under `/api`)
- `GET /api/posts` — List published posts with pagination. Query params: `page`, `per_page`, `language`.
- `GET /api/posts/<post_id>` — Get a specific post. Query param: `language`.
- `POST /api/posts` — Create a new post. Requires session auth. Body may include `title`, `english_title`, `english_content`, `hindi_title`, `hindi_content`, optional `schedule_date` (ISO).
- `PUT /api/posts/<post_id>` — Update a post (author only). Update `title` and translations.
- `DELETE /api/posts/<post_id>` — Delete a post (author only).
- `POST /api/posts/<post_id>/publish` — Publish a post (author only).
- `GET /api/user/posts` — Current user posts (auth required).
- `GET /api/stats` — Current user stats (auth required).

### Authentication API (under `/auth`)
- `POST /auth/api/login` — Session login with JSON `{ username, password }`.
- `POST /auth/api/register` — Create a user with JSON `{ username, email, password }`.
- `POST /auth/api/logout` — Logout current session.

## Authentication & Authorization

The application uses Flask-Login for session-based authentication. There is no token/JWT auth implemented.

- Session-based authentication for web interface and protected API endpoints
- `User.is_admin` flag exists; admin-only routes are not implemented yet
- Route protection with `login_required` decorator

## Deployment

The application supports multiple deployment options:

### Docker Deployment

The project includes Docker and Docker Compose configurations for containerized deployment:

- Web application container
- MySQL database container
- Redis container for message broker
- Celery worker container
- Nginx container for reverse proxy

### Vercel Deployment

The project includes Vercel configuration for serverless deployment:

- API routes via Vercel Python runtime
- Static assets via Vercel static hosting

## Local Development Setup

1. Create and activate a virtual environment.
   - Windows (PowerShell): `python -m venv .venv; .\\.venv\\Scripts\\Activate.ps1`
2. Install dependencies: `pip install -r requirements.txt`
3. Create a `.env` file in project root (see next section).
4. Initialize the database by running the app once: `python run.py`.
   - On first run, default languages `en` and `hi` are inserted.
5. Start the app: `python run.py`
   - Runs development config unless `VERCEL` is set.

Common URLs:
- Web: `/`, `/browse`, `/dashboard` (login), `/auth/login`, `/auth/register`
- Health: `/health`
- API: `/api/...` and `/auth/api/...`

## Configuration & Environment Variables

The app loads config from `config.py` and environment variables (via `python-dotenv`).

Required:
- `SECRET_KEY` — Flask secret key
- `DATABASE_URL` — SQLAlchemy URL (e.g., `sqlite:///poetry_hub.db`, `mysql+pymysql://user:pass@host/db`)
- `MAIL_USERNAME` — SMTP username
- `MAIL_PASSWORD` — SMTP password

Optional (defaults apply):
- `MAIL_SERVER` (default: `smtp.gmail.com`)
- `MAIL_PORT` (default: `587`)
- `MAIL_USE_TLS` (default: `True`)
- `MAIL_DEFAULT_SENDER`
- `FLASK_ENV` (`development` or `production`)

Notes:
- Development config enables `DEBUG=True` and will fall back to SQLite if `DATABASE_URL` is not set; however, base `Config` requires `SECRET_KEY` and mail credentials for startup, so include them locally.

## Runtime & Operations

- Database tables are created at startup (`db.create_all()` in `create_app`).
- Default languages are seeded in `run.py` on first run (`en`, `hi`).
- Health check: `GET /health` returns status/version JSON.
- Scheduling: `schedule_date` on `POST /api/posts` enqueues a Celery task if Celery/Redis are configured; otherwise scheduling will return an error while normal creation succeeds.
- Logging: basic console logging is configured in `run.py`.

## Production Readiness Assessment

Based on the current state of the project, here's an assessment of its production readiness:

### Strengths

- Well-structured modular architecture
- Comprehensive Docker configuration
- Environment variable management
- Database migration support
- Multilingual content support
- Background task processing

### Areas for Improvement

1. **Testing**: No test suite is currently implemented
2. **Logging**: Basic logging configuration needs enhancement
3. **Error Handling**: Error handling could be more comprehensive
4. **Security**: Additional security measures needed
5. **Monitoring**: No monitoring or alerting system
6. **CI/CD**: No continuous integration/deployment pipeline
7. **Documentation**: Limited code and API documentation
8. **Performance**: No caching or performance optimization
9. **Dependencies**: `requirements.txt` has conflict markers and duplicate entries

## Recommendations for Production

To make Poetry Hub production-ready, the following improvements are recommended:

### 1. Testing

- Implement unit tests for models and utilities
- Add integration tests for API endpoints
- Create end-to-end tests for critical user flows
- Set up test coverage reporting

```python
# Example test file structure
/tests
  /unit
    /models
      test_user.py
      test_post.py
    /utils
      test_tasks.py
  /integration
    test_api.py
    test_auth.py
  /e2e
    test_user_journey.py
  conftest.py
```

### 2. Logging and Monitoring

- Implement structured logging with proper log levels
- Add request ID tracking across services
- Set up centralized log collection (e.g., ELK stack)
- Implement health check endpoints
- Add application metrics collection (e.g., Prometheus)
- Set up alerting for critical errors

```python
# Example logging configuration
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
        'json': {
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s',
            'class': 'pythonjsonlogger.jsonlogger.JsonFormatter'
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'standard',
            'level': 'INFO'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'logs/app.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 10,
            'formatter': 'json',
            'level': 'INFO'
        }
    },
    'loggers': {
        '': {
            'handlers': ['console', 'file'],
            'level': 'INFO'
        }
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
```

### 3. Security Enhancements

- Implement rate limiting for API endpoints
- Add CSRF protection for all forms
- Set up Content Security Policy headers
- Implement proper CORS configuration
- Add request validation middleware
- Conduct security audit and penetration testing

```python
# Example rate limiting with Flask-Limiter
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@api.route('/posts', methods=['POST'])
@limiter.limit("5 per minute")
@login_required
def create_post():
    # Implementation
```

### 4. Performance Optimization

- Implement caching for frequently accessed data
- Optimize database queries and add indexes
- Set up CDN for static assets
- Implement database connection pooling
- Add pagination for all list endpoints

```python
# Example caching with Flask-Caching
from flask_caching import Cache

cache = Cache(app, config={'CACHE_TYPE': 'redis', 'CACHE_REDIS_URL': 'redis://localhost:6379/1'})

@api.route('/posts', methods=['GET'])
@cache.cached(timeout=60, query_string=True)
def get_posts():
    # Implementation
```

### 5. CI/CD Pipeline

- Set up automated testing with GitHub Actions or similar
- Implement code quality checks (linting, formatting)
- Add automated security scanning
- Configure automated deployment to staging/production
- Implement database migration automation

```yaml
# Example GitHub Actions workflow
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.11
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      - name: Lint with flake8
        run: flake8 .
      - name: Test with pytest
        run: pytest

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to production
        run: ./deploy.sh
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

### 6. Documentation

- Add comprehensive API documentation with Swagger/OpenAPI
- Implement docstrings for all functions and classes
- Create developer documentation for local setup
- Add user documentation for the application

```python
# Example API documentation with Flask-RESTX
from flask_restx import Api, Resource, fields

api = Api(app, version='1.0', title='Poetry Hub API', description='A poetry sharing platform API')

post_model = api.model('Post', {
    'id': fields.Integer(readonly=True, description='Post unique identifier'),
    'title': fields.String(required=True, description='Post title'),
    'content': fields.String(required=True, description='Post content'),
    'author_id': fields.Integer(required=True, description='Author ID')
})

@api.route('/posts')
class PostList(Resource):
    @api.doc('list_posts')
    @api.marshal_list_with(post_model)
    def get(self):
        '''List all posts'''
        return Post.query.all()
    
    @api.doc('create_post')
    @api.expect(post_model)
    @api.marshal_with(post_model, code=201)
    def post(self):
        '''Create a new post'''
        return create_post(api.payload), 201
```

### 7. Infrastructure as Code

- Implement infrastructure as code with Terraform or similar
- Create environment-specific configurations
- Set up automated backups for database
- Implement disaster recovery procedures

```hcl
# Example Terraform configuration
provider "aws" {
  region = "us-west-2"
}

resource "aws_db_instance" "poetry_hub_db" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  name                 = "poetry_hub"
  username             = "admin"
  password             = var.db_password
  parameter_group_name = "default.mysql8.0"
  backup_retention_period = 7
  skip_final_snapshot  = true
}

resource "aws_elasticache_cluster" "poetry_hub_redis" {
  cluster_id           = "poetry-hub-redis"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis6.x"
  engine_version       = "6.x"
  port                 = 6379
}
```

### 8. Scalability Improvements

- Implement horizontal scaling for web servers
- Set up load balancing
- Optimize database for read/write splitting
- Implement message queue for asynchronous processing
- Add caching layer for frequently accessed data

```yaml
# Example Docker Compose scaling configuration
version: '3.8'

services:
  web:
    build: .
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    environment:
      - DATABASE_URL=mysql://user:password@mysql:3306/poetry_hub
    depends_on:
      - mysql
      - redis

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    depends_on:
      - web
```

### 9. Feature Enhancements

- Implement social sharing functionality
- Add user commenting and interaction features
- Implement content recommendation system
- Add analytics for user engagement tracking
- Implement full-text search for poetry content

### 10. Accessibility and Internationalization

- Ensure WCAG 2.1 compliance for accessibility
- Implement proper internationalization (i18n)
- Add right-to-left (RTL) language support
- Implement content translation workflows

```python
# Example Flask-Babel integration for i18n
from flask_babel import Babel, gettext

babel = Babel(app)

@babel.localeselector
def get_locale():
    return request.accept_languages.best_match(['en', 'hi', 'fr'])

# In templates
# {{ _('Welcome to Poetry Hub') }}
```

By implementing these recommendations, Poetry Hub will be well-positioned for production deployment with improved reliability, security, performance, and maintainability.

## Known Issues

- `requirements.txt` contains Git conflict markers and duplicates (e.g., `Werkzeug`, `python-dotenv`). Clean before deployment.
- API auth is session-based only; no token/JWT support.
- `migrations/` is referenced but not initialized. Add a Flask CLI and run `flask db init`, or manage with Alembic directly.
- Some optional pages assume templates exist (e.g., community/events/forums). Verify before enabling in production.