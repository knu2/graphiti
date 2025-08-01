# syntax=docker/dockerfile:1.9
FROM python:3.12-slim

WORKDIR /app

# Install system dependencies as root
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user and set ownership of /app
RUN groupadd -r app && useradd -r -d /app -g app app && chown app:app /app

# Switch to app user
USER app

# Install uv using pip
RUN pip install --user uv

# Add uv to PATH
ENV PATH="/app/.local/bin:${PATH}"

# Configure uv for optimal Docker usage
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    MCP_SERVER_HOST="0.0.0.0" \
    PYTHONUNBUFFERED=1

# Copy project files for dependency installation from mcp_server/
COPY --chown=app:app mcp_server/pyproject.toml mcp_server/uv.lock ./

# Install dependencies with cache mount
RUN mkdir -p /app/.cache/uv && chmod -R u+rw /app/.cache
RUN --mount=type=cache,id=s/9a6e0da9-4875-4d16-99b9-338347609f5f-/app/.cache/uv,target=/app/.cache/uv \
    uv sync --frozen --no-dev

# Copy application code
COPY --chown=app:app mcp_server/graphiti_mcp_server.py ./

# Expose port
EXPOSE 8000

# Command to run the application
CMD ["uv", "run", "graphiti_mcp_server.py", "--transport", "sse"]