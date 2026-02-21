## Question 1

You have deployed a microservices-based application on Kubernetes (AKS). Suddenly one of the critical services is returning HTTP 500 errors and latency has spiked.

👉 Walk me through your first 10–15 minutes of incident response.
What exactly will you check and in what order?

(Answer this. I’ll ask the next question after your response.)

Once the issue is noticed by prometheus alert, First thing I do is acknowledging and take responsibility. 
Second thing I do is to investigate and find out as much as details about the issue like what is the blast radius, 
how many users are effected, is it region specific, finding out which team would be responsible for this for taking this forward, what is the recent release happened. 
I collect all these details and create a slack war room and pass this information to team. I would send all hands on desk mail to relevant teams 
if I am not sure of which team would handle this/ not sure about the issue. The SME's from different teams will join the slack or teams conference call and check the issue. 
Meanwhile I will be acting as incident commander and will post the updates in the slack. I will be on-call support untill the entire issue is resolved. 
Once the incident management is started, our priority is to mitigate the issue rather than finding out rca so that customer don't face any downtime like we increase quota, 
routing traffic to different service temporarily etc., Once the issue is mitigated we troubleshoot the issue to find out rca so that next time the issue will not occur. 
We fix the issue and document the entire incident for future response.

Alright — honest interviewer feedback for Question 1 before we move on.

👍 What you did well:

You covered:

  Alert acknowledgement
  
  Blast radius thinking
  
  Recent release check
  
  War room creation
  
  Incident commander role
  
  Prioritizing mitigation over RCA
  
  Communication & documentation

That shows you understand process.

❌ What you missed (this is where most 3-year candidates fail):

Your answer was too ITIL / management-heavy and not technical enough for the first 10–15 minutes.

If a Kubernetes production service is throwing HTTP 500s + latency spike, I expect you to immediately talk about:

Checking pod health

kubectl get pods -n <ns>
kubectl describe pod <pod>
kubectl logs <pod>

Checking:

CrashLoopBackOff?

OOMKilled?

Readiness/Liveness probe failures?

Recent pod restarts?

Service endpoints:

kubectl get svc
kubectl get endpoints

Node pressure:

kubectl top node
kubectl top pod

Any HPA scaling issues?

Network?

Istio / ingress errors?

5xx at ingress vs pod?

Recent rollout:

kubectl rollout history deployment <name>

Rollback decision:

kubectl rollout undo deployment <name>

👉 In the first 10 minutes, your job is not to find the responsible team —
your job is to answer:

Is this infra issue, application issue, networking issue, or capacity issue?

You should already be deep into:

logs

metrics

events

rollout status

before creating a war room.

Current Rating for this question:

Process understanding: 7/10
Technical incident handling depth: 4/10

You need to speak more in terms of:
➡️ signals
➡️ hypotheses
➡️ kubectl actions
➡️ rollback vs scale decision

## Question 2

Your CI/CD pipeline deploys a new version of an application to AKS using a rolling update strategy.

After deployment:

Some pods are running the new version

Some are still running the old version

Users report inconsistent behavior

👉 Why does this happen in rolling deployments and how will you ensure zero-downtime + version consistency in production?

(Answer this.)

This happens when the same user is experienced the old version and new version of the application, but this is completely normal as in rolling updates rather the uninstalling all the old version pods, kubernetes natively supports rolling updates strategy where the new pods are replaced by 25% of the current pods. for example if there are 4 replicas of old version, kubernetes go ahead and remove 1 pod at a time and replaces it with new version pod. If any user raises concern or we observe any issue with the new version, the rollout will be done so that the previous good version app is deployment to avoid almost zero downtime. To ensure zdero downtime, enterprise companies use canary deployment strategy where the old version pods are not replaced rather new version pod are deployed and the 10% of the traffic is pushed to new version and if there any issue with new version it is just matter of pointing the load balancer to old version pods to acheive zero downtime. but the drawback is the infrastructure cost.
Feedback for Question 2 (Rolling Updates)
👍 What you got right:

You correctly identified that during rolling updates both old and new versions run simultaneously.

You mentioned rollback as a mitigation strategy.

You brought up canary deployment as an enterprise approach for safer releases.

That shows surface-level familiarity with deployment strategies.

