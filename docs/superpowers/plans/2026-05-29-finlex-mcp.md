# finlex-mcp Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a FastMCP server that wraps the Finlex API, exposing `search_statutes` and `get_statute` tools so Claude plugins can retrieve live Finnish statute text instead of relying on training data.

**Architecture:** Python FastMCP server using the `mcp` package's HTTP transport. Two tools: `search_statutes(query, language)` for discovery and `get_statute(law_code, section)` for precision retrieval. Thin client layer over the Finlex data API. Deployed to `https://finlex-mcp.velvoite.eu/mcp` by Velvoite.

**Tech Stack:** Python 3.11+, `mcp[server]` (Anthropic MCP SDK), `httpx` (async HTTP), `fastapi`, `uvicorn`, `pytest`, `pytest-asyncio`, `pytest-httpx`, `pydantic`.

---

## Files created in this plan

```
finlex-mcp/
  src/finlex_mcp/
    __init__.py
    server.py          # FastMCP server + tool definitions
    client.py          # httpx client wrapping Finlex API
    models.py          # Pydantic response models
  tests/
    conftest.py
    test_client.py
    test_server.py
  pyproject.toml
  README.md
  Dockerfile
  .gitignore
```

New GitHub repo: `github.com/Velvoite/finlex-mcp`

---

## Task 1: Project scaffold

**Files:**
- Create: `finlex-mcp/pyproject.toml`
- Create: `finlex-mcp/src/finlex_mcp/__init__.py`
- Create: `finlex-mcp/.gitignore`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p finlex-mcp/src/finlex_mcp
mkdir -p finlex-mcp/tests
```

- [ ] **Step 2: Write `finlex-mcp/pyproject.toml`**

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "finlex-mcp"
version = "0.1.0"
description = "MCP server exposing Finnish statute data from Finlex"
requires-python = ">=3.11"
dependencies = [
    "mcp[server]>=1.0.0",
    "httpx>=0.27.0",
    "fastapi>=0.115.0",
    "uvicorn>=0.32.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.24.0",
    "pytest-httpx>=0.32.0",
    "pytest-cov>=5.0.0",
]

[project.scripts]
finlex-mcp = "finlex_mcp.server:main"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]

[tool.hatch.build.targets.wheel]
packages = ["src/finlex_mcp"]
```

- [ ] **Step 3: Write `finlex-mcp/src/finlex_mcp/__init__.py`**

```python
"""finlex-mcp — MCP server for Finnish statute data."""

__version__ = "0.1.0"
```

- [ ] **Step 4: Write `finlex-mcp/.gitignore`**

```
__pycache__/
*.pyc
*.egg-info/
dist/
.venv/
.env
*.log
.pytest_cache/
htmlcov/
```

- [ ] **Step 5: Install dependencies**

```bash
cd finlex-mcp && python -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]"
```

Expected: packages install without error.

- [ ] **Step 6: Commit**

```bash
git -C finlex-mcp init && git -C finlex-mcp add -A && git -C finlex-mcp commit -m "chore: project scaffold"
```

---

## Task 2: Finlex API client

**Files:**
- Create: `finlex-mcp/src/finlex_mcp/models.py`
- Create: `finlex-mcp/src/finlex_mcp/client.py`
- Create: `finlex-mcp/tests/conftest.py`
- Create: `finlex-mcp/tests/test_client.py`

**Finlex API notes:** The Finlex data API uses the ELI (European Legislation Identifier) scheme at `https://data.finlex.fi`. Before implementing, verify current endpoints at https://data.finlex.fi/en/. The law code mapping (e.g. `TSL` → `2001/55`) is maintained in `client.py`. If the API changes, only `client.py` needs updating.

- [ ] **Step 1: Write `finlex-mcp/src/finlex_mcp/models.py`**

```python
from pydantic import BaseModel


class StatuteSection(BaseModel):
    law_name: str
    law_code: str
    section: str | None
    text: str
    url: str
    last_amended: str | None = None


class SearchResult(BaseModel):
    results: list[StatuteSection]
    total: int
    query: str
```

- [ ] **Step 2: Write `finlex-mcp/tests/conftest.py`**

