# Azure DevSecOps Pipelines for Secure API Lifecycle Management

This repository contains a collection of YAML pipelines for Azure DevOps, developed as part of a university thesis project. The goal of this project is to demonstrate a comprehensive and automated DevSecOps workflow for managing the entire lifecycle of APIs on Microsoft Azure.

---

## The DevSecOps Workflow

These pipelines work together to form a complete cycle that enforces security and governance from development to production. The typical workflow is as follows:

1.  **Extractor**: An operator extracts the current configuration of a live API from Azure API Management. The pipeline converts this configuration into code and opens a Pull Request.
2.  **Static Security Testing**: After the code is in the repository, this pipeline runs a battery of static analysis tools (SAST, secret scanning, OpenAPI linting) to find vulnerabilities before deployment.
3.  **Publisher**: Once the code is approved and merged, this pipeline deploys the API configuration from the Git repository to a target API Management instance.
4.  **Reachability Test & Rollback**: Immediately after deployment, this pipeline runs health checks. If any test fails, it **automatically triggers a rollback** to the last known-good version.
5.  **Dynamic Security Testing**: This pipeline runs dynamic (DAST) scans against the live API endpoints to find runtime vulnerabilities.
6.  **Master Pipeline**: A central orchestrator that can call these other pipelines.

---

## The Pipelines

Here is a summary of each pipeline's role and key features.

### Extractor
This pipeline bridges the gap between the live Azure environment and the Git repository, enabling a GitOps workflow.
* **Automates the extraction** of live API configurations (APIs, operations, policies) from Azure API Management into code using the Azure APIOps tool.
* **Performs immediate quality control** by linting the extracted OpenAPI specifications with Spectral.
* **Enforces a review process** by creating a new branch and opening a Pull Request with the changes, instead of committing directly to the main branch.
* **Versions the configuration** by applying a Git tag to the main branch after a successful merge.

### Static Security Testing
This pipeline acts as a comprehensive security gate for the source code before deployment.
* **Integrates multiple security tools**: SonarQube (SAST), Spectral (OpenAPI linting), and TruffleHog (secret scanning).
* **Securely manages credentials** by fetching cryptographic keys from Azure Key Vault at runtime.
* **Guarantees evidence integrity** by cryptographically signing all generated reports with Cosign.
* **Creates an immutable audit trail** by archiving the signed evidence in an Azure Blob Storage (WORM) container.

### Publisher
This pipeline completes the GitOps cycle by deploying API configurations from Git back to Azure.
* **Automates deployment** from the version-controlled repository to any target Azure API Management instance (e.g., development or production).
* **Handles secrets securely** by creating references to Azure Key Vault instead of deploying secret values from the codebase.
* **Ensures traceability** by updating a version identifier within the target APIM instance after a successful deployment.

### Reachability Test & Rollback
This pipeline serves as a critical safety net to ensure service continuity after a deployment.
* **Performs post-deployment validation** by running a series of automated HTTP tests against the live API endpoints.
* **Validates API functionality** by checking for expected HTTP status codes, with support for chained requests.
* **Features an automated rollback capability**: If any API health check fails, the pipeline is designed to **automatically restore the environment to its last known-good version** by redeploying the code from the previous stable Git tag.

### Dynamic Security Testing
This pipeline scans the running application to find runtime vulnerabilities.
* **Performs dynamic security testing** against live API endpoints using the APIsec platform.
* **Features a highly configurable security gate** to automatically evaluate scan results based on CVSS scores, severity levels ('Critical', 'High', etc.), and specific vulnerability types.
* **Can act as a blocking quality gate**, failing the pipeline if the defined risk threshold is exceeded.
* **Signs and archives all DAST reports** to the immutable WORM storage, maintaining a secure audit trail.

### Master Pipeline (Orchestrator)
This pipeline acts as the central control system for the entire workflow.
* **Orchestrates the child pipelines** (Static Scan, Publisher, etc.) by triggering them via the Azure DevOps REST API.
* **Provides highly flexible workflows** through boolean parameters, allowing scans to be run either as blocking quality gates or as non-blocking, informational steps.
* **Supports human-in-the-loop governance** by integrating with Azure DevOps Environments to enforce manual approval checks before critical deployments.
