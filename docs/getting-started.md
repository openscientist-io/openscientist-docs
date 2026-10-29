# Getting started

This covers running OpenScientist as a Docker-based application. If you want to
work on OpenScientist's own code instead, see [Contributing](contributing.md).

## Prerequisites

- Python 3.12+
- Docker (for containerized deployment)
- `uv` package manager
- One of the following for model access:
  - **CBORG**: API token from [CBORG](https://cborg.lbl.gov)
  - **Vertex AI**: GCP project with Vertex AI enabled
  - **AWS Bedrock**: AWS account with Bedrock access
  - **Azure AI Foundry**: Azure subscription with Foundry resource

See [Configuration](#configuration) below for how to set up each provider.

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd openscientist

# Create .env file (copy from example and configure)
cp .env.example .env
# Edit .env with your provider credentials

# Build and start
make build
make start
```

## Access the UI

Open your browser to `http://localhost:8080`.

## Usage

1. Upload your data files (optional - supports CSV, TSV, Excel, Parquet, JSON, PDB, mmCIF, FASTA, images, and many other file types)
2. Enter your research question
3. Set maximum iterations (e.g., 10)
4. Click "Start Discovery"
5. Monitor progress and view results

## Configuration

### Model Providers

OpenScientist supports multiple model providers. Choose one and configure it in your `.env` file:

#### Option 1: CBORG (Lawrence Berkeley National Lab)

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=cborg

# CBORG credentials
ANTHROPIC_AUTH_TOKEN=your-cborg-token
ANTHROPIC_BASE_URL=https://api.cborg.lbl.gov
```

**Cost Tracking**: Real-time via CBORG API (`/key/info`, `/user/daily/activity`)

#### Option 2: Google Vertex AI

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=vertex

# Vertex AI configuration
ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
CLOUD_ML_REGION=us-east5
VERTEX_REGION_CLAUDE_4_5_SONNET=us-east5
VERTEX_REGION_CLAUDE_4_5_HAIKU=us-east5

# BigQuery billing export (for cost tracking)
GCP_BILLING_ACCOUNT_ID=XXXXXX-YYYYYY-ZZZZZZ
```

**Cost Tracking**: Via GCP BigQuery billing export (1-6 hour lag)

#### Option 3: AWS Bedrock

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=bedrock

# AWS configuration
AWS_REGION=us-east-1

# Authentication (choose one):
# Option A: Access keys
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key

# Option B: AWS profile
# AWS_PROFILE=your-profile-name

# Option C: Bedrock API key
# AWS_BEARER_TOKEN_BEDROCK=your-bedrock-api-key
```

**Cost Tracking**: Via AWS Cost Explorer (24-48 hour lag)

**Note**: Requires IAM permissions for `bedrock:InvokeModel` and `ce:GetCostAndUsage`

#### Option 4: Azure AI Foundry (Microsoft Foundry)

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=foundry

# Azure resource configuration
ANTHROPIC_FOUNDRY_RESOURCE=your-resource-name
# Or use full URL:
# ANTHROPIC_FOUNDRY_BASE_URL=https://your-resource.services.ai.azure.com/anthropic

# Authentication (choose one):
# Option A: API key (recommended for testing)
ANTHROPIC_FOUNDRY_API_KEY=your-azure-api-key

# Option B: Azure Entra ID (automatic - no API key needed)
# Run: az login
# Or configure managed identity for production

# Model deployment names (optional - defaults shown)
ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-5
ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5
ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-6

# For cost tracking (optional):
AZURE_SUBSCRIPTION_ID=your-subscription-id
```

**Cost Tracking**: Via Azure Cost Management API (implementation in progress)

**Setup Guide**: See [Claude Code Foundry docs](https://code.claude.com/docs/en/microsoft-foundry)

**Note**: Requires Azure RBAC permissions (`Azure AI User` or `Cognitive Services User` role)

### Budget Controls

Set application-level budget limits (optional):

```bash
# Maximum total spend across all jobs
MAX_PROJECT_SPEND_TOTAL_USD=1000

# Maximum spend in last 24 hours
MAX_PROJECT_SPEND_24H_USD=50
```

Budget limits are checked before job creation. The web UI displays:

- Total project spend
- Recent spend (last 24h)
- Budget remaining (if provider supports it)

### Other Settings

```bash
# Dev mode - enables mock OAuth login for development
OPENSCIENTIST_DEV_MODE=true

# Maximum concurrent jobs (default: 1)
OPENSCIENTIST_MAX_CONCURRENT_JOBS=1

# Directory for job data (default: jobs/)
OPENSCIENTIST_JOBS_DIR=jobs
```

## Next Steps

- [Deployment](deployment.md) — managing jobs, monitoring, production deployment
- [Architecture](architecture.md) — how OpenScientist works under the hood
