# Tech Stack (AI) — Session Log

A record of the prompts/questions used with Claude to research and fill out `tech-stack-ai.md`.

## 1a. Config Files

- "Can you please tell me what config files are present in workspace/lms?"
- "Can you tell me about each one first, and ask me to approve its addition?"
- "Can you please add each of these to the file?"
- "Can we please section 1a off by repo?"

Investigation approach: searched each repo (excluding `node_modules`/`.git`) for common config file patterns (`.env`, `docker-compose*`, `Dockerfile*`, `*.yml`/`.yaml`, `*.ini`, `*.conf`, `settings.py`), then read each file's contents to summarize its purpose. For `.env` files containing real secrets, reviewed the corresponding `.env.template` (or grepped variable names only) instead of the live file, to avoid exposing credential values in the doc.

## 1b. How to Start It

- "Can you tell me what you can see about how to start it?"
- "So I believe the how to start it question is really just about the command to bring the whole system up."
- "Can you tell me more about the mechanics behind the `make up` command according to the files?"
- "Maybe too much information now — can we put the first answer re `make up` with the Monarch caveat, plus just that it is doing `docker compose up --build -d` and `docker compose logs -f`?"

Investigation approach: read each repo's `README.md`, `learn-ops-infrastructure/Makefile`, `learn-ops-infrastructure/SETUP_README.md`, and `learn-ops-api/entrypoint.sh` to determine the actual startup commands and what happens during container boot.

## 1c. Where to Access It

- "Yes, tell me more about 1c."

Investigation approach: cross-referenced port mappings across all three `docker-compose.yml` files (`learn-ops-infrastructure`, `learn-ops-infrastructure/valkey`, `service-monarch`) to build the service → port → URL table.

## 1d. Service Dependencies

- "Yes, tell me more about 1d."

Investigation approach: read `depends_on` blocks across all three compose files; discovered Valkey lives in its own separate `docker-compose.yml` (not the main infrastructure one) by grepping for "valkey" across the repo and finding `learn-ops-infrastructure/valkey/docker-compose.yml`.

## 1e. Main Entry Points

- "Yes, tell me more about 1e."

Investigation approach: located `manage.py` and `urls.py` files for the Django API, `src/index.js` and the React Router usage in `src/components/LearnOps.js` for the client, and `service/main.py` plus `service/custom_logging/web_interface.py` for Monarch.

## 2. Services

- "Yes, tell me more about section 2."

Investigation approach: pulled version info from `learn-ops-api/Pipfile.lock` (Django, DRF), `learn-ops-client/package.json` (React, react-router-dom, Radix UI), and `service-monarch/requirements.txt` (Flask, pydantic, prometheus-client, etc.), combined with the base images from each Dockerfile.

## 3. System Overview

- "Yes, tell me more about section 3."

Investigation approach: synthesized the dependency graph and entry-point findings from 1d/1e, plus the architecture diagram in `learn-ops-infrastructure/README.md` — noting that the README's diagram includes a "Hashtagger Service" marked "this will be added," which doesn't exist in the codebase yet and was excluded from the current-state diagram.
