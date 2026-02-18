# KL.NAVI configuration and infrastructure

- [Purpose](#purpose)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Repository Structure](#repository-structure)
- [Workflows](#workflows)
- [References](#references)

## Purpose
This repository contains the complete infrastructure-as-code (IaC) implementation for KL.NAVI, featuring:

- Multi-environment deployment (Local/Production)
- Kubernetes cluster management via ArgoCD (GitOps)
- Infrastructure provisioning with Terraform

## Prerequisites
TOOLS REQUIRED:

- docker-compose setup:
    - Git
    - Docker v20.10+

- kubernetes setup:
    - ArgoCD CLI v2.4+
    - kubectl v1.25+
    - Terraform v1.3+
    - Packer v1.8+

CLOUD REQUIREMENTS:
- Registered domain name for production
- S3-compatible storage for graph files
- Hetzner Cloud account with API token (Only for Kubernetes setup)

## Installation
#### SETUP USING DOCKER-COMPOSE

1. Clone repository:

```
git clone https://github.com/goat-community/klnavi.git
cd klnavi
```

2. Create environment file:
```
cp env_example .env
Edit .env with your local values
```

3. Start services (prod):
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

4. Updating the graph (if needed):
- Delete any previously downloaded graph files
```
rm /download/standard/graph.obj
rm /download/vision-impaired/graph.obj
```
- Update .env to reference new graph files
`OTP_DEFAULT_GRAPH_URL`
`OTP_VISION_IMPAIRED_GRAPH_URL`
- Restart services (prod)
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml down --remove-orphans && docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build --force-recreate
```

Access endpoints:

OTP Standard: http://otp.localhost/pedestrian/standard

Digitransit UI: http://app.localhost

#### SETUP USING KUBERNETES (HETZNER CLOUD):

1. Prepare infrastructure:
```
cd terraform
terraform init -backend-config=backend.conf
terraform apply -var-file="terraform.tfvar
```

2. Configure ArgoCD:
```
argocd login <ARGOCD_SERVER_URL>
argocd app create -f argocd/apps/klnavi/base/kustomization.yml
argocd app sync klnavi
```

3. Verify deployment:
```
kubectl get pods -n klnavi
```

## Repository Structure

```
├── argocd/                  # ArgoCD application manifests
│   └── apps/klnavi/
│       ├── base/            # Base configurations for all environments
│       └── overlays/prod/   # Production-specific overrides
├── terraform/               # Infrastructure-as-Code definitions
│   ├── main.tf              # Primary cluster configuration
│   └── extra-manifests/     # Kubernetes CRDs and operators
├── docker-compose.yml       # Local development setup
├── docker-compose.prod.yml  # Production-like local setup
└── packer/                  # OS image builds for Hetzner
```
## Workflows

#### DATA PIPELINE (KUBERNETES):

1. Download GTFS feeds from [VRN](https://www.vrn.de/opendata/datasets) and [GTFS.de](https://gtfs.de/en/feeds/de_rv)
2. Process with OpenTripPlanner builder
3. Upload graph.obj to cloud storage
4. Update router-config.json files
5. ArgoCD automatically deploys updates

#### CODE DEPLOYMENT (KUBERNETES):
```
Local -> Commit to main branch -> ArgoCD sync -> Production rollout
```

## References
- [Hetzner Cloud Documentation: ](https://docs.hetzner.com)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io)
- [OpenTripPlanner Config Guide](https://docs.opentripplanner.org)
-  [Digitransit UI Docs](https://digitransit.fi/en/developers)
