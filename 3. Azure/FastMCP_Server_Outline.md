Local_FastMCP_Server_Standup.md
# Local_FastMCP_Server_Standup

# Local_FastMCP_Server_Standup_Index
Local_FastMCP_Server_Standup.md  
Local_FastMCP_Server_Standup  
Local_FastMCP_Server_Standup_Build_Decision  
Local_FastMCP_Server_Standup_Mental_Model  
Local_FastMCP_Server_Standup_Prerequisite_Install_uv  
Local_FastMCP_Server_Standup_Configuration_Checklist  
Local_FastMCP_Server_Standup_Skeleton  
Local_FastMCP_Server_Standup_Minimal_Server_File  
Local_FastMCP_Server_Standup_Search_Backend_Skeleton  
Local_FastMCP_Server_Standup_Lab_Metadata_Skeleton  
Local_FastMCP_Server_Standup_Static_Metadata_Example  
Local_FastMCP_Server_Standup_Client_Config_Skeleton  
Local_FastMCP_Server_Standup_Verification_Commands  
Local_FastMCP_Server_Standup_Rollback  
Local_FastMCP_Server_Standup_Failure_Checks  
Local_FastMCP_Server_Standup_Related_Labs  

# Local_FastMCP_Server_Standup_Build_Decision
| Decision | Operational Meaning |
|---|---|
| Local workstation role | The Surface is used for Python code, uv, local tests, Git, and MCP client testing |
| Local Docker role | Local Docker is optional, not required for this path |
| Container build role | GitHub Actions will build the container image on a GitHub-hosted runner |
| Container registry role | GitHub Actions will eventually push the image to Azure Container Registry |
| Azure runtime role | Azure Container Apps will run the image after the remote build and push succeed |
| Phase 1 boundary | Local FastMCP stdio, local HTTP, tools, tests, lint, and Git commit |
| Phase 2 boundary | Dockerfile is added, but image build happens in GitHub Actions |
| Phase 2.5 boundary | GitHub Actions runs lint, tests, Docker build, ACR push, and later ACA deployment |
| Phase 3 boundary | Terraform creates Azure infrastructure after the app works locally |
| Main rule | Do not block Phase 1 on Docker Desktop |

# Local_FastMCP_Server_Standup_Mental_Model
| Concept | Operational Meaning |
|---|---|
| MCP server | A process that exposes tools to MCP-capable clients |
| FastMCP | Python framework used to define MCP tools with decorators instead of hand-writing MCP protocol handlers |
| Local server | First build target; proves the tool surface works before Docker, Azure, auth, APIM, or backing services |
| stdio transport | Local development transport where the MCP client starts the server process and communicates over stdin/stdout |
| Streamable HTTP transport | Remote-ready transport where the server listens on a local HTTP port and exposes the MCP endpoint at `/mcp` |
| Tool surface | The named MCP tools the client can discover and call |
| Narrow tool | One discrete capability, not a giant all-purpose function |
| Shared retrieval backend | One internal search function used by multiple external tools |
| search_workbook | MCP tool for querying Workbook notes and lab material |
| search_azure_docs | MCP tool for querying Azure documentation corpus or Azure workbook notes |
| get_lab_metadata_static | MCP tool for returning static lab metadata before live CML or dynamic lab state exists |
| Input validation | Every tool treats client input as untrusted and validates query length, corpus, top_k, and path scope |
| Structured logging | Server emits useful logs for tool name, request ID, duration, result count, and errors |
| Local success state | Client can list tools, call each tool, get deterministic responses, and run both stdio and HTTP locally |
| GitHub Actions build path | Container builds happen remotely so weak local hardware does not block the project |

# Local_FastMCP_Server_Standup_Prerequisite_Install_uv
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Install uv | Workstation | `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"` | `uv.exe`, `uvx.exe`, and `uvw.exe` install under the user profile |
| 2 | Confirm uv is available | Workstation | `uv --version` | uv version is printed |
| 3 | Restart shell if needed | Workstation | Close and reopen PowerShell | PowerShell can find `uv` on PATH |

