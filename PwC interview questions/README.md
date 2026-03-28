# PwC Interview Questions and Recommended Answers

These answers are tailored to my profile as **Hrushikesh**, currently working as a **DevOps Engineer at Wipro** with around **4 years of experience**, with strong exposure to **secure CI/CD, Azure/AWS, Kubernetes, Terraform, automation, vulnerability scanning, IAM, monitoring, and incident support**.

## 1. Tell me brief introduction about yourself?

My name is Hrushikesh, and I’m currently working as a DevOps Engineer at Wipro with around 4 years of experience. My work has mainly focused on building and managing secure CI/CD pipelines, cloud infrastructure on Azure and AWS, Kubernetes-based deployments, and automation using Python, Bash, and PowerShell. Over time, I became more involved in the security side of DevOps, such as integrating SAST, SCA, and DAST into pipelines, improving secrets management, hardening containers, and supporting incident handling. That is one of the main reasons I’m interested in moving into a Senior Security Analyst role, because my experience has naturally grown toward DevSecOps and proactive security.

## 2. Tell me about your first project. Is the current project the one going on for the last 4 years?

I have been with Wipro for around 4 years, and most of my experience has been within the same broader client environment, but the scope evolved over time. Initially, I was more focused on Azure operations, infrastructure support, monitoring, and incident response. Later, I moved deeper into CI/CD, infrastructure as code, Kubernetes operations, and security integrations. So while the client or project continuity remained, my responsibilities expanded significantly across multiple areas.

## 3. Have you set up CI/CD pipelines yourself?

Yes, I have set up CI/CD pipelines myself using GitHub Actions. I worked on designing workflows for build, test, security scanning, infrastructure validation, and deployment. I also integrated controls like SonarQube, pip-audit, Checkov, Gitleaks, Dependabot, and OWASP ZAP based on the use case. In addition to creating pipelines, I also handled optimization, troubleshooting, secrets management, and branch protection to make them production-ready.

## 4. Can you tell me why containerization is required?

Containerization is required mainly to ensure consistency, portability, and faster deployment. It allows us to package the application along with its dependencies, so it behaves the same across development, testing, and production. It also improves scalability, resource efficiency, and simplifies deployment in microservices environments. From a security and operations perspective, containers also help standardize runtime behavior and make it easier to enforce hardened images, non-root users, and minimal attack surface.

## 5. You built a GenAI powered tool, tell me more about it?

I built a GenAI-powered internal tool in Python that auto-generated production-ready Dockerfiles for different programming languages. The goal was to reduce manual effort and standardize Dockerfile creation for developers. It used a local LLaMA model, so it could work in a controlled environment without exposing internal code externally. The tool generated Dockerfiles based on application type and best practices, and I designed it to include recommendations like multi-stage builds, smaller base images, and secure defaults. It helped reduce developer dependency on the DevOps team for repeated Dockerfile creation tasks.

## 6. Tell me about the complete flow of your work as a DevOps engineer?

My role covered the full DevOps lifecycle. I worked on source-control based workflows, CI/CD setup, infrastructure provisioning using Terraform, deployment to Azure/AWS and AKS, secrets handling, monitoring with Prometheus and Grafana, incident support, and security integration into pipelines. I was also involved in vulnerability scanning, container hardening, IAM-related support, log analysis, RCA participation, and automating recurring operational tasks. So practically, my experience was not limited to deployment alone, but included reliability, security, governance, and developer enablement.

## 7. What are the challenges you faced while CI/CD integration, like pipelines getting stuck? Give examples.

One common challenge was pipelines getting stuck due to runner issues, hanging test processes, network dependencies, or external scanning tools taking too long. In such cases, I first identified whether the bottleneck was in the GitHub runner, tool timeout, dependency download, container build stage, or deployment target. For example, I saw cases where security scans or dependency resolution increased pipeline time significantly, so I used caching, stage separation, timeout tuning, and better logging to isolate the issue. In another case, image build failures were caused by unstable dependency sources, so we pinned versions and improved retry logic. My approach was always to identify the exact failure point through logs, metrics, and reproducibility rather than treating the pipeline as one black box.

