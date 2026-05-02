# Complete Docker Compose Build Guide

## Problem Analysis

The original CI/CD workflows had several issues causing components to be "ignored" during builds:

### Root Causes Identified:

1. **Conditional Execution in `changes.yaml`**: The `changes.yaml` workflow uses path filtering (`dorny/paths-filter`) to determine which tests/jobs should run based on what files changed. This is efficient for PRs but can make it appear that jobs are "skipped" when no relevant files changed.

2. **Separate Build Jobs**: The `build.yaml` workflow has separate flags for `build_app` and `build_e2e`. When `build_app: false`, only the E2E test image is built, not the main application image.

3. **No Unified Output Artifact**: The existing workflows push images to registries but don't produce a downloadable Docker Compose artifact or tar file for local use.

4. **Workflow Dependencies**: In `push.yaml`, jobs depend on each other in a chain, and if any early job fails or is skipped, downstream jobs may not run.

## Solution: New `build-complete.yaml` Workflow

I've created a new workflow `.github/workflows/build-complete.yaml` that:

1. **Builds the complete application** - Always builds the full Docker image with all components (node builder, go builder, final image)
2. **Produces usable artifacts** - Saves the Docker image as a `.tar.gz` file and generates ready-to-use Docker Compose files
3. **No conditional skipping** - All build steps run unconditionally when triggered
4. **Multiple targets supported** - Can build for OSS, Premium, or Cloud targets
5. **Flexible output options** - Can push to registry OR save as artifact (or both)

## Usage

### Via GitHub Actions UI:

1. Go to Actions → "Complete Docker Compose Build"
2. Click "Run workflow"
3. Select your target (oss/premium/cloud)
4. Choose whether to push to registry and/or save as artifact
5. Click "Run workflow"

### Via GitHub CLI:

```bash
gh workflow run build-complete.yaml \
  --field target=oss \
  --field push_to_registry=false \
  --field save_artifact=true
```

## Artifacts Produced

After the workflow completes, you'll find:

1. **Docker Image** (`dekart-docker-image-{target}`): A `.tar.gz` file containing the complete Docker image
2. **Docker Compose Files** (`dekart-docker-compose-{target}`): Ready-to-use configuration including:
   - `docker-compose.yml` - Main compose file with PostgreSQL and Dekart services
   - `.env.example` - Example environment variables
   - `README.md` - Quick start instructions

## Installation Steps

### Option 1: Using Pre-built Images from Docker Hub

For the simplest setup, use the official images:

```bash
cd install/docker-compose

# For BigQuery setup
docker compose -f docker-compose.bigquery.yaml up -d

# For Snowflake with SQLite
docker compose -f docker-compose.snowflake-sqlite.yaml up -d

# For local development (PostgreSQL + Adminer only)
docker compose -f docker-compose.local.yaml up -d
```

### Option 2: Using Your Custom Built Image

1. Download the artifacts from GitHub Actions
2. Load the Docker image:
   ```bash
   docker load -i dekart-oss.tar.gz
   ```
3. Navigate to the extracted `output/` directory
4. Copy and configure environment:
   ```bash
   cp .env.example .env
   # Edit .env with your settings
   ```
5. Start the services:
   ```bash
   docker compose up -d
   ```
6. Access Dekart at http://localhost:8080

## Configuration Reference

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DEKART_MAPBOX_TOKEN` | Mapbox token for map visualization | Required |
| `DEKART_STORAGE` | Storage backend (USER, GCS, S3, SNOWFLAKE) | USER |
| `DEKART_DATASOURCE` | Data source (USER, BQ, ATHENA, SNOWFLAKE, PG) | USER |
| `DEKART_ALLOW_FILE_UPLOAD` | Enable CSV/GeoJSON uploads | 1 |
| `DEKART_CORS_ORIGIN` | CORS origin for frontend | http://localhost:3000 |
| `DEKART_LOG_DEBUG` | Enable debug logging | 0 |
| `DEKART_DEV_CLAIMS_EMAIL` | Dev email for local testing | user@example.com |

### Storage Backends

- **USER**: Local file storage (development/testing)
- **GCS**: Google Cloud Storage (production)
- **S3**: Amazon S3 (production)
- **SNOWFLAKE**: Snowflake internal stage

### Data Sources

- **USER**: File uploads only (CSV, GeoJSON)
- **BQ**: Google BigQuery
- **ATHENA**: AWS Athena
- **SNOWFLAKE**: Snowflake
- **PG**: PostgreSQL

## Existing Workflows Summary

### Push to Main (`.github/workflows/push.yaml`)

Runs on every push to `main`:
- `changes` - Detects which parts of code changed
- `node_tests` - Runs Node.js tests (if JS files changed)
- `go_tests` - Runs Go tests (if Go files changed)
- `build_e2e` - Builds E2E test image
- `e2e_tests` - Runs E2E tests
- `build_cloud` - Builds cloud deployment image
- `create_pulumi_pr` - Creates Pulumi deployment PR

### Pull Request (`.github/workflows/pull_request.yaml`)

Runs on PRs to `main`:
- Same jobs as push workflow, but conditionally based on changed files
- `comment_results` - Posts test results as PR comment

### Release (`.github/workflows/release.yaml`)

Runs on tag pushes:
- Runs all tests
- Builds both E2E and app images
- Pushes to both OSS and Premium registries

## Troubleshooting

### Jobs Showing as "Skipped"

This is expected behavior when:
- No relevant files changed (for PR workflows)
- Previous dependent jobs failed
- Conditional checks evaluated to false

To force a complete build, use the new `build-complete.yaml` workflow manually.

### Build Failures

Common causes:
1. **Missing NPM token**: Ensure `NPM_GH_TOKEN` secret is set
2. **License key required**: For premium/cloud builds, set `DEKART_LICENSE_KEY` secret
3. **Registry authentication**: Verify Docker Hub or GHCR credentials

### Local Build Alternative

You can also build locally using the Makefile:

```bash
# Build complete image
make docker

# Run with Docker Compose
make up

# Run tests
make nodetest
make gotest
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Dockerfile                               │
├─────────────────────────────────────────────────────────────┤
│  Stage 1: nodedeps    → Install Node dependencies          │
│  Stage 2: nodebuilder → Build React frontend               │
│  Stage 3: godeps      → Download Go modules                │
│  Stage 4: gobuilder   → Compile Go server                  │
│  Stage 5: e2etest     → Cypress E2E tests (CI only)        │
│  Stage 6: final       → Production image                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Docker Compose Setup                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐                        │
│  │  PostgreSQL │───▶│   Dekart    │                        │
│  │   (metadata)│    │   (app)     │                        │
│  └─────────────┘    └─────────────┘                        │
│                            │                                │
│                            ▼                                │
│                     ┌─────────────┐                         │
│                     │   External  │                         │
│                     │  Services   │                         │
│                     │ (BQ/S3/etc) │                         │
│                     └─────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

## Next Steps

1. **For Development**: Use `docker-compose.local.yaml` for database only, run app locally
2. **For Testing**: Use `docker-compose.snowflake-sqlite.yaml` for quick Snowflake testing
3. **For Production**: Configure appropriate compose file with cloud storage and proper auth

## Support

- Documentation: https://dekart.xyz/docs/
- GitHub Issues: https://github.com/dekart-xyz/dekart/issues
- Docker Hub: https://hub.docker.com/r/dekartxyz/dekart
