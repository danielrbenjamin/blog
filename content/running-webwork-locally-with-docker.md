# Running WeBWorK locally with Docker to test a Chrome extension

A local WeBWorK server gives you a place to create assignments, submit answers, inspect the page, and test extension changes without touching a university course. I wanted this for [WeBWorKer](https://github.com/danielrbenjamin/WeBWorKer), my Chrome extension for WeBWorK.

UBC publishes a prebuilt image at [`lthub/webwork`](https://hub.docker.com/r/lthub/webwork). This guide uses that image with MariaDB, persistent storage, and local accounts. Chrome runs normally on Windows; Docker runs the server in Linux containers.

> **Verification:** This guide was checked against UBC's source at commit [`3d9992faa9c06a8a044197e100f3442a47763fca`](https://github.com/ubc/webwork2/tree/3d9992faa9c06a8a044197e100f3442a47763fca) on November 13, 2026. The corresponding `sha-3d9992f` image tag is listed on Docker Hub. The configuration and commands were reviewed against the source, but the complete Docker startup has not been executed as part of preparing this article.

## What is in the image?

UBC's [GitHub Actions workflow](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/.github/workflows/main.yml) builds [`Dockerfile-prod`](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/Dockerfile-prod) and publishes the result as `lthub/webwork`.

The Dockerfile uses several build stages. It downloads PG, WeBWorK's problem engine, and UBC's Open Problem Library fork; prepares the application and PG browser assets with npm; and copies them into an Ubuntu 24.04 image with Perl modules, TeX, and other dependencies. The WeBWorK application itself comes from the repository being built. The startup command runs the Mojolicious application through Hypnotoad.

The database is separate. Docker Compose connects the services and provides storage that survives replacing a container.

| Service | Purpose | Host access |
| --- | --- | --- |
| `app` | WeBWorK website | `http://localhost:8080/webwork2/` |
| `db` | MariaDB database | Docker network only |
| `r` | Rserve for problems that use R | Docker network only |
| `worker` | Minion background jobs | Docker network only |

UBC's example also includes database and email development interfaces and an LTI worker. Those are unnecessary for this local extension-testing setup.

## 1. Install the prerequisites

On Windows, install:

- [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) using its WSL 2 backend and Linux containers.
- [Git for Windows](https://git-scm.com/downloads/win).
- Google Chrome.

Follow Docker Desktop's instructions for enabling WSL 2 and virtualization if prompted. Start Docker Desktop, then open **PowerShell** and check:

```powershell
docker version
docker compose version
git --version
```

`docker version` should show both client and server information. If the server is unavailable, make sure Docker Desktop has finished starting.

The WeBWorK image is roughly 2.65 GB compressed, and Docker needs additional space for extracted files and persistent data. Leave several gigabytes free beyond the download size.

## 2. Get the matching configuration files

Create a working directory and clone the source snapshot used for this guide:

```powershell
mkdir webwork-local
cd webwork-local
git clone https://github.com/ubc/webwork2.git source
git -C source checkout --detach 3d9992faa9c06a8a044197e100f3442a47763fca
mkdir config
Copy-Item source/docker-config/db/mariadb.cnf config/mariadb.cnf
Copy-Item source/conf/localOverrides.conf.dist config/localOverrides.conf
```

We are using a prebuilt application image. The checkout supplies matching configuration files and makes the implementation easy to inspect; it is not mounted over the application code.

The MariaDB configuration enables `utf8mb4` before the database is initialized.

## 3. Enable local login

UBC's default overrides select external authentication for ordinary courses. For a standalone test server, append these settings to `config/localOverrides.conf`:

```powershell
@'

# Local development: use database-backed accounts in every course.
$authen{user_module} = 'WeBWorK::Authen::Basic_TheLastOption';
$authen{admin_module} = 'WeBWorK::Authen::Basic_TheLastOption';

# Local disposable accounts only: simplify repeated browser testing.
$twoFA{enabled} = 0;

# WeBWorK configuration includes must return a true value.
1;
'@ | Add-Content -Encoding ASCII config/localOverrides.conf
```

The single quotes around the PowerShell here-string delimiter preserve the Perl `$` variables exactly. These settings deliberately simplify login for a server bound to your own computer. They are not production authentication settings.

Keep `1;` as the final statement in this configuration file. WeBWorK requires included configuration files to return a true value; ending with `$twoFA{enabled} = 0;` returns false and stops startup.

## 4. Create the environment file

Run this once to generate `.env` with separate random values for the database credentials and application session secret:

```powershell
@(
  'WEBWORK_IMAGE=lthub/webwork:sha-3d9992f'
  'WEBWORK_DB_USER=webworkWrite'
  "WEBWORK_DB_PASSWORD=$([guid]::NewGuid().ToString('N'))"
  "WEBWORK_MYSQL_ROOT_PASSWORD=$([guid]::NewGuid().ToString('N'))"
  "WEBWORK_SECRET=$([guid]::NewGuid().ToString('N'))$([guid]::NewGuid().ToString('N'))"
) | Set-Content -Encoding ASCII .env
```

Keep `.env` private. If you commit your setup to Git, add it to `.gitignore`:

```gitignore
.env
```

Do not regenerate `.env` on each startup. MariaDB only creates the initial database user and password when its data directory is empty; changing the file later does not change an existing database password.

The commit tag keeps this guide tied to a known build. For stronger reproducibility, use an image digest as described below.

## 5. Create `compose.yaml`

Save this file in `webwork-local`, beside `.env`:

```yaml
name: webwork-local

x-webwork-environment: &webwork-environment
  WEBWORK_URL: /webwork2
  WEBWORK_ROOT_URL: http://localhost:8080
  WEBWORK_DB_DRIVER: MariaDB
  WEBWORK_DB_HOST: db
  WEBWORK_DB_PORT: "3306"
  WEBWORK_DB_NAME: webwork
  WEBWORK_DB_USER: ${WEBWORK_DB_USER:?Set WEBWORK_DB_USER in .env}
  WEBWORK_DB_PASSWORD: ${WEBWORK_DB_PASSWORD:?Set WEBWORK_DB_PASSWORD in .env}
  WEBWORK_SECRET: ${WEBWORK_SECRET:?Set WEBWORK_SECRET in .env}
  WEBWORK_TIMEZONE: America/Vancouver
  SYSTEM_TIMEZONE: UTC
  ADD_APT_PACKAGES: "0"
  ADD_LOCALES: "0"
  SKIP_UPLOAD_OPL_STATISTICS: "1"
  MOJO_PUBSUB_EXPERIMENTAL: "1"
  DEBUG: "0"

x-webwork-volumes: &webwork-volumes
  - courses:/opt/webwork/courses
  - opl:/opt/webwork/libraries/webwork-open-problem-library
  - htdocs-data:/opt/webwork/webwork2/htdocs/DATA
  - ./config/localOverrides.conf:/opt/webwork/webwork2/conf/localOverrides.conf:ro

services:
  db:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: ${WEBWORK_MYSQL_ROOT_PASSWORD:?Set root password in .env}
      MYSQL_DATABASE: webwork
      MYSQL_USER: ${WEBWORK_DB_USER}
      MYSQL_PASSWORD: ${WEBWORK_DB_PASSWORD}
    volumes:
      - mysql:/var/lib/mysql
      - ./config/mariadb.cnf:/etc/mysql/conf.d/mariadb.cnf:ro
      - ./config/mariadb.cnf:/etc/mysql/mariadb.cnf:ro
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 30
      start_period: 30s

  r:
    image: ubcctlt/rserve

  app:
    image: ${WEBWORK_IMAGE}
    environment: *webwork-environment
    volumes: *webwork-volumes
    ports:
      - "127.0.0.1:8080:8080"
    depends_on:
      db:
        condition: service_healthy
      r:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "--fail", "--silent", "--output", "/dev/null", "http://127.0.0.1:8080/webwork2/"]
      interval: 15s
      timeout: 10s
      retries: 80
      start_period: 60s
    stop_grace_period: 15s

  worker:
    image: ${WEBWORK_IMAGE}
    environment: *webwork-environment
    volumes: *webwork-volumes
    # Wait for app initialization before running the worker entrypoint.
    command: ["sudo", "-E", "-u", "www-data", "bin/webwork2", "minion", "worker", "-m", "production"]
    depends_on:
      app:
        condition: service_healthy

volumes:
  mysql:
  courses:
  opl:
  htdocs-data:
```

The host binding `127.0.0.1:8080:8080` restricts access to your computer. There is no need to publish the database or Rserve ports.

**Use container port 8080.** Although `Dockerfile-prod` declares `EXPOSE 80`, the bundled Hypnotoad configuration listens on 8080. `EXPOSE` is metadata; it does not change where the application listens.

Named volumes keep the database, courses, problem library, and generated library metadata across container replacement. The first mount of a new named volume copies existing image content at that path into the volume. In particular, the image already contains the problem library.

## 6. Start WeBWorK

From `webwork-local`:

```powershell
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose logs -f app
```

The initial pull is large. First startup also creates the admin course, creates database tables, prepares the model course, and initializes problem-library metadata. Give it time to finish rather than repeatedly restarting it. Some initialization paths also need outbound internet access.

Press **Ctrl+C** to stop following the logs. The containers continue running because they were started with `-d`.

Check their status:

```powershell
docker compose ps --all
```

Once the app is healthy, open:

[http://localhost:8080/webwork2/admin/](http://localhost:8080/webwork2/admin/)

For a new database, the startup script creates:

| Field | Value |
| --- | --- |
| Username | `admin` |
| Password | `admin` |

Change the password after signing in. This is a local test installation with simplified authentication; keep it bound to loopback.

## 7. Create a test course

In the admin interface:

1. Open **Course Administration** and choose **Add Course**.
2. Use `extension_test` as the course ID.
3. Keep the `admin` user selected so it is copied into the new course.
4. Finish creating the course.
5. Open [http://localhost:8080/webwork2/extension_test/](http://localhost:8080/webwork2/extension_test/) and sign in with the copied account.

The copied account uses its password at the time of course creation. The admin course is for managing the server; create assignments in `extension_test`.

Inside the test course, use the **Classlist Editor** to add a normal student account, such as `student1`, and set its password. Use a separate Chrome profile or Incognito window for that student session. This avoids repeatedly switching between instructor and student accounts.

## 8. Add problems and make them available

Use the course's **Library Browser** to create a problem set, such as `extension_checks`, and add several problems from the Open Problem Library. Menu wording can vary slightly between releases.

Then use the **Hmwk Sets Editor** to:

1. Confirm the set contains the selected problems.
2. Make the set visible and set its open date in the past.
3. Set its due and answer dates in the future.
4. Assign the set to `student1` and any other accounts you want to test.

Log in as the student and confirm that a problem opens, an answer can be previewed, and a submission receives feedback. A running server alone does not establish that the course, dates, and assignments are configured correctly.

For an extension that interacts with answer inputs, include numerical answers, algebraic expressions, multiple answer boxes, and any other input types the extension supports. Test parentheses, long expressions, previews, incorrect submissions, and navigation between problems.

## 9. Load the extension locally

In the development copy of your extension, add localhost to the existing content-script match patterns:

```json
"matches": [
  "http://localhost/*"
]
```

This is a fragment, not a complete manifest. Add it to the appropriate existing `content_scripts` entry and retain your production patterns. Chrome's localhost match pattern covers all ports.

If the extension uses host permissions for operations that require them, also add localhost to that existing permission list. Declared content-script matching and host permissions serve different purposes; adding a host permission alone does not configure a content script to run.

Search the extension code for production hostnames or URL checks. A manifest match will not help if the script immediately exits because `location.hostname` is not a university domain.

Then:

1. Open `chrome://extensions`.
2. Enable **Developer mode**.
3. Choose **Load unpacked** and select the folder containing the extension's `manifest.json`. If the project has a build step, select its generated extension folder.
4. Open a local problem page.
5. Inspect the page with Chrome DevTools and check the Console for errors.

After changing the extension, rebuild if needed, reload it from `chrome://extensions`, and refresh the problem page so the new content script is injected. If testing in Incognito, enable **Allow in Incognito** for the extension first.

Use `localhost` consistently. `127.0.0.1` is a different hostname for URL matching and cookies; if you use it instead, update both the server root URL and extension patterns.

## Everyday commands

Start the existing installation:

```powershell
docker compose up -d
```

Stop it while retaining the containers and volumes:

```powershell
docker compose stop
```

Remove the containers and network while retaining named volumes:

```powershell
docker compose down
```

Apply changes to `.env` or `compose.yaml`:

```powershell
docker compose up -d
```

After editing the mounted `config/localOverrides.conf`, restart the app and worker so configuration is reloaded:

```powershell
docker compose restart app worker
```

Inspect logs:

```powershell
docker compose logs --tail 100 app worker db
```

## Troubleshooting

### `localOverrides.conf did not return a true value`

If the app exits with code 255 and this message, make sure the final statement in `config/localOverrides.conf` is `1;`. To repair a file created with an earlier version of this guide:

```powershell
Add-Content -Encoding ASCII config/localOverrides.conf "`n1;"
docker compose up -d
docker compose logs -f app
```

Keep the existing volumes. This configuration correction does not require deleting the database or downloading the images again.

### `Unknown option: db-layout`

The pinned image's startup script passes `--db-layout=sql_single` to `addcourse`, but that version of `addcourse` does not recognize the option. This message appeared alongside the configuration error above. Correct the configuration return value first, then inspect the next startup logs; successful initialization after that correction has not yet been confirmed.

### Docker cannot connect to its daemon

Start Docker Desktop and wait for its Linux engine to be ready. Check `docker version` again.

### Port 8080 is already in use

Change the published port to `127.0.0.1:8081:8080` and change `WEBWORK_ROOT_URL` to `http://localhost:8081`. Keep the internal health-check URL on port 8080. Run `docker compose up -d` and browse to port 8081.

### The app keeps waiting for the database

Inspect `docker compose logs db` and `docker compose ps --all`. Make sure `.env` exists and the database credentials match. If you changed passwords after the database volume was initialized, restore the original values or explicitly change the database account's password. Recreating the application container does not reset MariaDB credentials.

### Login asks for external authentication

Confirm the local override file is mounted and its final authentication settings use `Basic_TheLastOption`. Restart the app after editing it. A course-specific `course.conf` can override the global settings, so inspect that too if only one course is affected.

### The app is unhealthy during first startup

Read the app logs before resetting anything. Library metadata preparation can take time, and an external metadata download can fail. If it is still making progress, wait. If it exits, resolve the reported error and run `docker compose up -d` again. The health check can be inspected with:

```powershell
docker inspect --format '{{json .State.Health}}' (docker compose ps -q app)
```

### The extension does not run

Check its localhost match pattern, any hardcoded hostname checks, extension errors in `chrome://extensions`, and the page Console. Reload both the extension and the page. If two copies of the extension are installed, temporarily disable the store version while testing the unpacked copy.

### The student cannot see the assignment

Check that the set is assigned to that specific account, visible, and open. An instructor being able to open a problem does not prove students can access it.

### I want a completely fresh installation

The following command **deletes this Compose project's named volumes**, including accounts, course files, assignments, submissions, and the database:

```powershell
docker compose down --volumes
```

Only run it when that data is disposable. Then run `docker compose up -d` to initialize again. Your local configuration files and `.env` remain.

## Pinning and updating images

The guide uses a commit tag rather than `latest`. Tags can still be reassigned. To record the registry digest after pulling:

```powershell
docker image inspect lthub/webwork:sha-3d9992f --format '{{index .RepoDigests 0}}'
```

Copy the returned `lthub/webwork@sha256:...` value into `WEBWORK_IMAGE` in `.env`. For a fully fixed environment, pin the database and Rserve images by digest as well.

Treat an upgrade as a deliberate change: review the target version, back up the database and course files together, update the configuration from the matching source, and then recreate the containers. An existing problem-library volume retains its previous contents; pulling a new image does not replace that volume with the new image's library.

## What this setup covers

This provides a real local WeBWorK application for testing page structure, answer inputs, MathJax rendering, previews, feedback, and extension behavior across instructor and student accounts. It does not reproduce UBC's live course contents, institutional login, Canvas launches, or every production configuration choice. Matching the application build is useful, but production compatibility still needs checking against the actual environment you support.

## Source references

- [UBC image and tags](https://hub.docker.com/r/lthub/webwork/tags)
- [Pinned production Dockerfile](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/Dockerfile-prod)
- [Pinned image publishing workflow](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/.github/workflows/main.yml)
- [Pinned Compose example](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/docker-config/docker-compose.dist.yml)
- [Pinned startup script](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/docker-config/docker-entrypoint.sh)
- [Pinned authentication overrides](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/conf/localOverrides.conf.dist)
- [Pinned application server configuration](https://github.com/ubc/webwork2/blob/3d9992faa9c06a8a044197e100f3442a47763fca/conf/webwork2.mojolicious.dist.yml)
- [Chrome extension match patterns](https://developer.chrome.com/docs/extensions/develop/concepts/match-patterns)
- [MariaDB image health checks](https://mariadb.com/kb/en/using-healthcheck-sh/)
