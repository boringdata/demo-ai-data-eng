# An example using multi-stage image builds to create a final image without uv.

# First, build the application in the `/app` directory.
# See `Dockerfile` for details.
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim AS builder
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy

# Disable Python downloads, because we want to use the system interpreter
# across both images. If using a managed Python version, it needs to be
# copied from the build image into the final image; see `standalone.Dockerfile`
# for an example.
ENV UV_PYTHON_DOWNLOADS=0

WORKDIR /app
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-project --no-dev
ADD . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev


# Then, use a final image without uv
FROM python:3.12-slim-bookworm

# Copy the application from the builder
COPY --from=builder --chown=app:app /app /app

# vLLM requires some basic compilation tools
RUN apt-get update && apt-get install -y build-essential

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"

# Executable package. Check pyproject.toml
CMD ["run-batch-job"]