```python
import pytest
import httpx
from finlex_mcp.client import FinlexClient

@pytest.fixture
def client():
    return FinlexClient()

@pytest.fixture
def tsl_response():
    """Minimal mock response for TSL §3:5 (non-compete)."""
    return {
        "title": "Työsopimuslaki",
        "statute_id": "2001/55",
        "section": "3:5",
        "text": "Työnantaja ja työntekijä voivat sopia kilpailukiellosta...",
        "url": "https://www.finlex.fi/fi/laki/ajantasa/2001/20010055#L3P5",
        "last_amended": "2022-08-01",
    }
```

- [ ] **Step 3: Write failing tests in `finlex-mcp/tests/test_client.py`**

```python
import pytest
from pytest_httpx import HTTPXMock
from finlex_mcp.client import FinlexClient
from finlex_mcp.models import StatuteSection, SearchResult


LAW_CODES = {
    "TSL": ("2001", "55"),
    "OYL": ("2006", "624"),
    "TVL": ("1992", "1535"),
    "YTL": ("2021", "1333"),
    "LSL": ("2018", "595"),
    "VLL": ("2005", "162"),
}


async def test_get_statute_returns_section(httpx_mock: HTTPXMock):
    httpx_mock.add_response(
        url="https://data.finlex.fi/eli/sd/2001/55/section/3/5",
        json={
            "title": "Työsopimuslaki",
            "statute_id": "2001/55",
            "section": "3:5",
            "text": "Työnantaja ja työntekijä voivat sopia kilpailukiellosta...",
            "url": "https://finlex.fi/fi/laki/ajantasa/2001/20010055#L3P5",
            "last_amended": "2022-08-01",
        },
    )
    client = FinlexClient()
    result = await client.get_statute("TSL", "3:5")
    assert result.law_code == "TSL"
    assert result.section == "3:5"
    assert "kilpailukiellosta" in result.text
    assert result.url.startswith("https://finlex.fi")


async def test_get_statute_full_act(httpx_mock: HTTPXMock):
    httpx_mock.add_response(
        url="https://data.finlex.fi/eli/sd/2001/55",
        json={
            "title": "Työsopimuslaki",
            "statute_id": "2001/55",
            "section": None,
            "text": "Luku 1 - Yleiset säännökset...",
            "url": "https://finlex.fi/fi/laki/ajantasa/2001/20010055",
            "last_amended": "2022-08-01",
        },
    )
    client = FinlexClient()
    result = await client.get_statute("TSL")
    assert result.law_code == "TSL"
    assert result.section is None


async def test_unknown_law_code_raises():
    client = FinlexClient()
    with pytest.raises(ValueError, match="Unknown law code"):
        await client.get_statute("UNKNOWN")


async def test_search_statutes(httpx_mock: HTTPXMock):
    httpx_mock.add_response(
        url__contains="data.finlex.fi/search",
        json={
            "results": [
                {
                    "title": "Työsopimuslaki",
                    "statute_id": "2001/55",
                    "section": "3:5",
                    "text": "kilpailukielto...",
                    "url": "https://finlex.fi/fi/laki/ajantasa/2001/20010055#L3P5",
                    "last_amended": "2022-08-01",
                }
            ],
            "total": 1,
        },
    )
    client = FinlexClient()
    result = await client.search_statutes("kilpailukielto", language="fi")
    assert result.total == 1
    assert result.results[0].law_code == "TSL"
```

- [ ] **Step 4: Run tests — verify they fail**

```bash
cd finlex-mcp && source .venv/bin/activate && pytest tests/test_client.py -v
```

Expected: `ImportError` or `ModuleNotFoundError` — `client.py` does not exist yet.

- [ ] **Step 5: Write `finlex-mcp/src/finlex_mcp/client.py`**

