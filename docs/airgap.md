# Air-gapped mode

Air-gapped mode is for sensitive data, such as clinical records or unpublished results. When it is on, an analysis runs end to end with no route for data to leave your infrastructure:

- **Code execution has no network.** The containers that run the agent's analysis code start with networking disabled.
- **The agent container sits behind a default-deny firewall.** It can reach only the database, the internal execution broker, and the model endpoint (or a local proxy in front of it).
- **Literature search is local.** `search_pubmed` queries a local copy of the PubMed/MEDLINE baseline (about 40 million articles) in PostgreSQL instead of NCBI.

Air-gapped mode is available on self-hosted installs only, not on www.openscientist.io.

## Turn it on

1. Choose a model provider that can be air-gapped (see below), then run the setup wizard. It asks whether to enable air-gapped mode:

    ```bash
    uv run openscientist setup
    ```

    On a running instance, an admin can instead flip the **Air-gap** switch under **Admin → Settings**; it applies to jobs started from then on.

    To force it on so it can't be switched off in the panel, set it in `.env`:

    ```bash
    OPENSCIENTIST_AIRGAPPED=true
    ```

2. Start OpenScientist with `uv run openscientist`. If the PubMed mirror is empty, it offers to load it. You can also load it later:

    ```bash
    docker compose run --rm migrate python -m openscientist.pubmed_mirror
    ```

    The load is a large one-time download from NCBI, so run it before you disconnect the host.

## Model providers

| Provider | Air-gapped? | How the model is reached |
|---|---|---|
| Ollama, vLLM, llama.cpp | Yes | On your own hardware; nothing leaves the network |
| Anthropic API, OpenAI (API key), Azure OpenAI, Azure AI Foundry, CBORG, Bedrock (API key) | Yes | Through a proxy in the web container; the agent container never holds the key or reaches the internet |
| Vertex AI, Bedrock (AWS keys or profile) | Yes, with a private endpoint | The container reaches the provider directly, so you must front it with a fixed private endpoint (Private Google Access for Vertex, an AWS PrivateLink endpoint for Bedrock) |
| OpenAI via ChatGPT login (no API key) | No | Refused; use an `OPENAI_API_KEY` instead |

For no data leaving your infrastructure at all, use a [local model](getting-started.md#option-7-local-models-ollama-vllm-llamacpp). With a hosted provider, prompts still go to that provider; air-gapped mode guarantees nothing else leaves.

## What changes for users

Jobs run as usual. Literature search returns results from the local mirror, which is as current as its last load. In our benchmark (the OpenScientist paper, Case Study 3), air-gapped runs reached the same central conclusions as online runs, with somewhat fewer literature searches.