❌ What you missed (this is the actual production problem):

The question was not “what is rolling update?”
It was:

Why are users seeing inconsistent behavior and how do you ensure zero downtime + version consistency?

This happens due to:

1. Session Stickiness / Stateful Behavior

If your app is:

caching locally

using in-memory sessions

not externalizing session state (e.g., Redis)

Then:

Request 1 → old pod

Request 2 → new pod
→ user sees different behavior

So the real root cause is often:

Non-backward-compatible changes between versions

Examples:

DB schema changes

API contract changes

feature flags not gated

cache format changes

JWT signing key change

config drift

What I expected you to talk about:
🔹 Kubernetes Deployment Strategy Tuning:
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1

Ensures:

No pod goes down before a new one is ready.

🔹 Readiness Probes

Prevent traffic to new pods until they're actually ready:

readinessProbe:
🔹 Version Compatibility

To ensure consistency:

Backward-compatible APIs

Expand → Migrate → Contract DB strategy

External session store (Redis)

Feature flags

🔹 Use:

Blue-Green Deployment for strict consistency
(route 100% traffic only after full validation)

Or Service Mesh (Istio) traffic shifting with sticky sessions

Enterprise-grade Answer Should Mention:
Concern	Solution
User hitting different versions	Sticky sessions
Schema mismatch	Backward compatible migration
New pod receives traffic too early	Readiness probe
Partial rollout inconsistency	Blue-Green
Zero downtime	maxUnavailable=0
Rating for this question:

Concept Awareness: 6/10
Production Deployment Depth: 4.5/10

You're explaining what happens, but not how to engineer for safe releases in distributed systems.

## Question 3

Your application in AKS is suddenly going into CrashLoopBackOff state after a new deployment.

👉 How do you debug this issue step-by-step and identify whether it's:

application issue

configuration issue

infrastructure issue

Walk me through your approach.

If the application pod is in crashloopbackoff state after a new deployment. I would first check the pod status using kubectl describe. If it shows OOM killed, it means the pod is leaking memory and the developer should decide whether it is legit that pod is acceptable to leak memory and will change the resource quota for the application, this comes under application issue. If it says not enough space, it means there is not enough space in cluster to schedule the pod and it is infrastructure issue.
Feedback for Question 3 (CrashLoopBackOff)

I’m going to be blunt here — this answer is not sufficient for a 3-year DevOps engineer handling Kubernetes in production.

❌ Major Gaps in Your Debugging Flow:

You jumped straight to:

check kubectl describe → see OOMKilled → developer issue

That is only one of ~10 common CrashLoopBackOff causes.

Also:

"pod is acceptable to leak memory"

🚨 This is a red flag statement in an interview.
Pods are never “acceptable to leak memory” — you either:

fix the memory leak

increase limit temporarily

or autoscale

What a Real Step-by-Step Debug Should Look Like:
✅ Step 1 — Check Pod State
kubectl get pods -n <ns>
✅ Step 2 — Describe Pod (Events Section)
kubectl describe pod <pod>

Check for:

Failed Mount

ImagePullBackOff

Back-off restarting failed container

Liveness probe failure

ConfigMap / Secret missing

✅ Step 3 — Check Container Logs (MOST IMPORTANT)
kubectl logs <pod> --previous

CrashLoopBackOff means:

container started and exited repeatedly

Logs will tell you:

App crash?

Port already in use?

DB connection failure?

Missing ENV variable?

Syntax error?

Dependency timeout?

✅ Step 4 — Check Exit Code

Inside describe:

Last State: Terminated
Exit Code: 1 / 137 / 139
Exit Code	Meaning
137	OOMKilled
1	App crash
139	Segfault
✅ Step 5 — Check Probes

Many times it’s:

Liveness probe killing a healthy but slow-starting app

Check:

livenessProbe:
initialDelaySeconds:
timeoutSeconds:
✅ Step 6 — Check Config Issues

Wrong DB URL?

Secret not mounted?

ConfigMap updated but not reloaded?

✅ Step 7 — Check Node Issues (Infra)
kubectl get nodes
kubectl describe node

Disk pressure?

Memory pressure?

PID pressure?

✅ Step 8 — Check Deployment Diff

What changed?

kubectl rollout history deployment <name>

Rollback if needed:

