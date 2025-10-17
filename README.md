# GitHub Repository Dropbox Backup Workflow

This guide explains how to set up automatic backups of a GitHub repository to Dropbox using a reusable GitHub Actions workflow stored in the `personal-workflows` repository.

---

## **1. Prerequisites**

- A GitHub account
- A Dropbox account
- A repository in the `personal-workflows` organization that contains the reusable workflow:
  - `.github/workflows/backup-to-dropbox.yml`

---

## **2. Create a Dropbox App for API Access**

1. Go to the Dropbox App Console: [https://www.dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)
2. Click **"Create App"**
3. Select **Scoped Access**
4. Choose **App Folder** (so the app can only access a single folder in your Dropbox)
5. Enter an **App Name** (e.g., `GitHub Repositories Backup`)
6. Click **Create App**
7. Set **Permissions**:
   - In the app settings, go to the **Permissions** tab
   - Enable **files.content.write** and **files.content.read**
   - Click **Submit** to save
8. Generate an **Access Token**:
   - In the app settings, go to the **Settings** tab
   - Scroll to **OAuth 2** section
   - Click **Generate** under **Access token**
   - Copy the token — you will use it in GitHub

---

## **3. Add the Dropbox Token to Your Repository Secrets**

1. Go to your **personal repository** (the one you want to back up)
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name it: `DROPBOX_ACCESS_TOKEN`
5. Paste your **Dropbox access token** in the value field
6. Click **Add secret**

> ⚠️ Secrets are **per repository**. You must do this for every repository you want to back up.

---


## **4. Add the Backup Workflow to Your Repository (Manually via GitHub UI)**

If you prefer not to create files manually, you can set up the workflow entirely through GitHub’s web interface:

1. Go to your **personal repository** on GitHub.  
2. Click the **Actions** tab near the top of the page.  
3. Click **“New workflow”** or, if it suggests templates, scroll down and click **“set up a workflow yourself”**.  
4. Rename the workflow file at the top of the editor:
   - Example: `backup.yml`
5. Copy the following workflow code into the editor:

```yaml
name: Backup to Dropbox

on:
  push:
    branches:
      - '**'  # Run on all branches

jobs:
  backup_to_dropbox:
    uses: personal-workflows/backup-workflows/.github/workflows/backup.yml@main
    secrets:
      DROPBOX_ACCESS_TOKEN: ${{ secrets.DROPBOX_ACCESS_TOKEN }}
```

6. Commit and push
7. Once pushed, the workflow will automatically trigger on any push to any branch.

---

## 5. How It Works

- The workflow in `personal-workflows` does the following:  
  1. Checks out the calling repository  
  2. Creates a zip archive of the repository  
  3. Names the backup as: backup-<branch>-<commit>-<timestamp>.zip  
  4. Uploads the zip file to a Dropbox folder named after the repository

- Example Dropbox path for repository `my-repo`:  
  /my-repo/backup-main-a1b2c3d-2025-10-16T23-45-00.zip

---

## 6. Notes

- To backup only certain branches, modify the branches section in the workflow trigger  
- The workflow runs in the context of the calling repository, so no changes are needed in `personal-workflows` for each repository  

---

## 7. Verify the Setup

1. Push any commit to your repository  
2. Go to the Actions tab of your repository  
3. Check that the workflow ran successfully  
4. Verify that the zip file appeared in the corresponding Dropbox folder

---

## 8. Troubleshooting

- Error: DROPBOX_ACCESS_TOKEN is null  
  - Ensure the secret exists in the calling repository, not in `personal-workflows`  

- Workflow fails on push  
  - Ensure `.github/workflows/backup.yml` references the correct branch of `personal-workflows`  
  - Ensure Dropbox token has files.content.write permission  

---

## 9. Summary

1. Create Dropbox app & token  
2. Add token to repository secrets  
3. Add `.github/workflows/backup.yml` in the repository (manually via Actions tab or locally)  
4. Push to trigger backup  

> Your repository is now automatically backed up to Dropbox on every push.

