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

### Step 2: Configure the CI/CD Pipeline

Open the `.github/workflows/build-and-push.yml` file and make two changes:

1.  **Update the `HOSTNAME`:** Change the `env.HOSTNAME` variable to the desired public URL for your new app. For example:
    ```yaml
    env:
      HOSTNAME: my-new-app.apps.matthewcarroll.ca
    ```
2.  **Update the `tags`:** Change the Docker image tag to match your new application's name. This should match the name of your repository. For example:
    ```yaml
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: us-central1-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/app-images/my-new-app:latest
    ```

### Step 3: Add GitHub Secrets

Go to your new repository's **Settings > Secrets and variables > Actions** and add the following secrets. These are required for the pipeline to authenticate with Google Cloud, Cloudflare, and the Ansible server configuration repository.

* `ANSIBLE_SSH_PRIVATE_KEY`: The private SSH key (`~/.ssh/id_ed25519_gcp`) used to connect to the GCP server.
* `ANSIBLE_VAULT_PASSWORD`: The password for the Ansible Vault in the `gcp-server-config` repository.
* `CLOUDFLARE_API_TOKEN`: The Cloudflare API token with permissions to edit DNS and SSL settings.
* `CLOUDFLARE_EMAIL`: The email address for your Cloudflare account.
* `CLOUDFLARE_ZONE_ID`: The Zone ID for your `matthewcarroll.ca` domain from the Cloudflare dashboard.
* `GCP_PROJECT_ID`: Your Google Cloud Project ID (e.g., `matthewc`).
* `GCP_SERVICE_ACCOUNT`: The email address of the `github-actions-builder` service account.
* `GCP_WORKLOAD_IDENTITY_PROVIDER`: The full path of the Workload Identity Provider from your GCP project.
* `GH_PAT`: A GitHub Personal Access Token with `repo` scope, used to check out the private `gcp-server-config` repository.

### Step 4: Update the Server Configuration

The CI/CD pipeline will automatically deploy your application, but you need to tell the server how to run it by adding it to the main `docker-compose.yml` file.

1.  Clone your `gcp-server-config` repository to your local machine.
2.  Open the `templates/docker-compose.yml.j2` file.
3.  Add a new service definition for your application. The service name **must** be `<subdomain>-app`. For example, if your hostname is `my-new-app.apps.matthewcarroll.ca`, the service name is `my-new-app-app`:
    ```yaml
    # ... (caddy service is already here) ...

    template-app:
      image: us-central1-docker.pkg.dev/{{ gcp_project_id }}/app-images/gcp-service-template:latest
      restart: unless-stopped

    my-new-app-app:
      image: us-central1-docker.pkg.dev/{{ gcp_project_id }}/app-images/my-new-app:latest
      restart: unless-stopped
    ```
4.  Commit and push this change to your `gcp-server-config` repository.

### Step 5: Develop and Deploy

You are now ready to work on your application.

1.  Replace the contents of `index.html` and the `Dockerfile` with your actual application code.
2.  Commit and push your changes to the `main` branch.

The GitHub Actions pipeline will automatically trigger, build your new image, provision the hostname, and deploy your application.