# Local_FastMCP_Server_Standup_Configuration_Checklist
| Step | Task                                                   | Device      | Command                                                                          | PowerShell                                                                                                                                                                              | Expected Result                                                                           |     |
| ---: | ------------------------------------------------------ | ----------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | --- |
|    1 | Create project folder                                  | Workstation | `mkdir workbook-mcp-server && cd workbook-mcp-server`                            | `New-Item -ItemType Directory -Force -Path workbook-mcp-server; Set-Location workbook-mcp-server`                                                                                       | Empty project directory exists                                                            | x   |
|    2 | Initialize Python project                              | Workstation | `uv init`                                                                        | `uv init`                                                                                                                                                                               | `pyproject.toml` is created                                                               | x   |
|    3 | Add FastMCP dependency                                 | Workstation | `uv add fastmcp`                                                                 | `uv add fastmcp`                                                                                                                                                                        | FastMCP is added to project dependencies                                                  | x   |
|    4 | Add test and lint dependencies                         | Workstation | `uv add --dev pytest ruff`                                                       | `uv add --dev pytest ruff`                                                                                                                                                              | Test and lint tools are available                                                         | x   |
|    5 | Create source layout                                   | Workstation | `mkdir -p src/workbook_mcp tools data tests`                                     | `New-Item -ItemType Directory -Force -Path src\workbook_mcp, tools, data, tests`                                                                                                        | Project folders exist                                                                     | x   |
|    6 | Create package marker                                  | Workstation | `touch src/workbook_mcp/__init__.py`                                             | `New-Item -ItemType File -Force -Path src\workbook_mcp\__init__.py`                                                                                                                     | Python package is importable                                                              | x   |
|    7 | Create server file                                     | Workstation | `touch src/workbook_mcp/server.py`                                               | `New-Item -ItemType File -Force -Path src\workbook_mcp\server.py`                                                                                                                       | Server entry file exists                                                                  | x   |
|    8 | Create search backend file                             | Workstation | `touch src/workbook_mcp/search_backend.py`                                       | `New-Item -ItemType File -Force -Path src\workbook_mcp\search_backend.py`                                                                                                               | Shared retrieval backend file exists                                                      | x   |
|    9 | Create lab metadata file                               | Workstation | `touch src/workbook_mcp/lab_metadata.py`                                         | `New-Item -ItemType File -Force -Path src\workbook_mcp\lab_metadata.py`                                                                                                                 | Static lab metadata module exists                                                         | x   |
|   10 | Create settings file                                   | Workstation | `touch src/workbook_mcp/settings.py`                                             | `New-Item -ItemType File -Force -Path src\workbook_mcp\settings.py`                                                                                                                     | Runtime settings module exists                                                            | x   |
|   11 | Create local data folders                              | Workstation | `mkdir -p data/workbook data/azure_docs data/lab_metadata`                       | `New-Item -ItemType Directory -Force -Path data\workbook, data\azure_docs, data\lab_metadata`                                                                                           | Local corpus folders exist                                                                | x   |
|   12 | Define allowed corpus names                            | Workstation | `workbook`, `azure_docs`                                                         | `code src\workbook_mcp\settings.py`                                                                                                                                                     | Tools cannot search arbitrary paths                                                       | x   |
|   13 | Implement shared backend                               | Workstation | `search_corpus(query, corpus, top_k)`                                            | `code src\workbook_mcp\search_backend.py`                                                                                                                                               | One retrieval function supports both search tools                                         | x   |
|   14 | Implement workbook tool                                | Workstation | `search_workbook(query: str, top_k: int = 5)`                                    | `code src\workbook_mcp\server.py`                                                                                                                                                       | Tool calls shared backend with `corpus="workbook"`                                        | x   |
|   15 | Implement Azure docs tool                              | Workstation | `search_azure_docs(query: str, top_k: int = 5)`                                  | `code src\workbook_mcp\server.py`                                                                                                                                                       | Tool calls shared backend with `corpus="azure_docs"`                                      | x   |
|   16 | Implement lab metadata tool                            | Workstation | `get_lab_metadata_static(lab_id: str)`                                           | `code src\workbook_mcp\server.py; code src\workbook_mcp\lab_metadata.py`                                                                                                                | Tool returns static lab metadata by lab ID                                                |     |
|   17 | Add strict input checks                                | Workstation | `validate query, top_k, lab_id, corpus`                                          | `code src\workbook_mcp\server.py; code src\workbook_mcp\search_backend.py`                                                                                                              | Bad inputs fail cleanly before filesystem access                                          |     |
|   18 | Add structured logging                                 | Workstation | `logging.info(... extra={...})`                                                  | `code src\workbook_mcp\server.py`                                                                                                                                                       | Tool calls produce readable operational logs                                              |     |
|   19 | Add server identity                                    | Workstation | `FastMCP("Workbook MCP Server")`                                                 | `code src\workbook_mcp\server.py`                                                                                                                                                       | MCP client sees correct server name                                                       |     |
|   20 | Add tool docstrings                                    | Workstation | Python docstrings on every tool                                                  | `code src\workbook_mcp\server.py`                                                                                                                                                       | MCP clients can understand tool purpose                                                   |     |
|   21 | Add stdio run path                                     | Workstation | `mcp.run()`                                                                      | `code src\workbook_mcp\server.py`                                                                                                                                                       | Server can run as local stdio MCP server                                                  |     |
|   22 | Add HTTP run path                                      | Workstation | `mcp.run(transport="http", host="127.0.0.1", port=8000)`                         | `code src\workbook_mcp\server.py`                                                                                                                                                       | Server can expose `http://127.0.0.1:8000/mcp`                                             |     |
|   23 | Add `/health` route for HTTP mode                      | Workstation | `@mcp.custom_route("/health", methods=["GET"])`                                  | `code src\workbook_mcp\server.py`                                                                                                                                                       | Health endpoint returns `OK`                                                              |     |
|   24 | Verify FastMCP install                                 | Workstation | `uv run fastmcp version`                                                         | `uv run fastmcp version`                                                                                                                                                                | FastMCP version output is displayed                                                       |     |
|   25 | Run server over stdio                                  | Workstation | `uv run fastmcp run src/workbook_mcp/server.py:mcp`                              | `uv run fastmcp run src\workbook_mcp\server.py:mcp`                                                                                                                                     | Server starts under stdio transport                                                       |     |
|   26 | Run server over HTTP                                   | Workstation | `uv run fastmcp run src/workbook_mcp/server.py:mcp --transport http --port 8000` | `uv run fastmcp run src\workbook_mcp\server.py:mcp --transport http --port 8000`                                                                                                        | HTTP MCP server listens locally                                                           |     |
|   27 | Verify health endpoint                                 | Workstation | `curl http://127.0.0.1:8000/health`                                              | `Invoke-RestMethod http://127.0.0.1:8000/health`                                                                                                                                        | Response returns `OK`                                                                     |     |
|   28 | Create basic tool tests                                | Workstation | `touch tests/test_tools.py`                                                      | `New-Item -ItemType File -Force -Path tests\test_tools.py`                                                                                                                              | Test file exists                                                                          |     |
|   29 | Test shared backend                                    | Workstation | `uv run pytest`                                                                  | `uv run pytest`                                                                                                                                                                         | Search backend returns expected mock results                                              |     |
|   30 | Lint project                                           | Workstation | `uv run ruff check .`                                                            | `uv run ruff check .`                                                                                                                                                                   | No lint failures                                                                          |     |
|   31 | Add MCP client config                                  | Workstation | `.vscode/mcp.json` or client-specific MCP config                                 | `New-Item -ItemType Directory -Force -Path .vscode; New-Item -ItemType File -Force -Path .vscode\mcp.json`                                                                              | Client knows how to start server                                                          |     |
|   32 | Confirm tools list in client                           | MCP client  | Use MCP client tool discovery                                                    | Use MCP client tool discovery                                                                                                                                                           | `search_workbook`, `search_azure_docs`, and `get_lab_metadata_static` appear              |     |
|   33 | Call each tool once                                    | MCP client  | Tool call from client UI                                                         | Tool call from client UI                                                                                                                                                                | Each tool returns valid response                                                          |     |
|   34 | Add Git ignore file                                    | Workstation | `touch .gitignore`                                                               | `New-Item -ItemType File -Force -Path .gitignore`                                                                                                                                       | Local junk, virtual environment, cache files, and secrets can be excluded                 |     |
|   35 | Add standard Python ignores                            | Workstation | Edit `.gitignore`                                                                | `Add-Content .gitignore ".venv/"; Add-Content .gitignore "__pycache__/"; Add-Content .gitignore ".pytest_cache/"; Add-Content .gitignore ".ruff_cache/"; Add-Content .gitignore ".env"` | Git will not track virtual environment, cache folders, or local secrets                   |     |
|   36 | Confirm repo status before commit                      | Workstation | `git status`                                                                     | `git status`                                                                                                                                                                            | Expected project files are staged or ready to stage                                       |     |
|   37 | Commit working local baseline                          | Workstation | `git add . && git commit -m "Add local FastMCP server baseline"`                 | `git add .; git commit -m "Add local FastMCP server baseline"`                                                                                                                          | Phase 1 baseline is saved                                                                 |     |
|   38 | Push baseline to GitHub                                | Workstation | `git push origin main`                                                           | `git push origin main`                                                                                                                                                                  | GitHub has the working local MCP server baseline                                          |     |
|   39 | Mark container builds as GitHub Actions responsibility | GitHub repo | Future workflow path: `.github/workflows/container-build.yml`                    | Future workflow path: `.github\workflows\container-build.yml`                                                                                                                           | Container build will happen on GitHub-hosted runner, not weak local workstation           |     |
|   40 | Stop before Dockerfile and Azure deployment            | Workstation | N/A                                                                              | N/A                                                                                                                                                                                     | Local server is proven before Dockerfile, ACR, Container Apps, and Azure auth work begins |     |

