# GitHub Repository Dropbox Backup Workflow

This guide explains how to set up automatic backups of a GitHub repository to Dropbox using a reusable GitHub Actions workflow stored in the `personal-workflows` repository.

---

## **1. Prerequisites**

- A GitHub account
- A Dropbox account
- A repository in the `personal-workflows` organization that contains the reusable workflow:
  - `.github/workflows/backup-to-dropbox.yml`

---

## **2. Create a Dropbox App**

1. Open the Dropbox App Console: [https://www.dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)
2. Click **"Create App"**
3. Select **Scoped Access**
4. Choose **App Folder** (so the app can only access a single folder in your Dropbox)
5. Enter an **App Name** (e.g., `GitHub Repositories Backup`)
6. Click **Create App**
7. Set **Permissions**:
   - In the app settings, go to the **Permissions** tab
   - Enable **files.content.write** and **files.content.read**
   - Click **Submit** to save
8. Note the **App key** and **App secret**
   - In the app settings, go to the **Settings** tab
   - Copy the **App key** and **App secret** for the next steps

---

## **3. Generate a Dropbox Refresh Token (one-time)**

Dropbox does not show refresh tokens in the UI. They must be generated using a one-time OAuth authorization flow.

1. Generate an authorization code by opening the following URL in your browser  
   (replace `APP_KEY` with your Dropbox App key):

   ```
   https://www.dropbox.com/oauth2/authorize?client_id=APP_KEY&response_type=code&token_access_type=offline
   ```

2. Approve access when prompted
3. Click "Allow" when prompted
4. Dropbox will redirect you to a page showing an **Access code** — copy it  
5. Exchange that code for a refresh token by running this command in your terminal  
   (replace the values with your **App key**, **App secret**, and the **Access code**):

   ```bash
   curl https://api.dropboxapi.com/oauth2/token \
     -d code=YOUR_ACCESS_CODE \
     -d grant_type=authorization_code \
     -d client_id=YOUR_APP_KEY \
     -d client_secret=YOUR_APP_SECRET
   ```

6. The response will include several fields. Copy the value of `"refresh_token"`.
   - This is the token your GitHub Actions workflow will use from now on.
7. Have the **App key** and **App secret** (copied from the Dropbox app settings page) and the **refresh_token** (from the previous step) copied for the next steps.

---

## **4. Add the Dropbox Token to Your Repository Secrets**

You must add **three** secrets to every repository that uses the Dropbox backup workflow.

1. Go to the repository you want to automatically back up
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and fill in the prompts for each of the following repository secrets:

| Secret Name             | Value to Paste                            |
|-------------------------|-------------------------------------------|
| `DROPBOX_CLIENT_ID`     | Your Dropbox **App key**                  |
| `DROPBOX_CLIENT_SECRET` | Your Dropbox **App secret**               |
| `DROPBOX_REFRESH_TOKEN` | Your generated **refresh_token**          |

4. Save all three secrets

---

## **5. Add the Backup Workflow to Your Repository (Manually via GitHub UI)**

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
    uses: louiekotler/personal-workflows/.github/workflows/backup-to-dropbox.yml@main
    secrets:
      DROPBOX_CLIENT_ID: ${{ secrets.DROPBOX_CLIENT_ID }}
      DROPBOX_CLIENT_SECRET: ${{ secrets.DROPBOX_CLIENT_SECRET }}
      DROPBOX_REFRESH_TOKEN: ${{ secrets.DROPBOX_REFRESH_TOKEN }}
```

6. Commit and push
7. Once pushed, the workflow will automatically trigger on any push to any branch.
8. This process can be repeated using the same secrets for multiple repositories! Each repository will backup to its own sub-folder on Dropbox.

---

## 6. How It Works

- The workflow in `personal-workflows` does the following:  
  1. Checks out the calling repository  
  2. Creates a zip archive of the repository  
  3. Names the backup as: backup-<branch>-<commit>-<timestamp>.zip  
  4. Uploads the zip file to a Dropbox folder named after the repository

- Example Dropbox path for repository `my-repo`:  
  /my-repo/backup-main-a1b2c3d-2025-10-16T23-45-00.zip

---

## 7. Notes

- To backup only certain branches, modify the branches section in the workflow trigger  
- The workflow runs in the context of the calling repository, so no changes are needed in `personal-workflows` for each repository  

---

## 8. Verify the Setup

1. Push any commit to your repository  
2. Go to the Actions tab of your repository  
3. Check that the workflow ran successfully  
4. Verify that the zip file appeared in the corresponding Dropbox folder

---

## 9. Troubleshooting

- Error: DROPBOX_* secret is null  
  - Ensure **all three** secrets exist in the calling repository  

- Workflow fails on push  
  - Ensure `.github/workflows/backup.yml` references the correct branch of `personal-workflows`  
  - Ensure your Dropbox app has **files.content.write** permission  

---

## 10. Summary

1. Create Dropbox app  
2. Generate refresh token  
3. Add the three secrets (`DROPBOX_CLIENT_ID`, `DROPBOX_CLIENT_SECRET`, `DROPBOX_REFRESH_TOKEN`)  
4. Add `.github/workflows/backup.yml` in the repository  
5. Push to trigger backup  

> Your repository is now automatically backed up to Dropbox on every push.
