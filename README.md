# Website Change Monitor

[![Python Version](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12-blue?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Tests](https://img.shields.io/badge/tests-pytest-brightgreen?logo=pytest&logoColor=white)](https://docs.pytest.org/)
[![Framework: FastAPI](https://img.shields.io/badge/API-FastAPI-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)

**Website Change Monitor** is an automated, developer-focused monitoring engine designed to track textual and structural mutations across target web pages over time. It regularly scrapes target pages, eliminates volatile DOM elements (such as timestamps, rotating banners, and session tokens), computes semantic and raw diffs against baseline snapshots, and presents actionable audit records via a terminal user interface (TUI) or REST API.

---

## Problem Statement & Target Audience

### The Problem
Traditional uptime monitoring tools (e.g., Pingdom, UptimeRobot) only verify HTTP response codes and network availability. They fail when a page returns an `HTTP 200 OK` status while the underlying content has quietly broken, prices have shifted, or regulatory statements have changed. Conversely, naive diff tools trigger continuous false positives caused by dynamic scripts, relative timestamps, and volatile ad scripts.

### Target Audience
* **Compliance & Legal Teams**: Track unexpected modifications in Terms of Service, Privacy Policies, or regulatory disclosures across vendor ecosystems.
* **DevOps & Platform Engineers**: Monitor third-party API documentation and external status pages that lack programmatic RSS or webhook notifications.
* **Competitive Intelligence & Procurement**: Audit vendor pricing structures, feature matrix adjustments, and enterprise service catalogs.

---

## Key Features

* **Targeted DOM Extraction**: Utilize CSS selectors and XPath expressions to isolate target elements, ignoring volatile DOM noise (e.g., ads, timestamps, dynamic CSRF tokens).
* **Scheduled Ingestion**: Native integration with `APScheduler` supporting both standard cron syntax (`cron: "0 */2 * * *"`) and fixed polling intervals (`interval: 300s`).
* **Noise Stripping & Normalization**: Strips script tags, style sheets, and user-defined dynamic CSS classes before computing differences.
* **Structural & Semantic Diffing**: Computes exact text diffs, unified patch files, and structural DOM tree modifications with user-definable similarity thresholds.
* **Schema Validation**: Built with Pydantic v2 to enforce strict schema validation for targets, jobs, snapshot payloads, and diff reports.
* **Rich Terminal Interface (TUI)**: Terminal dashboard built with `Rich` to inspect active monitor statuses, historical execution logs, and colorized diff tables.
* **Programmatic REST API**: Built-in FastAPI server for remote job registration, baseline updates, and on-demand diff executions.

---

## Architecture & System Design

```
                     ┌──────────────────────────────────────────┐
                     │          CLI Entrypoint (main.py)         │
                     └─────┬──────────────────────────────┬─────┘
                           │                              │
                           ▼                              ▼
                 ┌───────────────────┐          ┌───────────────────┐
                 │     Rich TUI      │          │    FastAPI App    │
                 │     (src/tui)     │          │   (src/api.py)    │
                 └─────────┬─────────┘          └─────────┬─────────┘
                           │                              │
                           └──────────────┬───────────────┘
                                          ▼
                             ┌──────────────────────────┐
                             │   Job Execution Engine   │
                             │    (src/scheduler.py)    │
                             └────────────┬─────────────┘
                                          │
                                          ▼
                             ┌──────────────────────────┐
                             │       DOM Processor      │
                             │       (src/core.py)      │
                             └──────┬─────────────┬─────┘
                                    │             │
                [Noise Cleaning & Extraction]   [Diff Engine (Baseline vs New)]
                                    │             │
                                    └──────┬──────┘
                                           ▼
                             ┌──────────────────────────┐
                             │      Data Contract       │
                             │     (src/models.py)      │
                             └────────────┬─────────────┘
                                          ▼
                             ┌──────────────────────────┐
                             │     SQLite Storage       │
                             │     (src/storage.py)     │
                             └──────────────────────────┘
```

### Component Breakdown

* **`src/models.py`**: Defines Pydantic models for monitoring targets, extraction rules, baseline snapshot records, execution outcomes, and diff schemas.
* **`src/core.py`**: Handles page acquisition, HTML parsing via BeautifulSoup4, volatile noise stripping (cleaning unneeded tags), selector extraction, and text/structural diff computations.
* **`src/scheduler.py`**: Manages APScheduler instances, coordinating cron-based and interval-based job queues.
* **`src/storage.py`**: SQLite abstraction layer handling persistence for targets, baseline history, and diff audit logs.
* **`src/tui.py`**: Rich-powered console rendering tables, progress trackers, and visual git-style unified diffs.
* **`src/api.py`**: FastAPI routing layer exposing RESTful endpoints to query monitor states and register new targets.
* **`src/main.py`**: Top-level CLI orchestration interface supporting worker daemons, interactive dashboard modes, and standalone runs.

---

## Technology Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Language** | Python 3.10+ | Core language runtime |
| **DOM Engine** | BeautifulSoup4 / lxml | High-performance HTML parsing and tree navigation |
| **HTTP Transport** | Requests | Content retrieval with custom headers and timeout management |
| **Scheduler** | APScheduler | Cron-like and interval job scheduling engine |
| **Validation** | Pydantic v2 | Strict data modeling, schema definition, and configuration casting |
| **Presentation** | Rich | Colorized terminal dashboards, spinners, and diff output |
| **Interface** | FastAPI & Uvicorn | Management REST endpoints for automated integrations |
| **Testing** | Pytest | Unit and integration test runner |

---

## Installation & Virtual Environment Setup

### Prerequisites
* Python 3.10 or higher installed on your host system
* `git` CLI utility

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/website-change-monitor.git
cd website-change-monitor
```

### 2. Create and Activate a Virtual Environment
```bash
# On Linux/macOS
python3 -m venv venv
source venv/bin/activate

# On Windows
python -m venv venv
venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## Environment Variables Configuration

Copy the example environment file and adjust the values according to your operational environment:

```bash
cp .env.example .env
```

### Configuration Options

| Variable | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `MONITOR_DB_PATH` | String | `monitor.db` | Path to the local SQLite database file |
| `API_HOST` | String | `127.0.0.1` | Network interface for the FastAPI management server |
| `API_PORT` | Integer | `8000` | Port for the FastAPI management server |
| `DEFAULT_POLL_INTERVAL`| Integer | `3600` | Default fallback polling interval in seconds |
| `REQUEST_TIMEOUT` | Integer | `15` | Default HTTP request timeout (in seconds) |
| `USER_AGENT` | String | `WebsiteChangeMonitor/1.0` | Custom User-Agent header sent during scrapers |
| `SIMILARITY_THRESHOLD` | Float | `0.85` | Ratio below which a page change is marked as significant |

---

## Usage

Website Change Monitor can be launched in multiple operational modes: as a terminal UI dashboard, a background daemon worker, or a headless REST API server.

### 1. Running the CLI Interface

#### Launch the Interactive TUI Dashboard
Displays live scheduler statuses, target health, and recent diff triggers directly in the terminal:
```bash
python -m src.main --mode tui
```

#### Launch as a Headless Background Daemon
Executes the scheduled worker loop without rendering the TUI dashboard:
```bash
python -m src.main --mode daemon
```

#### Run an Immediate Ad-Hoc Check
Trigger an immediate extraction and diff comparison for a specific target URL:
```bash
python -m src.main --mode check --url "https://news.ycombinator.com" --selector ".titleline"
```

---

### 2. Running the REST API Server

Launch the FastAPI application:
```bash
uvicorn src.api:app --host 127.0.0.1 --port 8000 --reload
```
Interactive OpenAPI documentation will be accessible at: `http://127.0.0.1:8000/docs`.

#### API Endpoints Overview

* `GET /health` - Service health status check.
* `GET /jobs` - List all configured monitoring jobs.
* `POST /jobs` - Register a new monitoring target.
* `GET /jobs/{id}` - Fetch configuration, baseline state, and metadata for a specific job.
* `POST /jobs/{id}/run` - Manually trigger an immediate scrape and diff cycle.
* `GET /jobs/{id}/diffs` - Retrieve historical diff records and similarity scores for a job.

#### Register a Monitor Job Example
```bash
curl -X POST "http://127.0.0.1:8000/jobs" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/terms",
    "name": "Terms of Service Monitor",
    "selector": "main.content-area",
    "cron_expression": "0 */4 * * *",
    "ignored_selectors": [".cookie-banner", "footer", ".timestamp"],
    "similarity_threshold": 0.95
  }'
```

#### Execute an On-Demand Run
```bash
curl -X POST "http://127.0.0.1:8000/jobs/1/run"
```

---

## Automated Testing Instructions

The test suite validates selector parsing, volatile element sanitization, diff computation thresholds, and API endpoint schemas.

Run tests using `pytest`:

```bash
# Run all tests
pytest

# Run tests with standard stdout enabled
pytest -s

# Run tests with code coverage report
pytest --cov=src --cov-report=term-missing

# Run a specific test module
pytest tests/test_core.py
pytest tests/test_api.py
```

---

## Project Structure

```
website-change-monitor/
├── .env.example              # Sample environment configuration file
├── .gitignore                # Git ignore patterns for Python/vcs artifacts
├── requirements.txt          # Production and testing dependencies
├── src/
│   ├── __init__.py           # Package marker
│   ├── api.py                # FastAPI endpoints and route handlers
│   ├── core.py               # Extraction, cleaning, and diffing algorithms
│   ├── main.py               # CLI entrypoint (TUI, daemon, single-run)
│   ├── models.py             # Pydantic schemas (Target, Snapshot, DiffResult)
│   ├── scheduler.py          # APScheduler management interface
│   ├── storage.py            # SQLite database layer for persistent baselines
│   └── tui.py                # Rich terminal UI components and diff visualizer
└── tests/
    ├── __init__.py           # Test suite marker
    ├── test_api.py           # API integration tests
    └── test_core.py          # Core parsing, normalization, and diff engine tests
```

---

## Roadmap & Future Enhancements

- [ ] **Headless Browser Execution**: Integrate Playwright or Selenium to handle client-side Single Page Applications (SPAs) requiring JavaScript hydration.
- [ ] **Webhook Dispatchers**: Built-in notification providers for Slack, Discord, Microsoft Teams, and custom HMAC-signed webhooks.
- [ ] **Visual Screenshot Regression**: Structural pixel-based diffing using headless browser canvas captures to spot CSS/styling regressions.
- [ ] **Authentication Support**: Session handling for cookie-based auth, basic HTTP auth, and bearer tokens on protected endpoints.
- [ ] **Regex Trigger Filters**: Alert only when specific regular expressions match added or deleted content (e.g., pricing figures or compliance keywords).

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.