# Local_FastMCP_Server_Standup_Skeleton
| File | Purpose | Skeleton |
|---|---|---|
| `pyproject.toml` | Python project and dependency definition | `fastmcp`, `pytest`, `ruff` |
| `src/workbook_mcp/server.py` | Main FastMCP server and tool registration | Defines `mcp`, tools, `/health`, and run behavior |
| `src/workbook_mcp/search_backend.py` | Shared local retrieval backend | Defines `search_corpus(query, corpus, top_k)` |
| `src/workbook_mcp/lab_metadata.py` | Static lab metadata lookup | Defines `get_lab_metadata(lab_id)` |
| `src/workbook_mcp/settings.py` | Local paths and runtime settings | Defines corpus roots and allowed top_k limits |
| `data/workbook/` | Local Workbook corpus | Markdown files copied or symlinked from Workbook repo |
| `data/azure_docs/` | Local Azure corpus | Azure docs or Azure workbook markdown |
| `data/lab_metadata/labs.json` | Static lab metadata | JSON list of lab records |
| `tests/test_tools.py` | Tool contract tests | Confirms tools return stable response shapes |
| `.vscode/mcp.json` | VS Code MCP client config | Registers local server command |
| `.gitignore` | Local exclusion rules | Excludes `.venv/`, Python caches, test caches, ruff caches, and `.env` |
| `Dockerfile` | Future container definition | Added after local stdio and HTTP server paths work |
| `.github/workflows/container-build.yml` | Future GitHub Actions workflow | Builds container image remotely on GitHub-hosted runner |

