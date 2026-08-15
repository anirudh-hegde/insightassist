
# 🚀 InsightAssist

[![SigNoz](https://img.shields.io/badge/Observability-SigNoz-orange.svg)](https://signoz.io/)  
[![OpenTelemetry](https://img.shields.io/badge/Telemetry-OpenTelemetry-lightgrey.svg)](https://opentelemetry.io/)

**InsightAssist** is a Kubernetes-native observability helper that automatically instruments your Java, Node.js, or Python applications, ships traces, metrics & logs to SigNoz Cloud, and even suggests manual instrumentation via AI. All you do is point InsightAssist at your un-instrumented code (zip or GitHub repo) and hit **Instrument**—we handle the rest!

---

## Demo

https://youtu.be/78b0B-wF314

## 📋 Table of Contents

- [✨ Features](#✨-features)  
- [🏗️ Architecture](#️-architecture)  
- [⚙️ Prerequisites](#️-prerequisites)  
- [🚀 Quick Start](#-quick-start)  
  - [1. Clone & Configure](#1-clone--configure)  
  - [2. Deploy to Minikube](#2-deploy-to-minikube)  
  - [3. Instrument a User App](#3-instrument-a-user-app)  
- [🛠️ Configuration](#️-configuration)  
- [🧹 Cleanup](#️-cleanup)  
- [🤝 Contributing](#️-contributing)  
- [📄 License](#-license)  

---

## ✨ Features

- **Auto-Instrumentation** of Java / Node.js / Python apps via the OTel Operator  
- **AI-Driven Suggestions** for manual instrumentation (powered by OpenAI)  
- **Traces & Metrics → SigNoz Cloud** using OTLP sidecars  
- **Logs Tailored** via Collector DaemonSet & `filelog` receiver  
- **Host & Node Metrics** (CPU, memory, filesystem, network, load)  
- **One-Click Cleanup** script to tear down all resources  

---

## 🏗️ Architecture

```
User App (zip / Git Repo)
│
▼
InsightAssist Backend ──► Kubernetes Manifests (deployment+service .yaml)
│
├── OpenTelemetry Operator (auto-inject sidecars)
│
└── Collector DaemonSet
    ├─ Receivers: OTLP, filelog, hostmetrics
    ├─ Processors: batch
    └─ Exporter: OTLP → SigNoz Cloud
```

- **Backend**: FastAPI service that clones or uploads code, renders Jinja2 K8s templates, invokes `kubectl apply`, calls OpenAI.  
- **Operator + Instrumentation CR**: Auto-injects the correct OTel SDK for each runtime.  
- **Collector DaemonSet**: Runs on each node (hostNetwork), tails container logs, scrapes infra metrics, and exports everything to SigNoz.

---

## ⚙️ Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/) (with Docker driver)  
- `kubectl` CLI  
- `helm` CLI  
- Docker (local)  
- SigNoz Cloud account & **Ingestion Key**  
- OpenAI API Key (for AI suggestions)  

---

## 🚀 Quick Start

### 1. Clone & Configure

```bash
https://github.com/anirudh-hegde/insightassist.git
cd InsightAssist
cp .env.sample .env
# Edit .env:
#   OPENAI_API_KEY=sk-…
#   SIGNOZ_CLOUD_ENDPOINT=ingest.signoz.cloud:4317
#   SIGNOZ_CLOUD_API_KEY=<YOUR_SIGNOZ_INGESTION_KEY>
```

### 2. Deploy to Minikube

```bash
chmod +x ./run.sh
eval "$(minikube docker-env)"
./run.sh
```

This will:
- Build your backend, AI-agent & frontend images
- Install SigNoz (Helm chart) in `signoz` ns
- Install cert-manager & OTel Operator
- Apply your Collector DaemonSet and Instrumentation CR
- Deploy InsightAssist services in `InsightAssist` ns

### 3. Instrument a User App

1. Visit: `http://localhost:5173` (InsightAssist UI)
2. **Upload Zip** or **Clone GitHub Repo** (e.g. `https://github.com/heroku/node-js-getting-started.git`)
3. Click **Instrument** → watch your app deploy in `InsightAssist` ns
4. Visit **SigNoz** dashboard (logs / metrics / traces)

---

## 🛠️ Configuration

| Component              | File                                | Purpose                                        |
|------------------------|-------------------------------------|------------------------------------------------|
| AI Suggestions         | backend/.env                        | OPENAI_API_KEY                                 |
| SigNoz Exporter        | run.sh & Deployment YAMLs           | SIGNOZ_CLOUD_ENDPOINT & SIGNOZ_CLOUD_API_KEY   |
| Collector Config       | k8s/otel-collector-config.yaml      | Receivers, processors, exporters (logs+infra)  |
| Collector DaemonSet    | k8s/otel-collector-daemonset.yaml   | Mount hostFS, docker logs, privileged mode     |
| Instrumentation CR     | k8s/instrumentation.yaml            | Auto-inject SDK sidecars                       |

Adjust resource requests/limits, scrape intervals, and file patterns as needed.

---

## 🧹 Cleanup

```bash
chmod +x ./cleanup.sh
./cleanup.sh
```

This will:
- Stop lingering `kubectl port-forward`
- Delete all k8s resources & namespaces
- Uninstall Helm releases (`signoz`, `opentelemetry-operator`, `cert-manager`)
- Remove OTel Collector DaemonSet & ConfigMap
- Prune built Docker images in Minikube
- Remove local `k8s/` manifests & `user-apps/` directory

---

## 🤝 Contributing

- Fork the repo and create your feature branch
- Write code, tests & update documentation
- Submit a pull request — we’ll review & merge!

Please follow the Contributor Covenant and our code style guidelines.

---


Made by team members.
Empowering developers to instrument in one click!