kubectl rollout undo deployment <name>
Classification:
Type	Example
Application Issue	Null pointer crash
Config Issue	Missing secret
Infra Issue	Node memory pressure
Deployment Issue	Bad probe config
Your Current Rating:

K8s Troubleshooting Depth: 3.5/10

Right now you're diagnosing based on one signal (OOMKilled) instead of forming multiple hypotheses from:

logs

probes

exit codes

config

infra events


## Question 4

In a production AKS cluster, one of your nodes becomes NotReady.

👉 What could be the possible reasons and what exact steps will you take to investigate and recover the node without impacting running workloads?

 first reason is the node pressure, that can be checked by kubectl top node

I’m going to stop you right there — because this is exactly how candidates dig themselves into a hole in real interviews.

You mentioned one possible reason:

node pressure → check using kubectl top node

That’s just one signal, and even that alone does not explain why a node becomes NotReady.

When a Node goes NotReady in Production AKS — Possible Reasons:
🔹 1. Kubelet stopped reporting to API Server

kubelet crash

kubelet hung

TLS cert expired

systemd service stopped

🔹 2. Node Resource Pressure

Node conditions:

MemoryPressure

DiskPressure

PIDPressure

Check:

kubectl describe node <node>

Look under:

Conditions:
🔹 3. Container Runtime Failure

containerd/docker crashed

SSH into node:

systemctl status containerd
🔹 4. Network Issue

Node can't reach API server

NSG rule misconfiguration (AKS-specific)

DNS failure

CNI plugin crash

🔹 5. Azure Infra Issue

VM rebooted

Host maintenance

Underlying disk detached

Check in Azure:

VMSS instance health

Boot diagnostics

🚨 Now: Production-Safe Recovery Steps (Without Impacting Workloads)
Step 1 — Cordon the Node

Prevent new pods from scheduling:

kubectl cordon <node>
Step 2 — Drain the Node Safely

Evict workloads gracefully:

kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

This:

Reschedules pods to healthy nodes

Maintains service availability

Step 3 — Investigate Node

Now SSH into node and check:

journalctl -u kubelet
systemctl status kubelet
df -h
free -m
Step 4 — Restart Services if Needed
systemctl restart kubelet
systemctl restart containerd
Step 5 — If Infra-Level Issue (AKS)

Restart VMSS instance:

az vmss restart
Step 6 — Bring Node Back
kubectl uncordon <node>
Rating for this question:

Understanding of Node Failure Handling: 3/10

Right now you're:

naming one metric

but not considering kubelet, runtime, networking, infra

and missing the most important production-safe steps:
👉 cordon and drain

## Question 5

Your CI/CD pipeline suddenly starts failing at the Docker build stage in Azure DevOps, even though it was working fine earlier.

Error:

no space left on device

👉 What could be the possible reasons and how will you fix this in a production pipeline permanently (not just temporarily)?

Actually I am not aware of this scenario. I am thinking that it might be the images pileup in container registry and there is no space left to accomodate the new build image. My solution here is using the same image which is successful in development, integration environment so not to build the image.

Good — you admitted you don’t know instead of bluffing. That’s the right instinct.

But let’s break this down because this is a very common real production CI failure.

❌ First: Your Assumption is Incorrect

"images pileup in container registry"

🚫 ACR has nothing to do with this error

The error:

no space left on device

is happening during the Docker build stage, which means:

👉 The build agent machine disk is full, not your container registry.

📌 Where This Actually Happens

In Azure DevOps pipelines:

Microsoft-hosted agents OR

Self-hosted agents (very common in enterprises)

When Docker builds an image:
It creates:

layers

intermediate containers

build cache

dangling images

Over time:

/var/lib/docker

gets filled with:

old images

stopped containers

unused volumes

build cache

Eventually:
➡️ Disk is full
➡️ Docker build fails
➡️ Pipeline breaks

🔍 How You Confirm This

SSH into self-hosted agent:

df -h

Check Docker usage:

docker system df

You’ll usually see:

Build Cache: 18GB
Dangling Images: 12GB
⚠️ Temporary Fix (NOT ACCEPTABLE IN PROD)

Manually clean:

docker system prune -a

Pipeline works again.

But this is reactive firefighting.