## 8. How did you deal with SAST and DAST in your project?

I integrated SAST and DAST into the CI/CD process so vulnerabilities could be detected as early as possible. For SAST, we used it during build or PR stages to identify code-level issues early. For DAST, we usually ran it against deployed test or staging environments because it needs a running application. I also worked with developers to review findings, validate whether they were actionable or false positives, and decide whether the issue should block release or be tracked for remediation.

## 9. Which tools have you used for SAST, SCA, and DAST?

For SAST, I used SonarQube. For SCA, I used tools like pip-audit and Dependabot. For DAST, I worked with OWASP ZAP. For infrastructure and secrets-related security, I also used Checkov and Gitleaks.

## 10. You mentioned pip-audit and Dependabot. Do you think there is a difference between these two in terms of output or accuracy?

Yes, there is a difference. `pip-audit` is more of an on-demand scanning tool that checks Python dependencies against known vulnerability databases and gives a direct vulnerability report for the current environment or requirements. Dependabot is more of an ongoing GitHub-native alerting and remediation mechanism, which identifies vulnerable dependencies in repositories and can even propose upgrade PRs. In terms of output, Dependabot is more workflow-oriented and continuous, while pip-audit is more scan-oriented and explicit. In terms of accuracy, both depend on vulnerability data sources, but their practical use is different. I see them as complementary rather than competing tools.

## 11. Have you been part of creating the alerts and implementing Dependabot alerts? What was your experience and were there challenges?

Yes, I was involved in enabling Dependabot and integrating it into repository workflows. The main challenge was not just enabling alerts, but making sure the alerts were actionable and not ignored. We had to tune repository settings, define update strategy, align with branch protections, and make sure developers understood how to review the PRs. Another challenge was handling noisy alerts or cases where upgrades caused compatibility issues, so we needed prioritization and validation before merging.

## 12. If there is a case of transitive dependency, what would be your approach or suggestion to the developer?

For transitive dependency issues, first I identify which top-level package is pulling in the vulnerable library. Then I check whether upgrading the direct dependency resolves the transitive issue. If yes, I recommend that path to the developer. If not, I look at whether there is a patched version, whether the vulnerable component is actually reachable in our use case, and whether compensating controls are needed. My suggestion is always based on risk, exploitability, and available remediation path rather than blindly forcing upgrades.

## 13. Sometimes due to dependency there will be compatibility issues and at that time you cannot upgrade the dependency. What is your approach?

In that situation, I would take a risk-based approach. First, I validate the actual exploitability and impact in our environment. Then I look for mitigation options such as restricting exposure, disabling the vulnerable feature, adding WAF or runtime controls, isolating the service, or strengthening monitoring. At the same time, I would document the exception clearly, involve the security or application owner, and track it until a proper fix is possible. So the response is not to ignore it, but to reduce risk with compensating controls and governance.

## 14. Is there any security team that is there for developers to support them, or does the DevOps team support developers?

In my experience, there is usually shared responsibility. There may be a central security team for policy, governance, and high-risk issues, but in day-to-day delivery, DevOps also supports developers by integrating controls into the pipeline, helping interpret scan results, improving secrets handling, and guiding remediation. In my role, I often acted as the bridge between development, operations, and security expectations.

## 15. Have you been part of security patching as well?

Yes, I have been part of patching activities, especially from infrastructure, dependency, and container security perspectives. This included updating vulnerable dependencies, rebuilding images with patched base layers, supporting infrastructure patch cycles, and helping validate that patches did not break deployments. I was also involved in checking the impact of changes through pipeline validation and post-change monitoring.

## 16. What approach did you follow when creating the GitHub pipeline, like sequential approach or multi-layered approach?

I followed a multi-layered approach, but within that, the stages were logically sequenced. For example, first code checkout and basic validations, then build and test, then security checks like SAST, SCA, secrets scanning, and IaC scanning, then artifact creation, then deployment to the target environment, and finally post-deployment validation. I prefer this layered approach because it gives early feedback while keeping the pipeline structured and maintainable.