# Local_FastMCP_Server_Standup_Minimal_Server_File
| File | Content |
|---|---|
| `src/workbook_mcp/server.py` | See code block below |

```python
from __future__ import annotations

import logging
import os
from typing import Any

from fastmcp import FastMCP
from starlette.requests import Request
from starlette.responses import PlainTextResponse

from workbook_mcp.lab_metadata import get_lab_metadata
from workbook_mcp.search_backend import search_corpus

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("workbook_mcp")

mcp = FastMCP(
    name="Workbook MCP Server",
    instructions=(
        "Provides narrow tools for searching Workbook notes, searching Azure docs, "
        "and returning static lab metadata."
    ),
)


def _validate_query(query: str) -> str:
    query = query.strip()
    if not query:
        raise ValueError("query cannot be empty")
    if len(query) > 500:
        raise ValueError("query cannot exceed 500 characters")
    return query


def _validate_top_k(top_k: int) -> int:
    if top_k < 1:
        raise ValueError("top_k must be at least 1")
    if top_k > 10:
        raise ValueError("top_k cannot exceed 10")
    return top_k


@mcp.custom_route("/health", methods=["GET"])
async def health_check(request: Request) -> PlainTextResponse:
    return PlainTextResponse("OK")


@mcp.tool
def search_workbook(query: str, top_k: int = 5) -> list[dict[str, Any]]:
    """
    Search the local Workbook corpus for relevant notes and lab material.
    """
    query = _validate_query(query)
    top_k = _validate_top_k(top_k)

    logger.info(
        "tool_call",
        extra={"tool": "search_workbook", "top_k": top_k},
    )

    return search_corpus(query=query, corpus="workbook", top_k=top_k)


@mcp.tool
def search_azure_docs(query: str, top_k: int = 5) -> list[dict[str, Any]]:
    """
    Search the local Azure documentation corpus.
    """
    query = _validate_query(query)
    top_k = _validate_top_k(top_k)

    logger.info(
        "tool_call",
        extra={"tool": "search_azure_docs", "top_k": top_k},
    )

    return search_corpus(query=query, corpus="azure_docs", top_k=top_k)


@mcp.tool
def get_lab_metadata_static(lab_id: str) -> dict[str, Any]:
    """
    Return static metadata for a known lab ID.
    """
    lab_id = lab_id.strip()
    if not lab_id:
        raise ValueError("lab_id cannot be empty")
    if len(lab_id) > 100:
        raise ValueError("lab_id cannot exceed 100 characters")

    logger.info(
        "tool_call",
        extra={"tool": "get_lab_metadata_static", "lab_id": lab_id},
    )

    return get_lab_metadata(lab_id)


if __name__ == "__main__":
    transport = os.getenv("MCP_TRANSPORT", "stdio")

    if transport == "http":
        mcp.run(
            transport="http",
            host=os.getenv("MCP_HOST", "127.0.0.1"),
            port=int(os.getenv("MCP_PORT", "8000")),
        )
    else:
        mcp.run()
```

