# P0 - Project Scaffolding Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the FastAPI + Vue3/Vite project skeleton with SQLite data models, config management, and a working dark-theme sidebar layout so both frontend and backend can start up and respond to requests.

**Architecture:** FastAPI backend with SQLAlchemy async ORM over SQLite, serving REST API endpoints. Vue3 frontend with Vite build tool, Pinia state management, Vue Router, Axios HTTP client, and a custom dark theme sidebar layout. Both run locally on separate ports (8000 backend, 5173 frontend) with CORS bridging.

**Tech Stack:** FastAPI 0.115+, SQLAlchemy 2.0+, aiosqlite, uvicorn, pydantic-settings, python-dotenv; Vue3 3.5+, Vite 5+, Pinia, Vue Router 4, Axios

---

## File Structure

### Backend (`backend/`)

```
backend/
  app/
    __init__.py
    main.py              # FastAPI app entry point, CORS, lifespan, mount routers
    config.py             # Settings via pydantic-settings + .env
    database.py           # SQLAlchemy async engine, session factory, init_db()
    models/
      __init__.py         # Import all models for registration
      project.py          # Project ORM model
      script.py           # Script ORM model
      device.py           # Device ORM model
      execution.py        # Execution ORM model
      llm_config.py       # LLMConfig ORM model
      apk.py              # Apk ORM model
      report.py           # Report ORM model
    routers/
      __init__.py
      projects.py         # Project CRUD router (placeholder endpoints)
      scripts.py          # Script CRUD router (placeholder endpoints)
      devices.py          # Device router (placeholder endpoints)
      executions.py       # Execution router (placeholder endpoints)
      reports.py          # Report router (placeholder endpoints)
      apks.py             # APK router (placeholder endpoints)
      llm_configs.py      # LLM config router (placeholder endpoints)
      ai.py               # AI generation router (placeholder endpoints)
      dashboard.py        # Dashboard stats router (placeholder endpoints)
    schemas/
      __init__.py
      project.py          # Project Pydantic schemas (create/update/response)
      script.py           # Script Pydantic schemas
      device.py           # Device Pydantic schemas
      execution.py        # Execution Pydantic schemas
      llm_config.py       # LLM config Pydantic schemas
      apk.py              # Apk Pydantic schemas
      report.py           # Report Pydantic schemas
    services/
      __init__.py
    exceptions.py         # Custom exceptions + global error handler
  data/                   # SQLite DB + file storage (auto-created)
    droidbot.db
    scripts/
    apks/
    reports/
    screenshots/
    logs/
    uploads/
  .env                    # Environment config (DB path, CORS origins, etc.)
  .env.example            # Template .env for new setups
  requirements.txt        # Python dependencies
  alembic.ini             # (future, not implemented in P0)
```

### Frontend (`frontend/`)

```
frontend/
  src/
    main.js               # Vue app bootstrap
    App.vue               # Root component (renders router-view inside AppLayout)
    router/
      index.js            # All route definitions with placeholder components
    stores/
      index.js            # Pinia setup
      project.js          # Project store (empty skeleton)
      script.js           # Script store (empty skeleton)
      device.js           # Device store (empty skeleton)
      execution.js        # Execution store (empty skeleton)
    api/
      index.js            # Axios instance (baseURL, interceptors, error handling)
    layouts/
      AppLayout.vue       # Sidebar + main content layout
    components/
      Sidebar.vue         # 260px fixed sidebar with nav menu
      PlaceholderPage.vue # Reusable "coming soon" placeholder component
    views/
      DashboardView.vue         # Placeholder
      ProjectsView.vue          # Placeholder
      ProjectScriptsView.vue    # Placeholder
      ScriptEditView.vue        # Placeholder
      ScriptDetailView.vue      # Placeholder
      AiScriptView.vue          # Placeholder
      ExecutionNewView.vue      # Placeholder
      ExecutionDetailView.vue   # Placeholder
      DevicesView.vue           # Placeholder
      ReportsView.vue           # Placeholder
      ReportDetailView.vue      # Placeholder
      ApkManagerView.vue        # Placeholder
      LlmConfigView.vue         # Placeholder
      GeneralSettingsView.vue   # Placeholder
    styles/
      variables.css          # CSS custom properties (dark theme)
  index.html
  vite.config.js            # Vite config (dev server proxy to backend)
  package.json
```

---

## Task 1: Backend Directory Structure + Dependencies

**Files:**
- Create: `backend/app/__init__.py`
- Create: `backend/app/main.py`
- Create: `backend/app/config.py`
- Create: `backend/app/database.py`
- Create: `backend/app/exceptions.py`
- Create: `backend/.env`
- Create: `backend/.env.example`
- Create: `backend/requirements.txt`
- Create: `backend/data/` (directories only)

- [ ] **Step 1: Create backend directory tree**

```bash
cd C:\pythonworkspace\DroidBot
mkdir -p backend/app/models backend/app/routers backend/app/schemas backend/app/services
mkdir -p backend/data/scripts backend/data/apks backend/data/reports backend/data/screenshots backend/data/logs backend/data/uploads
touch backend/app/__init__.py backend/app/models/__init__.py backend/app/routers/__init__.py backend/app/schemas/__init__.py backend/app/services/__init__.py
```

- [ ] **Step 2: Write `backend/requirements.txt`**

```
fastapi==0.115.6
uvicorn[standard]==0.34.0
sqlalchemy[asyncio]==2.0.36
aiosqlite==0.20.0
pydantic-settings==2.7.1
python-dotenv==1.0.1
python-multipart==0.0.20
httpx==0.28.1
```

- [ ] **Step 3: Install backend dependencies**

