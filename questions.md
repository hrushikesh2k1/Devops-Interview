1. Explain about a bad day recently you faced in your work and how did you solve it?
   "A recent change introduced in-memory caching without proper limits, which caused memory usage to grow under high traffic. The container crossed its Kubernetes memory limit and was OOMKilled.
   I identified it using pod events and resource metrics, stabilized the service by increasing memory temporarily, then fixed the cache behavior and added monitoring to prevent it from happening again.”

   “I updated the Deployment resource to increase the container memory limits, which triggered a rolling update. Once the new pods came up with higher limits, the OOMKills stopped and the service stabilized.”

2. Did you face any issues with terraform drift? if yes, explain the scenario?
   “We faced infrastructure drift when an emergency automation script modified Azure network rules outside
   Terraform during a production incident. Since Terraform was our source of truth, this change wasn’t reflected in the state file.
    We later detected the drift during a scheduled Terraform plan in CI/CD, paused further applies, and
   evaluated whether the emergency change was temporary or required long term. We then updated the Terraform code to match
   the desired state and applied it to reconcile the infrastructure. To prevent recurrence, we enforced Terraform-only changes through CI/CD,
   restricted direct access via RBAC, and added scheduled drift detection."

3. What is terrform remote backend and why it is critical?
   “A Terraform backend defines where and how the state file is stored and managed. In team environments,
   we use a remote backend like Azure Blob or S3 because it enables shared state, locking, and consistency.
   This prevents multiple engineers or pipelines from modifying infrastructure at the same time and avoids
   race conditions, state corruption, and duplicate resource creation.”
   ```bash
   terraform {
        required_providers {
            azurerm = {
                 source = ""
                 version = ""
            }
        }
        backend "azurerm" {
           resource_group = ""
           storage_account_name = ""
           container_name = ""
           key = ""
        }
   }
   ```

   Advantages of using remote backend over local state file
   “Remote backends solve several problems that local state cannot.
    They prevent race conditions and concurrent applies by using state locking, so two users or pipelines can’t modify infrastructure at the same time.
    They enable shared state, so all engineers and CI/CD pipelines work against the same source of truth.
    They also provide versioning and backups, which allows us to recover the state if it gets corrupted or deleted.

4. Difference between git pull and fetch?
   “git fetch only downloads remote changes without modifying my branch, while git pull fetches and then merges those changes into the current branch.”

5. Explain CICD approach in your organization?
   “In our project, we implemented a CI/CD pipeline using Git, Jenkins, Docker, and Kubernetes, following a GitOps-based deployment approach.”
   We use GitHub/GitLab/Bitbucket as our source code repository.
   Developers raise PRs, code is reviewed, and once merged, a commit triggers the pipeline via a Git webhook.
   Pipeline stages:
   Checkout Stage
   Jenkins checks out the latest code from the Git repository.
   ### Build & Unit Testing
   We build the application (for example, using Maven for Java).
   Unit tests are executed as part of the pipeline.
   Optional static checks like linting/formatting can be included.
   ### Code Quality & Security Scan
   We perform static code analysis using tools like SonarQube.
   This helps detect code smells, bugs, and security vulnerabilities early.
   ### Container Image Build
   Since our target platform is Kubernetes, we build a container image using a Dockerfile from the repo.
   ### Image Security Scan
   The built image is scanned for vulnerabilities (base image + dependencies).
   ### Push to Image Registry
   Once validated, the image is pushed to an image registry like Docker Hub / ECR / private registry.
   After the image is pushed, we update Kubernetes manifests or Helm charts with the new image tag.
   These manifests are stored in a Git repository (same repo or a separate deployment repo).
   We use a GitOps tool like Argo CD.

   Argo CD continuously watches the manifest/Helm repo.

   When a new commit is detected, it automatically syncs and deploys the changes to the Kubernetes cluster.

  Git remains the single source of truth, and Argo CD ensures continuous reconciliation.

  (If GitOps is not used)

  Deployment can also be done via kubectl/Helm commands using shell scripts, Ansible, or Python from Jenkins.

5. Suddenly your build stage has stopped, what would you do?
   “When a build stage stops suddenly, I first check the pipeline logs to identify the exact step and error message.
   Common reasons include test failures, dependency issues, resource limits on the runner, or security scans blocking the build.
   Once I understand whether it’s a code issue or an infrastructure issue, I either fix the code, retry the build, or escalate appropriately.”

   “We had a CI pipeline where the build stage suddenly failed without any code changes.
   I first checked the pipeline logs and saw the failure occurred during dependency installation in the Docker build.
   The root cause was an unpinned dependency pulling an incompatible version from an external repository.
   I fixed it by pinning the dependency version, re-ran the pipeline, and confirmed the build succeeded.
   After that, we improved dependency management and caching to avoid similar issues.”

   