✅ Permanent Production Fix
🔹 1. Add Automated Cleanup in Pipeline

Before build stage:

docker system prune -af --volumes
🔹 2. Limit Docker Build Cache

Use:

--no-cache

OR better:
Use multi-stage builds to reduce layer size.

🔹 3. Use Ephemeral Agents (Best Practice)

Instead of long-running agents:

Use VMSS-based self-hosted agents

Agent destroyed after job

Clean environment every run

➡️ No disk buildup
➡️ No drift
➡️ No flaky builds

🔹 4. Set ACR Retention Policy (Separate Problem)

This prevents registry bloat:

Remove untagged manifests

Keep last N builds

But again:
📌 This fixes registry storage — not build agent disk

Rating for this question:

CI/CD Operational Understanding: 2.5/10

At 3 YOE, I expect you to know:

Docker builds consume agent disk, not registry disk

And permanent fixes like:

ephemeral agents

auto prune

cache strategy

## Question 6

Your Terraform pipeline accidentally deleted a production resource (e.g., an Azure Load Balancer).

👉 How do you design Terraform workflows to prevent accidental destructive changes in production environments?

In pipeline, at terraform plan we introduce approval so that no terraform script can accidentally delete the infra.
Feedback for Question 6 (Terraform Safety in Production)

You said:

Introduce approval at terraform plan

That’s a good start — but this is only a process control, not an engineering safeguard.

Manual approvals can:

be ignored

be rushed

or misunderstood (plan output is not easy to read)

In production environments, we never rely on approval alone to prevent destructive changes.

What I Expected in a 3 YOE Production-Safe Terraform Design:
✅ 1. lifecycle Block (MOST IMPORTANT)

Protect critical infra like:

Load Balancers

VNets

Databases

lifecycle {
  prevent_destroy = true
}

Now even if someone runs:

terraform destroy

Terraform will fail the plan itself.

✅ 2. Separate State Files per Environment

Never share:

dev.tfstate

prod.tfstate

Use:

different backend storage keys

separate Azure Storage Containers

So dev changes never impact prod infra.

✅ 3. Use:
terraform plan -out=tfplan
terraform apply tfplan

This ensures:

What you reviewed

is exactly what gets applied

(No re-computation during apply)

✅ 4. Sentinel / OPA Policy (Enterprise Level)

Prevent destructive changes automatically:

“Deny if any resource is getting destroyed in prod”

Example policy logic:

deny if change.actions contains "delete"

Now:
➡️ Even if someone approves
➡️ Policy blocks deletion

✅ 5. RBAC Separation

Devs → plan only

Platform team → apply in prod

Use Azure DevOps:

environment-level permission gating

✅ 6. -target is DISABLED in Prod

Prevents partial infra mutation.

Production Philosophy:
Method	Is it Enough?
Manual approval	❌ No
prevent_destroy	✅ Yes
Policy-as-Code	✅ Yes
State isolation	✅ Yes
Rating for this question:

IaC Safety Engineering: 4/10

You mentioned governance —
but missed:

lifecycle protection

policy-as-code

state isolation

immutable plan file

These are table stakes for production Terraform usage.

## Question 7

Your application is auto-scaling using HPA in AKS based on CPU usage.

But even when CPU usage crosses 80%, the pods are not scaling up.

👉 What could be the possible reasons and how will you debug this?

The possible reason is the HPA resource could not find the metrics of the pod usually metric server should be running to omit the metrics of pods, so metric server should be present for HPA to perform scaling. Another reason here is there is not enough disk/memory in cluster to schedule the scale out pods, eventually pods will be scheduled but goes to pending state.
Feedback for Question 7 (HPA not scaling)

This is better than your previous answers 👍
You identified:

Metrics Server dependency ✅

Cluster capacity issue (pods Pending) ✅

These are valid real-world causes.

But again — at 3 YOE, I expect a full HPA debugging flow, not just 2 guesses.

When HPA is NOT Scaling Even After CPU > 80%
🔹 Step 1 — Check HPA Status
kubectl get hpa
kubectl describe hpa <name>

Look for:

Conditions:
AbleToScale
ScalingActive
ScalingLimited
🔹 Step 2 — Check Metrics Availability

You already said this ✔️

kubectl top pod