Run: `cd backend && pip install -r requirements.txt`
Expected: All packages installed successfully

- [ ] **Step 4: Write `backend/app/config.py`**

```python
from pydantic_settings import BaseSettings
from pathlib import Path


class Settings(BaseSettings):
    """Application configuration loaded from .env file."""

    # Database
    db_path: Path = Path("data/droidbot.db")

    # CORS
    cors_origins: list[str] = ["http://localhost:5173", "http://127.0.0.1:5173"]

    # Server
    host: str = "0.0.0.0"
    port: int = 8000

    # Data directories
    data_dir: Path = Path("data")
    scripts_dir: Path = Path("data/scripts")
    apks_dir: Path = Path("data/apks")
    reports_dir: Path = Path("data/reports")
    screenshots_dir: Path = Path("data/screenshots")
    logs_dir: Path = Path("data/logs")
    uploads_dir: Path = Path("data/uploads")

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}


settings = Settings()
```

- [ ] **Step 5: Write `backend/.env.example`**

```
# DroidBot Backend Configuration
DB_PATH=data/droidbot.db
CORS_ORIGINS=["http://localhost:5173","http://127.0.0.1:5173"]
HOST=0.0.0.0
PORT=8000
```

- [ ] **Step 6: Write `backend/.env`**

Same content as `.env.example` for local development.

- [ ] **Step 7: Write `backend/app/database.py`**

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase
from app.config import settings


engine = create_async_engine(
    f"sqlite+aiosqlite:///{settings.db_path}",
    echo=False,
)

async_session = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)


class Base(DeclarativeBase):
    """SQLAlchemy declarative base for all models."""
    pass


async def init_db() -> None:
    """Create all tables in the SQLite database."""
    # Import all models so Base.metadata knows about them
    import app.models  # noqa: F401

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)


async def get_session() -> AsyncSession:
    """FastAPI dependency that yields an async database session."""
    async with async_session() as session:
        yield session
```

- [ ] **Step 8: Write `backend/app/exceptions.py`**

```python
from fastapi import Request
from fastapi.responses import JSONResponse


class AppError(Exception):
    """Base application error with status code and message."""

    def __init__(self, message: str, status_code: int = 400):
        self.message = message
        self.status_code = status_code


class NotFoundError(AppError):
    def __init__(self, resource: str, resource_id: int):
        super().__init__(f"{resource} with id {resource_id} not found", status_code=404)


class ConflictError(AppError):
    def __init__(self, message: str):
        super().__init__(message, status_code=409)


async def app_error_handler(request: Request, exc: AppError) -> JSONResponse:
    """Global handler for AppError exceptions."""
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.message},
    )
```

- [ ] **Step 9: Write `backend/app/main.py`**

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.exception_handlers import http_exception_handler

from app.config import settings
from app.database import init_db
from app.exceptions import AppError, app_error_handler
from app.routers import (
    projects,
    scripts,
    devices,
    executions,
    reports,
    apks,
    llm_configs,
    ai,
    dashboard,
)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Initialize database on startup."""
    await init_db()
    yield


app = FastAPI(
    title="DroidBot API",
    description="AI-driven Android compatibility testing platform",
    version="0.1.0",
    lifespan=lifespan,
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Exception handlers
app.add_exception_handler(AppError, app_error_handler)

# Register routers
app.include_router(projects.router, prefix="/api/projects", tags=["Projects"])
app.include_router(scripts.router, prefix="/api/scripts", tags=["Scripts"])
app.include_router(devices.router, prefix="/api/devices", tags=["Devices"])
app.include_router(executions.router, prefix="/api/executions", tags=["Executions"])
app.include_router(reports.router, prefix="/api/reports", tags=["Reports"])
app.include_router(apks.router, prefix="/api/apks", tags=["APKs"])
app.include_router(llm_configs.router, prefix="/api/llm-configs", tags=["LLM Configs"])
app.include_router(ai.router, prefix="/api/ai", tags=["AI"])
app.include_router(dashboard.router, prefix="/api/dashboard", tags=["Dashboard"])


@app.get("/api/health", tags=["Health"])
async def health_check():
    return {"status": "ok", "version": "0.1.0"}
```

- [ ] **Step 10: Verify backend starts**

Run: `cd backend && python -m uvicorn app.main:app --host 0.0.0.0 --port 8000`
Expected: Server starts, no import errors. Visit `http://localhost:8000/docs` — should show OpenAPI UI with all routers listed.

- [ ] **Step 11: Commit backend skeleton**

```bash
cd C:\pythonworkspace\DroidBot
git add backend/
git commit -m "feat(P0): add FastAPI backend skeleton with config, database, and error handling"
```

---

## Task 2: SQLAlchemy Data Models

**Files:**
- Create: `backend/app/models/project.py`
- Create: `backend/app/models/script.py`
- Create: `backend/app/models/device.py`
- Create: `backend/app/models/execution.py`
- Create: `backend/app/models/llm_config.py`
- Create: `backend/app/models/apk.py`
- Create: `backend/app/models/report.py`
- Modify: `backend/app/models/__init__.py`

- [ ] **Step 1: Write `backend/app/models/project.py`**

```python
from sqlalchemy import String, Text, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.database import Base


class Project(Base):
    __tablename__ = "projects"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    description: Mapped[str] = mapped_column(Text, nullable=False, default="")
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())
    updated_at: Mapped[str] = mapped_column(DateTime, server_default=func.now(), onupdate=func.now())

    scripts: Mapped[list["Script"]] = relationship(back_populates="project", cascade="all, delete-orphan")
```

- [ ] **Step 2: Write `backend/app/models/script.py`**

