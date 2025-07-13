# because of dotnet, we always build on amd64, and target platforms in cli
# dotnet doesn't support QEMU for building or running.
# (errors common in arm/v7 32bit) https://github.com/dotnet/dotnet-docker/issues/1537
# https://hub.docker.com/_/microsoft-dotnet
# hadolint ignore=DL3029
# to build for a different platform than your host, use --platform=<platform>
# for example, if you were on Intel (amd64) and wanted to build for ARM, you would use:
# docker buildx build --platform "linux/arm64/v8" .

# build compiles the program for the builder's local platform

# Change to Chainguard .NET SDK image, using the :latest tag
FROM cgr.dev/chainguard/dotnet-sdk:latest AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
#RUN ["/bin/bash", "-c", "echo \"I am running on $BUILDPLATFORM, building for $TARGETPLATFORM\""]

WORKDIR /source
COPY *.csproj .
RUN ["dotnet", "restore", "-r", "linux-x64"]

COPY . .
# MODIFIED: Publish output to a subdirectory within WORKDIR /source (e.g., './app-output')
RUN ["dotnet", "publish", "-c", "release", "-o", "./app-output", "--use-current-runtime", "--self-contained", "false", "--no-restore"]

# app image
FROM cgr.dev/chainguard/dotnet-runtime:latest
WORKDIR /app
# MODIFIED: Copy from the new output location in the build stage
COPY --from=build /source/app-output .
ENTRYPOINT ["dotnet", "Worker.dll"]