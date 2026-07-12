# 🚗 Vehicle Insurance Prediction — End-to-End MLOps Pipeline

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-S3%20%7C%20EC2%20%7C%20ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)

**A production-grade Machine Learning pipeline that takes a model from raw data to a live, auto-scaling web service — fully automated with CI/CD.**

[Overview](#-overview) • [Architecture](#-architecture) • [Tech Stack](#-tech-stack) • [Pipeline](#-ml-pipeline-stages) • [CI/CD](#-cicd--deployment) • [Setup](#-local-setup) • [Project Structure](#-project-structure)

</div>

---

## 📌 Overview

This project implements a complete **MLOps lifecycle** for a vehicle insurance response-prediction model — covering everything from data ingestion out of a live MongoDB database, through validation, transformation, training, evaluation, and versioned deployment to AWS S3, all the way to a FastAPI web application serving real-time predictions. The entire deployment pipeline is containerized with Docker and automated via a self-hosted GitHub Actions runner on an EC2 instance.

This isn't just a model in a notebook — it's a **modular, production-style pipeline** built with industry-standard software engineering practices: custom logging, custom exception handling, configuration-driven components, and infrastructure-as-code style deployment.

---

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────────┐
│  MongoDB    │────▶│    Data      │────▶│      Data      │────▶│      Data       │
│   Atlas     │     │  Ingestion   │     │   Validation   │     │  Transformation │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────────┘
                                                                          │
                                                                          ▼
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────────┐
│   FastAPI   │◀────│    Model     │◀────│     Model      │◀────│      Model      │
│  Web App    │     │    Pusher    │     │   Evaluation   │     │     Trainer      │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────────┘
       │                    │
       ▼                    ▼
┌─────────────┐     ┌──────────────┐
│  End Users  │     │   AWS S3     │
│  (Browser)  │     │ Model Registry│
└─────────────┘     └──────────────┘

        ⬇ Continuous Integration / Continuous Deployment ⬇

  GitHub Push → GitHub Actions → Build Docker Image → Push to ECR
        → Self-Hosted Runner (EC2) → Pull & Deploy Container
```

---

## 🛠️ Tech Stack

| Category | Tools & Services |
|---|---|
| **Language** | Python 3.10 |
| **Web Framework** | FastAPI + Uvicorn + Jinja2 Templates |
| **Database** | MongoDB Atlas (NoSQL, cloud-hosted) |
| **ML / Data** | Pandas, NumPy, Scikit-learn |
| **Cloud Storage** | AWS S3 (model registry & versioning) |
| **Cloud Compute** | AWS EC2 (Ubuntu 24.04) |
| **Container Registry** | AWS ECR |
| **Containerization** | Docker |
| **CI/CD** | GitHub Actions (self-hosted runner) |
| **IAM & Security** | AWS IAM (scoped access keys), GitHub Secrets |
| **Packaging** | setuptools, `pyproject.toml`, editable installs |
| **Logging & Monitoring** | Custom rotating file logger |
| **Error Handling** | Custom exception wrapper with full traceback context |

---

## 🔄 ML Pipeline Stages

The pipeline is broken into independently testable, config-driven components — the same pattern used in real production ML systems:

| Stage | Responsibility |
|---|---|
| **1. Data Ingestion** | Pulls raw data from MongoDB Atlas, converts to a structured DataFrame, and exports it into the feature store. |
| **2. Data Validation** | Validates incoming data against a defined `schema.yaml` — checks column names, types, and data drift. |
| **3. Data Transformation** | Applies preprocessing, encoding, and feature engineering pipelines; prepares train/test arrays. |
| **4. Model Trainer** | Trains the ML model and evaluates it against target performance metrics (F1, Precision, Recall). |
| **5. Model Evaluation** | Compares the newly trained model against the currently deployed production model in S3, using a configurable performance threshold. |
| **6. Model Pusher** | Pushes the new model to the AWS S3 model registry **only if it outperforms** the existing production model. |
| **7. Prediction Pipeline** | Loads the production model from S3 on-demand and serves live predictions through the FastAPI app. |

Every stage is wrapped in a custom **exception handler** that captures the exact file name and line number of any failure, and a custom **rotating logger** that writes timestamped logs to both console and disk — making the whole pipeline debuggable in production, not just in development.

---

## 🚀 CI/CD & Deployment

This project ships itself. Every push to the main branch triggers a fully automated deployment pipeline:

```
 Git Push
    │
    ▼
 GitHub Actions Workflow Triggered
    │
    ▼
 Build Docker Image
    │
    ▼
 Push Image to AWS ECR (Elastic Container Registry)
    │
    ▼
 Self-Hosted Runner on EC2 Pulls Latest Image
    │
    ▼
 Container Deployed & Live on Port 5080
```

**Infrastructure highlights:**
- 🔐 Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `ECR_REPO`) are managed securely through **GitHub Secrets** — never hardcoded.
- 🐳 The app is fully **containerized** with a slim Python base image for fast, reproducible builds.
- 🖥️ A **self-hosted GitHub Actions runner** on an EC2 instance handles deployment, avoiding third-party runner limitations.
- 🌐 Custom **inbound security group rules** expose the app on a dedicated port for public access.
- 📦 A dedicated **on-demand training route** allows retraining the model directly from the live web app.

---

## ⚙️ Local Setup

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd MLOps-project1

# 2. Create and activate a virtual environment
conda create -n vehicle python=3.10 -y
conda activate vehicle

# 3. Install dependencies
pip install -r requirements.txt
pip install -e .

# 4. Set up environment variables (create a .env file)
echo "MONGODB_URL=your_mongodb_connection_string" >> .env
echo "AWS_ACCESS_KEY_ID=your_aws_access_key" >> .env
echo "AWS_SECRET_ACCESS_KEY=your_aws_secret_key" >> .env

# 5. Run the training pipeline
python demo.py

# 6. Launch the web app
python app.py
```

Then open `http://localhost:5000` in your browser. 🎉

### 🐳 Run with Docker

```bash
docker build -t vehicle-insurance-app .
docker run --env-file .env -p 5000:5000 vehicle-insurance-app
```

---

## 📁 Project Structure

```
MLOps-project1/
├── src/
│   ├── components/          # Core pipeline stages (ingestion, validation, transformation, trainer, evaluation, pusher)
│   ├── configuration/        # MongoDB & AWS connection setup
│   ├── cloud_storage/         # S3 read/write abstraction layer
│   ├── data_access/            # MongoDB → DataFrame transformation layer
│   ├── entity/                   # Config & artifact schema classes + S3 estimator
│   ├── exception/                  # Custom exception handling
│   ├── logger/                       # Custom rotating file + console logger
│   ├── pipline/                        # Training & prediction pipeline orchestration
│   ├── constants/                        # Centralized project constants
│   └── utils/                              # Shared utility functions
├── notebook/                # EDA, feature engineering & MongoDB push notebooks
├── config/                  # schema.yaml — dataset validation schema
├── static/ & templates/     # Frontend assets for the FastAPI app
├── .github/workflows/       # CI/CD pipeline definition
├── app.py                   # FastAPI application entry point
├── demo.py                  # Training pipeline entry point
├── Dockerfile
├── requirements.txt
├── setup.py
└── pyproject.toml
```

---

## 🎯 Key Engineering Highlights

- ✅ **Modular, config-driven pipeline** — each stage reads its configuration from centralized entity classes, not hardcoded values.
- ✅ **Automated model governance** — new models are only promoted to production if they beat the current model on a defined performance threshold.
- ✅ **Secrets management** — zero credentials committed to source control; environment-based configuration throughout.
- ✅ **Full CI/CD automation** — from `git push` to a live, publicly accessible endpoint, with no manual deployment steps.
- ✅ **Cloud-native storage** — model artifacts are versioned and served directly from AWS S3, decoupling the app from local storage.
- ✅ **Production-style observability** — structured logging and traceable custom exceptions across every layer of the pipeline.

---

<div align="center">

Built as a hands-on demonstration of real-world MLOps practices — from data to deployment. 🚀

</div>