```python
from sqlalchemy import String, Text, Integer, ForeignKey, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.database import Base


class Script(Base):
    __tablename__ = "scripts"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(200), nullable=False)
    description: Mapped[str] = mapped_column(Text, nullable=False, default="")
    content: Mapped[str] = mapped_column(Text, nullable=False, default="")
    project_id: Mapped[int] = mapped_column(Integer, ForeignKey("projects.id"), nullable=False)
    target_os: Mapped[str] = mapped_column(String(20), nullable=False, default="Android")
    file_path: Mapped[str] = mapped_column(String(500), nullable=False, default="")
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())
    updated_at: Mapped[str] = mapped_column(DateTime, server_default=func.now(), onupdate=func.now())

    project: Mapped["Project"] = relationship(back_populates="scripts")
```

- [ ] **Step 3: Write `backend/app/models/device.py`**

```python
from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from app.database import Base


class Device(Base):
    __tablename__ = "devices"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    serial: Mapped[str] = mapped_column(String(100), unique=True, nullable=False)
    model: Mapped[str] = mapped_column(String(100), nullable=False, default="")
    brand: Mapped[str] = mapped_column(String(50), nullable=False, default="")
    os_type: Mapped[str] = mapped_column(String(20), nullable=False, default="Android")
    android_version: Mapped[str] = mapped_column(String(20), nullable=False, default="")
    ip_address: Mapped[str] = mapped_column(String(50), nullable=False, default="")
    connection_type: Mapped[str] = mapped_column(String(20), nullable=False, default="usb")
    status: Mapped[str] = mapped_column(String(20), nullable=False, default="idle")
    resolution: Mapped[str] = mapped_column(String(20), nullable=False, default="")
    last_seen_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())
```

- [ ] **Step 4: Write `backend/app/models/execution.py`**

```python
from sqlalchemy import String, Text, Integer, ForeignKey, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.database import Base


class Execution(Base):
    __tablename__ = "executions"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    script_id: Mapped[int] = mapped_column(Integer, ForeignKey("scripts.id"), nullable=False)
    device_id: Mapped[int] = mapped_column(Integer, ForeignKey("devices.id"), nullable=False)
    status: Mapped[str] = mapped_column(String(20), nullable=False, default="pending")
    log_path: Mapped[str] = mapped_column(String(500), nullable=False, default="")
    report_path: Mapped[str] = mapped_column(String(500), nullable=False, default="")
    started_at: Mapped[str | None] = mapped_column(DateTime, nullable=True)
    finished_at: Mapped[str | None] = mapped_column(DateTime, nullable=True)
    result: Mapped[str] = mapped_column(String(20), nullable=False, default="")
    error_message: Mapped[str] = mapped_column(Text, nullable=False, default="")

    script: Mapped["Script"] = relationship()
    device: Mapped["Device"] = relationship()
```

- [ ] **Step 5: Write `backend/app/models/llm_config.py`**

```python
from sqlalchemy import String, Float, Boolean, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from app.database import Base


class LlmConfig(Base):
    __tablename__ = "llm_configs"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    provider: Mapped[str] = mapped_column(String(50), nullable=False)
    model: Mapped[str] = mapped_column(String(100), nullable=False)
    api_key: Mapped[str] = mapped_column(String(500), nullable=False, default="")
    api_base: Mapped[str] = mapped_column(String(200), nullable=False, default="")
    temperature: Mapped[float] = mapped_column(Float, nullable=False, default=0.7)
    is_active: Mapped[bool] = mapped_column(Boolean, nullable=False, default=False)
```

- [ ] **Step 6: Write `backend/app/models/apk.py`**

```python
from sqlalchemy import String, Integer, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from app.database import Base


class Apk(Base):
    __tablename__ = "apks"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    filename: Mapped[str] = mapped_column(String(200), nullable=False)
    file_path: Mapped[str] = mapped_column(String(500), nullable=False)
    version: Mapped[str] = mapped_column(String(50), nullable=False, default="")
    package_name: Mapped[str] = mapped_column(String(200), nullable=False, default="")
    size: Mapped[int] = mapped_column(Integer, nullable=False, default=0)
    uploaded_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())
```

- [ ] **Step 7: Write `backend/app/models/report.py`**

```python
from sqlalchemy import String, Text, Integer, ForeignKey, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.database import Base


class Report(Base):
    __tablename__ = "reports"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    execution_id: Mapped[int] = mapped_column(Integer, ForeignKey("executions.id"), nullable=False)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    content: Mapped[str] = mapped_column(Text, nullable=False, default="")
    summary: Mapped[str] = mapped_column(Text, nullable=False, default="")
    screenshots: Mapped[str] = mapped_column(Text, nullable=False, default="[]")
    created_at: Mapped[str] = mapped_column(DateTime, server_default=func.now())

    execution: Mapped["Execution"] = relationship()
```

- [ ] **Step 8: Update `backend/app/models/__init__.py` to register all models**

```python
from app.models.project import Project
from app.models.script import Script
from app.models.device import Device
from app.models.execution import Execution
from app.models.llm_config import LlmConfig
from app.models.apk import Apk
from app.models.report import Report

__all__ = [
    "Project",
    "Script",
    "Device",
    "Execution",
    "LlmConfig",
    "Apk",
    "Report",
]
```

- [ ] **Step 9: Verify database creates all tables**

Run: `cd backend && python -c "import asyncio; from app.database import init_db; asyncio.run(init_db()); print('Tables created')" `
Expected: "Tables created" printed, `data/droidbot.db` file exists.

- [ ] **Step 10: Delete test DB, restart uvicorn, verify auto-init**

