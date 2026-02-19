# CI/CD Setup Instructions

This project uses GitHub Actions to automatically deploy to your Raspberry Pi via Tailscale.

## Required GitHub Secrets

You need to add the following secrets to your GitHub repository:

1. Go to your repository on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** for each of the following:

### Option 1: Using Tailscale AuthKey (Recommended)

Use the workflow file: `.github/workflows/deploy-authkey.yml`

**Required Secrets:**
- `TAILSCALE_AUTHKEY` - Your Tailscale auth key
  - Generate at: https://login.tailscale.com/admin/settings/keys
  - Make it reusable and set an appropriate expiration
  
- `SSH_KEY` - Your private SSH key
  - The contents of your `github-deploy` file (private key)
  - Make sure the corresponding public key is in `~/.ssh/authorized_keys` on your Raspberry Pi
  
- `RASPBERRY_PI_HOST` - Tailscale hostname or IP of your Raspberry Pi
  - Example: `raspberry-pi` or `100.x.x.x`
  - Find it by running `tailscale status` on your Raspberry Pi
  
- `RASPBERRY_PI_USER` - SSH username on Raspberry Pi
  - Example: `pi` or your custom username

### Option 2: Using Tailscale OAuth (Alternative)

Use the workflow file: `.github/workflows/deploy.yml`

**Required Secrets:**
- `TAILSCALE_OAUTH_CLIENT_ID` - OAuth client ID
- `TAILSCALE_OAUTH_SECRET` - OAuth client secret
  - Generate both at: https://login.tailscale.com/admin/settings/oauth
  - Create a new OAuth client with appropriate permissions
  
- `SSH_KEY` - Your private SSH key
- `RASPBERRY_PI_HOST` - Tailscale hostname or IP
- `RASPBERRY_PI_USER` - SSH username

## Raspberry Pi Setup

1. **Install Tailscale on your Raspberry Pi:**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```

2. **Install Docker and Docker Compose:**
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   ```

3. **Clone your repository:**
   ```bash
   cd ~
   git clone <your-repo-url> chic-deployment
   cd chic-deployment
   ```

4. **Add the public SSH key:**
   ```bash
   mkdir -p ~/.ssh
   cat github-deploy.pub >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

5. **Test the initial deployment manually:**
   ```bash
   docker-compose up -d
   ```

## Workflow Triggers

The deployment workflow runs automatically when:
- You push to the `main` branch
- You manually trigger it from the GitHub Actions tab

## Troubleshooting

- **Tailscale connection fails:** Check that your auth key is valid and hasn't expired
- **SSH connection fails:** Verify the SSH key is correctly added to authorized_keys on the Raspberry Pi
- **Docker commands fail:** Ensure your user is in the docker group on the Raspberry Pi
- **Port conflicts:** Make sure ports 3000, 8000, and 27017 are available on your Raspberry Pi

## Customization

You can modify the deployment script in the workflow file to:
- Change the deployment directory path
- Add environment-specific configurations
- Run database migrations
- Send deployment notifications
