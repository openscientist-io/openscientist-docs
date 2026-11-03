# Getting started

There are two ways to use OpenScientist:

1. **Run jobs on our live instance** at [www.openscientist.io](https://www.openscientist.io). Sign in, wait for an administrator to approve your account, and start a job. Nothing to install.
2. **Install your own instance on premises** from the [GitHub repository](https://github.com/openscientist-io/openscientist), as described below.

## Prerequisites

- Python 3.12+
- Docker (for containerized deployment)
- `uv` package manager
- One of the following for model access:
    - **Anthropic API**: an API key or a Claude subscription
    - **OpenAI**: an API key or a ChatGPT login
    - **Azure OpenAI**: an Azure OpenAI resource with a model deployment
    - **Vertex AI**: GCP project with Vertex AI enabled
    - **AWS Bedrock**: AWS account with Bedrock access
    - **Azure AI Foundry**: Azure subscription with Foundry resource
    - **Local models**: an Ollama, vLLM or llama.cpp server on your own hardware
    - **CBORG** (LBNL staff only): API token from [CBORG](https://cborg.lbl.gov)

See [Configuration](#configuration) below for how to set up each provider.

## Installation

```bash
git clone https://github.com/openscientist-io/openscientist.git
cd openscientist

# First run starts a setup wizard, then builds and starts the stack
uv run openscientist
```

The wizard asks for your model provider and its credentials, air-gapped mode, the site address and admin emails, and checks the provider with a live call. It writes `.env`, `.env.db` and `.env.owner` and generates the secret key and database passwords, so there is nothing to edit by hand. For scripted installs, `uv run openscientist setup --answers answers.toml` takes every answer from a file instead of prompting; `--write-answers answers.toml` saves a run's answers (never the secrets) for reuse. Limits and resources are set afterwards in the admin [Settings panel](#admin-settings).

Later runs start OpenScientist directly. Stop it with `uv run openscientist stop`, and re-run the wizard with `uv run openscientist setup`.

For sensitive data, see [Air-gapped mode](airgap.md).

## Access the UI

Open your browser to `http://localhost:8080`.

## Usage

1. Upload your data files (optional - supports CSV, TSV, Excel, Parquet, JSON, PDB, mmCIF, FASTA, images, and many other file types)
2. Enter your research question
3. Set maximum iterations (e.g., 10)
4. Click "Start Discovery"
5. Monitor progress and view results

To share a finished job, use the job's share dialog to share it with named users, or switch on **Anyone signed in can view this job** to make it public. Public jobs are reached by link and stay out of other users' job lists; readers get everything except your uploaded input files. Each run records its environment (OpenScientist commit, agent harness and version, container), shown on the job page, in the report footer and as `provenance.json` in the artifacts zip.

## Configuration

### Model Providers

The wizard writes these settings for you. The blocks below show what ends up in `.env` for each provider, for reference or for editing by hand.

#### Option 1: Anthropic API

```bash
OPENSCIENTIST_PROVIDER=anthropic

# Option A: API key from https://console.anthropic.com
ANTHROPIC_API_KEY=sk-ant-api03-your-key-here

# Option B: a Claude subscription login (run `claude setup-token`)
# CLAUDE_CODE_OAUTH_TOKEN=...
```

The simplest setup: one key, no cloud account. Runs on the Claude Code agent harness.

#### Option 2: OpenAI

```bash
OPENSCIENTIST_PROVIDER=openai

# Option A: API key
OPENAI_API_KEY=sk-your-openai-key

# Option B: a ChatGPT login (from `codex login`)
# CODEX_AUTH_FILE=/home/you/.codex/auth.json
```

Runs on the Codex agent harness.

#### Option 3: Azure OpenAI

```bash
OPENSCIENTIST_PROVIDER=azure-openai
# Required: your Azure deployment name
OPENSCIENTIST_MODEL=your-deployment-name

AZURE_OPENAI_API_KEY=your-azure-openai-key
# The <resource> in https://<resource>.openai.azure.com
AZURE_OPENAI_RESOURCE=your-resource
# Optional; defaults to 2025-04-01-preview
# AZURE_OPENAI_API_VERSION=2025-04-01-preview
```

OpenAI models through your Azure subscription, on the Codex agent harness.

#### Option 4: Google Vertex AI

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=vertex

# Vertex AI configuration
ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project
# Host path to the service-account key; Docker mounts it into the containers
GCP_CREDENTIALS_FILE=/path/to/service-account.json
CLOUD_ML_REGION=us-east5
VERTEX_REGION_CLAUDE_4_5_SONNET=us-east5
VERTEX_REGION_CLAUDE_4_5_HAIKU=us-east5

# BigQuery billing export (for cost tracking)
GCP_BILLING_ACCOUNT_ID=XXXXXX-YYYYYY-ZZZZZZ
```

**Cost Tracking**: Via GCP BigQuery billing export (1-6 hour lag)

#### Option 5: AWS Bedrock

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

#### Option 6: Azure AI Foundry (Microsoft Foundry)

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
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
```

**Cost Tracking**: Via Azure Cost Management (1-3 day lag)

**Setup Guide**: See [Claude Code Foundry docs](https://code.claude.com/docs/en/microsoft-foundry)

**Note**: Requires Azure RBAC permissions (`Azure AI User` or `Cognitive Services User` role)

#### Option 7: Local models (Ollama, vLLM, llama.cpp)

Open-weight models can run on your own hardware and be served to OpenScientist over an OpenAI-compatible endpoint. Three local servers are supported; all are driven by the omp agent harness. Pick one and point OpenScientist at it:

=== "Ollama"

    ```bash
    OPENSCIENTIST_PROVIDER=ollama
    OLLAMA_BASE_URL=http://host.docker.internal:11434/v1
    # Optional; defaults to gpt-oss:20b
    OPENSCIENTIST_MODEL=gpt-oss:20b
    # Set to the num_ctx Ollama runs with (it cannot report this before the model is loaded)
    OPENSCIENTIST_MODEL_CONTEXT_TOKENS=32768
    ```

=== "vLLM"

    ```bash
    OPENSCIENTIST_PROVIDER=vllm
    VLLM_BASE_URL=http://host.docker.internal:8000/v1
    # Required: the model name vLLM was launched with
    OPENSCIENTIST_MODEL=your-served-model
    # Only if vLLM was started with --api-key
    # VLLM_API_KEY=...
    ```

=== "llama.cpp"

    ```bash
    OPENSCIENTIST_PROVIDER=llamacpp
    # llama-server defaults to port 8080, which OpenScientist also uses; start it with --port 8081
    LLAMACPP_BASE_URL=http://host.docker.internal:8081/v1
    # Required: the model name llama-server serves
    OPENSCIENTIST_MODEL=your-served-model
    # Only if llama-server was started with --api-key
    # LLAMACPP_API_KEY=...
    # The context window is read from llama-server's /props automatically
    ```

Use `host.docker.internal` (as above) when the model server runs on the same machine as OpenScientist, because agents run inside containers; use `localhost` only when running OpenScientist from source outside Docker. No API key or internet access is required, which makes local models the natural choice for [air-gapped mode](airgap.md).

**Cost Tracking**: None needed; inference runs on your own hardware.

#### Option 8: CBORG (Lawrence Berkeley National Lab)

Only relevant inside Lawrence Berkeley National Laboratory, which runs CBORG as an internal model gateway.

```bash
# Provider selection
OPENSCIENTIST_PROVIDER=cborg

# CBORG credentials
ANTHROPIC_AUTH_TOKEN=your-cborg-token
ANTHROPIC_BASE_URL=https://api.cborg.lbl.gov
```

**Cost Tracking**: Real-time via CBORG API (`/key/info`, `/user/daily/activity`)

### Admin Settings

Once OpenScientist is running, admins set limits and resources under **Admin → Settings**. Changes take effect without a restart.

| Group | Settings |
| --- | --- |
| Budget | Job cost limit, Project spend total, Project spend (24 h), Project spend warning (24 h); each in USD or no limit |
| Containers | Agent and executor CPU, memory and timeout |
| Jobs | Max file size, Max concurrent jobs (0 pauses new starts), Scheduler window (hours of each user's recent use that decide whose queued job starts next) |
| Login | Session length (days) |
| Security | Air-gap switch (see [Air-gapped mode](airgap.md)) |

Budget limits are checked when a job is created, when a queued job starts, on every iteration, and on chat turns and report regeneration. A running job that hits a limit stops iterating, still writes its report from what it found, and is marked as stopped early with a note naming the limit. Spend and warnings appear on the admin **Billing** page. When any limit is set, hosted models without a known price are refused, since their spend can't be tracked; local models (Ollama, vLLM, llama.cpp) are exempt.

Under **Admin → Users**, admins approve new users and can mark a user as **priority**: their queued jobs start ahead of others' at the next free slot (running jobs are never interrupted).

The matching `.env` variables (for example `MAX_JOB_COST_USD`, `OPENSCIENTIST_MAX_CONCURRENT_JOBS`) only set the starting values on first start; after that the Settings panel is what counts.

## Next Steps

- [Deployment](deployment.md) — managing jobs, monitoring, production deployment
- [Architecture](architecture.md) — how OpenScientist works under the hood
