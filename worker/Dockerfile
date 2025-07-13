# ... (rest of your comments) ...

# build compiles the program for the builder's local platform
# Change to Chainguard .NET SDK image, using the :latest tag
FROM cgr.dev/chainguard/dotnet-sdk:latest AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
#RUN ["/bin/bash", "-c", "echo \"I am running on $BUILDPLATFORM, building for $TARGETPLATFORM\""]

WORKDIR /source
COPY *.csproj .
# MODIFIED: Tell dotnet restore to also target linux-x64 runtime
RUN ["dotnet", "restore", "-r", "linux-x64"]

COPY . .
# This line remains correct for publishing for the current runtime (linux-x64)
RUN ["dotnet", "publish", "-c", "release", "-o", "/app", "--use-current-runtime", "--self-contained", "false", "--no-restore"]

# app image
# Change to Chainguard .NET Runtime image, using the :latest tag
FROM cgr.dev/chainguard/dotnet-runtime:latest
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]