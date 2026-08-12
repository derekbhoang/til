# Docker

Docker is an open-source platform that lets developers package applications and their dependencies into lightweight, portable units called containers.

Containers can run consistently across different environments, such as a developer's laptop, a test server, or a production cloud environment.

## Why Docker is useful

Without Docker:

- An application may work on one machine but fail on another because of differences in operating systems, libraries, or configurations.

"It works on my machine" problem.

With Docker:

- The application and everything it needs are bundled together.
- The same container can run anywhere Docker is installed.
- Deployment becomes faster and more reliable.

## Key concepts

### 1. Dockerfile

A Dockerfile is a plain text file that contains instructions for building a Docker image.

Think of it as a recipe that tells Docker:

- Which base operating system or image to start from
- What software to install
- Which files to copy into the image
- Which commands to run during setup
- What command should execute when the container starts

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

#### What each instruction does

| Instruction  | Purpose                                       |
| ------------ | --------------------------------------------- |
| `FROM`       | Specifies the base image                      |
| `WORKDIR`    | Sets the working directory                    |
| `COPY`       | Copies files into the image                   |
| `RUN`        | Executes commands during image build          |
| `ENV`        | Sets environment variables                    |
| `EXPOSE`     | Documents which port the app uses             |
| `CMD`        | Default command when the container starts     |
| `ENTRYPOINT` | Defines the main executable for the container |


### 2. Image

A Docker image is a read-only template used to create containers.

It contains everything needed to run an application:

- Application code
- Dependencies
- Libraries
- Runtime (Node, Python, Java, etc.)
- Configuration

Think of it as a blueprint.

An image might contain:

```
+ Python 3.12
+ FastAPI
+ Application code
```

#### Create a Docker Image

This is my app:

```
my-app/
├── app.py
├── Dockerfile
└── requirements.txt
```

`app.py`:

```python
print("Hello from Docker!")
```

`Dockerfile`:

Each line is a layer.

```dockerfile
FROM python:3.12                    # Base image

WORKDIR /app                        # Create empty /app if not exist

COPY requirements.txt .             # Copy requirements.txt to /app (".")

RUN pip install -r requirements.txt # Docker executes this inside /app, so it looks for: /app/requirements.txt

COPY . .                            # Copy current dir to /app

CMD ["python", "app.py"]            # Startup command
```

Build a Docker Image by running:

```bash
docker build -t my-image .
```

**Layer 1**

**Purpose:** Defines the base image.

```dockerfile
FROM python:3.12
```

Docker starts from the official Python 3.12 image.

Includes:

- Linux operating system (typically Debian-based unless another tag is used)
- Python 3.12 interpreter
- `pip`
- Standard Python runtime files

```
Linux OS
+ Python 3.12
```

**Layer 2**

```dockerfile
WORKDIR /app
```

Sets the working directory inside the container.

- If `/app` doesn't exist, Docker creates it.
- Future commands run from `/app`.

After layer 2, we have:

```
/
└── app/
```

**Layer 3**

Copies only requirements.txt into the image.

After copying, we have:

```
/
└── app/
    └── requirements.txt
```

**Layer 4**

Installs Python dependencies.

Docker executes:

```bash
pip install -r requirements.txt
```

Packages get installed into the image, then we have:

```
/
└── usr
    └── local
        └── lib
            └── python3.12
                └── site-packages
                    ├── fastapi
                    ├── uvicorn
                    └── ...
```

**Layer 5**

```dockerfile
COPY app.py .
```

Copies the rest of the application code.

```
/
└── app/
    ├── app.py
    ├── Dockerfile
    └── requirements.txt
```

**Layer 6**

```dockerfile
CMD ["python", "app.py"]
```

Defines the default command when a container starts.

When run:

```bash
docker run my-image
```

Docker executes:

```bash
python app.py
```

#### Check Image

```bash
docker images
```

Output:

```
REPOSITORY   TAG       IMAGE ID
my-love      latest    i3o92oo6
```

### 3. Container

A Docker container is a lightweight, portable, and isolated environment that packages an application together with everything it needs to run:

- Application code
- Runtime (e.g., Java, Python, Node.js)
- Libraries and dependencies
- Configuration files

#### Create & Start the Container

```bash
docker run --name my-container my-image
```

Docker performs the following:

**Step 1 — Create a writable layer**

Docker does not modify the image.

Instead, it adds a new writable layer on top:

```
┌────────────────────────┐
│ Writable Layer         │  ← Container layer
├────────────────────────┤
│ Layer 6 (CMD)          │
│ Layer 5 (app.py)       │
│ Layer 4 (Packages)     │
│ Layer 3 (requirements) │
│ Layer 2 (/app)         │
│ Layer 1 (Python 3.12)  │
└────────────────────────┘
```

The image remains read-only.

**Step 2 — Allocate Container Resources**

Docker creates:

- Container ID
- Network interface
- Filesystem view
- Process namespace

Example:

```
Container ID: 1oi0oo1ii01

Filesystem:
/
├── bin/
├── etc/
├── usr/
│   └── local/
│       └── lib/
│           └── python3.12/
│               └── site-packages/
│                   ├── fastapi/
│                   └── ...
└── app/
    ├── app.py
    ├── requirements.txt
    └── Dockerfile
```

**Step 3 — Execute CMD**

Docker runs:

```bash
python app.py
```

Inside the container:

```
Container Process
└── python app.py
```

Output:

```
Hello from Docker!
```