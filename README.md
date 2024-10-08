# harness-lab

This lab will guide you through setting up and using Harness Open Source (referred to as "Harness" from now on), with a focus on managing a project, using GitSpaces, creating pipelines, and setting up an artifact registry. By the end of the lab, you'll be able to create a project, import a repository, work with GitSpaces, and automate build pipelines.

## Prerequisites

Before starting, ensure you have the following installed on your local machine:

- Docker
- VS Code (optional but recommended for working with GitSpaces)

If Docker is not already installed on your system, you can set it up using [Colima](https://github.com/abiosoft/colima?tab=readme-ov-file#installation), a lightweight container runtime for macOS and Linux. Note that you'll still need the Docker CLI (Docker client) to interact with Colima, so ensure that the Docker client is installed and configured on your machine.

## Setup and Installation

1. Create a common network:

```bash
docker network create harness
```

2. Run the following command to start a Harness instance that uses the `harness` network:

```bash
docker run -d \
  --network=harness \
  -e GITNESS_CI_CONTAINER_NETWORKS=harness \
  -e GITNESS_URL_REGISTRY=http://harness:3000 \
  -p 3000:3000 -p 3022:3022 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /tmp/harness:/data \
  --name harness \
  --restart always \
  harness/harness
```

This command starts the Harness server, exposes it on port 3000, and mounts necessary volumes for Docker and persistent data storage.

3. Follow these steps to create an admin user:

   - Once the container is running, open http://localhost:3000 in your browser.
   - Select **Sign Up**.
   - Enter a User ID (`admin`), Email (`admin@example.com`), and Password (`changeit`).
   - Select **Sign Up**. (You might see a warning to change your password. You can ignore that warning.)

## Project and Repository

### New Project

1. Select **New Project**.
2. Enter a project Name (**harness-lab**) and optional Description (**Open source code hosting, pipelines, artifact registry, dev environments**).
3. Select **Create Project**.
   > [!NOTE]
   > Harness can also [import projects](https://docs.gitness.com/administration/project-management#import-a-project) from external sources (such as GitLab groups or GitHub organizations).
4. You can organize your work in Harness by creating labels to categorize pull requests, artifacts, and more. To get started, add three labels to your project: "dev," "staging," and "prod." You can also assign specific values to each of these labels, helping to streamline project management and tracking.

### Import a Repository

1. Click on the drop-down under **Repositories**, and select "Import Repository".
2. Use `harness-community` for the organization and `podinfo` for the repository.
3. Click **Import Repository**.

### Webhook

You can send data to HTTP endpoints from actions in your repository, such as opened pull requests, new branches, and more. For this exercise, you’ll use [webhook.site](https://webhook.site) - a website that offers unique, random URLs to instantly receive and inspect all incoming HTTP requests and webhooks in real-time, facilitating testing and debugging. For free webhook.site users, the URL and its data are kept for 7 days. You can close the browser tab and still return to the same unique webhook.site URL.

1. Navigate to webhook.site and copy your unique URL.
2. Click on **Webhooks** under the podinfo repository and then **+ New Webhook**.
3. Give this webhook a name: `trigger_on_branch_created`.
4. Paste the unique URL you copied under Payload URL. You can leave out the Secret.
5. Choose **Let me select individual events** and select **Branch created**.
6. Click **Create Webhook**.

You'll need to reuse this webhook URL in a later section. Go to **Secrets --> + New Secret** and add a new secret called **webhook_url**. Use the value of the unique URL you have copied.

Now, continue to the next section to push a new branch. Once a new branch is pushed, you’ll see the trigger in action on this site.

### Create a Branch to Trigger Webhook

1. Within the podinfo repository, create a new branch named "feature".
2. On webhook.site, you should see a notification indicating that the webhook was triggered.

The response will look something like this:

```json
{
  "trigger": "branch_created",
  "repo": {
    "id": 1,
    "path": "testproj/podinfo",
    "identifier": "podinfo",
    "description": "",
    "default_branch": "master",
    "url": "http://159.203.33.47:3000/testproj/podinfo",
    "git_url": "http://159.203.33.47:3000/git/testproj/podinfo.git",
    "git_ssh_url": "ssh://git@159.203.33.47:3022/testproj/podinfo.git",
    "uid": "podinfo"
  },
  "principal": {
    "id": 4,
    "uid": "admin",
    "display_name": "Administrator",
    "email": "mail@example.com",
    "type": "user",
    "created": 1724895740977,
    "updated": 1724895740977
  },
  "ref": {
    "name": "refs/heads/feature3",
    "repo": {
      "id": 1,
      "path": "testproj/podinfo",
      "identifier": "podinfo",
      "description": "",
      "default_branch": "master",
      "url": "http://159.203.33.47:3000/testproj/podinfo",
      "git_url": "http://159.203.33.47:3000/git/testproj/podinfo.git",
      "git_ssh_url": "ssh://git@159.203.33.47:3022/testproj/podinfo.git",
      "uid": "podinfo"
    }
  },
  "sha": "dbf831f84f486243998a2f86cda9fa76d9f1b748",
  "head_commit": {
    "sha": "dbf831f84f486243998a2f86cda9fa76d9f1b748",
    "message": "Updated pipeline testpipe",
    "author": {
      "identity": { "name": "Administrator", "email": "mail@example.com" },
      "when": "2024-09-03T17:38:34Z"
    },
    "committer": {
      "identity": { "name": "Gitness", "email": "system@gitness.io" },
      "when": "2024-09-03T17:38:34Z"
    },
    "added": [],
    "removed": [],
    "modified": []
  },
  "commit": {
    "sha": "dbf831f84f486243998a2f86cda9fa76d9f1b748",
    "message": "Updated pipeline testpipe",
    "author": {
      "identity": { "name": "Administrator", "email": "mail@example.com" },
      "when": "2024-09-03T17:38:34Z"
    },
    "committer": {
      "identity": { "name": "Gitness", "email": "system@gitness.io" },
      "when": "2024-09-03T17:38:34Z"
    },
    "added": [],
    "removed": [],
    "modified": []
  },
  "old_sha": "0000000000000000000000000000000000000000",
  "forced": false
}
```

## GitSpaces

### Create a GitSpace for VS Code Desktop

1. Create a GitSpace for the `podinfo/master` branch and open it in VS Code Desktop.
2. You will need to create a token and add it to the Gitness extension on VSCode. To do so, click **Admin** and then **+ New Token**.
3. Build the binary for podinfo by running:

   ```bash
   go build ./cmd/podinfo
   ```

You should see the following error:

```
bash: go: command not found
```

GitSpaces come with an Ubuntu image (`mcr.microsoft.com/devcontainers/base:dev-ubuntu-24.04`) if you don’t have a DevContainer file with a base image defined.

In your gitspace, add the following file to your repo: `podinfo/files/master/~/.devcontainer/devcontainer.json`

```
{
    "image": "mcr.microsoft.com/devcontainers/go"
}
```

Merge the changes.

Stop and delete the GitSpace instance, then recreate it. Retry the above command, and this time, the Go build should succeed.

3. Run the app:

   ```bash
   ./podinfo
   ```

4. Open your browser and navigate to [http://localhost:9898](http://localhost:9898) to see the app running version `6.6.1`. 

### Create gitspaces for VS Code Browser

Make sure to merge your master branch into your feature branch before continuing.

1. Create a gitspaces for the `podinfo/feature` branch and open it in VS Code Browser.
2. Make a change to `pkg/version/version.go` and update the version to **6.6.2**. Save the file.
3. Build the binary for podinfo by running:

   ```bash
   go build ./cmd/podinfo
   ```

4. Run the app:

   ```bash
   ./podinfo
   ```

5. Open your browser and navigate to [http://localhost:9898](http://localhost:9898) to see the app running version `6.6.2`.

6. Commit and push the change to feature branch.

> [!NOTE]
> Observe that these gitspaces instances are already configured with git credentials from Harness Open Source so you don't have to configure git credentials.

## Secret Detection

From **Repositories --> podinfo --> Manage Repository --> Security**, enable **Secret Scanning**. Harness Open Source includes [gitleaks](https://github.com/gitleaks/gitleaks) integrations for detecting and preventing hardcoded secrets.

Now, from one of the gitspaces, create a new file called **config.yaml** and add the following:

```bash
pat.W3bJ9X4K2L8V7fH1pG0M5nQ.ZM1cP9gB5L2vJ8K6R3wY1N4z.X9V7cT3pB5M1nF2G4J0K
```

The above follows the same pattern as a Harness Personal Access Token. While this is not a valid token, the built-in scanner in Harness will detect the pattern and prevent you from pushing this commit.

This approach is much safer than detecting secrets after they've been committed.

## Pipeline

### Create a New Pipeline

1. In the podinfo repository, go to **Pipelines** and click **+ New Pipeline**.
2. Click "Generate" to let Harness automatically create a pipeline for your Go project. This pipeline should install dependencies, build the app, and run tests.
3. Click "Save and Run" to execute the pipeline and ensure all steps complete successfully.

## Artifact Registry

### Create an Artifact Registry

1. Navigate to **Artifact Registries --> + New Artifact Registry** and create a new docker artifact registry named "harness-reg".
2. In the registry settings, click **Set up client** to retrieve the connection credentials to that registry. Make a note of the username.
3. Click **Generate token** and make a note of the token.

### Add Secrets

1. Navigate to Secrets in the Harness dashboard.
2. Add two secrets:
   - `docker_username`: Use the registry username from the previous step. For most cases, this would be **admin**.
   - `docker_password`: Use the generated token from the previous step.

### Create a Pipeline for Build, Test, and Push

Create a new pipeline called "build-test-push" and use the following YAML configuration:

```yaml
kind: pipeline
spec:
  stages:
    - name: build-test-push-scan
      spec:
        platform:
          arch: amd64
          os: linux
        steps:
          - name: go_install
            spec:
              container:
                image: golang:1.23
            script:
              - go install ./...
            type: run
          - name: go_test
            spec:
              container:
                image: golang:1.23
            script:
              - go test -v ./...
            type: run
          - name: go_build_push
            type: plugin
            spec:
              name: docker
              inputs:
                insecure: true
                repo: host.docker.internal:3000/harness-lab/harness-reg/podinfo
                registry: host.docker.internal:3000
                username: ${{ secrets.get("docker_username") }}
                password: ${{ secrets.get("docker_password") }}
                tags: ${{ build.number }}
      type: ci
version: 1
```

Click **Save and Run** to execute the pipeline.

#### Troubleshooting

- For local installations, add `host.docker.internal:3000` as an insecure-registry in your docker config.
- For cloud VM installations, add `YOUR_IP:3000` as an insecure-registry in your docker config.

1. Locate your Docker configuration file:

- On Linux or macOS, this is typically located at `/etc/docker/daemon.json`.
- On Windows, you might find it at `C:\ProgramData\docker\config\daemon.json`.

2. Edit the `daemon.json` file. If the file does not exist, create it.
3. Add the following content, making sure to replace `IP:PORT` with the actual address of your insecure registry:

```json
{
  "insecure-registries": ["IP:PORT"]
}
```

4. Restart Docker: After saving your changes, restart the Docker service for the new configuration to take effect.

`sudo systemctl restart docker` or `colima restart`.

### Verify

Check the artifact registry to ensure the new image has been successfully pushed.

## Add Security Testing

In this step, we use a popular open source image scanning tool called [Grype](https://github.com/anchore/grype). The goal is to integrate Grype into the pipeline to scan the newly built image for vulnerabilities before promoting it to the prod environment.

### Updated Pipeline

Modify the pipeline to include a new step for Grype scanning:

```YAML
kind: pipeline
spec:
  stages:
    - name: build-test-push-scan
      spec:
        platform:
          arch: amd64
          os: linux
        steps:
          - name: go_install
            spec:
              container:
                image: golang:1.23
              script:
                - go install ./...
            type: run
          - name: go_test
            spec:
              container:
                image: golang:1.23
              script:
                - go test -v ./...
            type: run
          - name: go_build_push
            type: plugin
            spec:
              name: docker
              inputs:
                insecure: true
                repo: host.docker.internal:3000/harness-lab/harness-reg/podinfo
                registry: host.docker.internal:3000
                username: ${{ secrets.get("docker_username") }}
                password: ${{ secrets.get("docker_password") }}
                tags: ${{ build.number }}
          - name: Grype_Image_Scan
            type: run
            when: |
              build.event == "pull_request"
              and
              build.target == "master"
            spec:
              container: alpine
              script: |
                apk add --no-cache curl
                curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /tmp/grype-bin
                /tmp/grype-bin/grype host.docker.internal:3000/harness-lab/harness-reg/podinfo:${{ build.number }}
                echo "Image scan completed!"
      type: ci
version: 1
```

### Create a Pull Request to trigger the pipeline

From the "feature" branch, create a new Pull Request (PR) to the "master" branch. The conditions for the **Grype_Image_Scan** step will be met, and the pipeline will execute the Grype scan as part of the CI process.

### Pipeline Execution

The default trigger should automatically kick off the "build-test-push" pipeline. Ensure that a new image is built and pushed to the image registry.

### Notifications

Update the pipeline to add a notifications step on build failure.

```
kind: pipeline
spec:
  stages:
    - name: build-test-push-scan
      spec:
        platform:
          arch: amd64
          os: linux
        steps:
          - name: go_install
            spec:
              container:
                image: golang:1.23
              script:
                - go install ./...
            type: run
          - name: go_test
            spec:
              container:
                image: golang:1.23
              script:
                - go test -v ./...
            type: run
          - name: go_build_push
            type: plugin
            spec:
              name: docker
              inputs:
                insecure: true
                repo: host.docker.internal:3000/testproj/testreg/podinfo
                registry: host.docker.internal:3000
                username: ${{ secrets.get("docker_username") }}
                password: ${{ secrets.get("docker_password") }}
                tags: ${{ build.number }}
          - name: Grype_Image_Scan
            type: run
            when: |
              build.event == "pull_request"
              and
              build.target == "master"
            spec:
              container: alpine
              script: |
                apk add --no-cache curl
                curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /tmp/grype-bin
                /tmp/grype-bin/grype host.docker.internal:3000/testproj/testreg/podinfo:${{ build.number }}
                echo "Image scan completed!"
          - name: notify
            type: plugin
            when: failure()
            spec:
              name: webhook
              inputs:
                content_type: application/json
                urls: ${{ secrets.get("webhook_url") }}
                template: |
                  Name: Harness Build Notification
                  Repo Name: {{ repo.name }}
                  Build Number {{ build.number }}
                  Build Event: {{ build.event }}
                  Build Status: {{ build.status }}
      type: ci
version: 1
```

Since the Grype_Image_Scan step will fail, the notification step will be triggered and you'll see a notification like this on webhook.site:

```
Name: Harness Build Notification
Repo Name: podinfo
Build Number 9
Build Event: pull_request
Build Status: failure
```

## API

Check out the [Swagger API](http://localhost:3000/swagger) to programmatically create and manage Harness resources. You'll need to [generate a token](https://developer.harness.io/docs/open-source/administration/user-management#generate-user-token) to get started.

## Next Steps

This was just a teaser of what you can do with Harness Open Source. Check out the [docs](https://developer.harness.io/docs/open-source/overview) to build something awesome with Harness.
