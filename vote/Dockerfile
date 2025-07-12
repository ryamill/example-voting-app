# --- Stage 1: Builder ---
FROM cgr.dev/chainguard/python:latest-dev AS builder
WORKDIR /usr/local/app
USER root
# Install curl for healthcheck in the builder, it will not be in the final image
RUN apk update && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

# --- Stage 2: Final Production Image ---
FROM cgr.dev/chainguard/python:latest AS final_production
WORKDIR /usr/local/app
# Copy installed Python packages from the builder stage
COPY --from=builder /usr/lib/python3.12/site-packages /usr/lib/python3.12/site-packages
# Copy application code from the builder stage
COPY --from=builder /usr/local/app .
EXPOSE 80
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]

# --- Stage 3 (Optional): Development Image ---
# This stage is for local development with live code reloading.
# If you use 'target: dev' in docker-compose, this CMD will be used.
FROM builder AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
CMD ["python", "app.py"]