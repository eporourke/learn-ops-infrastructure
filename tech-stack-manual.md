# Tech Stack Manual

## 1. Run Questions

### 1a. Config Files

| Config File          | Location                                                                                     | Config Value                                                                 | What it's for                                                                                      | How it's used                                                                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `.env`               | `\\wsl.localhost\Ubuntu\home\erin\workspace\lms\learn-ops-infrastructure\.env`               | `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `DATA_SOURCE_NAME`      | To store database connection info locally so as not to reveal sensitive information in pushed code | To connect to a database — the information is called with an alias                                                                  |
| `docker-compose.yml` | `\\wsl.localhost\Ubuntu\home\erin\workspace\lms\learn-ops-infrastructure\docker-compose.yml` | `services`, `database`, `networks`, `container_name`, `ports`, etc.          | All the services that make up the system, their locations, port numbers and config values          | It's the specifications to build the Docker container                                                                               |
| `prometheus.yml`     | `\\wsl.localhost\Ubuntu\home\erin\workspace\lms\learn-ops-infrastructure\prometheus.yml`     | `scrape_interval`, `evaluation_interval`, `scrape_configs`, `job_name`, etc. | Sets global parameters and configs regarding scraping                                              | Research or ask instructor                                                                                                          |
| `Makefile`           | `\\wsl.localhost\Ubuntu\home\erin\workspace\lms\learn-ops-infrastructure\Makefile`           | `setup`, `doctor`, `teardown`, `pull`, `up`, `down`, etc.                    | It's a file that aliases Docker commands                                                           | The aliases can be used so that you don't have to type in full commands. For instance, `make down` instead of `docker compose down` |

### 1b. How to Start It

Start the Docker program.

Open PowerShell or VS Code and go to the caret to open an Ubuntu terminal. `cd` into `lms/learn-ops-infrastructure` and type `make up`.

Docker should start to build.

**Question for class:** Is this doing the same thing as pressing the play button in Docker? Are we always going to build instead of compose?

### 1c. Where to Access It

| Service           | Port           | URL                                              |
| ----------------- | -------------- | ------------------------------------------------ |
| database          | `5433`         | `http://localhost:5433`                          |
| api               | `8000`, `5678` | `http://localhost:8000`, `http://localhost:5678` |
| client            | `3000`         | `http://localhost:3000`                          |
| prometheus        | `9090`         | `http://localhost:9090`                          |
| grafana           | `3001`         | `http://localhost:3001`                          |
| postgres_exporter | `9187`         | `http://localhost:9187`                          |

### 1d. Service Dependencies

| Service           | Depends On | Why                                                                                                                                     |
| ----------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| api               | database   | The API's job is to pull data from the database. No database, no data. **Note to self: check whether "pull" is the right terminology.** |
| prometheus        | API        | Scrapes metrics from the API                                                                                                            |
| Grafana           | Prometheus | **I don't know — research**                                                                                                             |
| postgres_exporter | database   | Needs access to the data to export                                                                                                      |

### 1e. Main Entry Points

| Service           | Startup File         | Routes / URL Config File                                                     |
| ----------------- | -------------------- | ---------------------------------------------------------------------------- |
| API               | `entrypoint.sh`      | `\\wsl.localhost\Ubuntu\home\erin\workspace\lms\learn-ops-api\entrypoint.sh` |
| Client            | `Dockerfile` **(?)** | ?                                                                            |
| database          | ?                    | ?                                                                            |
| prometheus        | ?                    | ?                                                                            |
| grafana           | ?                    | ?                                                                            |
| postgres_exporter | ?                    | ?                                                                            |

**Note to self:** Check whether the full `entrypoint.sh` path belongs under "Routes / URL Config File" or whether the template is asking for something different.

---

## 2. Services

| Service Name | Tech Stack (including version) | Purpose                                                                      |
| ------------ | ------------------------------ | ---------------------------------------------------------------------------- |
| Database     | PostgreSQL 16                  | To store and structure data                                                  |
| Client       | learnopsclient 4.0.0           | The front end where people can see the data                                  |
| API          | Python 3.11.11                 | To communicate between the client and database to display data in the client |

---

## 3. System Overview

The system is comprised of three main services, the Database, the client, and the API. A Docker container provides the environment where everything runs with the requirements and specifications. There are global and local settings that manage things such as timeouts and how many times to try to connect. The database needs to be running for the other systems to run successfully. The client is there as a front end learning operations system. There are other packages **(note to self: confirm)** such as Prometheus, which scrape metrics.
