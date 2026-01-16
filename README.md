# Uptime Kuma Template

A template repository for deploying [Uptime Kuma](https://github.com/louislam/uptime-kuma) with [Kamal](https://kamal-deploy.org/) and GitHub Actions.

## Features

- 🐳 Docker-based deployment using Kamal 2
- 🗄️ MariaDB database backend
- 🔒 SSL enabled by default via Kamal Proxy
- 🚀 GitHub Actions for automated deployment
- 📦 Container registry integration with GitHub Container Registry (ghcr.io)

## Prerequisites

- A server with SSH access
- Docker installed on the server
- Ruby 4+ (managed via [mise](https://mise.jdx.dev/))
- A GitHub account with Container Registry access

## Quick Start

### 1. Clone and Configure

```bash
git clone https://github.com/<your-username>/uptime-template.git
cd uptime-template
```

### 2. Install Dependencies

Make sure you have Ruby installed (this project uses mise for version management):

```bash
# Install mise (if not already installed)
curl https://mise.run | sh

# Install Ruby
mise install

# Install gems
bundle install
```

### 3. Configure Deployment

#### Update `config/deploy.yml`

Replace `<username>` with your GitHub username:

```yaml
image: <your-github-username>/uptime-kuma
```

#### Update `config/deploy.production.yml`

Replace the server addresses and hostnames with your own:

```yaml
servers:
  web:
    - uptime.yourdomain.com

proxy:
  hosts:
    - uptime.yourdomain.com
    - status.yourdomain.com # Optional: additional hostname
```

### 4. Set Up Environment Variables

Create a `.env` file in the project root:

```bash
# GitHub Container Registry credentials
KAMAL_REGISTRY_USERNAME=your-github-username
KAMAL_REGISTRY_PASSWORD=ghp_your_personal_access_token

# Database credentials (for local deployment)
MARIADB_DATABASE=uptime_kuma
MARIADB_USER=uptime_kuma
MARIADB_PASSWORD=your_secure_password
```

> **Note:** The `.env` file is gitignored. Never commit credentials to version control.

### 5. Set Up GitHub Secrets

For GitHub Actions deployment, add these secrets to your repository:

| Secret                    | Description                                              |
| ------------------------- | -------------------------------------------------------- |
| `SSH_PRIVATE_KEY`         | SSH private key for server access                        |
| `KAMAL_REGISTRY_USERNAME` | GitHub username                                          |
| `KAMAL_REGISTRY_PASSWORD` | GitHub Personal Access Token with `write:packages` scope |
| `MARIADB_DATABASE`        | Database name (e.g., `uptime_kuma`)                      |
| `MARIADB_USER`            | Database username                                        |
| `MARIADB_PASSWORD`        | Database password                                        |

## Deployment

### Initial Setup

Run the setup workflow from GitHub Actions or deploy manually:

```bash
# First-time deployment (sets up containers and accessories)
./bin/kamal setup --destination=production
```

### Subsequent Deployments

```bash
# Deploy latest changes
./bin/kamal deploy --destination=production
```

### Using GitHub Actions

1. **Manual Deploy**: Go to Actions → Deploy → Run workflow
2. **Release Deploy**: Create a new release to trigger automatic deployment
3. **Setup**: Go to Actions → Setup → Run workflow (for first-time setup)

## Enabling Scheduled Deployments (Cron)

This template includes an optional scheduled deployment feature that can automatically deploy updates daily. This is useful for keeping Uptime Kuma up-to-date with the latest image.

To enable scheduled deployments, edit [.github/workflows/deploy.yml](.github/workflows/deploy.yml) and uncomment the schedule section:

```yaml
on:
  release:
    types: [published]
  # Uncomment to enable daily automatic deployments (runs at 7:00 AM UTC)
  schedule:
    - cron: "0 7 * * *"
  workflow_dispatch:
```

You can customize the cron schedule:

- `"0 7 * * *"` - Every day at 7:00 AM UTC
- `"0 0 * * 0"` - Every Sunday at midnight UTC
- `"0 */6 * * *"` - Every 6 hours

## Useful Commands

```bash
# View logs
./bin/kamal logs --destination=production

# Access the app container
./bin/kamal app exec -i bash --destination=production

# Check deployment status
./bin/kamal details --destination=production

# Rollback to previous version
./bin/kamal rollback --destination=production

# Restart the application
./bin/kamal app restart --destination=production

# Manage accessories (database)
./bin/kamal accessory logs db --destination=production
./bin/kamal accessory restart db --destination=production
```

## Architecture

```
┌─────────────────────────────────────────────┐
│                   Server                    │
├─────────────────────────────────────────────┤
│  ┌─────────────┐      ┌─────────────────┐   │
│  │ Kamal Proxy │──────│  Uptime Kuma    │   │
│  │ (SSL/HTTPS) │      │  (Port 3001)    │   │
│  └─────────────┘      └────────┬────────┘   │
│                                │            │
│                       ┌────────▼────────┐   │
│                       │    MariaDB      │   │
│                       │  (Port 3306)    │   │
│                       └─────────────────┘   │
└─────────────────────────────────────────────┘
```

## Data Persistence

Data is persisted in the following locations on the server:

- **Uptime Kuma data**: `/data/uptime-kuma`
- **MariaDB data**: Docker volume `uptime-kuma-db-data`

## Customization

### Using a Different Database

The default configuration uses MariaDB. Uptime Kuma also supports SQLite (default). To use SQLite instead, remove the database accessory and environment variables from `config/deploy.yml`.

### Multiple Environments

Create additional destination files (e.g., `config/deploy.staging.yml`) and deploy with:

```bash
./bin/kamal deploy --destination=staging
```

## Troubleshooting

### SSH Connection Issues

Ensure your SSH key is added and the server is accessible:

```bash
ssh -i ~/.ssh/your_key user@your-server.com
```

### Container Issues

Check container status and logs:

```bash
./bin/kamal details --destination=production
./bin/kamal logs --destination=production
```

### Lock Issues

If a deployment is stuck, release the lock:

```bash
./bin/kamal lock release --destination=production
```

## License

This template is open source. Uptime Kuma is licensed under the [MIT License](https://github.com/louislam/uptime-kuma/blob/master/LICENSE).
