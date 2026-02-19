# Submodule Auto-Deploy Setup

This setup automatically triggers deployment to your Raspberry Pi whenever you push to the frontend or backend submodules.

## Setup Instructions

### 1. Create a Personal Access Token (PAT)

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a name like "Submodule Deploy Trigger"
4. Select these permissions:
   - `repo` (Full control of private repositories)
5. Click "Generate token"
6. **Copy the token immediately** (you won't see it again)

### 2. Add Secrets to Submodule Repositories

For **both** the frontend and backend repositories:

1. Go to the repository on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Add these secrets:
   - `PARENT_REPO_TOKEN` - The PAT you just created
   - `PARENT_REPO` - Your parent repo in format `owner/repo-name` (e.g., `yourname/chic-deployment`)

### 3. How It Works

```
Frontend/Backend Push → Trigger Workflow → Repository Dispatch → Parent Repo Deploy
```

1. You push to `frontend` or `backend` main branch
2. The trigger workflow runs in that submodule repo
3. It sends a `repository_dispatch` event to the parent repo
4. The parent repo's deploy.yml workflow triggers
5. Deployment to Raspberry Pi happens automatically

### 4. Testing

Push to any submodule:
```bash
# In frontend or backend directory
git add .
git commit -m "Test deployment trigger"
git push
```

Check the Actions tab in:
- The submodule repo (should show "Trigger Parent Deployment" workflow)
- The parent repo (should show "Deploy to Raspberry Pi 5" workflow starting)

### 5. Manual Deployment

You can still trigger deployment manually from the parent repo's Actions tab using the "Run workflow" button.
