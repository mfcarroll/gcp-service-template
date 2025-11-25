# GCP Service Template

This repository is a template for deploying a containerized web application to a GCP server infrastructure managed by Ansible and Terraform.

The process is fully automated using a reusable GitHub Actions workflow. To deploy a new application, you only need to change **one line of code**.

## How to Deploy a New App

### Step 1: Create the New Repository
1.  Create a new repository on GitHub by selecting **"Use this template"**.
2.  Clone your new repository to your local machine.

### Step 2: Grant GitHub Actions Access to GCP

**This is a mandatory one-time setup step for each new repository.** The CI/CD pipeline uses GCP's Workload Identity Federation to securely access your cloud resources without needing a static secret key. You must authorize your new repository to use this identity.

1.  Go to the [IAM & Admin > Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts) page in the Google Cloud Console.
2.  Click on the service account used for GitHub Actions (e.g., `github-actions-builder@...`).
3.  Go to the **Principals with access** tab.
4.  Click **Grant Access**.
5.  In the **New principals** field, add the following line, replacing `<YOUR_GITHUB_ORG>/<YOUR_NEW_REPO_NAME>` with your repository's path:
    ```
    principalSet://iam.googleapis.com/projects/<PROJECT_NUMBER>/locations/global/workloadIdentityPools/github-pool/attribute.repository/<YOUR_GITHUB_ORG>/<YOUR_NEW_REPO_NAME>
    ```
    *(You can find your Project Number on the GCP Console Dashboard).*
6.  Assign the **Workload Identity User** role.
7.  Click **Save**.

### Step 3: Configure the CI/CD Pipeline
The only change required is in the `.github/workflows/build-and-push.yml` file. Update the `appname` to your desired application name, which will also be used as its public subdomain.

For example, to deploy your app to `my-new-app.apps.matthewcarroll.ca`:
```yaml
name: Build, Push, and Deploy

on:
  push:
    branches: ["main"]

jobs:
  call-reusable-workflow:
    uses: mfcarroll/gcp-server-config/.github/workflows/reusable-deploy.yml@main
    permissions:
      contents: 'read'
      id-token: 'write'
    with:
      appname: my-new-app # <-- CHANGE THIS LINE
    secrets:
      inherit
```

### Step 4: Add GitHub Secrets
The reusable deployment workflow requires secrets to access your cloud providers. Go to your new repository's **Settings > Secrets and variables > Actions** and add the following secrets.

**Core Infrastructure Secrets:**
* `SERVER_IP`: The public IP address of the application server.
* `DOMAIN_NAME`: The root domain for your applications (e.g., `apps.mydomain.com`).

**Provider Credentials:**
* `ANSIBLE_SSH_PRIVATE_KEY`: The private SSH key used to connect to the GCP server for deployments.
* `ANSIBLE_VAULT_PASSWORD`: The password for the Ansible Vault in the `gcp-server-config` repository.
* `CLOUDFLARE_API_TOKEN`: Cloudflare API token with permissions to edit DNS and SSL settings.
* `CLOUDFLARE_EMAIL`: The email address for your Cloudflare account.
* `CLOUDFLARE_ZONE_ID`: The Zone ID for your domain from the Cloudflare dashboard.
* `GCP_PROJECT_ID`: Your Google Cloud Project ID.
* `GCP_SERVICE_ACCOUNT`: The email of the service account used for GitHub Actions authentication.
* `GCP_WORKLOAD_IDENTITY_PROVIDER`: The full path of the Workload Identity Provider from your GCP project.
* `GH_PAT`: A GitHub Personal Access Token with `repo` scope, used to check out the private `gcp-server-config` repository.

### Step 5: Develop and Deploy
You are now ready to work on your application.

1.  Replace the contents of `index.html`, `Dockerfile`, and `compose.yml.template` with your actual application code.
2.  Commit and push your changes to the `main` branch.

The GitHub Actions pipeline will automatically trigger, build your image, provision the hostname, and deploy your application.

## To reuse this configuration elsewhere

1. Use the server-terraform repo to set up your server (needs to be checked out but does not have to be forked)

https://github.com/mfcarroll/gcp-server-terraform

2. Fork the server-config repos and configure ansible secrets. This repo now holds your specific server configuration, to be reused by all deployed apps.

https://github.com/mfcarroll/gcp-server-config

3. Fork the service-template repo and adjust `build-and-push.yml` to point back to your github account. This is now your template to use for deployment.

https://github.com/mfcarroll/gcp-service-template/

4. Make copies of your own service-template repo as needed to spin up apps.
