# Docker Compose Build Fix Documentation

## Problem Summary

The original GitHub Actions workflows had two main issues:

1. **Conditional Job Execution**: Workflows like `push.yaml` used a `changes` job that filtered which components to build based on file changes. This caused critical components (build_e2e, node_tests, e2e_tests, go_tests) to be skipped when unrelated files were modified.

2. **No Output Artifacts**: The build process completed but didn't produce downloadable Docker images or ready-to-use Docker Compose files for end users.

3. **Deprecated Actions**: Several actions were using Node.js 20 runtime which is deprecated and will be removed in September 2026.

## Solutions Implemented

### 1. New Complete Build Workflow

Created `.github/workflows/build-complete.yaml` - a standalone workflow that:
- **Always builds all components** without conditional filtering
- **Produces downloadable artifacts**: Docker image tarball and Docker Compose files
- **Works on-demand** via manual trigger (workflow_dispatch)

### 2. Updated Action Versions

Fixed all deprecated actions across all workflow files:
- `actions/checkout@v4` → `@v5`
- `dorny/paths-filter@v2` → `@v3`
- `actions/github-script@v7` → `@v8`
- `docker/setup-qemu-action@v2/v3` → `@v4`
- `docker/setup-buildx-action@v2/v3` → `@v4`
- `peter-evans/dockerhub-description@v2` → `@v4`

This eliminates the Node.js 20 deprecation warnings.

### 3. Fixed Artifact Upload

The workflow now:
- Always loads the Docker image after building (`load: true`)
- Saves the image as a compressed tarball
- Uploads both the Docker image and Docker Compose files as artifacts
- Uses `overwrite: true` to prevent upload conflicts

## How to Use

### Option 1: GitHub Actions (Recommended)

1. Go to your repository's **Actions** tab
2. Select **"Complete Docker Compose Build"** workflow
3. Click **"Run workflow"**
4. Configure options:
   - **Target**: Choose `oss`, `premium`, or `cloud`
   - **Push to registry**: Keep `false` to download artifacts
   - **Save artifact**: Keep `true` (default)
5. Click **"Run workflow"**
6. After completion, download artifacts from the workflow run page:
   - `dekart-docker-image-{target}` - Docker image tarball
   - `dekart-docker-compose-{target}` - Ready-to-use Docker Compose files

### Option 2: Using Pre-built Docker Compose Files

The repository includes pre-configured Docker Compose files in `install/docker-compose/`:

```bash
cd install/docker-compose

# For Snowflake + SQLite setup
docker compose -f docker-compose.snowflake-sqlite.yaml up -d

# For BigQuery setup
docker compose -f docker-compose.bigquery.yaml up -d

# For local development
docker compose -f docker-compose.local.yaml up -d
```

### Option 3: Local Build

```bash
# Build Docker image locally
make docker

# Run with Docker Compose
docker compose -f docker-compose.yml up -d
```

## Downloaded Artifact Contents

When you download `dekart-docker-compose-{target}`, you'll receive:

```
output/
├── docker-compose.yml    # Ready-to-use Docker Compose configuration
├── .env.example          # Example environment variables
└── README.md             # Usage instructions
```

### Quick Start with Downloaded Files

```bash
# Extract the artifact
tar -xzf dekart-docker-image-oss.tar.gz  # If using image artifact
unzip dekart-docker-compose-oss.zip

cd output

# Load the Docker image (if downloaded)
docker load -i ../dekart-oss.tar.gz

# Configure environment
cp .env.example .env
# Edit .env and set your DEKART_MAPBOX_TOKEN

# Start services
docker compose up -d

# Access the application
open http://localhost:8080
```

## Workflow Outputs

After successful execution, the workflow produces:

1. **Docker Image Artifact** (`dekart-docker-image-{target}`)
   - Compressed tarball (~500MB-1GB)
   - Contains the full Dekart application image
   - Can be loaded with `docker load`

2. **Docker Compose Artifact** (`dekart-docker-compose-{target}`)
   - `docker-compose.yml` - Pre-configured with correct image reference
   - `.env.example` - Template for environment variables
   - `README.md` - Setup instructions

3. **Build Summary**
   - Displayed in the workflow run page
   - Shows target, image tag, and download links

## Troubleshooting

### No Artifacts Appearing

1. Check that `save_artifact` input was set to `true`
2. Verify the workflow completed successfully (green checkmark)
3. Look for the "Artifacts" section at the bottom of the workflow run page
4. Check the debug step output to confirm files were created

### Docker Image Load Fails

```bash
# Verify the file exists and has content
ls -lh dekart-oss.tar.gz

# Check if it's a valid gzip file
file dekart-oss.tar.gz

# Try loading with verbose output
docker load -i dekart-oss.tar.gz --verbose

# List available images after loading
docker images | grep dekart
```

### Docker Compose Issues

```bash
# Validate the compose file
docker compose config

# Check service status
docker compose ps

# View logs
docker compose logs -f dekart

# Restart services
docker compose restart

# Full reset (WARNING: deletes data)
docker compose down -v
docker compose up -d
```

## Architecture Notes

### Why Separate Workflow?

The new `build-complete.yaml` is separate from CI/CD workflows because:
- CI workflows optimize for speed by skipping unchanged components
- Release/deployment workflows need complete, reproducible builds
- End users need downloadable artifacts, not just deployed containers

### Image Tagging Strategy

- **Local/Artifact builds**: Use commit SHA (`dekartxyz/dekart:abc123`)
- **Registry pushes**: Include target type (`dekartxyz/dekart:oss-abc123`)
- **Premium/Cloud**: Use appropriate registry (GHCR or GCP Artifact Registry)

## Security Considerations

1. **Secrets Required** (for premium/cloud builds):
   - `DEKART_LICENSE_KEY` - License validation
   - `PREMIUM_GHCR_USERNAME` / `PREMIUM_GHCR_TOKEN` - GHCR access
   - `NPM_GH_TOKEN` - Private npm packages

2. **Artifact Retention**: Set to 7 days by default to balance storage costs and availability

3. **Registry Access**: Only push to registries when explicitly enabled via workflow input

## Future Improvements

Consider these enhancements:
- Multi-platform builds (ARM64 support)
- Automated release tagging
- Integration with container vulnerability scanning
- Automatic changelog generation
- Deployment to staging environments
