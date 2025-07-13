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
# Modified line 13: Remove '-a "$TARGETARCH"'
RUN ["dotnet", "restore"]

COPY . .
# Modified line 16: Use '--use-current-runtime' instead of '-a "$TARGETARCH"'
RUN ["dotnet", "publish", "-c", "release", "-o", "/app", "--use-current-runtime", "--self-contained", "false", "--no-restore"]

# app image
# Change to Chainguard .NET Runtime image, using the :latest tag
FROM cgr.dev/chainguard/dotnet-runtime:latest
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]