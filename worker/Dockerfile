# ... (rest of your comments) ...

# build compiles the program for the builder's local platform
# Change to Chainguard .NET SDK image
# Removed --platform=${BUILDPLATFORM} from here to fix the parsing error
FROM cgr.dev/chainguard/dotnet-sdk:7.0 AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

WORKDIR /source
COPY *.csproj .
RUN dotnet restore -a $TARGETARCH

COPY . .
RUN dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

# app image
# Removed --platform=${BUILDPLATFORM} from here (if it was implicitly there or if you add it)
FROM cgr.dev/chainguard/dotnet-runtime:7.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]