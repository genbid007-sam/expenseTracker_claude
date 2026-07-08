This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

  Overview

  Spendly is a lightwaeight simple web application designed as an Expense Tracker. It uses Flask for the backend and Jinja templating for the frontend structure and SQLite. The architecture follows a standard Model-View-Controller pattern,
  though simplified:

  - Model/Data Layer: Handled by files within database/ (e.g., db.py). This layer is responsible for connecting to SQLite, initializing tables, and handling all data interactions.
  - Controller/Business Logic: The core logic resides in app.py. It maps HTTP requests to Python functions, executes the business rules, and renders the appropriate templates.
  - View/Presentation Layer: Templates are stored in the templates/ directory (e.g., landing.html, login.html). These files define the structure of the user interface using Jinja2 templating syntax.


# System prompt -- reinforces coding agent behavior and explicit

SYSTEM 
    """You are a senior software engineer operating as a coding agent.
    When working with code:
    - Read files before editing them. Never assume file contents.
    - Make one focused change at a time and verify it before proceeding.
    - When a tool call fails, examine the error carefully before retrying.
    Do not retry with identical parameters. Diagnose first.
    - Prefer surgical edits over full file rewrites.
    - Run tests after each meaningful change, not after a batch of changes.
    - If you are uncertain about the codebase structure, read more files
    rather than guessing.
    Be precise and methodical. Avoid explaining what you are about to do
    when you could simply do it."""

  Core Architecture & Structure

    spendly/
        ├── app.py              # All routes — single file, no blueprints
        ├── database/
        │   └── db.py           # SQLite helpers: get_db(), init_db(), seed_db()
        ├── templates/
        │   ├── base.html       # Shared layout — all templates must extend this
        │   └── *.html          # One template per page
        ├── static/
        │   ├── css/
        │   │   ├── style.css       # Global styles
        │   │   └── landing.css     # Landing-page-only styles
        │   └── js/
        │       └── main.js         # Vanilla JS only
        └── requirements.txt

  Where things belong:

    New routes → app.py only, no blueprints
    DB logic → database/db.py only, never inline in routes
    New pages → new .html file extending base.html
    Page-specific styles → new .css file, not inline <style> tags

  File Structure

  - app.py: The entry point for the application. Contains all route definitions and acts as the primary request handler (Controller).
    - It defines routes for core functions like /login, /register, /landing, and placeholder routes for expense management (/expenses/...).
    - Crucially, it handles the rendering of legal pages: /terms and /privacy.
  - database/: This package is intended to house all database logic.
    - db.py: Contains stubs for database utility functions (get_db, init_db, seed_db), managing the SQLite connection and schema creation (Model).

  Data Flow

  1. A request hits a route in app.py (e.g., /login).
  2. The corresponding function calls database helpers from database/ to retrieve or modify data using a database session established by get_db().
  3. The business logic uses this retrieved data, processes it, and then calls render_template() with the resulting data context.
  4. The relevant HTML file in templates/ uses Jinja2 to render the final view for the user.

 Common Development Tasks & Commands

  Since development is actively progressing, these commands assume standard Flask setup:

  Code style
    Python: PEP 8, snake_case for all variables and functions
    Templates: Jinja2 with url_for() for every internal link — never hardcode URLs
    Route functions: one responsibility only — fetch data, render template, done
    DB queries: always use parameterized queries (? placeholders) — never f-strings in SQL
    Error handling: use abort() for HTTP errors, not bare return "error string"

Tech constraints
    Flask only — no FastAPI, no Django, no other web frameworks
    SQLite only — no PostgreSQL, no SQLAlchemy ORM, no external DB
    Vanilla JS only — no React, no jQuery, no npm packages
    No new pip packages — work within requirements.txt as-is unless explicitly told otherwise
    Python 3.10+ assumed — f-strings and match statements are fine

Subagent Policy
    Always use a builtin explore subagent for codebase exploration before implementing any new feature
    Always use a subagent to verify test results after any implementation
    When asked to plan, delegate codebase research to a subagent before presenting the plan
    always use a builtin plan subagent in plan mode
Commands
# Setup
    python -m venv venv
    venv\Scripts\activate
    pip install -r requirements.txt

# Run dev server (port 5001)
    python app.py

# Run all tests
    pytest

# Run a specific test file
    pytest tests/test_foo.py

# Run a specific test by name
    pytest -k "test_name"

# Run tests with output visible
    pytest -s
    Implemented vs stub routes

Route	Status
    GET /	Implemented — renders landing.html
    GET /register	Implemented — renders register.html
    GET /login	Implemented — renders login.html
    GET /logout	Stub — Step 3
    GET /profile	Stub — Step 4
    GET /expenses/add	Stub — Step 7
    GET /expenses/<id>/edit	Stub — Step 8
    GET /expenses/<id>/delete	Stub — Step 9
    Do not implement a stub route unless the active task explicitly targets that step.

Warnings and things to avoid
    Never use raw string returns for stub routes once a step is implemented — always render a template
    Never hardcode URLs in templates — always use url_for()
    Never put DB logic in route functions — it belongs in database/db.py
    Never install new packages mid-feature without flagging it — keep requirements.txt in sync
    Never use JS frameworks — the frontend is intentionally vanilla
    database/db.py is currently empty — do not assume helpers exist until the step that implements them
    FK enforcement is manual — SQLite foreign keys are off by default; get_db() must run PRAGMA foreign_keys = ON on every connection
    The app runs on port 5001, not the Flask default 5000 — don't change this

Development Notes

  - Linting/Formatting: Use a dedicated linter (e.g., flake8) targeting app.py and files in database/.
  - Database Initialization: After making changes to the database schema, run the intended initialization procedure (e.g., calling init_db() within a test script or temporary setup flow).

Rules & Best Practices

  - Feature Scoping: The application uses placeholders for major features (e.g., Step 7: Add Expense Route; Step 8: Edit Expense); new feature development must fill these gaps logically and incrementally, starting
  with data model changes in database/ before touching routes in app.py.
  - Separation of Concerns: All database access logic MUST be encapsulated within the database/ package. Never execute raw SQL or state changes directly from app.py.