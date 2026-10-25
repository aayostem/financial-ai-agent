<div align="center">

<br />

# Financial AI Agent

**A production-grade Financial RAG system built phase by phase: SEC EDGAR ingestion, pgvector hybrid retrieval, LLM agents with function calling, and a full AWS-native deployment.**

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-844FBA?style=flat-square&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Terragrunt](https://img.shields.io/badge/Terragrunt-DRY_IaC-4A5568?style=flat-square)](https://terragrunt.gruntwork.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://aws.amazon.com/eks/)
[![Helm](https://img.shields.io/badge/Helm-packaged-0F1689?style=flat-square&logo=helm&logoColor=white)](https://helm.sh/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)](./LICENSE)

<br />

[**Docs**](./docs/) · [**Helm Guide**](./README-HELM.md) · [**Roadmap**](./ROADMAP.md) · [**Changelog**](./CHANGELOG.md) · [**Contributing**](./CONTRIBUTING.md)

<br />

</div>

---

## What This Is

`financial-ai-agent` is a complete financial RAG system, built in 12 phases from an empty repository to a running production workload on AWS EKS. It is designed as a teaching project and a reference implementation.

Every phase is self-contained. You can stop at any phase and have a working system. You can resume at any phase and the previous state is reproducible through Git, Terraform state, and Helm releases.

The pipeline:

- Downloads 10-K and 10-Q filings from SEC EDGAR
- Parses, cleans, and chunks the documents
- Embeds the chunks and stores them in PostgreSQL with pgvector
- Retrieves relevant context with a hybrid of vector similarity and BM25
- Answers financial questions with an LLM using function calling
- Records every query in an audit trail
- Runs on EKS behind an ALB with TLS
- Autoscales with HPA and provisions nodes with Karpenter
- Enforces L3/L4/L7 network policy with Cilium
- Encrypts pod-to-pod traffic with Istio mTLS
- Detects runtime threats with Falco
- Exposes traces, metrics, and logs through an observability stack
- Manages cost through resource limits, spot instances, and SLO-based alerting

---

## Phases

| Phase | Title | Deliverable |
|-------|-------|-------------|
| 1 | Foundation, Config & Database Layer | Docker stack, settings, database and cache clients |
| 2 | SEC EDGAR Ingestion Pipeline | Rate-limited EDGAR client, HTML and text parsers |
| 3 | Text Processing, Chunking & Hybrid Search | Token-aware chunking, embeddings, pgvector + BM25 |
| 4 | LLM Agent, Function Calling & Audit Trail | Multi-provider LLM client, agent with tools, analysis history |
| 5 | FastAPI Layer, Auth & Rate Limiting | REST API, API key auth, rate limiting, Prometheus |
| 6 | Vault Zero-Trust Secrets | Dynamic DB credentials, IRSA, Vault Agent sidecar |
| 7 | CI/CD Pipeline | GitHub Actions, Trivy, gitleaks, OPA Rego |
| 8 | Helm, Karpenter & EKS Infrastructure | Helm chart, Karpenter NodePools, Terragrunt modules |
| 9 | GitOps & eBPF Networking | ArgoCD App of Apps, Cilium, Istio mTLS |
| 10 | Runtime Security & Threat Detection | Falco, Falcosidekick, security alerting |
| 11 | LGTM Observability Stack | Loki, Tempo, Mimir, Grafana, OpenTelemetry |
| 12 | FinOps & Cost Optimization | Kubecost, VPA, resource quotas, cost alerts |

---

## Architecture


```
┌──────────────────────────────────────────────────────────────────────────┐
│                        financial-rag-agent                               │
├────────────────────────────────────────────────────────────────────────── │
│                                                                          │
│   ┌─────────────────┐     ┌──────────────────┐     ┌────────────────┐   │
│   │  Data Ingestion │────▶│  Vector Store /  │────▶│  RAG Pipeline  │   │
│   │                 │     │  Retrieval Layer  │     │                │   │
│   │  • Real-time    │     │                  │     │  • Reranking   │   │
│   │  • Batch docs   │     │  • Embeddings    │     │  • Context     │   │
│   │  • Multi-modal  │     │  • Semantic search│     │    assembly    │   │
│   └─────────────────┘     └──────────────────┘     └───────┬────────┘   │
│                                                            │             │
│                                                   ┌────────▼────────┐   │
│                                                   │  Agent Layer    │   │
│                                                   │                 │   │
│                                                   │  • Foundation   │   │
│                                                   │  • Multi-agency │   │
│                                                   │  • Predictive   │   │
│                                                   │  • Multi-modal  │   │
│                                                   └────────┬────────┘   │
│                                                            │             │
│                                                   ┌────────▼────────┐   │
│                                                   │    Response     │   │
│                                                   │    Synthesis    │   │
│                                                   └─────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
```


Runtime on EKS:

```
ALB (TLS) → Istio Gateway → FastAPI → QueryEngine → DocumentRetriever
                                               ├── pgvector (StatefulSet)
                                               ├── Redis (StatefulSet)
                                               └── LLM (Groq / OpenAI)

Observability: OpenTelemetry → Collector → Tempo / Loki / Mimir → Grafana
Security:      Cilium L7 policy → Istio mTLS → Falco runtime detection
Secrets:       Vault → dynamic DB creds → IRSA for AWS access
```

---

## Repository Structure

```
financial-ai-agent/
├── .github/workflows/                # CI/CD pipelines
├── argocd/                           # ApplicationSets, AppProject, notifications
├── cilium/                           # CiliumNetworkPolicies, ServiceMonitors
├── docs/                             # Architecture and runbooks
├── falco/                            # Custom rules and Falcosidekick config
├── infrastructure/
│   ├── docker/                       # Dockerfile, compose files, init SQL
│   ├── helm/                         # Helm chart
│   │   └── karpenter/                # NodePools and EC2NodeClass
│   └── terraform/                    # Terragrunt + modules
│       ├── modules/                  # VPC, EKS, RDS, ElastiCache, ECR, IAM, S3, observability
│       └── environments/             # dev, staging, prod
├── istio/                            # mTLS, AuthorizationPolicy, Gateway, DestinationRule
├── monitoring/                       # SLO rules, alerts, dashboards
├── observability/                    # LGTM stack, tracer config, Grafana datasources
├── policies/rego/                    # OPA policies (K8s, Helm, Terraform, Vault)
├── scripts/                          # Utility and diagnostic scripts
├── src/financial_ai/                 # Application package
│   ├── agents/                       # FinancialAgent with tool use
│   ├── api/                          # FastAPI routes, models, middleware
│   ├── config/                       # Settings (Pydantic v2)
│   ├── ingestion/                    # SEC EDGAR client, parsers
│   ├── monitoring/                   # Prometheus metrics
│   ├── processing/                   # Chunker
│   ├── retrieval/                    # Embeddings, hybrid search, query engine
│   ├── security/                     # Vault client
│   ├── storage/                      # Database, cache, repositories, vector store
│   ├── telemetry/                    # OpenTelemetry wiring
│   └── utils/                        # Exceptions
├── tests/
│   ├── integration/                  # Phase tests (require running services)
│   └── unit/                         # Fast isolated tests
├── docker-compose.yml
├── pyproject.toml
└── README.md
```

---

## Getting Started

### Prerequisites

```
Python 3.11+
Docker + Docker Compose
kubectl
Helm 3.12+
Terraform 1.7+ and Terragrunt 0.50+
AWS CLI v2 with a configured profile
```

### 1. Clone and configure

```bash
git clone https://github.com/aayostem/financial-ai-agent-v2.git
cd financial-ai-agent

cp .env.example .env
# Add POSTGRES_PASSWORD, REDIS_PASSWORD, and your LLM API key
```

### 2. Install dependencies

```bash
python -m venv .venv
source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

### 3. Start the local stack

```bash
docker compose -f docker-compose.dev.yml up -d
docker compose -f docker-compose.dev.yml logs -f app
```

Verify:

```bash
curl http://localhost:8000/health
curl http://localhost:8000/
```

### 4. Run the Phase 1 integration test

```bash
pytest tests/integration/test_phase1_foundation.py -v
```

If all tests pass, the foundation is intact and you can proceed to Phase 2.

---

## Testing

Each phase has a test file that verifies its specific deliverable.

```bash
# All tests
pytest tests/ -v

# A single phase
pytest tests/integration/test_phase1_foundation.py -v

# Only tests that do not need Docker or network
pytest tests/unit/ -v

# Only integration tests
pytest tests/ -v -m integration
```

---

## Deployment

### Local Docker

```bash
docker compose -f docker-compose.dev.yml up -d
```

### Kubernetes via Helm

Prerequisites: EKS cluster running, `kubectl` configured, cluster controllers installed (Phase 8 Part 6).

```bash
helm upgrade --install financial-ai infrastructure/helm/ \
  --namespace financial-ai \
  --create-namespace \
  -f infrastructure/helm/values.yaml \
  -f infrastructure/helm/values.prod.yaml
```

Verify:

```bash
kubectl -n financial-ai get pods
kubectl -n financial-ai get hpa
kubectl -n financial-ai get ingress
```

### Infrastructure via Terragrunt

```bash
cd infrastructure/terraform/environments/staging

# Bootstrap state bucket once (per environment)
cd s3 && terragrunt apply

# Apply modules in dependency order
cd ../vpc   && terragrunt apply
cd ../eks   && terragrunt apply
cd ../ecr   && terragrunt apply
cd ../rds   && terragrunt apply
cd ../elasticache && terragrunt apply
cd ../iam   && terragrunt apply
```

### GitOps via ArgoCD

After Phase 9 is complete:

```bash
kubectl apply -f argocd/projects/financial-ai-project.yaml
kubectl apply -f argocd/appsets/apps-appset.yaml
argocd app sync financial-ai-staging
```

---

## API Overview

| Endpoint | Method | Purpose |
|---|---|---|
| `/health` | GET | Database and cache health |
| `/` | GET | Service metadata |
| `/query` | POST | Financial question, returns answer with sources |
| `/ingest/sec` | POST | Ingestion job (returns 202, runs in background) |
| `/stats` | GET | Global vector store statistics |
| `/stats/{ticker}` | GET | Per-ticker statistics |
| `/metrics` | GET | Prometheus metrics |
| `/docs` | GET | OpenAPI UI (development only) |

Example query:

```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What were Apple main sources of revenue growth in FY2023?",
    "ticker": "AAPL",
    "analysis_style": "analyst",
    "search_type": "hybrid"
  }'
```

---

## Security

- **Secrets** are never committed. `.env` is in `.gitignore`. gitleaks runs on every push.
- **Runtime** credentials are injected from Vault Agent as files, not environment variables.
- **Database** access uses dynamic credentials with a 1-hour TTL.
- **AWS access** uses IRSA — pods assume IAM roles through their service account, not through static keys.
- **Network** access is enforced with Cilium at L3/L4/L7 and Istio mTLS.
- **Vulnerability** scanning runs in CI (Trivy) and at admission (OPA).

See [SECURITY.md](./SECURITY.md) for the disclosure process.

---

## Roadmap

See [ROADMAP.md](./ROADMAP.md).

---

## Changelog

See [CHANGELOG.md](./CHANGELOG.md).

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for branch strategy, commit conventions, and PR requirements.

---

## License

[MIT](./LICENSE) — Copyright © 2026 Ayo
```
