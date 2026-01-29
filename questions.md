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

6. How to check open ports?
   “I use ss -tulnp to see all listening TCP/UDP ports along with the process using them. It helps identify which service is bound to which port.”

7. What is hard link and soft link?
   “A hard link points to the same inode so it still works if the original file is deleted."
   “A soft link is like a shortcut that lets applications access files from a fixed path even if the real file lives somewhere else.”
   ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/app.conf


9. What is cherry picking?
   “Cherry-pick is used to apply a specific commit from one branch to another, usually for moving a hotfix without merging the full branch.”
   git cherry-pick <commit_id>

10. WHat is WebHook in CICD?
    “A webhook is used to notify the CI/CD system when a code change happens, like a Git push.
    It automatically triggers the pipeline to build, test, and deploy the application.”
   
11. What branching strategy do you follow, and how do you handle merges to avoid breaking the release branch? If a bug appears in production, what’s your approach to resolving it?
    --Typical flow:
    Developers work on feature/* branches
    They raise PRs into develops
    After testing, develop is merged into main for release
    --To protect production:
    main is read-only
    Only PRs from develop or hotfix are allowed
    CI must pass before merge
    --If a prod bug happens:
    Create hotfix/bug-123 from main
    Fix the bug
    Merge into main
    cherry-pick the same commit into develop
    This keeps prod fixed without breaking ongoing work.

12. Image scanning in CICD
    “Trivy doesn’t just scan the image tag — it extracts every layer, reads OS packages and application dependencies,
    and compares them against CVE databases, so we catch vulnerable libraries before deployment.”
    trivy image myapp:1.0
    trivy scan four types of scanning inside the docker image
    | Layer         | What it checks              |
    | ------------- | --------------------------- |
    | OS packages   | apt, apk, yum packages      |
    | App libraries | npm, pip, maven, etc        |
    | Config files  | Dockerfile, Kubernetes YAML |
    | Secrets       | hardcoded passwords, tokens |
    Compares all versions against vulnerability DB (NVD, GitHub, RedHat CVEs)
    trivy image --exit-code 1 --severity CRITICAL,HIGH myapp:1.0 ---- [pipeline code]
    Kubernetes YAML Scanning
    --[Code] trivy config deployment.yaml
    --It checks:
    Privileged container
    Running as root
    No resource limits
    Open ports
    This prevents bad Kubernetes security before deployment.

13. Difference between Github hosted runners and self-hosted runners?
    --Github hosted runners
    No setup needed
    Fresh VM for every job
    Has preinstalled tools (Docker, Node, Java, etc)
    Deleted after the job finishes
    When to use this? when you want quick setup, clean environments, and standard builds

    --self-hosted runners
    Full control of OS and tools
    Persistent environment
    Can access private networks
    Can run custom software

14. How rollbacks are done?
    --In CICD, we 
    Tag Docker images with build ID
    Store them in ACR
    Deploy via Helm

    every helm deployment created a release, to check the release history
    helm history <image>
    helm rollback <image> <revision_no.>

    helm rollback <image> --this rollback to the previous revision.

15. What checks do you include before allowing a deployment to production?
    “We allow production deployment only after passing automated tests, security scans, quality gates,
    artifact validation, and manual approval, followed by health checks and rollback protection.”

16. Explain SonarQube in detail?
    SonarQube is a Static Application Security Testing (SAST) and code quality platform.
    It analyses source code before it is compiled without running the application to find bugs, security vulnerabilities, bad coding practises.
    How SonarQube detects vulnerabilities:
    --it uses
    OWASP Top 10--Top 10 most dangerous web application vulnerabilities.
    CWE standards--It is a huge list of coding mistakes that cause security issues.
    Secure code rules
    --So it detects
    Hardcoded passwords
    Unsafe encryption
    Broken auth
    Command injection
    Secrets in code
    “OWASP defines the top security risks in applications, and CWE classifies the exact coding mistakes that lead to those risks — SonarQube uses both to detect vulnerabilities in code.”

    Sonar quality gate checks
    No critical bugs
    No security vulnerabilities
    Coverage > 80%
    Duplications < 3%

    
