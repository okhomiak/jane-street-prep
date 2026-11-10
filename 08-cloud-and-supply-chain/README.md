# 08 · Cloud, containers, CI/CD and supply chain

[← Windows and endpoints](../07-windows-and-endpoints/README.md) · [Index](../README.md) · [Next: Incident response →](../09-incident-response/README.md)

Jane Street builds most things in house, so expect fewer "which AWS service" questions than at a cloud-native firm and more "how would you design the control". The questions below are the ones security engineers report at Google, Amazon, Instacart and in cloud security loops, plus the supply-chain questions that October 2026 made unavoidable. Tags: [TW] tadwhitaker, [GN] gracenolan, [GD] Glassdoor, [DF] Dataford guides, [own] a question you were asked in a past interview.

---

## IAM and AWS

<details>
<summary><b>What are the biggest AWS security vulnerabilities?</b> · [TW]</summary>

In order of how often they cause breaches: leaked long-lived access keys (in repos, laptops, CI logs); over-privileged IAM (wildcards, `AdministratorAccess` on service roles); public S3 buckets and snapshots; SSRF to the instance metadata service handing out role credentials (Capital One); security groups open to `0.0.0.0/0` on admin ports; no MFA on root or on humans; missing CloudTrail in some region; secrets in environment variables and user data; cross-account trust policies that are too broad. The pattern they want: identity is the perimeter now, and most cloud breaches are credential misuse, not exploits.
</details>

<details>
<summary><b>IAM role versus policy. How are allow and deny evaluated?</b> · [DF Instacart] [GD] [own: you implemented this in an interview]</summary>

A policy is a document of statements (effect, action, resource, condition). A role is an identity that can be assumed to get temporary credentials; policies attach to roles, users and groups, and resource policies attach to the resource. Evaluation: an explicit Deny anywhere wins; otherwise you need an Allow from an identity policy (and from the resource policy for cross-account); otherwise implicit deny. Permission boundaries and SCPs cap what an Allow can grant. Say the practical rule: humans assume roles through SSO with short-lived credentials, workloads get roles, nobody gets long-lived keys.

You've been asked to code this before. The logic, tested:

```python
import re

def _match(pattern, value):
    regex = "^" + ".*".join(re.escape(p) for p in pattern.split("*")) + "$"   # '*' only; '?' stays literal
    return re.match(regex, value) is not None

def evaluate_access(policies, action, resource):
    allowed = False
    for policy in policies:
        for stmt in policy.get("Statement", []):
            actions = stmt.get("Action", []); resources = stmt.get("Resource", [])
            actions = [actions] if isinstance(actions, str) else actions
            resources = [resources] if isinstance(resources, str) else resources
            if not (any(_match(a, action) for a in actions) and any(_match(r, resource) for r in resources)):
                continue
            if stmt.get("Effect") == "Deny":
                return False                 # explicit deny wins immediately
            if stmt.get("Effect") == "Allow":
                allowed = True
    return allowed                           # default deny
```

Why not `fnmatch`: it treats `?` and `[...]` as wildcards, and ARNs can contain them.
</details>

<details>
<summary><b>How do you protect S3 buckets, and how would you detect and fix a public one automatically?</b> · [GD] [DF Instacart]</summary>

Protect: account-level Block Public Access, bucket owner enforced (ACLs off), default encryption with a customer-managed KMS key, a bucket policy that denies non-TLS and unexpected principals, versioning plus Object Lock for records, access logging and CloudTrail data events, VPC endpoints with endpoint policies for internal access. Detect and fix: AWS Config rule `s3-bucket-public-read-prohibited` or Access Analyzer finding → EventBridge → Lambda that re-applies Block Public Access and notifies the owner; or run it as a scheduled scan with `boto3` listing buckets and checking `get_public_access_block` and policy status. Guardrail above it all: an SCP that forbids disabling Block Public Access.
</details>

<details>
<summary><b>How would you secure an EC2 instance? Which services do central monitoring in AWS?</b> · [GD]</summary>

