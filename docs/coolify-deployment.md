# Deploying Automaker to Coolify

This guide covers deploying Automaker to Coolify using your Claude Pro/Max subscription (no API key required).

## Prerequisites

- A Coolify server with SSH access
- Claude Pro or Claude Max subscription
- Your fork of this repository

## Step 1: Authenticate Claude CLI on Coolify Server

SSH into your Coolify server and set up Claude Code CLI:

```bash
# SSH into your Coolify server
ssh user@your-coolify-server

# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Authenticate with your Claude account
claude login

# Verify authentication
claude --version

# Note the .claude config path (you'll need this)
ls -la ~/.claude
# Usually: /root/.claude or /home/username/.claude
```

## Step 2: Add Repository in Coolify

1. Go to your Coolify dashboard
2. Click **Add Resource** → **Docker Compose**
3. Select **GitHub** as the source
4. Connect your GitHub account if not already connected
5. Select your forked repository: `omniwave-ai/automaker`
6. Set the **Compose File** to: `docker-compose.coolify.yml`

## Step 3: Configure Environment Variables

In Coolify's environment variables section, add:

### Required

| Variable             | Value                     | Description                                  |
| -------------------- | ------------------------- | -------------------------------------------- |
| `CLAUDE_CONFIG_PATH` | `/root/.claude`           | Path to .claude on your server (from Step 1) |
| `CORS_ORIGIN`        | `https://your-domain.com` | Your Coolify domain                          |

### Recommended

| Variable            | Value                         | Description                             |
| ------------------- | ----------------------------- | --------------------------------------- |
| `AUTOMAKER_API_KEY` | `<random-string>`             | API authentication key                  |
| `TERMINAL_PASSWORD` | `<password>`                  | Protect terminal access                 |
| `VITE_SERVER_URL`   | `https://api.your-domain.com` | Backend URL (if using separate domains) |

### Optional

| Variable                 | Default     | Description                     |
| ------------------------ | ----------- | ------------------------------- |
| `ALLOWED_ROOT_DIRECTORY` | `/projects` | Restrict file access            |
| `TERMINAL_ENABLED`       | `true`      | Enable/disable terminal         |
| `GH_TOKEN`               | -           | GitHub token for git operations |

## Step 4: Configure Storage

In Coolify, ensure persistent storage is configured:

1. Go to **Storage** settings
2. Add a volume mount:
   - **Volume Name**: `automaker-data`
   - **Mount Path**: `/data`

## Step 5: Configure Domains

### Option A: Single Domain (Recommended)

Map your domain to the `ui` service on port `3007`:

- `automaker.yourdomain.com` → `ui:80`

The UI proxies API requests to the server internally.

### Option B: Separate Domains

If you need separate domains for UI and API:

1. UI: `automaker.yourdomain.com` → `ui:80`
2. API: `api.automaker.yourdomain.com` → `server:3008`

Set `VITE_SERVER_URL=https://api.automaker.yourdomain.com` in environment variables.

## Step 6: Deploy

1. Click **Deploy** in Coolify
2. Wait for the build to complete (first build takes ~5-10 minutes)
3. Access your Automaker instance at your configured domain

## Verification

After deployment:

1. Open your Automaker URL
2. Go to **Settings** → **Setup**
3. Click **Verify CLI Authentication**
4. Should show: ✅ Claude CLI authenticated

## Troubleshooting

### "Claude CLI not authenticated"

1. SSH into your Coolify server
2. Run `claude login` again
3. Verify: `claude --version`
4. Check the mount path matches `CLAUDE_CONFIG_PATH`

### Container can't access .claude directory

1. Check file permissions: `ls -la ~/.claude`
2. Ensure the container user (automaker:1001) can read it
3. Try: `chmod -R 755 ~/.claude`

### Health check failing

1. Check server logs in Coolify
2. Verify port 3008 is exposed
3. Test: `curl http://localhost:3008/api/health`

### CORS errors

1. Verify `CORS_ORIGIN` matches your exact domain (including https://)
2. Check for trailing slashes (don't include them)

## Updating

To update your deployment:

1. Push changes to your fork
2. In Coolify, click **Redeploy**

Or enable automatic deployments:

1. Go to **Settings** → **Webhooks**
2. Enable GitHub webhook for auto-deploy on push

## Security Recommendations

1. **Set `AUTOMAKER_API_KEY`** - Protects API endpoints
2. **Set `TERMINAL_PASSWORD`** - Protects terminal access
3. **Use HTTPS** - Configure SSL in Coolify
4. **Restrict `ALLOWED_ROOT_DIRECTORY`** - Limit file system access
5. **Regular updates** - Keep your fork synced with upstream