# Local_FastMCP_Server_Standup_Search_Backend_Skeleton
| File | Content |
|---|---|
| `src/workbook_mcp/search_backend.py` | See code block below |

```python
from __future__ import annotations

from pathlib import Path
from typing import Any

CORPUS_ROOTS = {
    "workbook": Path("data/workbook"),
    "azure_docs": Path("data/azure_docs"),
}


def search_corpus(query: str, corpus: str, top_k: int = 5) -> list[dict[str, Any]]:
    """
    Minimal local retrieval backend.

    Phase 1 behavior:
    - search local markdown files
    - return simple keyword matches
    - keep response shape stable for later Azure AI Search swap
    """
    if corpus not in CORPUS_ROOTS:
        raise ValueError(f"unsupported corpus: {corpus}")

    root = CORPUS_ROOTS[corpus]
    if not root.exists():
        return []

    query_terms = query.lower().split()
    results: list[dict[str, Any]] = []

    for path in root.rglob("*.md"):
        text = path.read_text(encoding="utf-8", errors="ignore")
        lowered = text.lower()

        score = sum(lowered.count(term) for term in query_terms)
        if score <= 0:
            continue

        snippet_start = max(0, lowered.find(query_terms[0]) - 120)
        snippet_end = min(len(text), snippet_start + 500)

        results.append(
            {
                "corpus": corpus,
                "path": str(path),
                "score": score,
                "snippet": text[snippet_start:snippet_end].strip(),
            }
        )

    results.sort(key=lambda item: item["score"], reverse=True)
    return results[:top_k]
```

# Local_FastMCP_Server_Standup_Lab_Metadata_Skeleton
| File | Content |
|---|---|
| `src/workbook_mcp/lab_metadata.py` | See code block below |

```python
from __future__ import annotations

import json
from pathlib import Path
from typing import Any

LAB_METADATA_PATH = Path("data/lab_metadata/labs.json")


def get_lab_metadata(lab_id: str) -> dict[str, Any]:
    if not LAB_METADATA_PATH.exists():
        raise FileNotFoundError("data/lab_metadata/labs.json does not exist")

    labs = json.loads(LAB_METADATA_PATH.read_text(encoding="utf-8"))

    for lab in labs:
        if lab.get("lab_id") == lab_id:
            return lab

    raise KeyError(f"lab_id not found: {lab_id}")
```

# Local_FastMCP_Server_Standup_Static_Metadata_Example
| File | Content |
|---|---|
| `data/lab_metadata/labs.json` | See code block below |

```json
[
  {
    "lab_id": "mcp-local-001",
    "title": "Local FastMCP Server Baseline",
    "domain": "MCP Server Development",
    "difficulty": "beginner",
    "repo_path": "MCP/Local_FastMCP_Server_Standup.md",
    "related_notes": [
      "Local_FastMCP_Server_Standup.md",
      "MCP_Tool_Surface_Design.md",
      "MCP_Streamable_HTTP_Transport.md"
    ],
    "expected_services": [
      "FastMCP",
      "stdio transport",
      "Streamable HTTP transport"
    ],
    "skills_tested": [
      "Python project setup",
      "MCP tool registration",
      "local retrieval backend",
      "tool validation",
      "local MCP client testing"
    ]
  }
]
```

# Local_FastMCP_Server_Standup_Client_Config_Skeleton
| Client  | File               | Content                   |
| ------- | ------------------ | ------------------------- |
| VS Code | `.vscode/mcp.json` | Local stdio server config |

