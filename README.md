<div align="center">
  <h1 align="center">Dekart</h1>
  <h3>Create Kepler.gl maps with SQL <div>for BigQuery, Snowflake, Postgres</div></h3>
  <div><code>🤖 🛑 Every line in this repo is reviewed by a human</code></div>
</div>

<br/>

<div align="center">
  <a href="https://dekart.xyz/self-hosted/?ref=github-license"><img alt="License" src="https://img.shields.io/badge/license-AGPLv3-purple"></a>
</div>

<br/>

Self-hosted alternative to CARTO & Foursquare Studio for your data warehouse.

## Features

* Shareable map links
* Manage data access & sharing
* Up-to-date maps from BigQuery, Snowflake, Wherobots datasets

<br/>
<p align="center"><a href="https://dekart.xyz/?ref=github-pic"><img alt="Self-hosted alternative to CARTO & Foursquare Studio for your data warehouse." src=".github/images/github-screencast.gif"></a></p>
<div align="center">
  <a href="https://dekart.xyz/?ref=github-try-live-demo"><img alt="Live Demo" src="https://img.shields.io/badge/Live%20Demo-blue?style=for-the-badge"></a>
</div>

## Live Map Examples

* [BigQuery](https://dekart.xyz/docs/about/overture-maps-examples/)
* [Snowflake](https://dekart.xyz/docs/about/snowflake-kepler-gl-examples/)
* [Postgres](https://dekart.xyz/docs/self-hosting/keycloak-reverse-proxy/)
* [Wherobots](https://dekart.xyz/docs/usage/wherobots-sql-tutorial/)


## How it works

Dekart is a self-hosted backend for Kepler.gl, built with Golang and React. It connects to your data warehouse, caches query results, and serves them to the frontend for visualization.

## Deployment Options

Dekart is single Docker container that can be deployed to any cloud provider or on-premises server. It requires a PostgreSQL database to store user data and Cloud Storage for caching query results.

👉 [Documentation](https://dekart.xyz/docs/configuration/environment-variables/)

⭐️ **Press GitHub Star to Get Notified of Updates**

### Quick Start with Docker Compose

For the fastest way to get started, use one of the pre-configured Docker Compose setups:

```bash
# Navigate to Docker Compose examples
cd install/docker-compose

# For BigQuery setup (requires GCP credentials)
docker compose -f docker-compose.bigquery.yaml up -d

# For Snowflake with SQLite (simplest for testing)
docker compose -f docker-compose.snowflake-sqlite.yaml up -d

# For local development (PostgreSQL + Adminer only)
docker compose -f docker-compose.local.yaml up -d
```

See [install/docker-compose/README.md](install/docker-compose/README.md) for all available configurations.

### Build Your Own Custom Image

To build a custom Docker image with your specific configuration:

1. Go to Actions → "Complete Docker Compose Build" workflow
2. Click "Run workflow" and select your target (oss/premium/cloud)
3. Download the generated artifacts (Docker image + Docker Compose files)
4. Follow the included README for installation instructions

See [DOCKER_COMPOSE_BUILD_GUIDE.md](DOCKER_COMPOSE_BUILD_GUIDE.md) for detailed instructions.

### Deployment Guides:

- [Run with Docker](https://dekart.xyz/docs/self-hosting/docker/?ref=github)
- [Run with Docker Compose profiles](https://dekart.xyz/docs/self-hosting/docker-compose/?ref=github)
- [Docker Compose examples by setup](install/docker-compose/README.md)
- [Deploy to AWS/ECS (Terraform)](https://dekart.xyz/docs/self-hosting/aws-ecs-terraform/?ref=github)
- [Deploy to Google App Engine](https://dekart.xyz/docs/self-hosting/app-engine/?ref=github)
- [Enable SSO for self-hosted instance](https://dekart.xyz/docs/self-hosting/enable-sso-open-source-instance/?ref=github)

## Support

* [Slack Community](https://slack.dekart.xyz)
## License

This project is open source under the GNU Affero General Public License Version 3 (AGPLv3) or any later version.

[Commercial Licenses Available](https://dekart.xyz/self-hosted/)

Copyright (c) 2025 Volodymyr Bilonenko
