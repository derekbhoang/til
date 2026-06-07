# Alembic

The standard database migration tool for Python applications that use SQLAlchemy.

It helps you manage changes to your database schema over time in a controlled, versioned way.

## Workflow

### 1. Install Alembic + SQLAlchemy

```bash
pip install sqlalchemy alembic
```

### 2. Initialize Alembic

```bash
alembic init migrations
```

This creates:

- `migrations/` folder (migration scripts live here)
- `alembic.ini` (configuration file)
- `env.py` (connects Alembic to your app)

## 3. Set the database URL

In `alembic.ini`:

```ini
sqlalchemy.url = postgresql+psycopg://user:password@localhost:5432/mydb
```

For real apps, prefer env vars inside `migrations/env.py`.

### 4. Create app models

`app/models.py`:

```py
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import String

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
```

### 5. Connect Alembic to your models

Edit `migrations/env.py`:

```py
from app.models import Base

target_metadata = Base.metadata
```

This enables autogeneration, where Alembic compares your models to the database. Alembic’s docs note autogenerate is useful but must be reviewed manually.

### 6. Create your first migration

```bash
alembic revision --autogenerate -m "create users table"
```

Alembic creates a file in:

`migrations/versions/`

Example:

`migrations/versions/e1n3n0e9l_version_name`

Review it before running it.

### 7. Apply the migration

```bash
alembic upgrade head
```

Your database now has the users table.

### 8. Check current migration state

```bash
alembic current
```

See history:

```bash
alembic history
```

### 9. Make a schema change

Change the model:

```py
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
    name: Mapped[str | None] = mapped_column(String(100))
```

Generate migration:

```bash
alembic revision --autogenerate -m "add user name"
```

Apply it:

```bash
alembic upgrade head
```

### 10. Downgrade / upgrade

Rollback one migration:

```bash
alembic downgrade -1
```

Rollback to base:

```bash
alembic downgrade base
```

Rollback/upgrade to a specific revision:

```bash
alembic upgrade <revision_id>
alembic downgrade <revision_id>
```

## Resources:

[Alembic Introduction - Migrations and Auto-Generating Revisions from SQLAlchemy Models](https://www.youtube.com/watch?v=i9RX03zFDHU)