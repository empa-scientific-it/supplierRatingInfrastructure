# Supplier Rating Software - Infrastructure (Local openBIS)

This repository provides a **local openBIS instance** (via Docker Compose) for developing the
Supplier Rating Software on your own machine, and documents how to run the **full stack**
(openBIS + Backend + Frontend) locally.

It is the counterpart to the [Backend](https://github.com/empa-scientific-it/supplierRatingBackend)
and [Frontend](https://github.com/empa-scientific-it/supplierRatingFrontend) repositories.

## 1. What's in this repository

- `docker-compose.yml` – starts a local openBIS stack: the application server (`openbis-app`),
  a PostgreSQL database (`openbis-db`), and an Nginx ingress (`openbis-ingress`) for HTTPS access.
- `.env` – selects the openBIS version that is started (see [Section 3.1](#31-choosing-the-openbis-version)).
- `certs/` – self-signed certificates used by the ingress.

## 2. Prerequisites

- Docker
- Docker Compose

## 3. Start the local openBIS

From this repository, run:

```bash
docker compose up -d
```

This starts the openBIS version set in `.env` (default: **20.10.13.1**, the current production version).
openBIS takes a minute or two to fully start. On the first start of a version, Docker also downloads
the image, which takes a few more minutes.

### 3.1. Choosing the openBIS version

The openBIS version is set in the `.env` file:

```bash
OPENBIS_VERSION=20.10.13.1
```

| Value        | Matches                                                  |
|--------------|----------------------------------------------------------|
| `20.10.13.1` | Current Empa production (lab303) – **default**           |
| `7.0.0`      | Workshop instance (dev308) and all Empa instances from 2027 |

The value is used for the Docker image (`openbis/openbis-app:<version>`) and for the names of the
Docker volumes (e.g. `openbis-db-data-20.10.13.1`). **Each version therefore has its own data.**
Switching the version does not delete anything: the data of the other version stays in its volumes
and is available again when you switch back.

To switch the version:

1. Stop the running stack: `docker compose down`
2. Change `OPENBIS_VERSION` in `.env`.
3. Check the result (optional): `docker compose config | grep -E "image: openbis/openbis-app|name: openbis-db"`
4. Start again: `docker compose up -d`

> **Note:** Run only one version at a time – all versions use the same container names and ports.
> A version that is started for the first time is empty and needs the one-time configuration
> described in [Section 5](#5-one-time-openbis-configuration).

> **Backend setting:** The schema files in the Backend repository define the name property as `NAME`.
> For both local versions, set `OPENBIS_NAME_PROPERTY=NAME` in the Backend's `.env.development`.
> (The production instance lab303 uses the internal property `$NAME` instead.)

## 4. Access openBIS

Log in with the openBIS admin account: user `admin`, password `123456789` (configurable via the `OPENBIS_ADMIN_PASS` environment variable in `docker-compose.yml`; it only takes effect on a fresh initialization).

The URLs differ slightly between the versions:

| UI                               | openBIS 20.10.13.1                                    | openBIS 7.0.0                         |
|----------------------------------|-------------------------------------------------------|---------------------------------------|
| Entry page                       | http://localhost:9090/openbis/ (classic login page)   | **http://localhost:9090/openbis/**    |
| ELN (Lab Notebook & Inventory)   | **http://localhost:9090/openbis/webapp/eln-lims/**    | via the entry page                    |
| Admin UI (import, PATs)          | **http://localhost:9090/openbis/webapp/openbis-ng-ui/** | via the entry page                  |

The stack exposes the following ports:

| Port      | Purpose                                       |
|-----------|-----------------------------------------------|
| 443 / 80  | openBIS UI via the Nginx ingress (HTTPS/HTTP) |
| 9090      | Direct browser access to `openbis-app`        |
| 8081      | `openbis-app` without ingress                 |
| 8085      | Data Store Server (DSS)                        |

## 5. One-time openBIS configuration

The first time you set up a fresh openBIS instance, you need to import the data model and create
the storage hierarchy:

1. **Import the object types (schema):** The `.xlsx` schema files live in the
   [Backend repository](https://github.com/empa-scientific-it/supplierRatingBackend) under
   `openbis-schema/`. Import them via the openBIS Admin UI (**TOOLS → Import → All**).
2. **Create the storage hierarchy** (Space, Projects, Collections) with the required codes.

The Backend README's *"How to configure openBIS"* section lists the exact steps and the required
Space, Project, and Collection codes.

## 6. Personal Access Tokens (PAT)

The Supplier Rating Software authenticates users with openBIS **Personal Access Tokens**. PAT support
is already configured in `docker-compose.yml`:

```yaml
- personal-access-tokens-enabled=true
- personal-access-tokens-max-validity-period=31536000   # 365 days in seconds
- personal-access-tokens-file-path=/data/openbis/personal-access-tokens.json
```

> **Why the dashes matter:** openBIS reads these settings **only** in the hyphenated form
> (`personal-access-tokens-...`). The dotted form (e.g. `personal-access-tokens.max-validity-period`)
> is ignored, and openBIS falls back to its default maximum validity of **30 days**. Keep the
> hyphenated form so that tokens with up to one year of validity can be created.

### Create a PAT

1. Open openBIS and log in as `admin`.
2. Go to the profile menu (top right) → **Personal Access Tokens**.
3. Create a new token with a validity of up to **1 year** and copy it.
4. Use this token together with your openBIS username to log in to the Supplier Rating Frontend.

### Persistence

PATs are stored in the openBIS **database** (volume `openbis-db-data-<version>`, e.g.
`openbis-db-data-20.10.13.1`). A PAT is only valid for the version it was created in.
PATs survive both container restarts and a full `docker compose down` / `up`. They are only lost if
you delete the volumes (see [Section 9](#9-stopping-and-data)).

## 7. Important configuration notes

- **Configure openBIS only through `docker-compose.yml`.** On every start, openBIS regenerates its
  internal `service.properties` from the environment variables in this file. Editing that file inside
  the running container has no lasting effect — it is overwritten on the next restart.
- **Use the hyphenated property form** (see Section 6). The dotted form is silently ignored.
- **Volume names contain the openBIS version.** Changing `OPENBIS_VERSION` switches to a different set
  of data — your previous data is not lost (see [Section 3.1](#31-choosing-the-openbis-version)).

## 8. Running the full stack locally

Run the three parts in this order:

1. **Start openBIS** (this repository):

   ```bash
   docker compose up -d
   ```

   Once it is up, open openBIS (URLs see [Section 4](#4-access-openbis)) and log in as `admin` / `123456789`.

2. **Configure openBIS and create a PAT** (first time only — see Sections 5 and 6).

3. **Start the Backend** — clone and configure the
   [Backend repository](https://github.com/empa-scientific-it/supplierRatingBackend) (see its README),
   then build and start it:

   ```bash
   docker compose up -d --build
   ```

   It joins openBIS's shared Docker network (`openbis-shared-network`) and is available at `http://localhost:8080`.

4. **Start the Frontend** — install the dependencies and start the dev server in the
   [Frontend repository](https://github.com/empa-scientific-it/supplierRatingFrontend):

   ```bash
   npm install
   npm run start:dev
   ```

   It runs at `http://localhost:4200`.

5. Open `http://localhost:4200` and log in with your openBIS username and PAT.

6. **When you're finished**, stop the Frontend dev server with `Ctrl+C`, then shut down the Docker stacks
   so openBIS shuts down cleanly. In both the Backend and the infrastructure folder, run:

   ```bash
   docker compose stop
   ```

   `docker compose stop` keeps the containers so they restart quickly; `docker compose down` removes them
   instead — your data is preserved either way. Next time, bring everything back up with
   `docker compose up -d`. See [Section 9](#9-stopping-and-data) for fully removing the containers or
   resetting all data.

## 9. Stopping and data

- `docker compose down` — stops and removes the containers but **keeps** the volumes. Your openBIS
  data and PATs are preserved.
- `docker compose down -v` — additionally **deletes** the volumes of the version currently set in
  `.env`. **All openBIS data and PATs of this version are lost.** Use this only for a clean reset.

## 10. License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file f