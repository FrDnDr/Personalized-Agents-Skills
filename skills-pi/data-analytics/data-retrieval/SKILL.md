---
name: data-retrieval
description: Querying data from databases, APIs, flat files such as CSV and Excel, and basic web scraping patterns.
---
# data-retrieval.md

## Purpose

Define conventions for acquiring data from diverse sources — covering database queries,
API consumption, flat file ingestion (CSV/Excel/JSON), web scraping basics,
and modular connector patterns for reusable data acquisition.

---

## Conventions

### Connector Architecture

Standardize data retrieval through a modular connector pattern:

```
/data
  /connectors
    base.py              # Abstract connector interface
    database.py          # SQL database connector (PostgreSQL, SQLite)
    api.py               # REST API connector with auth and pagination
    file.py              # CSV, Excel, JSON file readers
    scraper.py           # Web scraping connector
  /config
    sources.yaml         # Source registry with connection details
```

### Connector Interface

```python
# connectors/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any

@dataclass
class RetrievalResult:
    data: Any                   # DataFrame, list, or dict
    row_count: int
    source: str
    retrieved_at: datetime
    metadata: dict = None       # Source-specific metadata

class BaseConnector(ABC):
    @abstractmethod
    def connect(self) -> None: ...

    @abstractmethod
    def fetch(self, **kwargs) -> RetrievalResult: ...

    @abstractmethod
    def close(self) -> None: ...

    def __enter__(self): self.connect(); return self
    def __exit__(self, *args): self.close()
```

### Database Retrieval

```python
# connectors/database.py
import pandas as pd
from sqlalchemy import create_engine, text

class DatabaseConnector(BaseConnector):
    def __init__(self, connection_string: str):
        self.engine = create_engine(connection_string)

    def fetch(self, query: str, params: dict = None) -> RetrievalResult:
        df = pd.read_sql(text(query), self.engine, params=params)
        return RetrievalResult(
            data=df, row_count=len(df),
            source=str(self.engine.url), retrieved_at=datetime.utcnow(),
        )
```

**Rules:**
- Always use parameterized queries — never string interpolation for SQL
- Always specify column lists — never `SELECT *` in production
- Always limit result sets for exploratory queries (`LIMIT 1000`)
- Use `read_sql` with `chunksize` for large result sets (>1M rows)

### API Retrieval

```python
# connectors/api.py
import httpx

class APIConnector(BaseConnector):
    def __init__(self, base_url: str, auth_token: str):
        self.client = httpx.Client(
            base_url=base_url,
            headers={"Authorization": f"Bearer {auth_token}"},
            timeout=30.0,
        )

    def fetch(self, endpoint: str, params: dict = None) -> RetrievalResult:
        all_data = []
        page = 1
        while True:
            response = self.client.get(endpoint, params={**(params or {}), "page": page})
            response.raise_for_status()
            batch = response.json()["data"]
            if not batch:
                break
            all_data.extend(batch)
            page += 1

        return RetrievalResult(
            data=pd.DataFrame(all_data), row_count=len(all_data),
            source=f"{self.client.base_url}{endpoint}", retrieved_at=datetime.utcnow(),
        )
```

**Rules:**
- Always implement pagination — never assume single-page responses
- Always set request timeouts — never leave connections open indefinitely
- Always handle rate limits with retry + backoff
- Always store API keys in environment variables — never hardcode

### File Retrieval

```python
# connectors/file.py
import pandas as pd
from pathlib import Path

class FileConnector(BaseConnector):
    READERS = {
        ".csv": lambda p, **kw: pd.read_csv(p, **kw),
        ".xlsx": lambda p, **kw: pd.read_excel(p, engine="openpyxl", **kw),
        ".xls": lambda p, **kw: pd.read_excel(p, engine="xlrd", **kw),
        ".json": lambda p, **kw: pd.read_json(p, **kw),
        ".jsonl": lambda p, **kw: pd.read_json(p, lines=True, **kw),
        ".parquet": lambda p, **kw: pd.read_parquet(p, **kw),
    }

    def fetch(self, file_path: str, **read_kwargs) -> RetrievalResult:
        path = Path(file_path)
        ext = path.suffix.lower()
        reader = self.READERS.get(ext)
        if not reader:
            raise ValueError(f"Unsupported file format: {ext}")

        df = reader(path, **read_kwargs)
        return RetrievalResult(
            data=df, row_count=len(df),
            source=str(path), retrieved_at=datetime.utcnow(),
            metadata={"format": ext, "size_bytes": path.stat().st_size},
        )
```

**Rules:**
- Always specify `encoding="utf-8"` for CSV unless source encoding is known
- Always use `dtype` parameter to enforce column types on read
- For Excel: always specify `sheet_name` explicitly — never rely on default
- For large files (>100MB): use `chunksize` for chunked reading

### Source Registry

Centralize all source configurations:

```yaml
# config/sources.yaml
sources:
  shopify_db:
    type: database
    connection: ${SHOPIFY_DB_URL}
    description: "Shopify PostgreSQL production database"
  stripe_api:
    type: api
    base_url: "https://api.stripe.com/v1"
    auth: ${STRIPE_API_KEY}
    rate_limit: 100/sec
  monthly_report:
    type: file
    path: "./data/reports/monthly_sales.xlsx"
    format: xlsx
    sheet: "Sales Data"
```

---

## Anti-Patterns

- Never use `SELECT *` in production queries — always specify columns
- Never hardcode connection strings or API keys — use environment variables
- Never fetch unlimited API results — always paginate
- Never skip request timeouts — hanging connections waste resources
- Never read large files entirely into memory — use chunked reading
- Never assume file encoding — always specify or detect
- Never mix retrieval logic with transformation — retrieve first, transform separately

---

## Cross-References

- **data-analytics/data-cleaning** → clean data immediately after retrieval
- **data-engineering/etl-pipeline** → connectors used in the extract stage
- **core/security** → credential management for database and API connections
- **database/sql** → query patterns for database retrieval

---

## Ready-to-Use Prompt

```
Task: Retrieve data from [source type] for [analysis purpose]
Skill: data-analytics/data-retrieval, data-analytics/data-cleaning

REQUIREMENTS:
- Source: [database / API / CSV / Excel / JSON]
- Connection: [describe source and credentials]
- Scope: [what data, what date range, what filters]

CONSTRAINTS:
- Use modular connector pattern
- Parameterized queries for databases
- Pagination for API sources
- Chunked reading for large files
- Environment variables for credentials
- Return RetrievalResult with metadata

DONE WHEN:
- Data retrieved successfully into DataFrame
- Source metadata captured (row count, timestamp)
- No hardcoded credentials
- Code review skill applied before commit
```

