# Deployment

This guide explains how to deploy OpenScientist using Docker.

## Prerequisites

- Docker and Docker Compose installed
- LLM provider credentials (see `.env.example` for options)

## Quick Start

### 1. Configure, build and start

From the repository root:

```bash
uv run openscientist
```

On first run a setup wizard writes `.env`, `.env.db` and `.env.owner`, generating the database passwords and the master secret; then it builds the images and starts the stack. The web refuses to start if `OPENSCIENTIST_SECRET_KEY` is missing, still the `.env.example` placeholder, or shorter than 64 characters. Later runs skip the wizard. Stop with `uv run openscientist stop`; re-run the wizard with `uv run openscientist setup`.

### 2. Access the web interface

Open your browser and navigate to:

```
http://localhost:8080
```

## Configuration

### Environment Variables

See `.env.example` for the full list of configuration options, and
[Getting started](getting-started.md#configuration) for provider setup.

### Volumes

The Docker setup mounts the following directories:

- `./jobs`: Persists discovery job data
- `./data`: Stores uploaded data files

The web runs as a non-root user in the jobs group (gid 1001), so `./jobs` must be group-owned by gid 1001 with the setgid bit. On a fresh checkout:

```bash
chgrp 1001 ./jobs && chmod 2770 ./jobs
```

### Ports

- `8080`: Web interface, published on `127.0.0.1` by default; set `OPENSCIENTIST_BIND_ADDRESS` to publish it on another host address

### Container launcher

A separate `launcher` service is the only part of the stack with access to the Docker socket. The web asks it, over a unix socket, to start, stop and list job and executor containers; neither the web nor the job containers can reach Docker themselves. The launcher builds every container from one of two fixed templates, accepts only the image digests recorded at build time (in `.env.owner`) and resource values within the `LAUNCHER_*` bounds in `.env`, and only acts on containers it created. `uv run openscientist` builds the images and records their digests for you.

## Managing Jobs

### View jobs

```bash
docker compose exec openscientist python -m openscientist.job_manager list
```

### Get job details

```bash
docker compose exec openscientist python -m openscientist.job_manager get <job_id>
```

### Delete a job

```bash
docker compose exec openscientist python -m openscientist.job_manager delete <job_id>
```

### Clean up old jobs

```bash
# Delete jobs older than 7 days (keep completed)
docker compose exec openscientist python -m openscientist.job_manager cleanup --days 7

# Delete all old jobs including completed
docker compose exec openscientist python -m openscientist.job_manager cleanup --days 7 --delete-completed
```

## Monitoring

### View logs

```bash
# Follow logs
docker compose logs -f

# View last 100 lines
docker compose logs --tail 100
```

### Check container status

```bash
docker compose ps
```

### Health check

The container includes a health check that pings the web interface every 30 seconds.

```bash
docker inspect openscientist-openscientist-1 --format='{{.State.Health.Status}}'
```

## Backup and Restore

### Backup jobs

```bash
# Create backup
tar -czf openscientist-jobs-backup-$(date +%Y%m%d).tar.gz jobs/

# Copy out of container
docker cp openscientist-openscientist-1:/app/jobs ./jobs-backup
```

### Restore jobs

```bash
# Extract backup
tar -xzf openscientist-jobs-backup-YYYYMMDD.tar.gz

# Copy into container
docker cp ./jobs openscientist-openscientist-1:/app/
```

## Troubleshooting

### Container won't start

1. Check logs:
   ```bash
   docker compose logs
   ```

2. Verify environment variables:
   ```bash
   docker compose config
   ```

3. Rebuild and restart (keeps job data):
   ```bash
   uv run openscientist stop
   uv run openscientist
   ```

### Budget exceeded

Check spend on the admin **Billing** page, and raise or clear the limits in the admin **Settings** panel. See [Admin Settings](getting-started.md#admin-settings).

### Job stuck in "running" state

1. Check job manager:
   ```bash
   docker compose exec openscientist python -m openscientist.job_manager get <job_id>
   ```

2. Check job logs:
   ```bash
   cat jobs/<job_id>/claude_iterations.log
   ```

3. Cancel the job through the [REST API](api.md):
   ```bash
   curl -X POST -H "Authorization: Bearer $OS_KEY" \
     http://localhost:8080/api/v1/jobs/<job_id>/cancel
   ```

## Production Deployment

### Security Considerations

1. **Use secrets management**: Don't commit `.env` to git
   ```bash
   # Add to .gitignore
   echo ".env" >> .gitignore
   ```

2. **Restrict access**: Use a reverse proxy (nginx) with authentication
   ```nginx
   location / {
       auth_basic "OpenScientist";
       auth_basic_user_file /etc/nginx/.htpasswd;
       proxy_pass http://localhost:8080;
   }
   ```

3. **Use HTTPS**: Set up SSL/TLS certificates
   ```bash
   certbot --nginx -d openscientist.yourdomain.com
   ```

4. **Limit resource usage**: Add resource limits to docker-compose.yml
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 4G
   ```

### Scaling

For multiple concurrent jobs, raise **Max concurrent jobs** under Admin → Settings (default 1); container CPU and memory are set there too. See [Admin Settings](getting-started.md#admin-settings).

### Monitoring and Alerting

Set up monitoring with Prometheus/Grafana:

1. Add metrics endpoint to web app
2. Configure Prometheus to scrape metrics
3. Create Grafana dashboards for job status, costs, etc.

## Updating OpenScientist

```bash
# Pull latest code
git pull origin main

# Rebuild the images and restart
uv run openscientist stop
uv run openscientist
```

To update a remote server, run the same commands there over SSH.

## Uninstalling

```bash
# Stop and remove containers
docker compose down

# Remove volumes (WARNING: deletes all job data)
docker compose down -v

# Remove images
docker rmi openscientist-openscientist
```

## Support

For issues or questions, open an issue on GitHub.
