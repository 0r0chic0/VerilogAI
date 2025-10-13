FROM python:3.14

ENV PYTHONUNBUFFERED=1

WORKDIR /app/

# Install uv
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#installing-uv
COPY --from=ghcr.io/astral-sh/uv:0.5.11 /uv /uvx /bin/

# Place executables in the environment at the front of the path
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#using-the-environment
ENV PATH="/app/.venv/bin:$PATH"

# Compile bytecode
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#compiling-bytecode
ENV UV_COMPILE_BYTECODE=1

# uv Cache
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#caching
ENV UV_LINK_MODE=copy

# Install dependencies
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#intermediate-layers
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-project

ENV PYTHONPATH=/app

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      build-essential autoconf gperf bison flex git \
      meson ninja-build desktop-file-utils pkg-config \
      libgtk-3-dev libgtk-4-dev libbz2-dev libjudy-dev libgirepository1.0-dev verilator \
 && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/steveicarus/iverilog.git /app/iverilog \
 && cd /app/iverilog \
 && sh autoconf.sh \
 && ./configure --prefix=/usr/local \
 && make -j"$(nproc)" \
 && make install \
 && rm -rf /app/iverilog


RUN git clone https://github.com/gtkwave/gtkwave.git /app/gtkwave \
 && cd /app/gtkwave \
 && meson setup build --prefix=/usr/local \
 && cd build \
 && meson install \
 && rm -rf /app/gtkwave

COPY ./scripts /app/scripts

COPY ./pyproject.toml ./uv.lock ./alembic.ini /app/

COPY ./app /app/app

# ─── NEW LINES TO ADD ───────────────────────────────────────────────────────────
# 1) Copy your service-account key into the image
#    (make sure you have a folder "keys" next to this Dockerfile, containing service-account.json)
COPY ./keys/service-account.json /app/keys/service-account.json

# 2) Point ADC to the copied JSON
ENV GOOGLE_APPLICATION_CREDENTIALS="/app/keys/service-account.json"
# ────────────────────────────────────────────────────────────────────────────────

# Sync the project
# Ref: https://docs.astral.sh/uv/guides/integration/docker/#intermediate-layers
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync

CMD ["fastapi", "run", "--workers", "4", "app/main.py"]
