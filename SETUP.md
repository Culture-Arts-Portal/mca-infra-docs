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
Open a **normal** PowerShell (NOT as administrator).

We use a short folder at the top of the C: drive. Don't use the Desktop:
OneDrive often moves it, and then `cd ~\Desktop` fails.

```
mkdir C:\Culture-Arts-Portal
cd C:\Culture-Arts-Portal
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
```

### 4a. Is PostgreSQL already installed on your PC?
Check before you start the container:

```
Get-Service *postgres*
```

- **Nothing listed:** keep `POSTGRES_PORT=5432` in `.env`.
- **A service is listed** (for example `postgresql-x64-16`): it already uses port 5432.
  Open `.env` and change the last line to `POSTGRES_PORT=5433`.

**Why this matters:** if both use 5432, the backend talks to the PostgreSQL installed on
Windows instead of our container. Our `mca_user` doesn't exist there, so Prisma fails with
`P1000: Authentication failed`.

**Use the same port in the backend.** The backend's `.env` has a `DATABASE_URL`, and its port must match
`POSTGRES_PORT`:

```
DATABASE_URL="postgresql://mca_user:mca_password@localhost:5432/mca_portal?schema=public"
```

(With `POSTGRES_PORT=5433`, write `localhost:5433` instead.)

### 4b. Start it

If you ran an older version of this guide, remove the old container first. If it says
`No such container`, that's fine:

```
docker rm -f mca-postgres
```

Then:

```
docker compose up -d
docker ps
```

You should see `mca-postgres` and `mca-adminer` running. After a few seconds `mca-postgres`
shows `(healthy)`, which means the database is ready to accept connections. Adminer waits for
that before it starts.

## 5. Check the database in Adminer
Open http://localhost:8080 and log in:

| Field    | Value        |
|----------|--------------|
| System   | PostgreSQL   |
| Server   | db           |
| Username | mca_user     |
| Password | mca_password |
| Database | mca_portal   |

Server is `db` (the name inside Docker), whatever your `POSTGRES_PORT` is.

## 6. Backend and frontend

### Backend

```powershell
cd mca-backend
npm install
Copy-Item .env.example .env
```

Open `.env` and check the port in `DATABASE_URL`: it must match your `POSTGRES_PORT`
(5432, or 5433 if you changed it in step 4a). Then:

```powershell
npx prisma migrate dev
npx prisma db seed
npm run dev
```

In a second terminal, check that it works:

```powershell
curl.exe http://localhost:5000/api/services
```

You should see `"success":true` and a list of services.

### Frontend

Coming soon (Person 4).

## Useful Docker commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start the database |
| `docker compose stop` | Stop it (data is kept) |
| `docker ps` | See what's running, and whether the database is `(healthy)` |
| `docker compose down -v` | Delete everything and start fresh (**data is lost**) |

## Common problems

| Error | Fix |
|---|---|
| `Permission denied` when cloning | You're in `C:\WINDOWS\system32`. Open a normal PowerShell and `cd C:\Culture-Arts-Portal` |
| `Cannot find path '...\Desktop'` | OneDrive moved your Desktop. Use `C:\Culture-Arts-Portal` (step 2) |
| `Could not resolve host: github.com` | No internet connection. Reconnect and try again |
| `no configuration file provided` | You're in the wrong folder. `cd` into `mca-infra-docs` first |
| `Cannot connect to the Docker daemon` | Open Docker Desktop and wait for **Engine running** |
| `Port 5432 is already allocated` | PostgreSQL is already installed. Set `POSTGRES_PORT=5433` in `.env` (step 4a), then `docker compose up -d` |
| `P1000: Authentication failed` from Prisma | The backend reached the Windows PostgreSQL. Make the port in the backend's `DATABASE_URL` match `POSTGRES_PORT` (step 4a) |
| `container name "/mca-postgres" is already in use` | Run `docker rm -f mca-postgres`, then `docker compose up -d` |
