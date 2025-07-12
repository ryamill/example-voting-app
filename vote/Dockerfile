# --- Stage 1: Builder ---
# This stage uses the 'latest-dev' image because it contains a shell (apk)
# and development tools needed to install dependencies like pip.
FROM cgr.dev/chainguard/python:latest-dev AS builder

# Set the application directory for the build process
WORKDIR /usr/local/app

# Switch to root user to install system packages with apk
USER root
# Install build-time dependencies and your application's requirements.
# Use 'apk' for system packages (like curl, if truly needed for a script here)
# and 'pip' for Python packages.
RUN apk update && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

# It's good practice to switch back to a non-root user if one exists in the base image
# and subsequent commands don't require root. For Chainguard images, a common
# non-root user or UID might be 'nonroot' or '65532'. However, for simplicity
# in this builder stage, and since pip install often works as root, we can proceed
# without explicitly changing back here. The important part is that the final image
# will inherently run as a non-root user unless you explicitly set it to root.

# Copy requirements and install them
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of your application code to the builder stage
COPY . .

# --- Stage 2: Final Production Image ---
# This stage uses the *truly minimal*, shell-less Chainguard Python runtime.
# It only copies the essential artifacts from the 'builder' stage.
FROM cgr.dev/chainguard/python:latest AS final_production

# Set the application directory in the final stage
WORKDIR /usr/local/app

# Copy only the installed Python packages from the builder stage.
# The path for site-packages in Chainguard Python images is typically /usr/lib/pythonX.Y/site-packages.
# Adjust the 'python3.12' part if your Python version is different.
COPY --from=builder /usr/lib/python3.12/site-packages /usr/lib/python3.12/site-packages

# Copy your application source code from the builder stage
COPY --from=builder /usr/local/app .

# Make port 80 available for links and/or publish
EXPOSE 80

# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]

# --- Stage 3 (Optional): Development Image ---
# If you still want a 'dev' image for local development, it should be based
# on the 'builder' stage (or a specific dev image) but is NOT used as a base
# for your 'final_production' image. This keeps your dev environment separate.
FROM builder AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
ENTRYPOINT ["sh"]
CMD ["-c", "python app.py"]