```python
import httpx
from .models import StatuteSection, SearchResult

BASE_URL = "https://data.finlex.fi"

# Map short law codes to (year, statute_number) for ELI URLs
LAW_CODES: dict[str, tuple[str, str]] = {
    "TSL": ("2001", "55"),     # Employment Contracts Act
    "OYL": ("2006", "624"),    # Companies Act
    "YTL": ("2021", "1333"),   # Co-operation Act (YT-laki)
    "TVL": ("1992", "1535"),   # Income Tax Act (§66 ESOP)
    "LSL": ("2018", "595"),    # Business Secrets Act
    "VLL": ("2005", "162"),    # Annual Holidays Act
    "TAL": ("2019", "872"),    # Working Hours Act
    "HETIL": ("2018", "1050"), # Data Protection Act
    "KSL": ("1978", "38"),     # Consumer Protection Act
    "HankL": ("2016", "1397"), # Procurement Act
}


def _eli_url(law_code: str, section: str | None = None) -> str:
    if law_code not in LAW_CODES:
        raise ValueError(f"Unknown law code: {law_code}. Known codes: {sorted(LAW_CODES)}")
    year, number = LAW_CODES[law_code]
    base = f"{BASE_URL}/eli/sd/{year}/{number}"
    if section:
        # Convert "3:5" → "section/3/5"
        parts = section.replace(":", "/").replace("§", "")
        return f"{base}/section/{parts}"
    return base


def _parse_response(law_code: str, data: dict) -> StatuteSection:
    return StatuteSection(
        law_name=data["title"],
        law_code=law_code,
        section=data.get("section"),
        text=data["text"],
        url=data["url"],
        last_amended=data.get("last_amended"),
    )


class FinlexClient:
    async def get_statute(self, law_code: str, section: str | None = None) -> StatuteSection:
        url = _eli_url(law_code, section)
        async with httpx.AsyncClient(timeout=10.0) as http:
            response = await http.get(url, headers={"Accept": "application/json"})
            response.raise_for_status()
            return _parse_response(law_code, response.json())

    async def search_statutes(self, query: str, language: str = "en") -> SearchResult:
        params = {"q": query, "lang": language, "limit": 10}
        async with httpx.AsyncClient(timeout=10.0) as http:
            response = await http.get(
                f"{BASE_URL}/search",
                params=params,
                headers={"Accept": "application/json"},
            )
            response.raise_for_status()
            data = response.json()
            results = [
                _parse_response(
                    r.get("statute_id", "").split("/")[-1],
                    r,
                )
                for r in data.get("results", [])
            ]
            return SearchResult(results=results, total=data.get("total", len(results)), query=query)
```

- [ ] **Step 6: Run tests — verify they pass**

```bash
pytest tests/test_client.py -v
```

Expected: 4 tests pass.

- [ ] **Step 7: Commit**

```bash
git add src/finlex_mcp/models.py src/finlex_mcp/client.py tests/ && git commit -m "feat: Finlex API client with search and get_statute"
```

---

## Task 3: FastMCP server

**Files:**
- Create: `finlex-mcp/src/finlex_mcp/server.py`
- Create: `finlex-mcp/tests/test_server.py`

- [ ] **Step 1: Write failing tests in `finlex-mcp/tests/test_server.py`**

```python
import pytest
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from finlex_mcp.server import create_server


async def test_server_lists_tools():
    server = create_server()
    tools = await server.list_tools()
    tool_names = [t.name for t in tools]
    assert "search_statutes" in tool_names
    assert "get_statute" in tool_names


async def test_get_statute_tool_schema():
    server = create_server()
    tools = {t.name: t for t in await server.list_tools()}
    schema = tools["get_statute"].inputSchema
    assert "law_code" in schema["properties"]
    assert "section" in schema["properties"]
    assert schema["required"] == ["law_code"]


async def test_search_statutes_tool_schema():
    server = create_server()
    tools = {t.name: t for t in await server.list_tools()}
    schema = tools["search_statutes"].inputSchema
    assert "query" in schema["properties"]
    assert schema["required"] == ["query"]
```

- [ ] **Step 2: Run tests — verify they fail**

```bash
pytest tests/test_server.py -v
```

Expected: `ImportError` — `server.py` doesn't exist yet.

- [ ] **Step 3: Write `finlex-mcp/src/finlex_mcp/server.py`**