## 17. In which stage should you check for security scan results?

Security should be checked at multiple stages, not only one stage. Secret scanning, linting, and some SAST checks should happen very early, ideally at PR level. Dependency and IaC scanning should happen during build and validation stages. DAST should run after deployment in a test or staging environment. In short, security should be shifted left, but some checks also need runtime or deployed context.

## 18. How did you configure the pipeline for production, integration, and test? Did you identify vulnerabilities in development or in production also? What tests do you perform in production CI?

We maintained environment-specific workflows and controls. In development and integration, we ran broader validation and security checks because that is the best place to catch issues early. In production, we were more controlled and focused on validated build artifacts, approvals, deployment safety checks, and smoke tests rather than heavy intrusive scanning. Vulnerabilities were mainly identified in development and staging because production should ideally receive already validated artifacts. In production CI/CD, I prefer lightweight checks such as artifact integrity, policy validation, deployment verification, and smoke tests, rather than aggressive scans that can impact live services.

## 19. When you get the security test result, you said you will do false positive analysis. What are the major challenges and how do you do that?

The biggest challenge is distinguishing theoretical findings from practically exploitable issues. Tools may flag code patterns or dependencies without understanding business context, runtime usage, compensating controls, or reachability. My process is to review the evidence, reproduce where possible, check severity and CVE details, validate whether the vulnerable path is actually used, and discuss with developers if needed. I try to avoid closing issues too quickly, but also avoid wasting time on purely tool-generated noise.

## 20. How do you get the security report? Did you set up anything in the CI step to download it, dump it, or how does the process work?

It depends on the tool. In most cases, the security tools were integrated as CI steps and produced logs, artifacts, or dashboard-linked reports. For example, some results were visible directly in GitHub Actions logs, some in SonarQube dashboards, and some as PR alerts like Dependabot. In practice, I made sure the pipeline either surfaced the result clearly in the job output, stored reports as artifacts when needed, or linked to the scanning platform for review.

## 21. If you identify one as false positive, do you go and close that or what do you do?

I would not close it blindly. First I would validate and document why it is a false positive, including evidence if possible. Then depending on the process and tool, I would mark it as false positive, suppressed, accepted with justification, or discuss it with the security owner before closure. The key point is that the decision should be traceable and defensible.

## 22. Let's say you checked the report and identified vulnerabilities, but later developers say these are false positives. Have you encountered these cases?

Yes, that happens quite often. I treat it as a collaborative review, not an argument. I ask for technical reasoning, validate the scan evidence, check whether the vulnerable code path is reachable, and confirm whether any controls already reduce the risk. Sometimes developers are correct, and sometimes the issue is real but misunderstood. The important part is having a structured validation process.

## 23. How many applications have you assessed so far?

I have worked across multiple repositories and services, including more than 10 repositories directly from a CI/CD and security integration perspective. My involvement included pipeline setup, dependency and security scanning, infrastructure validation, and deployment support across several applications and microservices.

## 24. You said you have worked on microservices. What is the codebase for those microservices you used?

My primary role was not as an application developer, but I worked closely with microservice-based applications from the DevOps side. The codebases varied depending on the service, and my involvement focused on build and deploy patterns, Dockerization, dependency scanning, and pipeline integration rather than feature development. I was comfortable understanding repository structure, build requirements, package managers, Dockerfiles, manifests, and security tooling around those codebases.

## 25. When you said you built the GenAI-powered application, was it your individual contribution or team effort?

It was primarily my individual contribution, though it supported team needs. I identified the recurring bottleneck around Dockerfile creation, designed the solution, built the Python-based tool using a local LLaMA model, and shaped it around practical developer workflows. Of course, I also took feedback from the team to improve usability and relevance.

## Closing Positioning Statement

A strong way to position this profile in the interview is:

> I come from a DevOps background, but a large part of my work has already been security-focused. I have practical experience in integrating security into pipelines, handling vulnerability management, securing secrets, hardening containers, and supporting remediation. So I see this role as a natural progression from DevOps into a more dedicated security role.
