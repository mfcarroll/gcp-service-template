# GCP Service Template

This repository is a template for creating and deploying a containerized web application to the GCP server infrastructure. It is designed to be a self-contained unit, managing its own deployment configuration and network provisioning through a fully automated CI/CD pipeline.

The pipeline will:
1.  **Build** the application into a Docker image.
2.  **Push** the image to a private Google Artifact Registry.
3.  **Provision** a public-facing hostname in Cloudflare for the app, including the necessary SSL certificates.
4.  **Deploy** the application by copying its configuration to the server and triggering a central Ansible playbook.

## Repository Structure

* **`.github/workflows/build-and-push.yml`**: The core CI/CD pipeline. **This is the main file you will edit.**
* **`compose.yml.template`**: A template for the application's Docker Compose configuration. This is generated into the final `compose.yml` by the pipeline.
* **`Dockerfile`**: Instructions for building the application into a Docker image.
* **`index.html`**: The sample application code. Replace this with your own.

---

## How to Use This Template for a New App

Follow these steps to create and deploy a new application. The key principle is that you only need to make changes within your new application's repository.

### Step 1: Create the New Repository

1.  Create a new repository on GitHub using this one as a template.
2.  Clone the new repository to your local machine.

### Step 2: Configure Your New Application

You only need to define your application's name in **one place**.

1.  **Update the CI/CD Workflow (`.github/workflows/build-and-push.yml`):**
    * Open this file and change the `APPNAME` environment variable to the desired name for your new application (e.g., `my-cool-app`). This will be used for the subdomain and the Docker image tag.

        ```yaml
        env:
          # --- THIS IS THE ONLY LINE TO CHANGE FOR A NEW APP ---
          APPNAME: my-cool-app
        ```

2.  **Update the Compose Template (`compose.yml.template`):**
    * This file uses a placeholder, `__APP_NAME__`, which is automatically replaced by the `APPNAME` from your workflow during deployment. You only need to edit this if your application requires specific environment variables or other Docker Compose settings.

### Step 3: Add GitHub Secrets

Go to your new repository's **Settings > Secrets and variables > Actions** and add the following secrets. These are required for the pipeline to authenticate with all the necessary services.

* `ANSIBLE_SSH_PRIVATE_KEY`: The **contents** of the private SSH key (`~/.ssh/id_ed25519_gcp`) used to connect to the GCP server.
* `ANSIBLE_VAULT_PASSWORD`: The password for the Ansible Vault in the `gcp-server-config` repository.
* `CLOUDFLARE_API_TOKEN`: The Cloudflare API token with `Zone:SSL and Certificates:Edit`, `Zone:Zone:Read`, and `Zone:DNS:Edit` permissions.
* `CLOUDFLARE_EMAIL`: The email address for your Cloudflare account.
* `CLOUDFLARE_ZONE_ID`: The Zone ID for your `matthewcarroll.ca` domain (found on the Cloudflare dashboard's "Overview" page).
* `GCP_PROJECT_ID`: Your Google Cloud Project ID (e.g., `matthewc`).
* `GCP_SERVICE_ACCOUNT`: The email address of the `github-actions-builder` service account.
* `GCP_WORKLOAD_IDENTITY_PROVIDER`: The full path of the Workload Identity Provider from your GCP project (e.g., `projects/123.../providers/github-provider`).
* `GH_PAT`: A GitHub Personal Access Token with `repo` scope, used to check out the private `gcp-server-config` repository.

### Step 4: Develop and Deploy

You are now ready to work on your application.

1.  Replace the contents of `index.html` and the `Dockerfile` with your actual application code.
2.  Commit and push your changes to the `main` branch.

The GitHub Actions pipeline will automatically trigger, build your image, provision the hostname, generate and copy your `compose.yml` to the server, and trigger the central Ansible playbook to deploy the changes.