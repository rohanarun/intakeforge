# IntakeForge

Accessible versioned forms with deterministic logic, privacy controls, corrections, and grounded summaries.

IntakeForge is a focused, public MIT distribution for the `forms` module in [managed-oss-cloud](https://github.com/rohanarun/managed-oss-cloud). It includes a product web UI, a product-scoped HTTP client, the `intakeforge` CLI, and a stdio MCP server exposing only this product's 16 typed actions.

## Current boundary

This repository is runnable, but it is intentionally not a second database server. Authentication, workspace isolation, shared PostgreSQL storage, plan enforcement, AI execution, and audit records remain behind the managed-oss-cloud API. This product receives a scoped API token and cannot receive database credentials or run database migrations.

- Hosted backend: `https://cloud.getsupers.com`
- Self-hosted backend: any compatible managed-oss-cloud v0.4.2 deployment
- Hosted minimum plan: `starter`
- Resource class: `shared`
- Pinned backend source: [v0.4.2](https://github.com/rohanarun/managed-oss-cloud/tree/v0.4.2) at `20c4a704c77cbbbff1da995e1d91b937625a8aa4`

## AI-native by construction

- draft review-only forms
- summarize aggregate results
- suggest cited form improvements

AI actions use their own `ai` token scope, preserve the typed action contract, and return durable backend job evidence. They do not grant the model database credentials or bypass approval, plan, tenant, or action boundaries.

## Run the CLI

Node.js 20 or newer is the only local dependency.

```bash
npm install
npm link
export INTAKEFORGE_TOKEN="a-scoped-workspace-token"
export INTAKEFORGE_URL="https://cloud.getsupers.com"
intakeforge actions
intakeforge workspace
intakeforge action form-create '{"name":"Sample Form name"}'
```

The generic `SUPERSUITE_TOKEN` and `SUPERSUITE_URL` variables are supported as fallbacks. Create a token in the workspace dashboard with only the `read`, `write`, and/or `ai` scopes the client needs.

## Run the web UI

The UI proxies requests through the local Node server so the workspace API token is never sent to the browser. Browser access is protected by a separate key of at least 24 characters.

```bash
export INTAKEFORGE_TOKEN="a-scoped-workspace-token"
export INTAKEFORGE_URL="https://cloud.getsupers.com"
export INTAKEFORGE_WEB_KEY="a-separate-random-browser-key"
npm start
```

Open `http://127.0.0.1:4173`. Put the service behind TLS and an authenticated reverse proxy before exposing it to a network.

Docker runs the same server:

```bash
docker build -t intakeforge:0.1.0 .
docker run --rm -p 4173:4173 \
  -e INTAKEFORGE_TOKEN \
  -e INTAKEFORGE_URL \
  -e INTAKEFORGE_WEB_KEY \
  intakeforge:0.1.0
```

## Connect the MCP server

The MCP server uses newline-delimited JSON-RPC over stdio and implements `initialize`, `ping`, `tools/list`, and `tools/call`. It advertises four product utilities plus the 16 product action tools with their pinned JSON input schemas.

```json
{
  "mcpServers": {
    "intakeforge": {
      "command": "intakeforge-mcp",
      "env": {
        "INTAKEFORGE_TOKEN": "a-scoped-workspace-token",
        "INTAKEFORGE_URL": "https://cloud.getsupers.com"
      }
    }
  }
}
```

## Self-host the backend

```bash
git clone https://github.com/rohanarun/managed-oss-cloud.git
cd managed-oss-cloud
git checkout v0.4.2
# Follow that repository's PostgreSQL, migration, TLS, and runtime instructions.
```

Then point `INTAKEFORGE_URL` at the self-hosted control-plane origin. All products may share the same backend and PostgreSQL cluster while the backend preserves workspace and module boundaries.

## Typed action catalogue

| Action ID | Capability | Token scope | Operation |
|---|---|---|---|
| `form-create` | Create form | `write` | `create` |
| `form-list` | List forms | `read` | `read` |
| `form-draft` | Draft form release | `write` | `command` |
| `schema-validate` | Validate form schema | `read` | `read` |
| `logic-validate` | Validate form logic | `read` | `read` |
| `release-diff` | Diff form release | `read` | `read` |
| `release-publish` | Publish form release | `write` | `command` |
| `submission-validate` | Validate form submission | `read` | `read` |
| `submission-create` | Create form submission | `write` | `command` |
| `submission-get` | Read form submission | `read` | `read` |
| `submission-correct` | Correct form submission | `write` | `command` |
| `results-query` | Query form results | `read` | `read` |
| `results-summarize` | Summarize form results | `read` | `read` |
| `export-preview` | Preview form export | `write` | `command` |
| `export-create` | Create form export | `write` | `command` |
| `rights-preview` | Preview respondent rights request | `read` | `read` |

The complete machine-readable module definition, JSON input schemas, MCP tool names, risk metadata, examples, and release provenance are pinned in [product-manifest.json](./product-manifest.json).

## Clean-room statement

Original clean-room implementation of the data collection software category, designed and written independently. Public category behavior informed the requirements, but the product name, implementation, UI, CLI, MCP surface, tests, and documentation in this repository are original. No third-party product source code, assets, copied interface, trademarks, or branding are included.

## Security

- Use a distinct, least-privilege workspace API token per deployment.
- Never place the API token in browser code, Git history, container images, or logs.
- Keep the web server on loopback unless it is behind TLS and authentication.
- Rotate a token immediately if it is exposed.
- Treat AI output as a proposal unless the action contract explicitly records approval and execution boundaries.

See [SECURITY.md](./SECURITY.md) for vulnerability reporting and the trust boundary.

## Development

```bash
npm test
npm run verify
npm pack --dry-run
```

The tests run against a fake API and prove bearer authentication, fixed module routing, input validation, CLI execution, stdio MCP discovery/calls, web-key protection, and server-side token handling.

## License

[MIT](./LICENSE)
