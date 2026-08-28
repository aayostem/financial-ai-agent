FROM python:3.11-slim AS builder

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        build-essential \
        libpq-dev \
        && apt-get clean && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR /app

# Copy ONLY pyproject.toml (remove README.md)
COPY pyproject.toml ./

# Create virtual environment and install dependencies
RUN python -m venv /app/venv && \
    /app/venv/bin/pip install --upgrade pip setuptools wheel && \
    /app/venv/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu --no-cache-dir

# Install dependencies from pyproject.toml without installing the package itself
RUN /app/venv/bin/pip install $(/app/venv/bin/python -c "import tomllib; print(' '.join(tomllib.load(open('pyproject.toml','rb'))['project']['dependencies']))") --no-cache-dir --timeout 1000

# Copy source code
COPY src/ ./src/

# ── Stage 2: production ───────────────────────────────────────────────────────
FROM python:3.11-slim-bookworm AS production

WORKDIR /app

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libpq5 \
        curl \
        ca-certificates \
        && apt-get clean && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd -r finrag && useradd -r -g finrag finrag && \
    mkdir -p /app/data && \
    chown -R finrag:finrag /app

# Copy virtual environment and source code from builder
COPY --from=builder --chown=finrag:finrag /app/venv /app/venv
COPY --from=builder --chown=finrag:finrag /app/src /app/src

ENV PATH="/app/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH="/app/src" \
    TZ="UTC"

RUN find /app -type f -name "*.pyc" -delete && \
    find /app -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true

USER finrag

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1

EXPOSE 8000

CMD ["uvicorn", "financial_ai.api.server:app", \
     "--host", "0.0.0.0", \
     "--port", "8000", \
     "--workers", "2", \
     "--loop", "asyncio", \
     "--access-log"]