```bash
rm backend/data/droidbot.db
cd backend && python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Expected: Server starts, `data/droidbot.db` auto-created on startup, `/api/health` returns `{"status": "ok"}`.

- [ ] **Step 11: Commit data models**

```bash
cd C:\pythonworkspace\DroidBot
git add backend/app/models/
git commit -m "feat(P0): add SQLAlchemy data models for all 8 tables (projects, scripts, devices, executions, llm_configs, apks, reports)"
```

---

## Task 3: Pydantic Schemas + Placeholder API Routers

**Files:**
- Create: `backend/app/schemas/project.py`
- Create: `backend/app/schemas/script.py`
- Create: `backend/app/schemas/device.py`
- Create: `backend/app/schemas/execution.py`
- Create: `backend/app/schemas/llm_config.py`
- Create: `backend/app/schemas/apk.py`
- Create: `backend/app/schemas/report.py`
- Create: `backend/app/routers/projects.py`
- Create: `backend/app/routers/scripts.py`
- Create: `backend/app/routers/devices.py`
- Create: `backend/app/routers/executions.py`
- Create: `backend/app/routers/reports.py`
- Create: `backend/app/routers/apks.py`
- Create: `backend/app/routers/llm_configs.py`
- Create: `backend/app/routers/ai.py`
- Create: `backend/app/routers/dashboard.py`
- Modify: `backend/app/schemas/__init__.py`

- [ ] **Step 1: Write `backend/app/schemas/project.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class ProjectCreate(BaseModel):
    name: str
    description: str = ""


class ProjectUpdate(BaseModel):
    name: str | None = None
    description: str | None = None


class ProjectResponse(BaseModel):
    id: int
    name: str
    description: str
    created_at: datetime
    updated_at: datetime
    script_count: int = 0

    model_config = {"from_attributes": True}
```

- [ ] **Step 2: Write `backend/app/schemas/script.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class ScriptCreate(BaseModel):
    name: str
    description: str = ""
    content: str = ""
    project_id: int
    target_os: str = "Android"


class ScriptUpdate(BaseModel):
    name: str | None = None
    description: str | None = None
    content: str | None = None
    target_os: str | None = None


class ScriptResponse(BaseModel):
    id: int
    name: str
    description: str
    content: str
    project_id: int
    target_os: str
    file_path: str
    created_at: datetime
    updated_at: datetime

    model_config = {"from_attributes": True}
```

- [ ] **Step 3: Write `backend/app/schemas/device.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class DeviceResponse(BaseModel):
    id: int
    serial: str
    model: str
    brand: str
    os_type: str
    android_version: str
    ip_address: str
    connection_type: str
    status: str
    resolution: str
    last_seen_at: datetime

    model_config = {"from_attributes": True}
```

- [ ] **Step 4: Write `backend/app/schemas/execution.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class ExecutionCreate(BaseModel):
    script_id: int
    device_id: int


class ExecutionResponse(BaseModel):
    id: int
    script_id: int
    device_id: int
    status: str
    log_path: str
    report_path: str
    started_at: datetime | None
    finished_at: datetime | None
    result: str
    error_message: str

    model_config = {"from_attributes": True}
```

- [ ] **Step 5: Write `backend/app/schemas/llm_config.py`**

```python
from pydantic import BaseModel


class LlmConfigCreate(BaseModel):
    provider: str
    model: str
    api_key: str = ""
    api_base: str = ""
    temperature: float = 0.7


class LlmConfigUpdate(BaseModel):
    provider: str | None = None
    model: str | None = None
    api_key: str | None = None
    api_base: str | None = None
    temperature: float | None = None


class LlmConfigResponse(BaseModel):
    id: int
    provider: str
    model: str
    api_key: str
    api_base: str
    temperature: float
    is_active: bool

    model_config = {"from_attributes": True}
```

- [ ] **Step 6: Write `backend/app/schemas/apk.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class ApkResponse(BaseModel):
    id: int
    filename: str
    file_path: str
    version: str
    package_name: str
    size: int
    uploaded_at: datetime

    model_config = {"from_attributes": True}
```

- [ ] **Step 7: Write `backend/app/schemas/report.py`**

```python
from pydantic import BaseModel
from datetime import datetime


class ReportResponse(BaseModel):
    id: int
    execution_id: int
    title: str
    content: str
    summary: str
    screenshots: str
    created_at: datetime

    model_config = {"from_attributes": True}
```

- [ ] **Step 8: Update `backend/app/schemas/__init__.py`**

```python
from app.schemas.project import ProjectCreate, ProjectUpdate, ProjectResponse
from app.schemas.script import ScriptCreate, ScriptUpdate, ScriptResponse
from app.schemas.device import DeviceResponse
from app.schemas.execution import ExecutionCreate, ExecutionResponse
from app.schemas.llm_config import LlmConfigCreate, LlmConfigUpdate, LlmConfigResponse
from app.schemas.apk import ApkResponse
from app.schemas.report import ReportResponse

__all__ = [
    "ProjectCreate", "ProjectUpdate", "ProjectResponse",
    "ScriptCreate", "ScriptUpdate", "ScriptResponse",
    "DeviceResponse",
    "ExecutionCreate", "ExecutionResponse",
    "LlmConfigCreate", "LlmConfigUpdate", "LlmConfigResponse",
    "ApkResponse",
    "ReportResponse",
]
```

- [ ] **Step 9: Write all placeholder routers**

Each router returns placeholder responses. Here's `projects.py` as template — all others follow the same pattern:

`backend/app/routers/projects.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List projects")
async def list_projects():
    return {"message": "Projects list endpoint - implementation pending", "data": []}


@router.post("/", summary="Create project")
async def create_project():
    return {"message": "Create project endpoint - implementation pending"}