```json
{
  "servers": {
    "workbook-local": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "run",
        "fastmcp",
        "run",
        "src/workbook_mcp/server.py:mcp"
      ]
    }
  }
}
```

# Local_FastMCP_Server_Standup_Verification_Commands
| Command | PowerShell | Purpose | Good Output |
|---|---|---|---|
| `uv --version` | `uv --version` | Confirms uv is installed | uv version is printed |
| `uv run python --version` | `uv run python --version` | Confirms Python runtime works | Python version is printed |
| `uv run fastmcp version` | `uv run fastmcp version` | Confirms FastMCP is installed | FastMCP and MCP versions are printed |
| `uv run ruff check .` | `uv run ruff check .` | Lints project | No lint errors |
| `uv run pytest` | `uv run pytest` | Runs local tests | Tests pass |
| `uv run fastmcp run src/workbook_mcp/server.py:mcp` | `uv run fastmcp run src\workbook_mcp\server.py:mcp` | Runs local stdio server | Server starts without import errors |
| `uv run fastmcp run src/workbook_mcp/server.py:mcp --transport http --port 8000` | `uv run fastmcp run src\workbook_mcp\server.py:mcp --transport http --port 8000` | Runs local HTTP server | Server listens on port 8000 |
| `curl http://127.0.0.1:8000/health` | `Invoke-RestMethod http://127.0.0.1:8000/health` | Verifies HTTP health route | Returns `OK` |
| `curl -i http://127.0.0.1:8000/mcp` | `Invoke-WebRequest http://127.0.0.1:8000/mcp` | Confirms MCP endpoint exists | HTTP response comes from MCP endpoint |
| MCP client tool list | MCP client tool list | Confirms tool discovery | `search_workbook`, `search_azure_docs`, and `get_lab_metadata_static` appear |
| MCP client call `search_workbook` | MCP client call `search_workbook` | Confirms workbook search tool works | Returns list of result objects |
| MCP client call `search_azure_docs` | MCP client call `search_azure_docs` | Confirms Azure docs search tool works | Returns list of result objects |
| MCP client call `get_lab_metadata_static` | MCP client call `get_lab_metadata_static` | Confirms metadata lookup works | Returns matching lab metadata object |
| `git status` | `git status` | Confirms local Git state before commit | Expected files are visible |
| `git add . && git commit -m "Add local FastMCP server baseline"` | `git add .; git commit -m "Add local FastMCP server baseline"` | Saves Phase 1 baseline | Local commit exists |
| `git push origin main` | `git push origin main` | Pushes Phase 1 baseline to GitHub | GitHub repo has local server code |
| N/A | GitHub Actions workflow run in future Phase 2.5 | Confirms container image builds remotely | GitHub-hosted runner builds image without relying on local Docker |

# Local_FastMCP_Server_Standup_Rollback
| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Stop running stdio or HTTP server | Workstation | `Ctrl+C` | `Ctrl+C` | Server process exits |
| 2 | Remove broken local environment | Workstation | `rm -rf .venv` | `Remove-Item -Recurse -Force .venv` | Virtual environment is removed |
| 3 | Recreate environment | Workstation | `uv sync` | `uv sync` | Dependencies reinstall from project definition |
| 4 | Remove bad dependency change | Workstation | `git restore pyproject.toml uv.lock` | `git restore pyproject.toml uv.lock` | Dependency files return to last committed state |
| 5 | Remove bad code change | Workstation | `git restore src tests` | `git restore src tests` | Source and tests return to last committed state |
| 6 | Confirm rollback state | Workstation | `git status` | `git status` | Working tree shows expected clean or known changes |
| 7 | Re-run verification | Workstation | `uv run pytest && uv run ruff check .` | `uv run pytest; uv run ruff check .` | Tests and lint pass |
| 8 | Re-run server | Workstation | `uv run fastmcp run src/workbook_mcp/server.py:mcp` | `uv run fastmcp run src\workbook_mcp\server.py:mcp` | Local server starts again |
| 9 | Roll back bad GitHub Actions workflow later | GitHub repo | `git revert <BAD_COMMIT_SHA>` | `git revert <BAD_COMMIT_SHA>` | Broken CI/CD change is reversed |
| 10 | Re-run GitHub Actions later | GitHub repo | Push corrected commit | Push corrected commit | Remote container build workflow runs again |