EC2: IMDSv2 required, an instance role with least privilege, no public IP unless it's a load balancer target, security groups default deny, SSM Session Manager instead of SSH keys, patched AMIs built in a pipeline, EBS encrypted, CloudWatch agent and an EDR agent, no secrets in user data. Monitoring: CloudTrail (API audit, into an org trail with log file validation), GuardDuty (threat detection on CloudTrail, VPC flow logs and DNS), Config (configuration drift and rules), Security Hub (aggregation and standards), Inspector (vulnerabilities), Access Analyzer (external exposure), VPC Flow Logs, Macie for sensitive data in S3.
</details>

<details>
<summary><b>How does SSRF to the metadata service work, and why does IMDSv2 help?</b> · [GN] [GD Google loop: ssrf]</summary>

An app that fetches user-supplied URLs can be pointed at `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` and returns the instance role's temporary keys. IMDSv2 requires a `PUT` with a token first and a TTL header, and defaults the hop limit to 1, so a simple forwarded `GET` fails and containers can't reach the host's metadata. Plus: least-privilege roles so the stolen credential buys little, GuardDuty's `InstanceCredentialExfiltration` findings, and egress controls.
</details>

<details>
<summary><b>KMS and envelope encryption; what's your view on serverless security?</b> · [GD]</summary>

KMS holds customer-managed keys that never leave the HSM-backed service; you generate a data key, encrypt data with it locally, store the data key encrypted by the master key next to the data, and KMS access is audited and controllable through key policies and grants. Serverless: smaller attack surface (no OS to patch, short-lived execution), but the risks move to IAM (each function's role), dependencies (the function is mostly third-party code), event injection from untrusted triggers, and secrets handling; and logging is harder to get right. Net: good when the roles are tight.
</details>

## Containers and Kubernetes

<details>
<summary><b>How would you secure a containerised environment? Kubernetes RBAC: can you deny permissions?</b> · [GD]</summary>

Images: minimal base, pinned by digest, scanned, signed (Sigstore/cosign) with admission control that refuses unsigned images. Runtime: non-root, read-only filesystem, drop capabilities, seccomp and AppArmor profiles, no privileged pods, no host namespaces. Cluster: RBAC with least privilege, network policies default deny, secrets in a secrets manager not in manifests, audit logging, workload identity instead of node credentials, pod security standards enforced. Kubernetes RBAC is additive only: there is no deny rule, so you deny by not granting, and you use admission controllers (OPA Gatekeeper, Kyverno) for "must not" rules.
</details>

<details>
<summary><b>Containers versus VMs versus clusters from a security standpoint; escape techniques; hyperjacking.</b> · [GN]</summary>

A VM has a hardware-enforced boundary (the hypervisor); a container shares the host kernel and relies on namespaces, cgroups and capabilities, so a kernel bug or a misconfiguration (privileged mode, mounted docker socket, host PID namespace) is an escape. Clusters add the control plane as a target: a stolen service account token, an open kubelet, or a compromised etcd. Hyperjacking is taking over the hypervisor itself, which owns every guest. Defence in depth: sandboxed runtimes (gVisor, Kata) for untrusted workloads, minimal privileges, patching the host, and treating the node as the blast radius.
</details>

<details>
<summary><b>Spectre and Meltdown, and what is site isolation?</b> · [GN]</summary>

Speculative execution leaks data through cache timing: Meltdown let user code read kernel memory, Spectre tricks the branch predictor into leaking across processes or sandboxes. Fixes were kernel page-table isolation, microcode and retpolines, at a performance cost. Site isolation is Chrome putting each site in its own renderer process so a Spectre-style read inside the renderer can only see that site's data. The interview point: shared hardware is a shared fate, which matters for multi-tenant cloud and for a trading firm's latency-sensitive hosts where mitigations cost microseconds.
</details>

## CI/CD, secrets and supply chain

<details>
<summary><b>You have a pipeline for Docker images. How do you design it so the right checks happen?</b> · [TW]</summary>

Source: branch protection, signed commits, required review, secret scanning on push. Build: hermetic builds on ephemeral runners, pinned base images and dependencies with hashes, SBOM generated, SAST and dependency scanning, no network during build except the internal mirror. Artefact: image scanned, signed with the build's identity (keyless Sigstore from the CI's OIDC token), provenance attestation (SLSA). Deploy: admission control verifies signature and provenance, images referenced by digest, deploy credentials held by CI with OIDC federation rather than static keys. Runtime: drift detection, and the ability to answer "which images contain package X" in minutes (the Log4j test).
</details>

