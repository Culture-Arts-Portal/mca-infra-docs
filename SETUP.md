# Setup Guide – Culture & Arts e-Services Portal

Follow these steps to run the project on your own laptop.
Run commands **one at a time** and stop if you get an error.

## 1. Install these first
- [Git](https://git-scm.com/downloads)
- [Node.js LTS](https://nodejs.org)
- [VS Code](https://code.visualstudio.com)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)

**Windows users:** Docker needs WSL. Before installing Docker, open
PowerShell **as administrator**, run `wsl --install`, then restart
your computer.

## 2. Create a projects folder
Open a **normal** PowerShell (NOT as administrator):

```
cd ~\Desktop
mkdir projects
cd projects
```

## 3. Clone the repos

```
git clone https://github.com/Culture-Arts-Portal/mca-infra-docs.git
git clone https://github.com/Culture-Arts-Portal/mca-backend.git
git clone https://github.com/Culture-Arts-Portal/mca-frontend.git
```

## 4. Start the database
Open Docker Desktop first and wait for **Engine running**. Then:

```
cd mca-infra-docs
cp .env.example .env
docker compose up -d
docker ps
```

You should see `mca-postgres` and `mca-adminer` running.

## 5. Check the database in Adminer
Open http://localhost:8080 and log in:

| Field    | Value        |
|----------|--------------|
| System   | PostgreSQL   |
| Server   | db           |
| Username | mca_user     |
| Password | mca_password |
| Database | mca_portal   |

## 6. Backend and frontend
Coming soon – these steps will be added when the backend and
frontend skeletons are ready.

## Useful Docker commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start the database |
| `docker compose stop` | Stop it (data is kept) |
| `docker ps` | See what's running |
| `docker compose down -v` | Delete everything and start fresh (**data is lost**) |

## Common problems

| Error | Fix |
|---|---|
| `Permission denied` when cloning | You're in `C:\WINDOWS\system32`. Open a normal PowerShell and `cd ~\Desktop\projects` |
| `Could not resolve host: github.com` | No internet connection. Reconnect and try again |
| `no configuration file provided` | You're in the wrong folder. `cd` into `mca-infra-docs` first |
| `Cannot connect to the Docker daemon` | Open Docker Desktop and wait for **Engine running** |
| `Port 5432 is already allocated` | PostgreSQL is already installed. In your `.env`, set `POSTGRES_PORT=5433` |
