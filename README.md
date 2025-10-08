# GCP Service Template

This repository is a template for creating and deploying a containerized web application to the GCP server infrastructure. It is designed to be a self-contained unit, managing its own deployment configuration and network provisioning through a fully automated CI/CD pipeline.

The pipeline will:
1.  **Build** the application into a Docker image.
2.  **Push** the image to a private Google Artifact Registry.
3.  **Provision** a public-facing hostname in Cloudflare for the app, including the necessary SSL certificates.
4.  **Deploy** the application by copying its configuration to the server and triggering a central Ansible playbook.

## Repository Structure

* **`.github/workflows/build-and-push.yml`**: The core CI/CD pipeline that automates the entire build and deployment process.
* **`compose.yml`**: A Docker Compose file that defines how this specific application should run on the server.
* **`Dockerfile`**: Instructions for building the application into a Docker image.
* **`index.html`**: The sample application code. Replace this with your own.

---

## How to Use This Template for a New App

Follow these steps to create and deploy a new application. The key principle is that you will **only need to make changes within this new repository.**

### Step 1: Create the New Repository

1.  Create a new repository on GitHub using this one as a template.
2.  Clone the new repository to your local machine. Let's assume the new repository is named `my-new-app`.

### Step 2: Configure Your New Application

You need to update three key pieces of information to match your new app's name. Let's say your app's subdomain will be `new-app`.

1.  **Update the CI/CD Workflow (`.github/workflows/build-and-push.yml`):**
    * Change the `HOSTNAME` environment variable to the desired public URL.
        ```yaml
        env:
          HOSTNAME: new-app.apps.matthewcarroll.ca
        ```
    * Update the Docker image `tags` to match your new repository's name.
        ```yaml
        - name: Build and push Docker image
          # ...
          with:
            # ...
            tags: us-central1-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/app-images/my-new-app:latest
        ```

2.  **Update the Docker Compose File (`compose.yml`):**
    * The service name **must** follow the pattern `<subdomain>-app`.
    * Update the `image` to match the tag from your workflow file.
        ```yaml
        services:
          new-app-app:
            image: us-central1-docker.pkg.dev/matthewc/app-images/my-new-app:latest
            restart: unless-stopped
        ```

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

The GitHub Actions pipeline will automatically trigger, build your new image, provision the hostname, copy your `compose.yml` to the server, and trigger the central Ansible playbook to deploy the changes.