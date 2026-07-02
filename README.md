<div align="center">

# ❤️ Heart Disease Prediction — ML + Cloud Native Deployment

**An end-to-end, production-grade Machine Learning application that predicts heart disease risk with 100% test accuracy and 0.003s inference time — deployed on AWS EKS with full GitOps automation, autoscaling, and live observability.**

[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/r/waleedcodearch/heart-disease)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-326CE5?logo=kubernetes&logoColor=white)](#)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?logo=argo&logoColor=white)](#)
[![AWS](https://img.shields.io/badge/AWS-EKS%20%7C%20EC2%20%7C%20ALB%20%7C%20ASG-FF9900?logo=amazonaws&logoColor=white)](#)
[![Flask](https://img.shields.io/badge/Flask-Backend-000000?logo=flask&logoColor=white)](#)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](#)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus&logoColor=white)](#)
[![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?logo=grafana&logoColor=white)](#)

</div>

---

## 📖 Overview

This project predicts whether a patient has heart disease — along with a **severity score** — from 13 basic clinical inputs (age, cholesterol, blood pressure, ECG results, etc.). A user fills out a simple web form and receives an instant prediction, powered by a **Gradient Boosting Classifier** trained on 1,025 combined medical records from the UCI Heart Disease dataset (Cleveland, Hungarian, Statlog, and Long Beach sources).

Beyond the ML model itself, this project demonstrates a **complete production deployment pipeline**: containerization, Kubernetes orchestration on AWS EKS, GitOps-driven continuous delivery with ArgoCD, automatic horizontal scaling under load, and real-time infrastructure monitoring with Prometheus and Grafana.

> 🎯 **Why this project matters:** It's not just a Jupyter notebook model — it's a fully deployed, self-healing, auto-scaling, monitored cloud application, built to mirror real-world production ML systems.

---

## ✨ Key Highlights

| | |
|---|---|
| 🎯 **Model Accuracy** | 100% (Gradient Boosting) |
| ⚡ **Inference Time** | 0.003 seconds |
| 📊 **Dataset Size** | 1,025 combined medical records |
| ☁️ **Infrastructure** | AWS EKS, EC2, ALB, ASG |
| 🔄 **Deployment Strategy** | GitOps via ArgoCD (auto-sync every 3 min) |
| 📈 **Autoscaling** | HPA — scales pods 2 → 10 based on CPU load |
| 📡 **Monitoring** | Prometheus + Grafana real-time dashboards |
| 🐳 **Containerization** | Docker (image on Docker Hub) |

---

## 🧠 Machine Learning Pipeline

### Dataset
- **1,025 records** combined from the Cleveland, Hungarian, Statlog, and Long Beach heart disease datasets (Kaggle/UCI).
- Data cleaning to remove invalid values in the `ca` and `thal` columns.
- Target variable: `0` = No heart disease, `1` = Heart disease present.

### Feature Engineering
- **StandardScaler** for feature normalization
- **SelectKBest (ANOVA F-test)** for statistical feature selection
- **Principal Component Analysis (PCA)** — dimensionality reduced to 8 components
- **GridSearchCV** for hyperparameter tuning
- **5-fold cross-validation** for robust, generalizable evaluation
- **Parallel processing (Joblib)** to speed up training

### Model Comparison

| Algorithm | Accuracy | Precision | Recall | F1-Score | Training Time | Testing Time |
|---|---|---|---|---|---|---|
| Logistic Regression | 86% | 86% | 86% | 86% | 0.047s | 0.011s |
| Naive Bayes | 81% | 81% | 81% | 81% | 0.015s | 0.004s |
| Decision Tree | 91% | 91% | 91% | 91% | 0.013s | 0.003s |
| Random Forest | 97% | 97% | 97% | 97% | 0.189s | 0.011s |
| **Gradient Boosting ✅** | **100%** | **100%** | **100%** | **100%** | 0.255s | **0.003s** |
| Stacking Classifier | 99% | 99% | 99% | 99% | 2.094s | 0.009s |

**Gradient Boosting** was selected as the final production model for its perfect accuracy and near-instant inference time — critical for a responsive user-facing web application. Severity is derived using `predict_proba` to give a percentage-based risk score rather than a flat binary output.

### Input Features (13 clinical parameters)

| Feature | Range / Values |
|---|---|
| Age | 29 – 77 |
| Sex | 0 = Female, 1 = Male |
| Chest Pain Type | 0 – 3 |
| Resting BP | 94 – 200 mmHg |
| Cholesterol | 126 – 564 mg/dL |
| Fasting Blood Sugar | 0 or 1 (>120 mg/dL) |
| Resting ECG | 0 – 2 |
| Max Heart Rate | 71 – 202 bpm |
| Exercise-Induced Angina | 0 or 1 |
| ST Depression | 0 – 6.2 |
| Slope of ST Segment | 0 – 2 |
| Major Vessels | 0 – 3 |
| Thal | 1 = Normal, 2 = Fixed Defect, 3 = Reversible |

---

## 🏗️ Cloud Architecture

**End-to-end flow — from a `git push` to a live prediction in the browser:**

```
Developer → GitHub → GitHub Actions (CI) → Docker Hub
                                                │
GitOps: ArgoCD watches k8s/ folder ─────────────┘
                │
                ▼
        AWS EKS Cluster (t2.medium worker nodes)
                │
   ┌────────────┼─────────────┐
   ▼            ▼             ▼
 Flask Pod   Flask Pod    HPA (scales 2→10 on CPU > 50%)
   │            │
   └────────────┴──► ClusterIP Service (port 80 → 5000)
                              │
                              ▼
                 AWS ALB (Internet-facing, public URL)
                              │
                              ▼
                      User's Browser
```

**Monitoring loop:** Prometheus scrapes pod/node metrics via ServiceMonitor → Grafana visualizes CPU, memory, network, and pod count in real time → metrics also feed the Horizontal Pod Autoscaler.

### Request Flow
1. User opens the app in their browser and submits the health form.
2. Request hits the **AWS Application Load Balancer** on port 80.
3. ALB routes traffic to the Kubernetes **ClusterIP Service**.
4. Service forwards the request to a **Flask pod** on port 5000.
5. Flask converts form data into a pandas DataFrame and passes it to the pre-loaded ML model (same container — no extra network hop).
6. The Gradient Boosting model returns a prediction and severity probability in **0.003 seconds**.
7. Flask renders the result back to the browser.

---

## ⚙️ Tech Stack & Infrastructure

### Application Layer
| Layer | Technology |
|---|---|
| Frontend | HTML + CSS (dark "medical luxury" theme, fully responsive) |
| Backend | Flask (Python) |
| ML Libraries | scikit-learn, pandas, numpy, seaborn, matplotlib, plotly, joblib |
| Model Serving | Gradient Boosting Classifier (`.joblib`) loaded in-process with Flask |

### DevOps & Cloud Infrastructure
| Component | Purpose |
|---|---|
| **Docker** | Packages app, model, and dependencies into a single portable image |
| **GitHub Actions** | CI pipeline — builds and pushes Docker image to Docker Hub on every push to `main` |
| **Docker Hub** | Image registry (`waleedcodearch/heart-disease:latest`) |
| **ArgoCD** | GitOps continuous delivery — auto-syncs `k8s/` manifests to EKS every 3 minutes |
| **Amazon EKS** | Managed Kubernetes cluster running the Flask deployment |
| **EC2 (t2.medium)** | Worker nodes hosting the application pods |
| **AWS ALB** | Internet-facing Application Load Balancer, provisioned via Kubernetes Ingress |
| **Auto Scaling Group (ASG)** | Ensures worker node availability; replaces failed nodes automatically |
| **HPA (Horizontal Pod Autoscaler)** | Scales pods from 2 → 10 based on CPU utilization (>50% trigger) |
| **IAM + OIDC** | Secure trust bridge allowing the ALB Controller to manage AWS Load Balancers from within Kubernetes |
| **Prometheus + Grafana** | Real-time cluster and pod-level monitoring (via `kube-prometheus-stack` Helm chart) |

### Kubernetes Manifests (`k8s/`)
| File | Purpose |
|---|---|
| `deployment.yaml` | Runs 2 replicas of the Flask app with resource limits; `imagePullPolicy: Always` |
| `service.yaml` | Internal `ClusterIP` service, forwards port 80 → 5000 |
| `ingress.yaml` | Provisions a public-facing AWS ALB via ALB Controller annotations |
| `hpa.yaml` | Autoscaling policy: 2–10 pods, scale-up at 50% CPU |
| `servicemonitor.yaml` | Tells Prometheus which services/pods to scrape |

---

## 📊 Live Infrastructure Snapshot

| Resource | Status |
|---|---|
| EKS Cluster | `heart-disease-cluster` — Active, Kubernetes v1.34 |
| ALB | Internet-facing, Active |
| ASG | 2 nodes running, scaling limits 1–2 |
| K8s Workloads | 2 nodes · 25 pods · 19 workloads (via Grafana) |
| HPA Test | Verified: pods scaled 2 → 10 under simulated load, and back to 2 at idle |
| Docker Image | 1.9 GB · 246+ pulls on Docker Hub |

---

## 🚀 Getting Started

### Option 1 — Run Locally
```bash
git clone https://github.com/Waleedcoderarch/waleed.git
cd waleed
pip install -r requirements.txt
python app.py
```

### Option 2 — Run with Docker (Recommended)
```bash
docker build -t heart-predictor .
docker run -p 5000:5000 heart-predictor
```
Then open **http://localhost:5000** in your browser.

### Option 3 — Deploy to Kubernetes (Production)
```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/

# ArgoCD will auto-sync future changes pushed to the k8s/ folder
```

Prerequisites: an EKS cluster, `kubectl` + `eksctl` configured, AWS Load Balancer Controller installed, and ArgoCD deployed in-cluster.

---

## 📸 Screenshots

| Prediction Form | Prediction Result |
|---|---|
| Clean, dark-themed health input form | Instant result with severity percentage (e.g., "Yes — 60.92%") |

| ArgoCD Sync View | Grafana Live Dashboard |
|---|---|
| GitOps deployment status: service, deployment, ingress, pods | Real-time CPU, memory, and network metrics across nodes and pods |

*(See the `/docs` or `/screenshots` folder for full-resolution images of the form, prediction output, Docker Hub repo, CI/CD pipeline, EKS cluster, ALB, ASG, ArgoCD, and Grafana dashboards.)*

---

## 📄 Documentation

Full project documentation is available in this repository:

- 📘 [**Project Documentation**](./Documentation-Heart%20Disease%20Prediction.pdf) — architecture, deployment flow, AWS infrastructure, Kubernetes objects, and full setup walkthrough with screenshots.
- 🔬 [**Scientific Documentation**](./Scientific-Documentation.pdf) — dataset details, feature engineering pipeline, and model comparison methodology.

---

## 📂 Project Structure
```
.
├── app.py                          # Flask application entry point
├── templates/
│   └── index.html                  # Web form + prediction UI
├── gradient_boosting_model.joblib  # Trained ML model
├── Dockerfile
├── requirements.txt
├── .github/
│   └── workflows/
│       └── docker.yml              # CI/CD pipeline (build + push to Docker Hub)
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── servicemonitor.yaml
```

---

## 🔭 Future Improvements
- Add HTTPS via ACM + ALB TLS termination
- Introduce a model registry (MLflow) for versioned retraining
- Add integration tests to the CI pipeline before image push
- Migrate secrets to AWS Secrets Manager / External Secrets Operator
- Add Grafana alerting rules (Slack/email) for pod crash loops or high latency

---

## 👤 Author

**Waleed Ahmed**
GitHub: [github.com/Waleedcoderarch/waleed](https://github.com/Waleedcoderarch/waleed)

---

<div align="center">

*Built as a capstone project demonstrating end-to-end ML system design — from model training to a fully automated, self-healing, monitored cloud deployment.*

</div>
