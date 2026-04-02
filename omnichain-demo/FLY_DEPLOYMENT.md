# Fly.io Deployment Guide

## Prerequisites

1. Install the Fly CLI:
   ```bash
   # Windows (PowerShell)
   iwr https://fly.io/install.ps1 -useb | iex
   
   # macOS/Linux
   curl -L https://fly.io/install.sh | sh
   ```

2. Authenticate with Fly.io:
   ```bash
   fly auth login
   ```

## Initial Deployment

1. Navigate to the project directory:
   ```bash
   cd omnichain-demo
   ```

2. Launch the app (this will create the app and deploy it):
   ```bash
   fly launch
   ```
   
   When prompted:
   - Choose your app name (or use the default: `blockchain-escrow`)
   - Select a region (default: `iad` - Ashburn, Virginia)
   - Don't create a Postgres database
   - Don't create a Redis database
   - The `fly.toml` is already configured, so it will use that

3. Alternatively, if you want to create the app separately:
   ```bash
   fly apps create blockchain-escrow
   fly deploy
   ```

## Subsequent Deployments

After the initial deployment, simply run:
```bash
fly deploy
```

## Monitoring

View logs:
```bash
fly logs
```

Check app status:
```bash
fly status
```

Open the app in your browser:
```bash
fly open
```

## Scaling

Adjust resources if needed:
```bash
# Scale to 2GB RAM
fly scale memory 2048

# Scale to 2 CPUs
fly scale count 2
```

## Secrets Management

If you need to add environment variables as secrets:
```bash
fly secrets set MY_SECRET=value
```

## Troubleshooting

### View running processes
```bash
fly ssh console
supervisorctl status
```

### Restart the app
```bash
fly apps restart blockchain-escrow
```

### Check supervisor logs inside the container
```bash
fly ssh console
tail -f /var/log/supervisor/supervisord.log
```

## Architecture

This deployment uses **supervisord** to manage multiple processes in a single container:

1. **Anvil** (priority 1): Local Ethereum node on port 8545
2. **Deploy Contract** (priority 2): One-time deployment script
3. **Relayer** (priority 3): API server and dashboard on port 3001

The processes start in order based on priority, ensuring Anvil is ready before contract deployment, and the contract is deployed before the relayer starts.
