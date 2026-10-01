
Local Development Guide
This guide explains how to run and manage the local services used by the Event Ticketing System.

Local architecture
During early development:

The web, API, and worker applications run directly on Windows using pnpm.
PostgreSQL and Redis run inside Linux containers managed by Docker Desktop.
PostgreSQL is available on localhost:5432.
Redis is available on localhost:6379.
Docker Compose reads compose.yaml from the repository root and manages both services as one project.

Essential commands
Validate the configuration
docker compose config
Reads compose.yaml, resolves environment variables, validates it, and prints the final configuration. It does not start containers.

Start services
docker compose up -d
up creates and starts the services. -d runs them in the background.

Check status
docker compose ps
PostgreSQL and Redis should eventually report healthy.

Test PostgreSQL
docker compose exec postgres psql -U ticketing -d ticketing -c "SELECT current_database(), current_user;"
Test Redis
docker compose exec redis redis-cli ping
A working Redis service responds with PONG.

Logs
View all logs:

docker compose logs
Follow new logs continuously:

docker compose logs -f
Follow one service:

docker compose logs -f postgres
docker compose logs -f redis
Press Ctrl+C to stop following logs. This does not stop the containers.

Stop, start, and remove services
Stop services while preserving containers and data:

docker compose stop
Start previously stopped containers:

docker compose start
Stop and remove containers and their default network:

docker compose down
Named volumes remain, so PostgreSQL and Redis data is preserved. Recreate the containers with docker compose up -d.

Delete all local service data
docker compose down -v
Warning: -v deletes the named PostgreSQL and Redis volumes. Use it only when you intentionally want a clean local environment.

Other useful commands
Restart every service:

docker compose restart
Restart one service:

docker compose restart postgres
Run a command inside a container:

docker compose exec <service></service> <command></command>
Examples:

docker compose exec postgres sh
docker compose exec redis redis-cli
View live resource usage:

docker stats
Pull newer configured images:

docker compose pull
docker compose up -d
Do not change major PostgreSQL versions without reviewing the database upgrade process.

Daily workflow
Start infrastructure:

docker compose up -d
docker compose ps
Start the applications after they have been initialized:

pnpm dev
Pause infrastructure when finished:

docker compose stop
Configuration files
compose.yaml describes services, ports, volumes, and health checks.
.env.example documents safe example variables and is committed to Git.
.env contains local values and must not be committed.
Confirm that .env is ignored:

git check-ignore .env
Common troubleshooting
Docker daemon is unavailable
Start Docker Desktop, wait for its engine, and run:

docker version
Port 5432 or 6379 is already in use
Stop the conflicting local service, or change POSTGRES_PORT or REDIS_PORT in .env and update the corresponding application connection URL.

A service is unhealthy
docker compose ps
docker compose logs postgres
docker compose logs redis
Configuration changes are not reflected
docker compose up -d --force-recreate
Start with an empty database
Only when data loss is intentional:

docker compose down -v
docker compose up -d
Command summary
Command	Purpose	Deletes data?
docker compose config	Validate and print resolved configuration	No
docker compose up -d	Create and start services in the background	No
docker compose ps	Show service and health status	No
docker compose logs -f	Follow service logs	No
docker compose stop	Stop services while preserving containers	No
docker compose start	Start stopped containers	No
docker compose restart	Restart services	No
docker compose down	Remove containers and the default network	No; named volumes remain
docker compose down -v	Remove containers, network, and named volumes	Yes
docker compose exec	Run a command inside a service	No
docker compose pull	Download newer configured images	No