@router.get("/{project_id}", summary="Get project detail")
async def get_project(project_id: int):
    return {"message": f"Get project {project_id} endpoint - implementation pending"}


@router.put("/{project_id}", summary="Update project")
async def update_project(project_id: int):
    return {"message": f"Update project {project_id} endpoint - implementation pending"}


@router.delete("/{project_id}", summary="Delete project")
async def delete_project(project_id: int):
    return {"message": f"Delete project {project_id} endpoint - implementation pending"}
```

`backend/app/routers/scripts.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List scripts")
async def list_scripts():
    return {"message": "Scripts list endpoint - implementation pending", "data": []}


@router.post("/", summary="Create script")
async def create_script():
    return {"message": "Create script endpoint - implementation pending"}


@router.get("/{script_id}", summary="Get script detail")
async def get_script(script_id: int):
    return {"message": f"Get script {script_id} endpoint - implementation pending"}


@router.put("/{script_id}", summary="Update script")
async def update_script(script_id: int):
    return {"message": f"Update script {script_id} endpoint - implementation pending"}


@router.delete("/{script_id}", summary="Delete script")
async def delete_script(script_id: int):
    return {"message": f"Delete script {script_id} endpoint - implementation pending"}


@router.post("/upload", summary="Upload script file")
async def upload_script():
    return {"message": "Upload script endpoint - implementation pending"}


@router.get("/{script_id}/download", summary="Download script file")
async def download_script(script_id: int):
    return {"message": f"Download script {script_id} endpoint - implementation pending"}
```

`backend/app/routers/devices.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List devices")
async def list_devices():
    return {"message": "Devices list endpoint - implementation pending", "data": []}


@router.post("/scan", summary="Scan for new devices")
async def scan_devices():
    return {"message": "Scan devices endpoint - implementation pending"}


@router.get("/{device_id}", summary="Get device detail")
async def get_device(device_id: int):
    return {"message": f"Get device {device_id} endpoint - implementation pending"}


@router.delete("/{device_id}", summary="Delete device")
async def delete_device(device_id: int):
    return {"message": f"Delete device {device_id} endpoint - implementation pending"}
```

`backend/app/routers/executions.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.post("/", summary="Create execution")
async def create_execution():
    return {"message": "Create execution endpoint - implementation pending"}


@router.get("/", summary="List executions")
async def list_executions():
    return {"message": "Executions list endpoint - implementation pending", "data": []}


@router.get("/{execution_id}", summary="Get execution detail")
async def get_execution(execution_id: int):
    return {"message": f"Get execution {execution_id} endpoint - implementation pending"}
```

`backend/app/routers/reports.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List reports")
async def list_reports():
    return {"message": "Reports list endpoint - implementation pending", "data": []}


@router.get("/{report_id}", summary="Get report detail")
async def get_report(report_id: int):
    return {"message": f"Get report {report_id} endpoint - implementation pending"}


@router.get("/{report_id}/html", summary="Get report HTML content")
async def get_report_html(report_id: int):
    return {"message": f"Get report {report_id} HTML endpoint - implementation pending"}
```

`backend/app/routers/apks.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List APKs")
async def list_apks():
    return {"message": "APKs list endpoint - implementation pending", "data": []}


@router.post("/upload", summary="Upload APK")
async def upload_apk():
    return {"message": "Upload APK endpoint - implementation pending"}


@router.delete("/{apk_id}", summary="Delete APK")
async def delete_apk(apk_id: int):
    return {"message": f"Delete APK {apk_id} endpoint - implementation pending"}


@router.post("/{apk_id}/install/{device_id}", summary="Install APK to device")
async def install_apk(apk_id: int, device_id: int):
    return {"message": f"Install APK {apk_id} to device {device_id} endpoint - implementation pending"}
```

`backend/app/routers/llm_configs.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/", summary="List LLM configs")
async def list_llm_configs():
    return {"message": "LLM configs list endpoint - implementation pending", "data": []}


@router.post("/", summary="Create LLM config")
async def create_llm_config():
    return {"message": "Create LLM config endpoint - implementation pending"}


@router.put("/{config_id}", summary="Update LLM config")
async def update_llm_config(config_id: int):
    return {"message": f"Update LLM config {config_id} endpoint - implementation pending"}


@router.delete("/{config_id}", summary="Delete LLM config")
async def delete_llm_config(config_id: int):
    return {"message": f"Delete LLM config {config_id} endpoint - implementation pending"}


@router.put("/{config_id}/activate", summary="Activate LLM config")
async def activate_llm_config(config_id: int):
    return {"message": f"Activate LLM config {config_id} endpoint - implementation pending"}
```

`backend/app/routers/ai.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.post("/generate", summary="Submit AI script generation request")
async def generate_script():
    return {"message": "AI generate endpoint - implementation pending"}


@router.get("/tasks/{task_id}", summary="Get AI generation task status")
async def get_task_status(task_id: str):
    return {"message": f"AI task {task_id} status endpoint - implementation pending"}
```

`backend/app/routers/dashboard.py`:
```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/stats", summary="Get dashboard statistics")
async def get_stats():
    return {
        "message": "Dashboard stats endpoint - implementation pending",
        "data": {
            "project_count": 0,
            "script_count": 0,
            "device_count": 0,
            "execution_count": 0,
            "success_rate": 0.0,
        },
    }
```

- [ ] **Step 10: Verify all routers appear in OpenAPI docs**

Run: `cd backend && python -m uvicorn app.main:app --host 0.0.0.0 --port 8000`
Visit `http://localhost:8000/docs`
Expected: All 9 router groups visible, each with placeholder endpoints. `/api/health` returns `{"status": "ok"}`.

- [ ] **Step 11: Commit schemas and routers**

```bash
cd C:\pythonworkspace\DroidBot
git add backend/app/schemas/ backend/app/routers/
git commit -m "feat(P0): add Pydantic schemas and placeholder API routers for all resources"
```

---

## Task 4: Vue3 + Vite Frontend Skeleton

**Files:**
- Create: `frontend/package.json`
- Create: `frontend/vite.config.js`
- Create: `frontend/index.html`
- Create: `frontend/src/main.js`
- Create: `frontend/src/App.vue`
- Create: `frontend/src/router/index.js`
- Create: `frontend/src/stores/index.js`
- Create: `frontend/src/stores/project.js`
- Create: `frontend/src/stores/script.js`
- Create: `frontend/src/stores/device.js`
- Create: `frontend/src/stores/execution.js`
- Create: `frontend/src/api/index.js`
- Create: `frontend/src/styles/variables.css`

- [ ] **Step 1: Create frontend directory and initialize Vue3 project**

```bash
cd C:\pythonworkspace\DroidBot
npm create vite@latest frontend -- --template vue
cd frontend
npm install
```

- [ ] **Step 2: Install additional dependencies**

```bash
cd frontend
npm install vue-router@4 pinia axios
```

- [ ] **Step 3: Write `frontend/vite.config.js`**

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
      '/ws': {
        target: 'ws://localhost:8000',
        ws: true,
      },
    },
  },
})
```

- [ ] **Step 4: Write `frontend/src/styles/variables.css`**

```css
:root {
  /* Backgrounds */
  --bg-primary: #0a0e17;
  --bg-secondary: #111827;
  --bg-card: #1a2332;
  --bg-hover: #1e293b;

  /* Borders */
  --border-color: rgba(255, 255, 255, 0.1);
  --border-radius: 8px;
  --border-radius-lg: 12px;

  /* Text */
  --text-primary: #ffffff;
  --text-secondary: #9ca3af;
  --text-muted: #6b7280;

  /* Accent gradient (purple) */
  --primary-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  --primary-color: #667eea;
  --primary-hover: #764ba2;

  /* Status colors */
  --success: #10b981;
  --success-bg: rgba(16, 185, 129, 0.15);
  --warning: #f59e0b;
  --warning-bg: rgba(245, 158, 11, 0.15);
  --error: #ef4444;
  --error-bg: rgba(239, 68, 68, 0.15);
  --info: #0ea5e9;
  --info-bg: rgba(14, 165, 233, 0.15);

  /* Layout */
  --sidebar-width: 260px;
  --sidebar-padding: 24px;
  --content-padding: 24px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.4);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.5);

  /* Transitions */
  --transition-fast: 0.15s ease;
  --transition-normal: 0.3s ease;
}
```

- [ ] **Step 5: Write `frontend/src/api/index.js`**

```javascript
import axios from 'axios'

const api = axios.create({
  baseURL: '/api',
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Response interceptor — unwrap data and handle errors
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    const message = error.response?.data?.detail || error.message || 'Unknown error'
    console.error('[API Error]', message)
    return Promise.reject(new Error(message))
  }
)

export default api
```

- [ ] **Step 6: Write `frontend/src/stores/index.js`**

```javascript
import { createPinia } from 'pinia'

export const pinia = createPinia()
```

- [ ] **Step 7: Write `frontend/src/stores/project.js`**

```javascript
import { defineStore } from 'pinia'
import api from '../api'

export const useProjectStore = defineStore('project', {
  state: () => ({
    projects: [],
    currentProject: null,
    loading: false,
  }),
  actions: {
    async fetchProjects() {
      this.loading = true
      try {
        const res = await api.get('/projects')
        this.projects = res.data || []
      } finally {
        this.loading = false
      }
    },
  },
})
```

- [ ] **Step 8: Write `frontend/src/stores/script.js`**

```javascript
import { defineStore } from 'pinia'
import api from '../api'

export const useScriptStore = defineStore('script', {
  state: () => ({
    scripts: [],
    currentScript: null,
    loading: false,
  }),
  actions: {
    async fetchScripts(projectId) {
      this.loading = true
      try {
        const res = await api.get(`/projects/${projectId}/scripts`)
        this.scripts = res.data || []
      } finally {
        this.loading = false
      }
    },
  },
})
```

- [ ] **Step 9: Write `frontend/src/stores/device.js`**

```javascript
import { defineStore } from 'pinia'
import api from '../api'

export const useDeviceStore = defineStore('device', {
  state: () => ({
    devices: [],
    loading: false,
  }),
  actions: {
    async fetchDevices() {
      this.loading = true
      try {
        const res = await api.get('/devices')
        this.devices = res.data || []
      } finally {
        this.loading = false
      }
    },
  },
})
```

- [ ] **Step 10: Write `frontend/src/stores/execution.js`**

```javascript
import { defineStore } from 'pinia'
import api from '../api'

export const useExecutionStore = defineStore('execution', {
  state: () => ({
    executions: [],
    currentExecution: null,
    loading: false,
  }),
  actions: {
    async fetchExecutions() {
      this.loading = true
      try {
        const res = await api.get('/executions')
        this.executions = res.data || []
      } finally {
        this.loading = false
      }
    },
  },
})
```

- [ ] **Step 11: Write `frontend/src/router/index.js`**

```javascript
import { createRouter, createWebHistory } from 'vue-router'

// Placeholder views — will be implemented in later phases
const PlaceholderPage = () => import('../components/PlaceholderPage.vue')

