ARG PYTHON_VERSION=3.11

# Stage 1: сборка зависимостей
FROM python:${PYTHON_VERSION} AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
COPY . .

# Stage 2: финальный образ
FROM python:${PYTHON_VERSION}-slim AS run
WORKDIR /app
ENV PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y --no-install-recommends libmariadb3 \
    && rm -rf /var/lib/apt/lists/*
COPY --from=builder /install /usr/local
COPY --from=builder /app .
CMD ["sh", "-c", "python manage.py migrate && python manage.py runserver 0.0.0.0:8080"]