If this fails:
➡️ Metrics Server issue
➡️ HPA cannot calculate utilization

🔹 Step 3 — Check Resource Requests (MOST COMMON ROOT CAUSE 🚨)

HPA scales based on:

CPU % of request, not actual usage

If your pod spec has:

resources:
  limits:
    cpu: 500m

but no request, HPA cannot compute target utilization.

You must define:

resources:
  requests:
    cpu: 200m

No request = ❌ No scaling

This is the #1 interview answer people miss.

🔹 Step 4 — Check Max Replicas

Maybe HPA already reached:

maxReplicas: 3

So even at 90% CPU:
➡️ No scale

🔹 Step 5 — Check Cooldown Period

HPA doesn't scale instantly:

behavior:
  scaleUp:
    stabilizationWindowSeconds: 300

You may be in cooldown window.

🔹 Step 6 — Pending Pods (You mentioned ✔️)

Check:

kubectl get pods

If Pending:
➡️ Cluster Autoscaler issue
➡️ Node pool maxed out

🔹 Step 7 — Check API Version Mismatch

Wrong target reference:

Deployment renamed?

HPA pointing to old object?

Classification:
Cause	Type
Metrics server missing	Infra
No CPU request	Config
Max replicas reached	Config
Cooldown window	HPA behavior
Pods Pending	Capacity
Rating for this question:

Autoscaling Understanding: 5.5/10

You’re getting closer — but you missed the CPU request dependency, which is the most common real-world blocker.

## Question 8

Your Kubernetes cluster suddenly cannot pull images from Azure Container Registry (ACR).

Error:

ImagePullBackOff
unauthorized: authentication required

👉 What could be the possible reasons and how will you troubleshoot this?

The first possible reason is Image tag is not present in the container registry. Second reason is the secret is corrupted so the connection to ACR is not happening. Third reason is forgot to mention the secret in the manifest itself, resulting container cannot know from where to pull the image.
Feedback for Question 8 (AKS ↔ ACR Image Pull Unauthorized)

You mentioned:

Wrong image tag ✅ (this causes manifest unknown, not unauthorized though)

Secret corrupted ✅

Secret not referenced in manifest ✅

These are valid for generic Kubernetes.

But this is AKS + ACR — and in Azure, most production clusters do not use imagePullSecrets at all.

They use:

Managed Identity–based ACR integration

So the most common real-world reason for:

unauthorized: authentication required

is NOT secret-related.

AKS–ACR Native Auth Flow (What Interviewers Expect)

AKS pulls from ACR using:

System-assigned Managed Identity OR

User-assigned Managed Identity

That identity must have:

➡️ AcrPull role on ACR

🔴 Most Common Causes in Production:
🔹 1. AKS MI Lost AcrPull Role

Someone removed RBAC:

Check:

az role assignment list \
--assignee <AKS_MI_CLIENT_ID> \
--scope <ACR_RESOURCE_ID>

Fix:

az role assignment create \
--assignee <AKS_MI_CLIENT_ID> \
--role AcrPull \
--scope <ACR_RESOURCE_ID>
🔹 2. ACR Not Attached to AKS

Check:

az aks show --name <aks> --resource-group <rg> \
--query "servicePrincipalProfile.clientId"

Attach:

az aks update \
--name <aks> \
--resource-group <rg> \
--attach-acr <acr-name>
🔹 3. Using Wrong Tenant / Subscription

ACR in:

Subscription A
AKS in:

Subscription B

➡️ MI doesn't have access

🔹 4. Node Needs Restart After RBAC Change

Token cached at kubelet level.

Restart nodepool VMSS:

az vmss restart
🔹 5. ACR Firewall Blocking AKS

If:

Private endpoint enabled

Public access disabled

Ensure:

AKS VNet has access

Troubleshooting Flow I Expect:

kubectl describe pod → confirm auth error

Check AKS–ACR attachment

Verify MI has AcrPull

Check ACR firewall rules

Restart node if RBAC recently changed

Rating for this question:

AKS–Azure Integration Depth: 4.5/10

You answered from a:

vanilla Kubernetes (DockerHub secret) mindset

But missed:

Azure-native Managed Identity auth model

Which is heavily expected if your resume says AKS experience.

## Question 9

In Kubernetes, what is the difference between:

