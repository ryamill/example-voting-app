# version is now using "compose spec"
# v2 and v3 are now combined!
# docker-compose v1.27+ required

services:
  vote:
    build:
      context: ./vote
      # CHANGE 1: Target the final_production stage for the vote app
      target: final_production
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"] # Note: curl might not be in final_production image
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      # This volume mount will override the contents of /usr/local/app
      # with your host's ./vote directory. This is fine for local development
      # but in a true production scenario, you'd typically bake the app code
      # directly into the image in the final stage without a volume mount.
      - ./vote:/usr/local/app
    ports:
      - "8080:80"
    networks:
      - front-tier
      - back-tier

  result:
    build: ./result
    # use nodemon rather than node for local dev
    entrypoint: nodemon --inspect=0.0.0.0 server.js
    depends_on:
      db:
        condition: service_healthy
      # CHANGE 2: Add dependency on worker to ensure DB schema is created
      worker:
        condition: service_started # Wait for worker to start, assuming it creates the table on startup
    volumes:
      - ./result:/usr/local/app
    ports:
      - "8081:80"
      - "127.0.0.1:9229:9229"
    networks:
      - front-tier
      - back-tier

  worker:
    build:
      context: ./worker
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_healthy
    networks:
      - back-tier

  redis:
    image: redis:alpine
    volumes:
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/redis.sh
      interval: "5s"
    networks:
      - back-tier

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    volumes:
      - "db-data:/var/lib/postgresql/data"
      - "./healthchecks:/healthchecks"
      # IMPORTANT: If your original app had SQL init scripts, they would go here.
      # For example: - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
      # However, the example-voting-app relies on the worker to create the table.
    healthcheck:
      test: /healthchecks/postgres.sh
      interval: "5s"
    networks:
      - back-tier

  # this service runs once to seed the database with votes
  # it won't run unless you specify the "seed" profile
  # docker compose --profile seed up -d
  seed:
    build: ./seed-data
    profiles: ["seed"]
    depends_on:
      vote:
        condition: service_healthy
    networks:
      - front-tier
    restart: "no"

volumes:
  db-data:

networks:
  front-tier:
  back-tier: