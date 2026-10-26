# Code Conventions

Organisation-wide conventions for contributing to repositories of the **Urban Futures Collective** (Urban Model Platform, UMP-X frontend, MCP server, model servers and related projects).

This document is deliberately high-level and language-agnostic. It describes the standard we are converging on, not only the current state of every repository.
Where a repository does not (yet) meet a rule, new and changed code should follow the rule; existing code is migrated opportunistically ("leave it better than you found it"), not in unrelated pull requests.

---

## 0. How to read this document

### 0.1 Normative language

The key words below are used in the sense of [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119):

| Keyword | Meaning |
|---|---|
| **MUST** / **MUST NOT** | Required. Deviations need an explicit, documented reason in the PR. |
| **SHOULD** / **SHOULD NOT** | Recommended best practice. Deviate only with good reason. |
| **MAY** | Optional; at the discretion of the repository maintainers. |

### 0.2 Precedence

1. Repository-specific rules (`CONTRIBUTING.md`, tool configuration such as `pyproject.toml`, `eslint.config.*`, `.editorconfig`) take precedence over this document **where they are more specific**.
2. This document applies wherever a repository is silent.
3. Automated tooling (formatter, linter, CI) is the source of truth for formatting. Style is not debated in code review.

Each repository's `CONTRIBUTING.md` SHOULD link to this document and describe only what is specific to that repository (setup, commands, architecture).

---

## 1. Language and communication

- All code, identifiers, comments, commit messages, pull requests, issues and technical documentation **MUST** be written in **English**.
- User-facing text in applications **MUST NOT** be hard-coded in a single language; it goes through the project's i18n mechanism (e.g. `i18n/` in the frontend). German and English are the primary UI languages.
- Be precise and neutral in reviews: comment on the code, not the person;
  distinguish blocking remarks from suggestions (e.g. prefix with `nit:`).

---

## 2. Repository baseline

Every repository **SHOULD** contain:

| File | Purpose |
|---|---|
| `README.md` | What it is, how to run it locally, how to test, links to docs. |
| `LICENSE` | An OSI-approved licence. Licence choice is per repository but MUST be explicit. |
| `CONTRIBUTING.md` | Repository-specific setup and workflow; links to this document. |
| `CHANGELOG.md` | Human-readable changes (see §11). |
| `.env.example` | Every supported environment variable with a safe default or placeholder. |
| `.gitignore` / `.dockerignore` | Keep build artefacts, local data and secrets out. |
| Agent instruction file (`CLAUDE.md`, `AGENTS.md`) | MAY; if present, see §13. |

Organisation-wide files live in the `.github` repository and apply automatically unless a repository overrides them.

Repositories **MUST NOT** contain: secrets, credentials, private keys,
production configuration with real values, personal data, large binary data
sets (use Git LFS, releases or external storage), or IDE-specific settings that
override project tooling. Shared editor recommendations (e.g. `.vscode/extensions.json`)
MAY be committed.

Dead or legacy code (e.g. a previous major version kept "for reference")
SHOULD NOT live on the default branch; use a tag or a branch instead. If it
must stay temporarily, it MUST be clearly marked and excluded from linting,
tests and packaging.

---

## 3. Version control workflow

### 3.1 Branches

- The default branch is `main`. It **MUST** always be releasable (build and
  tests green).
- Work happens on short-lived branches created from the current integration
  branch and merged via pull request. Direct pushes to `main` **SHOULD NOT**
  happen (enforce via branch protection).
- An integration branch (`dev`, `staging`) MAY be used where deployment
  environments require it; long-lived feature branches SHOULD be avoided.
- Branch names: `<type>/<short-kebab-description>`, optionally with an issue
  number, using the same types as commits:

  ```
  feat/mcp-tool-catalog
  fix/123-results-accept-header
  docs/provider-configuration
  ```

  Release lines MAY use `release/<version>` (e.g. `release/3.0`).

### 3.2 Commits

- Commit messages **SHOULD** follow
  [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/):

  ```
  <type>(<optional scope>): <imperative summary, ≤ 72 chars>

  <optional body: what and why, wrapped at 72 chars>

  <optional footer: BREAKING CHANGE: …, Refs #123, Co-authored-by: …>
  ```

  Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`,
  `chore`, `revert`.
- Use the imperative mood ("Add retry", not "Added retry").
- One logical change per commit. Formatting-only changes go in separate commits.
- Breaking changes **MUST** be marked (`feat!:` or `BREAKING CHANGE:` footer).

### 3.3 Pull requests

- A PR **SHOULD** address one concern and stay reviewable (guideline: < 400
  changed lines excluding generated files and lockfiles).
- The PR description **MUST** state *what* changes and *why*, how it was
  tested, and link the related issue. Screenshots for UI changes.
- At least **one approving review** from someone other than the author is
  required before merging into `main`. Merges into an integration branch
  (`staging`, `dev`) MAY be done by the author without review, so that work
  can be tried in a shared environment without waiting. Changes to security-relevant code (auth,
  authorization, input handling, secrets) SHOULD get a second review.
- CI **MUST** be green before merge.
- Preferred merge strategy: **squash merge** with a Conventional-Commit title,
  unless the repository relies on individual commits (then rebase merge).
- Delete the branch after merging.
- Draft PRs are encouraged for early feedback.

---

## 4. Code style

### 4.1 General rules (all languages)

- Formatting **SHOULD** be automated. Every repository configures a formatter
  and a linter, runs them in CI, and SHOULD run them via `pre-commit`
  (or an equivalent hook such as `lint-staged`).
- One formatter and one linter per language per repository. Do not configure
  overlapping tools that disagree (e.g. two formatters with different line
  lengths).
- UTF-8, LF line endings, final newline, no trailing whitespace.
- No commented-out code in merged changes; Git keeps history.
- `TODO`/`FIXME` comments **SHOULD** reference an issue (`TODO(#123): …`).
- No debug output (`print`, `console.log`, `breakpoint()`) in merged code;
  use the logging facility.

### 4.2 Naming

- Names describe intent: `job_status`, not `js`; `fetchProcesses`, not `getData`.
- Abbreviations only when established in the domain or the language
  (`id`, `url`, `ctx`, `crs`, `ogc`).
- Follow the casing conventions of the language; never mix styles within one
  codebase:

  | Language | Variables / functions | Types / classes | Constants | Files / modules |
  |---|---|---|---|---|
  | Python | `snake_case` | `PascalCase` | `UPPER_SNAKE_CASE` | `snake_case.py` |
  | TypeScript / JS | `camelCase` | `PascalCase` | `UPPER_SNAKE_CASE` | `kebab-case.ts` or framework convention |
  | Vue components | — | `PascalCase` | — | `PascalCase.vue` |
  | Shell | `snake_case` | — | `UPPER_SNAKE_CASE` (env) | `kebab-case.sh` |
  | Env variables | — | — | `PROJECT_PREFIX_NAME` (e.g. `UMP_AUTH_ENABLED`) | — |
  | Helm / YAML keys | `camelCase` (Helm) or as defined by the consumed standard | — | — | `kebab-case.yaml` |

- Booleans read as predicates: `is_public`, `has_results`, `anonymousAccess`.
- Interfaces/ports are named after their role (`JobRepositoryPort`,
  `HttpClientPort`); implementations after their technology
  (`SqlJobRepository`, `AiohttpClientAdapter`).

### 4.3 Comments and docstrings

- Code explains *what*; comments explain *why* (constraints, workarounds,
  references to standards or issues).
- Every public module, class and function **SHOULD** have a docstring / JSDoc
  stating purpose, parameters, return value and raised errors where not obvious.
- Python: one docstring style per repository (Google or reST; Google
  recommended). TypeScript: TSDoc.
- Reference specifications precisely (e.g. "OGC API – Processes Part 1,
  Req. 34") rather than paraphrasing them.

### 4.4 Functions and modules

- Single responsibility: if a function cannot be summarised in one sentence,
  split it.
- Prefer early returns over deep nesting; keep cyclomatic complexity low
  (linters SHOULD enforce a limit, e.g. 10).
- Prefer pure functions and immutable data where practical; isolate side
  effects at the edges.
- Avoid global mutable state. If unavoidable, encapsulate and synchronise it.

### 4.5 Language-specific baselines

These are defaults for new repositories and the target for existing ones.

#### Python

- Supported versions declared in `pyproject.toml` (`requires-python`).
- Data at boundaries is modelled with **Pydantic** (or dataclasses for pure
  internal data); configuration via `pydantic-settings`.
- `async` code MUST NOT call blocking I/O; never call `asyncio.run()` inside
  a running event loop or request handler.
- Packaging: `pyproject.toml` with a committed lockfile (Poetry `poetry.lock`
  or uv `uv.lock`). One package manager per repository.
- Source layout: `src/<package>/`, tests in `tests/`.

#### TypeScript / Vue / Nuxt

- TypeScript with `strict: true`. Avoid `any`; use `unknown` and narrow.
- Vue: Composition API with `<script setup lang="ts">`; reusable logic in
  `composables/` (`useXyz`); components small and presentational where
  possible.
- Validate external data (API responses, LLM/tool outputs) at runtime, e.g.
  with **Zod**, not only with static types.
- Server-side secrets (API keys, LLM tokens) MUST stay in server routes /
  runtime config and never reach the client bundle.
- User interfaces SHOULD meet **WCAG 2.1 level AA** (in Germany BITV 2.0):
  keyboard operable, sufficient contrast, labelled form fields and controls,
  text alternatives for images and maps where meaningful.
- `package-lock.json` (or the lockfile of the chosen manager) is committed;
  pin exact versions for fast-moving SDKs where breaking minor releases are
  common.

#### Shell

- `#!/usr/bin/env bash` and `set -euo pipefail`. Check with `shellcheck`.
- Prefer Python or Make targets for anything beyond simple glue.

#### Docker

- Multi-stage builds, minimal base images with pinned tags (ideally digests),
  non-root user, `HEALTHCHECK` where meaningful. 


#### SQL / database

- Schema changes **MUST** go through migrations (e.g. Alembic). No ad-hoc DDL
  in application code.
- Queries **MUST** be parameterised; string interpolation into SQL is
  forbidden.

---

## 5. Architecture and design

- **Standards first.** Where an open standard exists, implement it rather than
  inventing an API: OGC API – Processes / Features / Records, MCP for agent
  interfaces, OpenID Connect / OAuth 2.0 for auth, GeoJSON for vector data.
  Document deviations explicitly.
- **Design decisions** with lasting impact (architecture, technology choice,
  API contract) SHOULD be recorded as short Architecture Decision Records
  (`docs/adr/NNNN-title.md`: context, decision, consequences). Design notes
  and specs MUST be kept in sync with the code or removed when obsolete.
- Prefer composition over inheritance; prefer small, explicit pipelines or
  strategies over large conditional blocks.

---

## 6. APIs and interfaces

### 6.1 General

- Public HTTP APIs **MUST** be described by a machine-readable contract
  (OpenAPI; JSON Schema for tool/process inputs).
- APIs are **versioned** (e.g. `/v1/…`). Breaking changes require a new major
  API version and a deprecation period for the old one.
- Errors are returned in a consistent, documented format (e.g. RFC 9457
  *Problem Details*, or the exception format mandated by the implemented OGC
  standard) with correct HTTP status codes.
- Validate all input at the boundary; reject unknown or malformed input with a
  clear error rather than coercing it silently.
- Timeouts **MUST** be set for every outbound network call; retries use
  bounded exponential backoff and are only applied to idempotent operations.
- Geodata: state the coordinate reference system explicitly (EPSG code / CRS
  URI). GeoJSON is WGS 84 (EPSG:4326) per RFC 7946; other CRSs must be
  declared.

### 6.2 Process descriptions (OGC API – Processes)

Model servers are consumed by generic clients (UMP, the UMP-X frontend, the MCP
server) that build forms and tool schemas from the process description alone.
A description that cannot be satisfied makes the process unusable for all of
them. Therefore:

- Every input **MUST** state `minOccurs` explicitly. Optional inputs have
  `minOccurs: 0`; without it, clients treat the input as required.
- A `default` **MUST** be valid against the input's own schema (an integer
  under `type: integer`, never a placeholder such as `'auto'`). If the
  library has an automatic mode, leave the input out (`minOccurs: 0`, no
  default) or offer the mode as an explicit `enum` value.
- Fixed choices **MUST** be declared as `enum`, so that clients can offer a
  selection instead of free text.
- Titles and descriptions of processes and inputs SHOULD be available in
  English and German.
- Results **MUST** be returned with a correct `Content-Type`, also when the
  client sends no `Accept` header or `*/*`.
- Expensive work belongs in the job, not in the request that creates it: an
  asynchronous execution request SHOULD be accepted within seconds.

---

## 7. Error handling and logging

- Fail loudly and early. Do not swallow exceptions: no bare `except:` /
  `except Exception: pass` / empty `catch {}`. Catch the narrowest exception
  you can handle meaningfully; otherwise let it propagate.
- Use domain-specific exception types in the core; translate them to transport
  errors (HTTP status, MCP error) in the adapter layer.
- Use the project's logging facility, never `print`. Prefer **structured
  logging** (key–value / JSON) in services.
- Log levels: `DEBUG` diagnostics, `INFO` lifecycle events, `WARNING`
  recoverable anomalies, `ERROR` failed operations that need attention.
- **Never log** secrets, tokens, passwords, full authorisation headers or
  personal data. Mask identifiers where they are not needed.
- Include correlation identifiers (request ID, job ID) to make distributed
  flows traceable.

---

## 8. Testing

- Every behavioural change **MUST** come with tests; every bug fix SHOULD
  include a regression test that fails without the fix.
- Test pyramid: many fast unit tests, fewer integration tests, few end-to-end
  tests. Unit tests MUST NOT require network, external services or a specific
  machine.
- Prefer **hand-written fakes implementing a port/interface** over mocking
  concrete classes or patching internals; mock only at system boundaries
  (HTTP, clock, randomness).
- Tests **MUST** be deterministic and independent of execution order
  (randomised order SHOULD be enabled, e.g. `pytest-randomly`).
- Naming: `test_<unit>_<behaviour>_<condition>` (Python),
  `describe/it('…')` with readable sentences (TS). Test files mirror the
  structure of the code under test.
- Coverage is reported in CI and treated as a signal, not a target. New code
  SHOULD be covered; coverage MUST NOT decrease without justification.
- Tooling defaults: `pytest` (+ `pytest-asyncio`) for Python, `vitest` for
  TypeScript/Vue, Playwright for browser end-to-end tests where needed.

---

## 9. Security

- **Secrets** never enter the repository, logs, error messages, container
  images or client bundles. Use environment variables, CI secrets or a secret
  manager. Enable GitHub secret scanning and push protection.
- **Authentication and authorisation** use established libraries (OIDC/JWT
  libraries with signature, issuer, audience and expiry validation); never
  parse or trust tokens manually. Authorisation is enforced server-side,
  centrally and deny-by-default.
- **Input** from users, remote providers, files and AI models is untrusted:
  validate it, encode output, use parameterised queries.
- **Agent/LLM interfaces** (MCP tools, chat features): tools MUST respect the
  calling user's permissions; tool descriptions and tool results are untrusted
  input (prompt injection); actions with side effects SHOULD require explicit
  confirmation.
- **Dependencies**: lockfiles committed; automated updates via Dependabot or
  Renovate; vulnerability scanning (e.g. `pip-audit`, `npm audit`, GitHub
  dependency review, container image scanning) in CI.
- **Personal data**: new processing of personal data (new fields, logs,
  third-party services the browser or server contacts) MUST be recorded in
  the project's privacy inventory in the same PR, so that the privacy policy
  can be kept accurate.
- Vulnerabilities are reported privately as described in `SECURITY.md`,
  never in public issues.

---

## 10. Dependencies

- Add a dependency only if it provides clear value over a small amount of own
  code; prefer well-maintained libraries with compatible licences.
- Declare version ranges in the manifest, pin exact versions via the lockfile.
- Document non-obvious upper bounds with a comment explaining why
  (e.g. an API removed in the next major release).
- Separate runtime, development and documentation dependencies
  (dependency groups / `devDependencies`).
- Licences of dependencies MUST be compatible with the repository licence.
- **Data sources** (e.g. OpenStreetMap under ODbL, terrain or weather data)
  MUST be attributed as their licence requires, in the user interface where
  the data is shown and in the documentation.

---

## 11. Documentation, versioning and releases

### 11.1 Documentation

- Documentation lives next to the code (docs-as-code, Markdown) and is updated
  **in the same PR** as the change it describes. A PR that makes documentation
  wrong is incomplete.
- `README.md` stays short and current; longer material goes to `docs/`.
- Document every configuration option (name, type, default, effect), ideally
  generated from or validated against the settings model.
- Prefer diagrams as code (Mermaid, PlantUML) or editable SVG over raster
  images.

### 11.2 Versioning

- Releases follow [Semantic Versioning 2.0.0](https://semver.org/):
  `MAJOR.MINOR.PATCH`, pre-releases as `3.0.0-alpha.8`, `3.0.0-rc.1`
  (note the dot before the pre-release number, which keeps SemVer ordering
  correct beyond 9).
- Git tags are `v<version>` (e.g. `v3.0.0`).
- Versions are bumped by tooling (e.g. `bump-my-version`, release-please,
  semantic-release), never by hand in multiple files.
- Independently released artefacts (application, Helm chart, client library)
  MAY carry independent versions; their compatibility MUST be documented.

### 11.3 Changelog and releases

- `CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
  with an `Unreleased` section updated in each user-relevant PR (or generated
  from Conventional Commits).
- Releases are created from tags via CI; container images are published with
  immutable version tags (plus moving tags like `latest` only as convenience)
  and build provenance/attestation.

---

## 12. Continuous integration and delivery

Every repository's CI **SHOULD** run on each pull request:

1. Format check and lint
2. Type check (where applicable)
3. Unit and integration tests with coverage report
4. Build (package, container image, frontend bundle)
5. Dependency and security checks
6. For charts/manifests: `helm lint` / schema validation

Additional rules:

- Pin third-party GitHub Actions to a major version or commit SHA; grant
  workflows minimal `permissions`.
- The same commands run locally and in CI (via `make`, `npm run`, `poetry run`
  or `uv run` targets); document them in the README.
- Deployments to shared environments happen only from CI, never from
  developer machines.

---

## 13. AI-assisted contributions

AI coding assistants are welcome tools. The following applies:

- The **human author is responsible** for every line submitted: they MUST
  understand, review and test AI-generated code exactly like their own.
- AI-generated code MUST meet all conventions in this document; "the assistant
  wrote it" is not a justification for deviations.
- Significant AI assistance SHOULD be disclosed (e.g. `Co-authored-by:`
  trailer or a note in the PR description).
- Never paste secrets, credentials, personal data or non-public data into
  external AI services.
- Repository instruction files for agents (`CLAUDE.md`, `AGENTS.md`,
  `.github/copilot-instructions.md`) MAY be maintained. They MUST be consistent
  with this document and the repository's `CONTRIBUTING.md`, SHOULD reference
  rather than duplicate them, and MUST be updated when the referenced
  conventions or architecture change.

---

## 14. Pull request checklist

Copy into PR templates or use as a self-check:

- [ ] Branch name and PR title follow Conventional Commits
- [ ] Change is focused on one concern
- [ ] Formatter, linter and type checker pass locally
- [ ] Tests added/updated; all tests pass
- [ ] No secrets, debug output or commented-out code
- [ ] Public APIs / config options documented; `CHANGELOG.md` updated
- [ ] Breaking changes marked and migration path described
- [ ] Security implications considered (auth, input validation, logging)
- [ ] Personal data and data sources: privacy inventory and attribution updated
- [ ] AI assistance reviewed and, if significant, disclosed

---

## 15. Changing these conventions

These conventions evolve. Propose changes via a pull request to the
[`.github`](https://github.com/Urban-Futures-Collective/.github) repository,
explaining the motivation. Changes that require migration in existing
repositories SHOULD include a migration note and are announced to all
maintainers.