Requests

Limits

And how do they impact:

scheduling

performance

autoscaling?

In kubernetes, Every deployment has resource quotas so that the pods do not use up entire cluster. The requests are the minimum cpu, memory requirements that the deployment needs in the node so that it is scheduled and for it to be running. coming to limits, the maximum cpu and memory requirements upon which the deployment will not be allowed to use resources in the cluster. The pods are schedule in a node if that satisfies the requests cpu and memory. The performance depends on limits mentioned, like if the node has free cpu and memory that pod can use  upto it's limits, the performance of the application will be smooth. The autoscaling will work only if the requests quotas are mentioned in configuration of manifest, because the metrics work on requests.
Feedback for Question 9 (Requests vs Limits)

This is your best answer so far. You got the core ideas mostly right.

✅ What You Said Correctly:

✔️ Requests

Minimum CPU/Memory needed for scheduling

Scheduler places pod on a node based on requests

✔️ Limits

Max resources container can consume

✔️ Autoscaling

HPA works based on % of CPU requests
(not limits)

This is correct:

Metrics = usage / request

Now let’s make this Production-Correct:
🔹 Impact on Scheduling

Kubernetes scheduler ONLY looks at requests:

"Can this node guarantee this much?"

If:

requests:
  memory: 512Mi

Node must have 512Mi available.

Limits are ignored during scheduling.

🔹 Impact on Performance
CPU:

If app crosses CPU limit
➡️ It gets throttled (not killed)

App becomes:

slow

latency spikes

timeouts happen

Memory:

If memory crosses limit
➡️ Container is OOMKilled

Which leads to:

Restart

CrashLoopBackOff

Pod instability

🔹 Impact on Autoscaling

HPA uses:

CPU Utilization % = current usage / requested CPU

Example:

Usage	Request	Result
400m	200m	200% → scale
400m	500m	80% → no scale

Same workload
Different request
Different scaling behavior

This is why wrong requests = broken HPA.

Rating for this question:

K8s Resource Management: 6.5/10

You understand:

scheduling

autoscaling dependency

But didn’t mention:

CPU throttling

Memory OOM kill behavior

which are real production performance impacts.

## Final Question (Question 10)

You have deployed a stateful application (e.g., database) in Kubernetes using a Deployment instead of StatefulSet.

👉 What problems can this cause in production and why is StatefulSet preferred for such workloads?

Honestly I dont have any idea about this.

That’s fair — but this is a very important Kubernetes workload design concept that a 3 YOE DevOps Engineer working with AKS is expected to know.

Let me explain what your production risk would be if you deploy a DB using a Deployment instead of a StatefulSet.

🚨 Problems of Using Deployment for Databases
🔹 1. Pod Names are NOT Stable

Deployment creates pods like:

mysql-6d8f9c7d9b-xk29z
mysql-6d8f9c7d9b-a92kd

If pod restarts:
➡️ New name
➡️ New identity

But DB clusters (like MySQL, MongoDB, Kafka) depend on:

stable network identity

Example:
Replica 2 needs to always talk to:

mysql-0.mysql-headless

Deployment cannot guarantee this.

🔹 2. Storage is NOT Persistent per Pod

Deployment uses shared PVC model.

If:

pod crashes

rescheduled to another node

➡️ It may attach a different volume
➡️ Data inconsistency
➡️ Replica corruption
➡️ Split brain

🔹 3. No Ordered Startup / Shutdown

DB clusters must start like:

Primary

Secondary 1

Secondary 2

Deployment:
➡️ Starts all pods randomly in parallel
➡️ Replicas may start before primary
➡️ Cluster fails to elect leader

✅ Why StatefulSet is Preferred?
Feature	StatefulSet Benefit
Stable Pod Name	mysql-0, mysql-1
Persistent Volume per Pod	unique PVC
Ordered Startup	0 → 1 → 2
Sticky Identity	Pod always same
Safe Scale Down	last pod deleted first

Each pod gets:

mysql-data-mysql-0
mysql-data-mysql-1

So even if pod moves node:
➡️ It gets its own disk back

🧠 Production Philosophy:
Workload	Use
Stateless App	Deployment
Database	StatefulSet
Kafka	StatefulSet
Redis Cluster	StatefulSet
