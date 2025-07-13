# --- Stage 1: Build ---
# This stage uses the Chainguard .NET SDK image to build your application.
# It includes the necessary tools for 'dotnet restore' and 'dotnet publish'.
FROM --platform=${BUILDPLATFORM} cgr.dev/chainguard/dotnet-sdk:latest AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
# Removed: RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

WORKDIR /source
COPY *.csproj .
# Already modified: Removed "-a", "$TARGETARCH" from dotnet restore
RUN ["dotnet", "restore"]

COPY . .
# Modified: Removed "-a", "$TARGETARCH" from dotnet publish
RUN ["dotnet", "publish", "-c", "release", "-o", "/app", "--self-contained", "false", "--no-restore"]

# --- Stage 2: App Image (Final Production Image) ---
# This stage uses the minimal Chainguard .NET Runtime image.
# It only copies the compiled application from the 'build' stage.
FROM cgr.dev/chainguard/dotnet-runtime:latest
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]