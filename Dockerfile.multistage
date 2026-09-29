# CP2 — Multi-stage production Dockerfile

# ─────────────────────────────────────────────
# Stage 1: Builder
# ─────────────────────────────────────────────
FROM python:3.11-slim AS builder

WORKDIR /app

# Copy requirements trước để tận dụng Docker cache
COPY requirements.txt .

# Cài dependencies vào thư mục riêng
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt


# ─────────────────────────────────────────────
# Stage 2: Runtime
# ─────────────────────────────────────────────
FROM python:3.11-slim

WORKDIR /app

# Copy dependencies từ builder
COPY --from=builder /install /usr/local

# Copy source code sau khi dependencies đã được cài
COPY . .

# Tạo user thường
RUN useradd --create-home --shell /bin/bash appuser

# Container chạy bằng user thường, không phải root
USER appuser

# Cloud sẽ cung cấp PORT qua environment variable
ENV PORT=8000

EXPOSE 8000

# Health check endpoint
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:' + __import__('os').environ.get('PORT', '8000') + '/health')" || exit 1

# Dùng PORT từ environment variable
CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}"]