const routes = [
  { path: '/', name: 'dashboard', component: PlaceholderPage, meta: { title: 'Dashboard' } },
  { path: '/projects', name: 'projects', component: PlaceholderPage, meta: { title: '项目管理' } },
  { path: '/projects/:id/scripts', name: 'project-scripts', component: PlaceholderPage, meta: { title: '脚本管理' } },
  { path: '/scripts/:id/edit', name: 'script-edit', component: PlaceholderPage, meta: { title: '脚本编辑' } },
  { path: '/scripts/:id/detail', name: 'script-detail', component: PlaceholderPage, meta: { title: '脚本详情' } },
  { path: '/ai-script', name: 'ai-script', component: PlaceholderPage, meta: { title: 'AI脚本生成' } },
  { path: '/execution/new', name: 'execution-new', component: PlaceholderPage, meta: { title: '新建执行' } },
  { path: '/execution/:id', name: 'execution-detail', component: PlaceholderPage, meta: { title: '执行监控' } },
  { path: '/devices', name: 'devices', component: PlaceholderPage, meta: { title: '设备管理' } },
  { path: '/reports', name: 'reports', component: PlaceholderPage, meta: { title: '报告管理' } },
  { path: '/reports/:id', name: 'report-detail', component: PlaceholderPage, meta: { title: '报告详情' } },
  { path: '/apk-manager', name: 'apk-manager', component: PlaceholderPage, meta: { title: 'APK管理' } },
  { path: '/settings/llm', name: 'llm-config', component: PlaceholderPage, meta: { title: 'LLM配置' } },
  { path: '/settings/general', name: 'general-settings', component: PlaceholderPage, meta: { title: '系统设置' } },
]

const router = createRouter({
  history: createWebHistory(),
  routes,
})

router.beforeEach((to) => {
  document.title = `${to.meta.title || 'DroidBot'} - DroidBot`
})

export default router
```

- [ ] **Step 12: Write `frontend/src/main.js`**

```javascript
import { createApp } from 'vue'
import { pinia } from './stores'
import router from './router'
import App from './App.vue'
import './styles/variables.css'

const app = createApp(App)
app.use(pinia)
app.use(router)
app.mount('#app')
```

- [ ] **Step 13: Verify frontend starts**

Run: `cd frontend && npm run dev`
Expected: Vite dev server starts on `http://localhost:5173`, no errors.

- [ ] **Step 14: Commit frontend skeleton**

```bash
cd C:\pythonworkspace\DroidBot
git add frontend/
git commit -m "feat(P0): add Vue3+Vite frontend skeleton with Pinia, Router, Axios, and dark theme variables"
```

---

## Task 5: AppLayout + Sidebar + Placeholder Components

**Files:**
- Create: `frontend/src/layouts/AppLayout.vue`
- Create: `frontend/src/components/Sidebar.vue`
- Create: `frontend/src/components/PlaceholderPage.vue`
- Modify: `frontend/src/App.vue`

- [ ] **Step 1: Write `frontend/src/components/Sidebar.vue`**

```vue
<template>
  <aside class="sidebar">
    <div class="logo">
      <h1>DroidBot</h1>
      <p>AI 驱动 Android 兼容性测试平台</p>
    </div>

    <nav class="nav-menu">
      <router-link to="/" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">📊</span>
        <span class="nav-text">Dashboard</span>
      </router-link>

      <router-link to="/projects" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">📁</span>
        <span class="nav-text">项目管理</span>
      </router-link>

      <router-link to="/ai-script" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">🤖</span>
        <span class="nav-text">AI脚本生成</span>
      </router-link>

      <router-link to="/execution/new" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">⚡</span>
        <span class="nav-text">脚本执行</span>
      </router-link>

      <router-link to="/devices" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">📱</span>
        <span class="nav-text">设备管理</span>
      </router-link>

      <router-link to="/reports" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">📋</span>
        <span class="nav-text">报告管理</span>
      </router-link>

      <router-link to="/apk-manager" class="nav-item" active-class="nav-item--active">
        <span class="nav-icon">📦</span>
        <span class="nav-text">APK管理</span>
      </router-link>

      <div class="nav-section-title">⚙️ 系统设置</div>

      <router-link to="/settings/llm" class="nav-item nav-item--sub" active-class="nav-item--active">
        <span class="nav-text">LLM配置</span>
      </router-link>

      <router-link to="/settings/general" class="nav-item nav-item--sub" active-class="nav-item--active">
        <span class="nav-text">通用设置</span>
      </router-link>
    </nav>
  </aside>
</template>

<style scoped>
.sidebar {
  width: var(--sidebar-width);
  background: var(--bg-secondary);
  border-right: 1px solid var(--border-color);
  padding: var(--sidebar-padding);
  position: fixed;
  height: 100vh;
  overflow-y: auto;
}

.logo {
  margin-bottom: 32px;
}

.logo h1 {
  font-size: 1.25rem;
  font-weight: 700;
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.logo p {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 4px;
}

.nav-menu {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 14px;
  color: var(--text-secondary);
  text-decoration: none;
  border-radius: var(--border-radius);
  transition: all var(--transition-fast);
  font-size: 14px;
}

.nav-item:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}

.nav-item--active {
  background: var(--bg-hover);
  color: var(--text-primary);
  font-weight: 600;
}

.nav-item--sub {
  padding: 10px 14px 10px 40px;
  font-size: 13px;
}

.nav-icon {
  font-size: 16px;
  width: 20px;
  text-align: center;
}

.nav-section-title {
  padding: 16px 14px 8px;
  color: var(--text-muted);
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
}
</style>
```

- [ ] **Step 2: Write `frontend/src/components/PlaceholderPage.vue`**