```python
from mcp.server.fastmcp import FastMCP
from .client import FinlexClient

_client = FinlexClient()


def create_server() -> FastMCP:
    mcp = FastMCP(
        "finlex",
        instructions=(
            "Provides live Finnish statute data from Finlex (finlex.fi). "
            "Use search_statutes to find relevant sections by keyword, "
            "then get_statute to retrieve the full authoritative text of a specific article. "
            "Law codes: TSL=Employment Contracts Act, OYL=Companies Act, "
            "TVL=Income Tax Act (§66 for ESOP), YTL=Co-operation Act, "
            "LSL=Business Secrets Act, VLL=Annual Holidays Act, TAL=Working Hours Act."
        ),
    )

    @mcp.tool()
    async def search_statutes(query: str, language: str = "en") -> dict:
        """
        Search Finnish statutes by keyword. Returns matching sections with text excerpts.
        Use this for discovery when you don't know the exact law code and section.

        Args:
            query: Search terms (e.g. "non-compete compensation" or "kilpailukielto")
            language: "en" for English results, "fi" for Finnish. Default "en".

        Returns:
            Dict with 'results' list (each: law_name, law_code, section, text, url) and 'total' count.
        """
        result = await _client.search_statutes(query, language=language)
        return result.model_dump()

    @mcp.tool()
    async def get_statute(law_code: str, section: str | None = None) -> dict:
        """
        Get the full text of a Finnish statute by law code and optional section.
        Use this for precision retrieval once you know the exact article.

        Args:
            law_code: Short law code. Supported: TSL, OYL, TVL, YTL, LSL, VLL, TAL, HETIL, KSL.
            section: Optional section in "chapter:paragraph" format (e.g. "3:5", "6:3").
                     Omit to get the full act.

        Returns:
            Dict with law_name, law_code, section, text (full statute text), url, last_amended.
        """
        result = await _client.get_statute(law_code, section)
        return result.model_dump()

    return mcp


def main():
    import uvicorn
    import os
    server = create_server()
    app = server.get_asgi_app()
    uvicorn.run(app, host="0.0.0.0", port=int(os.environ.get("PORT", "8080")))


if __name__ == "__main__":
    main()
```

- [ ] **Step 4: Run tests — verify they pass**

```bash
pytest tests/test_server.py -v
```

Expected: 3 tests pass.

- [ ] **Step 5: Commit**

```bash
git add src/finlex_mcp/server.py tests/test_server.py && git commit -m "feat: FastMCP server with search_statutes and get_statute tools"
```

---

## Task 4: Dockerfile + README

**Files:**
- Create: `finlex-mcp/Dockerfile`
- Create: `finlex-mcp/README.md`

- [ ] **Step 1: Write `finlex-mcp/Dockerfile`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY pyproject.toml .
COPY src/ src/

RUN pip install --no-cache-dir -e .

EXPOSE 8080
CMD ["finlex-mcp"]
```

- [ ] **Step 2: Write `finlex-mcp/README.md`**

```markdown
# finlex-mcp

MCP server exposing live Finnish statute data from [Finlex](https://finlex.fi).

## Tools

- `search_statutes(query, language?)` — find statute sections by keyword
- `get_statute(law_code, section?)` — retrieve specific statute text

## Law codes

| Code | Act | Finnish name |
|---|---|---|
| TSL | Employment Contracts Act | Työsopimuslaki 55/2001 |
| OYL | Companies Act | Osakeyhtiölaki 624/2006 |
| TVL | Income Tax Act | Tuloverolaki 1535/1992 |
| YTL | Co-operation Act | YT-laki 1333/2021 |
| LSL | Business Secrets Act | Liikesalaisuuslaki 595/2018 |
| VLL | Annual Holidays Act | Vuosilomalaki 162/2005 |
| TAL | Working Hours Act | Työaikalaki 872/2019 |

## Run locally

```bash
pip install -e .
finlex-mcp
# Server at http://localhost:8080/mcp
```

## Deploy

Hosted by Velvoite at `https://finlex-mcp.velvoite.eu/mcp`. No auth required.

## License

Apache 2.0 — Copyright 2026 Silly Pilot Oy
```

- [ ] **Step 3: Run full test suite**

```bash
pytest -v --cov=finlex_mcp --cov-report=term-missing
```

Expected: all tests pass, coverage > 80%.

- [ ] **Step 4: Final commit and push**

```bash
git add Dockerfile README.md && git commit -m "chore: add Dockerfile and README"
gh repo create Velvoite/finlex-mcp --public --description "MCP server for Finnish statute data (Finlex)"
git remote add origin https://github.com/Velvoite/finlex-mcp.git
git push -u origin main
```

---

## Important note on Finlex API

The ELI URL structure (`data.finlex.fi/eli/sd/{year}/{number}`) is based on the Finlex linked data API. **Before implementing, verify current API docs at https://data.finlex.fi/en/.** If the actual endpoint differs:
1. Only `client.py` needs updating — `_eli_url()` and the `BASE_URL` constant
2. Mock responses in tests use the assumed structure — update `conftest.py` to match actual responses
3. The MCP tool interface (tool names, parameters, return shape) stays the same regardless
