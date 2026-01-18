# Application CI – Expense Project
This repository contains the Continuous Integration (CI) pipeline for the Expense Backend/frontend application.

* The purpose of this CI pipeline is to:
    * Build the backend/frontend application
    * Create a Docker image
    * Push the image to Amazon ECR
    * Optionally trigger the CD pipeline for deployment

Deployment logic is intentionally separated into a different repository (CD) to keep responsibilities clean and manageable.
---
## CI Pipeline Responsibilities
```
Layer	Responsibility
CI	    Build, test, package, push image
CD      Deploy image to Kubernetes (EKS)
```
**This CI pipeline performs the following steps:**
* Read application version
* Install application dependencies
* Build Docker image
* Push image to Amazon ECR
* Optionally trigger CD pipeline

**Benefits:**
* CI can run frequently (every code change)
* CD can be controlled manually or per environment
* Easier debugging and rollback
* Cleaner pipelines and repositories
---
## Jenkins Pipeline Flow
1)  Read Application Version: 
    * Reads the version from package.json
    * Stores it in a Jenkins environment variable (env.appVersion)
    * This version is used as the Docker image tag

Ensures version consistency and Same version flows from CI → CD → Deployment

2) Docker Build & Push (CI Core):
    * Logs into Amazon ECR
    * Builds Docker image using the application version
    * Pushes the image to ECR

Image format used:
```
<account-id>.dkr.ecr.<region>.amazonaws.com/<project>/<env>/<component>:<version>
```
**This ensures:**
- Clear project ownership
- Environment isolation (dev)
- Versioned images for traceability

3) Optional Deployment Trigger:
Deployment is NOT automatic, Controlled using a boolean parameter (deploy)
```
When:
deploy = false → Only build & push image
deploy = true → Triggers the CD pipeline
```
This gives full control over when deployment happens.

4) Environment Strategy Used
This CI pipeline is environment-fixed: environment = 'dev'

* CI pipelines are environment-agnostic by design
* Deployment decisions belong to CD
* Keeps CI simple and predictable
* Environment selection is handled inside the CD pipeline, not here.
---
## Key Design Decisions
**Dynamic Version Handling**
* Version is read at runtime
* Assigned using env.appVersion
* Avoids hardcoding or manual tagging

**CI/CD Decoupling**
* CI does not deploy by default
* CD is triggered only when explicitly requested

**Reproducible Builds**
* Same version = same image
* Easier rollback and debugging
---
## CI → CD Interaction
When deployment is enabled: CI triggers Backend-CD job

**Passes:**
* Application version
* Target environment (dev)

**CD pipeline then:**
Pulls the exact image and Deploys it to Kubernetes using Helm
---
## Typical Usage Scenarios

**Build only (most common)**
* Developer pushes code
* CI builds & pushes image
* No deployment

**Build + Deploy**
* Release-ready change
* CI builds image
* CD pipeline deploys to EKS