<details>
<summary><b>How would you use CI/CD to improve security?</b> · [TW]</summary>

Put the controls where developers already are: pre-commit and CI secret scanning, dependency and container scanning that fails only on reachable critical findings, semgrep rules for your own bug classes, infrastructure-as-code scanning (checkov, tfsec) with policy as code (OPA) so a public bucket can't merge, signed artefacts, and automatic PRs for dependency updates. Your Bell Integration story fits here: a centralised GitHub Actions scanning workflow (Trivy, TruffleHog, Gitleaks) that any repo adopted with one file and zero configuration. Say the principle: paved roads, not gates, and measure adoption rather than findings.
</details>

<details>
<summary><b>How would you create a secret storage system?</b> · [TW]</summary>

A vault service (HashiCorp Vault, AWS Secrets Manager, or an internal one) with: envelope encryption under an HSM or KMS root key; authentication by workload identity (Kubernetes service account, IAM role, SPIFFE) not by another secret; short-lived dynamic credentials where possible (database users created on demand); fine-grained policies; every access audited; secret versioning and rotation; the client fetches at runtime into memory, never into an image or a repo. Break-glass procedure for the vault being down. Say the hardest part out loud: the secret zero, how the first credential is bootstrapped, which is why you lean on platform identity.
</details>

<details>
<summary><b>If you had to set up supply chain attack prevention, how would you do it?</b> · [TW] [GN: Log4j] [October 2026 LiteLLM incident]</summary>