```vue
<template>
  <div class="placeholder-page">
    <div class="placeholder-card">
      <div class="placeholder-icon">🚧</div>
      <h2>{{ title }}</h2>
      <p>此页面功能将在后续版本中实现</p>
      <div class="placeholder-phase">规划阶段：{{ phaseLabel }}</div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

const title = computed(() => route.meta.title || '页面')

const phaseMap = {
  dashboard: 'P6',
  projects: 'P1',
  'project-scripts': 'P1',
  'script-edit': 'P1',
  'script-detail': 'P1',
  'ai-script': 'P4',
  'execution-new': 'P2',
  'execution-detail': 'P2',
  devices: 'P3',
  reports: 'P5',
  'report-detail': 'P5',
  'apk-manager': 'P6',
  'llm-config': 'P4/P6',
  'general-settings': 'P6',
}

const phaseLabel = computed(() => phaseMap[route.name] || '待定')
</script>

<style scoped>
.placeholder-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
}

.placeholder-card {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius-lg);
  padding: 48px 64px;
  text-align: center;
}

.placeholder-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.placeholder-card h2 {
  font-size: 20px;
  color: var(--text-primary);
  margin-bottom: 8px;
}

.placeholder-card p {
  color: var(--text-secondary);
  font-size: 14px;
}

.placeholder-phase {
  margin-top: 16px;
  padding: 8px 16px;
  background: var(--info-bg);
  color: var(--info);
  border-radius: var(--border-radius);
  font-size: 13px;
  font-weight: 600;
}
</style>
```

- [ ] **Step 3: Write `frontend/src/layouts/AppLayout.vue`**

```vue
<template>
  <div class="app-container">
    <Sidebar />
    <main class="main-content">
      <router-view />
    </main>
  </div>
</template>

<script setup>
import Sidebar from '../components/Sidebar.vue'
</script>

<style scoped>
.app-container {
  display: flex;
  min-height: 100vh;
}

.main-content {
  margin-left: var(--sidebar-width);
  padding: var(--content-padding);
  width: calc(100% - var(--sidebar-width));
  min-height: 100vh;
  background: var(--bg-primary);
}
</style>
```

- [ ] **Step 4: Write `frontend/src/App.vue`**

```vue
<template>
  <AppLayout />
</template>

<script setup>
import AppLayout from './layouts/AppLayout.vue'
</script>

<style>
/* Global resets — variables.css already loaded via main.js */
body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: var(--bg-primary);
  color: var(--text-primary);
  margin: 0;
  padding: 0;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
```

- [ ] **Step 5: Verify frontend renders correctly**

Run: `cd frontend && npm run dev`
Visit `http://localhost:5173`
Expected: Sidebar with navigation links visible on the left, placeholder card "Dashboard 🚧" on the right. Clicking any nav item shows corresponding placeholder with phase label.

- [ ] **Step 6: Verify proxy to backend works**

Ensure backend is also running (`cd backend && python -m uvicorn app.main:app --port 8000`).
Visit `http://localhost:5173/api/health` through browser.
Expected: Returns `{"status": "ok", "version": "0.1.0"}` — confirming Vite proxy forwards to FastAPI.

- [ ] **Step 7: Commit layout and components**

```bash
cd C:\pythonworkspace\DroidBot
git add frontend/src/layouts/ frontend/src/components/ frontend/src/App.vue
git commit -m "feat(P0): add AppLayout with sidebar navigation and placeholder pages for all routes"
```

---

## Task 6: P0 Integration Verification + Final Commit

**Files:**
- Modify: `frontend/index.html` (ensure proper meta tags)

- [ ] **Step 1: Update `frontend/index.html` to set language and title**

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <title>DroidBot - AI驱动Android兼容性测试平台</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

- [ ] **Step 2: Full integration test — start both servers**

```bash
# Terminal 1: Backend
cd backend && python -m uvicorn app.main:app --host 0.0.0.0 --port 8000

# Terminal 2: Frontend
cd frontend && npm run dev
```

Visit `http://localhost:5173`:
- Expected: Sidebar visible with 9 nav items + 2 sub-items
- Click "项目管理" → placeholder shows "项目管理 🚧 规划阶段：P1"
- Click "设备管理" → placeholder shows "设备管理 🚧 规划阶段：P3"
- Visit `http://localhost:8000/docs` → OpenAPI UI with all 9 router groups
- Visit `http://localhost:8000/api/health` → `{"status": "ok"}`

- [ ] **Step 3: Commit final P0 polish**

```bash
cd C:\pythonworkspace\DroidBot
git add frontend/index.html
git commit -m "feat(P0): set Chinese language and title in frontend index.html"
```

- [ ] **Step 4: P0 complete — verify clean state**

```bash
cd C:\pythonworkspace\DroidBot
git log --oneline -10
git status
```

Expected: 4-5 commits for P0, clean working tree.

---

## P0 Verification Checklist

| # | Check | How to verify |
|---|-------|---------------|
| 1 | Backend starts without errors | `cd backend && python -m uvicorn app.main:app` |
| 2 | SQLite DB auto-created on startup | `ls backend/data/droidbot.db` |
| 3 | OpenAPI docs accessible | Visit `http://localhost:8000/docs` |
| 4 | Health check responds | `curl http://localhost:8000/api/health` |
| 5 | All placeholder routers respond | Click each endpoint in `/docs` |
| 6 | Frontend starts without errors | `cd frontend && npm run dev` |
| 7 | Sidebar renders with all nav items | Visit `http://localhost:5173` |
| 8 | Placeholder pages show per-route titles | Click each sidebar link |
| 9 | Vite proxy forwards API requests | `curl http://localhost:5173/api/health` |
| 10 | CSS variables apply dark theme | Visual: dark background, purple gradient logo |