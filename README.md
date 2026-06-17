# Dev Stack (MySQL + Mailpit + Adminer)

A standalone Docker Compose stack that gives you MySQL, Mailpit (mail catcher),
and Adminer (DB admin UI) for local development — independent of any
specific app. PHP, Composer, Node, and your Laravel app(s) all stay on your
host machine as usual; this just replaces the "infrastructure" containers
that Sail would normally spin up.

You can keep this folder anywhere on your machine (e.g. `~/dev-stack`) and
point every project at it, or copy it next to whichever project needs it.

## Folder structure

```
docker-dev/
├── compose.yaml        # the 3 services: mysql, mailpit, adminer
├── .env                # actual values used by compose (gitignored)
├── .env.example        # template to copy from
├── mysql/
│   └── init/           # optional: drop .sql or .sh files here to run
│                        # once on first container creation (e.g. extra
│                        # database creation scripts)
└── README.md
```

## Prerequisites

- Docker Desktop (or Docker Engine + Compose plugin) installed and running.

## First-time setup

```bash
cd docker-dev
cp .env.example .env   # already done for you, just edit values if needed
```

Edit `.env` if you want different credentials or ports (useful if 3306,
1025, 8025, or 8080 are already used by something else on your machine).

## Everyday commands

Run these from inside the `docker-dev` folder (where `compose.yaml` lives).

```bash
# start everything in the background
docker compose up -d

# check status
docker compose ps

# tail logs (all services, or one)
docker compose logs -f
docker compose logs -f mysql

# stop containers (keeps data volume)
docker compose down

# stop AND delete the MySQL data volume (full reset)
docker compose down -v

# restart a single service
docker compose restart mysql

# rebuild/pull latest images
docker compose pull
```

## Connecting from your Laravel app(s)

Since your app runs natively (`php artisan serve`, `composer`, `npm run dev`,
etc.) and only the services run in Docker, point your app's `.env` at
`127.0.0.1` using the ports from this stack's `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306              # matches FORWARD_DB_PORT
DB_DATABASE=app           # matches DB_DATABASE
DB_USERNAME=sail          # matches DB_USERNAME
DB_PASSWORD=password      # matches DB_PASSWORD

MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025             # matches FORWARD_MAILPIT_PORT
MAIL_ENCRYPTION=null
```

Then open the dashboards in your browser:

- Adminer (DB admin): http://localhost:8080 — server `mysql`... actually
  use `127.0.0.1` as the server name since you're connecting from outside
  the Docker network (your browser, not a container).
- Mailpit (caught emails): http://localhost:8025

> Note: `ADMINER_DEFAULT_SERVER: mysql` only pre-fills the login form. From
> your browser, the server field can stay as `127.0.0.1`/`mysql` and it
> will still resolve correctly because Adminer itself runs inside the same
> Docker network as the `mysql` container.

## Using multiple databases per project

You don't need a separate stack per project. Just create another database
inside the same MySQL container:

```bash
docker compose exec mysql mysql -uroot -p"${DB_PASSWORD}" \
  -e "CREATE DATABASE IF NOT EXISTS another_app;"
```

Or drop a `.sql`/`.sh` file into `mysql/init/` before the *first* `docker
compose up` (init scripts only run once, when the data volume is first
created).

## Notes

- Data persists in the named volume `devstack-mysql` across `docker compose
  down` / `up` cycles. Only `down -v` wipes it.
- Container names (`devstack-mysql`, `devstack-mailpit`, `devstack-adminer`)
  are fixed so you always know what you're looking at in `docker ps`,
  regardless of which folder you ran compose from.
- `.env` is meant to be gitignored if you version-control this folder;
  commit `.env.example` instead.