Inventory first: an SBOM for every build so you can answer "are we affected" in minutes. Then the controls that would have stopped the recent cases. LiteLLM 1.107.0 (21 October 2026) was a poisoned release that ran a credential stealer at every Python start through a `.pth` file, live on PyPI for under two hours ([Rescana](https://www.rescana.com/post/litellm-1-107-0-supply-chain-compromise-october-2026)): pin with hashes, an internal mirror with a cooldown of a few days before new releases are allowed, no outbound internet from build and research hosts, and credentials that are short-lived so a stolen one dies quickly. For your own releases: trusted publishing from CI with OIDC, signed artefacts, branch protection that an org owner can't silently disable, and alerts on release-workflow changes. For vendors and actions: pin third-party GitHub Actions to a commit SHA, review new dependencies, and watch for maintainer takeovers. Log4j is the drill: how fast could you list every affected artefact and redeploy. The newest wing of this is the **agentic supply chain**: MCP servers and tool registries are dependencies too, and a tool definition can carry an injection or be swapped after you trust it, so the same pin-and-diff, provenance and least-privilege thinking applies to tools your agents load ([block 16](../16-ai-and-llm-security/README.md)).
</details>

<details>
<summary><b>How can cloud service accounts enable lateral movement? What is BeyondCorp?</b> · [GN]</summary>

A workload's identity is often broader than its job: a build runner that can assume a deploy role, a monitoring account that can read every bucket, a Kubernetes service account token mounted into every pod by default. An attacker on one workload uses that identity to reach the next; the Binance theft in October 2026 ran through a stolen container deploy token on an engineer's laptop ([BleepingComputer](https://www.bleepingcomputer.com/news/security/binance-links-63-billion-crypto-theft-to-compromised-devops-laptop/)). Fix: one identity per job, no wildcard trust policies, tokens that expire in minutes, and alerts on first-time role assumptions. BeyondCorp: trust the device and the user, not the network; every request is authenticated and authorised on device health and identity, so being "inside" buys nothing.
</details>

<details>
<summary><b>WAF: why would you use one and what did it actually solve?</b> · [GD]</summary>

A WAF buys time against known attack patterns and bots at the edge, gives virtual patching while a fix ships, and rate limits. It does not fix logic bugs, it struggles with encoded payloads, and teams turn it off when false positives hit customers. Use case to cite: block a known exploit path for an unpatched vendor product for the week until the patch lands.
</details>

<details>
<summary><b>What is infrastructure as code and why does security like it?</b> · [GD]</summary>

Terraform, CloudFormation, Pulumi: infrastructure defined in reviewed, versioned text. Security gets review on every change, scanning before deploy, drift detection, reproducible environments, and the ability to encode guardrails (modules that only produce encrypted, private buckets). Risk: state files and plan outputs contain secrets, and the CI that applies IaC is the most powerful identity in the company.
</details>

## The 2025 and 2026 supply-chain incidents, and the pattern

You need two you can tell in a sentence, and the pattern.

| When | What | The lesson |
| --- | --- | --- |
| August 2025 | Stolen OAuth tokens for a sales-chat integration were used to export data from many companies' CRM tenants ([Google](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift)) | A third-party OAuth grant is a standing, broadly scoped credential. Inventory them and be able to revoke by integration |
| August 2025 | Poisoned releases of a build tool used developers' own AI coding tools to hunt for secrets ([Wiz](https://www.wiz.io/blog/s1ngularity-supply-chain-attack)) | An agent with its permission checks off is a credential harvester |
| Autumn 2025 | Self-spreading npm worms republished victims' packages with stolen publish tokens ([Semgrep](https://semgrep.dev/blog/2025/digging-for-secrets-sha1-hulud-the-second-coming-of-the-npm-worm)) | Long-lived publish tokens turn one compromise into hundreds |
| March 2026 | A security scanner's GitHub Action was hijacked; a pipeline running it leaked the publishing token of LiteLLM, the model-gateway library ([Snyk](https://snyk.io/blog/poisoned-security-scanner-backdooring-litellm/)) | Security tools in CI are privileged dependencies |
| August 2026 | An npm worm across 400+ packages, some published through victims' own build identity and so able to carry valid provenance, planted agent and editor configuration files ([Microsoft](https://www.microsoft.com/en-us/security/blog/2026/08/04/chaindrop-supply-chain-compromise-anatomy-self-propagating-worm/)) | Provenance proves where something was built, not that the build was honest |
| October 2026 | About 1,900 GitHub accounts pushed malicious workflow files to tens of thousands of repositories, after CI secrets including AI provider keys ([The Hacker News](https://thehackernews.com/2026/10/ghostaction-compromises-about-1900.html)) | Alert on new workflow files. Rotate after a compromise, every time |
| October 2026 | LiteLLM poisoned a second time, through its own release pipeline (above) | Hours of exposure is enough. Mirror with a cooldown |

The pattern in three lines: the way in is the pipeline (a workflow that runs untrusted input, or a step that isn't pinned); the amplifier is a long-lived token; the fix is short-lived credentials issued per job to an identified workload, plus pinning and a cooldown on new releases.

One more thing to know about CI identity: when the build system issues OIDC tokens, it has become an identity provider. Jenkins' OIDC plugin had a critical 2025 flaw that let someone who could configure a job shape the token that job received and impersonate a trusted one ([advisory listing](https://advisories.gitlab.com/pkg/maven/io.jenkins.plugins/oidc-provider/)). Bind cloud and vault roles to the job's subject, not to claims a build can influence, and keep the plugin patched.

---

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| Security groups versus NACLs? | SGs are stateful and per instance; NACLs are stateless, per subnet, and evaluated in order. |
| What is an SCP? | Organisation-level policy that caps what any account can do, even root. |
| How do you give a developer temporary prod access safely? | Just-in-time elevation with approval, time-boxed role, session recording, and an alert on use; never standing admin. |
| CloudTrail versus VPC Flow Logs versus GuardDuty? | API calls, network metadata, and the detection engine that reads both. |
| What is SLSA? | A framework of build-integrity levels: provenance, hermetic builds, two-person review. |
| Why pin GitHub Actions to a SHA? | A tag can be moved to malicious code; the tj-actions compromise did exactly that. |
| Difference between signing a commit and signing an artefact? | Commit proves authorship of source; artefact signature proves the build output came from a known builder. |
| What is workload identity? | Platform-issued, short-lived identity for software (IAM roles, Kubernetes SA tokens, SPIFFE) replacing static secrets. |
| CI issues OIDC tokens. What does that make it? | An identity provider: bind roles to the job's subject and guard who can configure jobs. |
| What's a dependency confusion attack? | Publishing a public package with the same name as an internal one so the resolver pulls the public version; fix with scoped registries and explicit sources. |
| How would you find every host running a vulnerable package? | SBOM search, EDR or osquery fleet query, image registry scan, and the honest answer that you need all three. |

[← Windows and endpoints](../07-windows-and-endpoints/README.md) · [Index](../README.md) · [Next: Incident response →](../09-incident-response/README.md)
