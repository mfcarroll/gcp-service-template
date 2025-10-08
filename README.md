# GCP Service Template

This repository is a template for creating and deploying a containerized web application to the GCP server infrastructure. It includes a sample web page and a fully automated CI/CD pipeline using GitHub Actions.

The pipeline will:
1.  **Build** the application into a Docker image.
2.  **Push** the image to a private Google Artifact Registry.
3.  **Provision** a public-facing hostname in Cloudflare for the app.
4.  **Deploy** the application to the server by triggering an Ansible playbook.

## How to Use This Template for a New App

Follow these steps to create and deploy a new application based on this template.

### Step 1: Create the New Repository

1.  Create a new repository on GitHub by using this one as a template.
2.  Clone the new repository to your local machine.

### Step 2: Grant GitHub Actions Access to GCP

**This is a mandatory one-time setup step for each new repository.** The CI/CD pipeline uses GCP's Workload Identity Federation to securely access your cloud resources without needing a static secret key. You must authorize your new repository to use this identity.

1.  Go to the [IAM & Admin > Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts) page in the Google Cloud Console.
2.  Click on the service account used for GitHub Actions (e.g., `github-actions-builder@...`).
3.  Go to the **Permissions** tab.
4.  Click **Grant Access**.
5.  In the **New principals** field, add the following line, replacing `<YOUR_GITHUB_ORG>/<YOUR_NEW_REPO_NAME>` with your repository's path:
    ```
    principalSet://[iam.googleapis.com/projects/](https://iam.googleapis.com/projects/)<PROJECT_NUMBER>/locations/global/workloadIdentityPools/github-pool/attribute.repository/<YOUR_GITHUB_ORG>/<YOUR_NEW_REPO_NAME>
    ```
    *(You can find your Project Number on the GCP Console Dashboard).*
6.  Assign the **Workload Identity User** role.
7.  Click **Save**.

### Step 3: Configure the CI/CD Pipeline

Open the `.github/workflows/build-and-push.yml` file and update the `APPNAME` environment variable to the desired subdomain for your new app. For example:
```yaml
env:
  # --- THE ONLY LINE TO CHANGE FOR A NEW APP ---
  APPNAME: my-new-app
```

### Step 4: Add GitHub Secrets

Go to your new repository's **Settings > Secrets and variables > Actions** and add the following secrets.

* `ANSIBLE_SSH_PRIVATE_KEY`: The **contents** of the **passphrase-free** private SSH key (`~/.ssh/id_ed25519_cicd`) used for automated deployments.
* `ANSIBLE_VAULT_PASSWORD`: The password for the Ansible Vault in the `gcp-server-config` repository.
* `CLOUDFLARE_API_TOKEN`: The Cloudflare API token with `Zone:SSL and Certificates:Edit`, `Zone:Zone:Read`, and `Zone:DNS:Edit` permissions.
* `CLOUDFLARE_EMAIL`: The email address for your Cloudflare account.
* `CLOUDFLARE_ZONE_ID`: The Zone ID for your `matthewcarroll.ca` domain.
* `GCP_PROJECT_ID`: Your Google Cloud Project ID (e.g., `matthewc`).
* `GCP_SERVICE_ACCOUNT`: The email address of the `github-actions-builder` service account.
* `GCP_WORKLOAD_IDENTITY_PROVIDER`: The full path of the Workload Identity Provider from your GCP project.
* `GH_PAT`: A GitHub Personal Access Token with `repo` scope, used to check out the private `gcp-server-config` repository.

### Step 5: Develop and Deploy

You are now ready to work on your application.

1.  Replace the contents of `index.html` and the `Dockerfile` with your actual application code.
2.  Commit and push your changes to the `main` branch.

The GitHub Actions pipeline will automatically trigger, build your new image, provision the hostname, deploy your application.