# Local_FastMCP_Server_Standup_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `ModuleNotFoundError: fastmcp` | Dependency not installed in active environment | `uv run fastmcp version` | Run `uv add fastmcp` then `uv sync` |
| Server starts but client sees no tools | Tools not registered with `@mcp.tool` | Inspect `server.py` | Add `@mcp.tool` above each tool function |
| Client cannot start stdio server | Bad client command or wrong working directory | Check `.vscode/mcp.json` command and args | Use `uv run fastmcp run src\workbook_mcp\server.py:mcp` |
| HTTP server does not expose `/mcp` | Server not running with HTTP transport | Check startup command | Run with `--transport http --port 8000` |
| `/health` fails | Custom route missing or HTTP server not running | `Invoke-RestMethod http://127.0.0.1:8000/health` | Add route and restart HTTP server |
| Search returns empty results | Corpus folder empty or wrong path | `Get-ChildItem -Recurse data\workbook -Filter *.md` | Copy or symlink Workbook markdown files into corpus folder |
| Search tool reads outside project | Unsafe path handling | Review backend path logic | Restrict corpus to fixed allowlist only |
| Bad query crashes server | Missing input validation | Call tool with empty query | Add query validation before backend call |
| `top_k` returns too much data | Missing top_k limit | Call tool with `top_k=999` | Reject values above 10 |
| Lab metadata tool fails | Missing `labs.json` | `Test-Path data\lab_metadata\labs.json` | Create static lab metadata file |
| Lab ID not found | Unknown or mistyped lab ID | Check `data\lab_metadata\labs.json` | Add lab record or correct lab ID |
| Logs are useless | No structured tool logging | Review terminal output | Log tool name, top_k, corpus, duration, and errors |
| Client hangs on stdio | Server prints protocol-breaking noise to stdout | Review print statements | Use logging, do not print random output to stdout |
| HTTP port already in use | Port 8000 occupied | `netstat -ano \| findstr :8000` | Stop conflicting process or use another port |
| `docker` command is missing locally | Docker Desktop is not installed | `docker --version` | Not a Phase 1 failure; GitHub Actions will handle container builds later |
| Local Docker Desktop runs poorly | Weak Surface hardware | Check RAM, CPU virtualization, and disk space | Do not rely on local Docker; build image in GitHub Actions |
| GitHub Actions build fails later | Dockerfile, workflow, or dependency issue | Check failed workflow logs | Fix workflow or Dockerfile and push new commit |
| Works locally but not ready for Azure | Phase boundary crossed too early | Confirm tests, lint, stdio, and HTTP work first | Finish local tool tests before Dockerfile, ACR, Container Apps, and Azure auth |

# Local_FastMCP_Server_Standup_Related_Labs
| Lab | Relationship |
|---|---|
| `mcp-local-001` | Baseline local FastMCP server |
| `mcp-tools-001` | Tool surface design and narrow tool contracts |
| `mcp-retrieval-001` | Shared local retrieval backend |
| `mcp-http-001` | Streamable HTTP transport and `/mcp` endpoint |
| `mcp-client-001` | VS Code or Claude Desktop MCP client wiring |
| `mcp-validation-001` | Tool input validation and failure handling |
| `mcp-ci-container-build-001` | Future GitHub Actions container image build |
| `mcp-acr-push-001` | Future push from GitHub Actions to Azure Container Registry |
| `mcp-aca-deploy-001` | Future deploy from ACR image to Azure Container Apps |
``


# Local_FastMCP_Server_Standup_Settings_File

| File | Content |
|---|---|
| `src/workbook_mcp/settings.py` | See code block below |

```python
from __future__ import annotations

from pathlib import Path

PROJECT_ROOT = Path(__file__).resolve().parents[2]

DATA_ROOT = PROJECT_ROOT / "data"
WORKBOOK_CORPUS_ROOT = DATA_ROOT / "workbook"
AZURE_DOCS_CORPUS_ROOT = DATA_ROOT / "azure_docs"
LAB_METADATA_PATH = DATA_ROOT / "lab_metadata" / "labs.json"

ALLOWED_CORPORA = {
    "workbook": WORKBOOK_CORPUS_ROOT,
    "azure_docs": AZURE_DOCS_CORPUS_ROOT,
}

MAX_QUERY_LENGTH = 500
MIN_TOP_K = 1
MAX_TOP_K = 10
MAX_LAB_ID_LENGTH = 100
```