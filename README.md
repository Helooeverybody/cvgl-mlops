# 🛸Cross-View Geo-Localization - MLOPs Platform

[![MLOps](https://img.shields.io/badge/Architecture-MLOps-20232A?style=for-the-badge&logo=git)](https://mlops.org/)
[![Computer Vision](https://img.shields.io/badge/Domain-Computer_Vision-5C3EE8?style=for-the-badge&logo=opencv)](https://opencv.org/)
[![CI/CD Pipeline](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-blue?style=for-the-badge&logo=github)](https://github.com/)
[![Serving](https://img.shields.io/badge/Serving-FastAPI%20%7C%20ONNX-009688?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Tracking](https://img.shields.io/badge/Tracking-MLflow-0194E2?style=for-the-badge&logo=mlflow)](https://mlflow.org/)



## 📖 Executive Summary
MLOps pipeline for Cross-View Geo-Localization for UAV simulation

## 🏗️Architecture

<img width="1396" height="881" alt="pipeline" src="https://github.com/user-attachments/assets/d4438f95-3f30-40a1-a998-3c1f54f91564" />

## 🛠️Tech Stack 

This project strictly adheres to open-source, enterprise-grade tooling. 

## 🗄️ Data Engineering & Versioning
| Technology | Role | Justification |
| :--- | :--- | :--- |
| **CVAT** | Data Annotation | Enterprise image/video annotation tool for computer vision. |
| **DVC** | Data Version Control | Couples data lineage tightly with Git commits. Ensures full reproducibility. |
| **MinIO** | Object Storage | High-performance, S3-compatible object storage for datasets and model binaries. |
| **PostgreSQL**| Relational Database | Persistent backend store for MLflow metrics and Airflow metadata. |

## 🧠 Model Training & Orchestration
| Technology | Role | Justification |
| :--- | :--- | :--- |
| **Apache Airflow** | DAG Orchestration | Schedules and manages complex dependencies between data extraction, training, and deployment. |
| **MLflow** | Experiment Tracking & Registry | Centralizes model metadata, versioning, and lifecycle management (Staging -> Production). |
| **ONNX** | Model Quantization | Standardizes model formats and reduces precision (e.g., FP32 to INT8) for faster edge inference. |
| **Jupyter** | Experimentation | Interactive environment for initial EDA and model prototyping. |

## 🚀 High-Performance Serving
| Technology | Role | Justification |
| :--- | :--- | :--- |
| **FastAPI** | Inference API | Asynchronous, high-throughput Python web framework for serving predictions. |
| **NGINX** | Reverse Proxy | Handles SSL termination, load balancing, and rate limiting for UAV client requests. |
| **Redis** | In-Memory Cache | Caches frequent identical predictions to reduce GPU/CPU compute overhead. |
| **FAISS** | Similarity Search | Meta's library for ultra-fast dense vector similarity clustering and retrieval. |

## 👁️ Observability & CI/CD
| Technology | Role | Justification |
| :--- | :--- | :--- |
| **Prometheus** | Metric Scraping | Collects system metrics (CPU/Memory) and ML metrics (prediction distributions, drift). |
| **Grafana / Loki** | Dashboards & Logs | Unified observability UI. Loki handles log aggregation from the FastAPI pods. |
| **GitHub Actions** | CI/CD | Automates code formatting, unit testing, Docker image builds, and deployments. |

---


## 📁Project Structure

```
mlo-uav-project/
│
├── .github/                      # CI/CD Versioning (GitHub Actions as per diagram)
│   ├── workflows/
│   │   ├── train_pipeline.yaml   # Triggers training via Airflow/Runners
│   │   ├── deploy_serving.yaml   # Builds FastAPI/Nginx images and deploys
│   │   └── data_validation.yaml  # Runs tests on new data pushes
│
├── .dvc/                         # DVC Configuration (Data versioning)
│   ├── config                    # Connects to MinIO backend
│   └── .gitignore
│
├── notebooks/                    # Model Experiment (Jupyter as per diagram)
│   ├── 01_data_exploration.ipynb
│   ├── 02_model_prototyping.ipynb
│   └── exploratory_faiss_tests.ipynb
│
├── orchestration/                # Apache Airflow orchestration
│   ├── dags/
│   │   ├── model_training_dag.py # Orchestrates extraction, training, ONNX export
│   │   └── auto_retrain_dag.py   # Triggered by webhook if Drift > Threshold
│   ├── plugins/
│   └── requirements_airflow.txt
│
├── src/                          # Main source code
│   ├── training/                 # Training Code & Model evaluating
│   │   ├── conf/                 # Hydra or YAML configs (hyperparameters)
│   │   ├── data_prep/
│   │   │   ├── cvat_export.py    # Scripts to pull annotated data from CVAT
│   │   │   └── dataset.py        # PyTorch/TensorFlow dataset loaders
│   │   ├── models/
│   │   │   └── architecture.py   # Neural network definitions
│   │   ├── train.py              # Main training loop (Logs to MLflow)
│   │   ├── evaluate.py           # Evaluation logic
│   │   ├── onnx_export.py        # Model Quantization & ONNX conversion script
│   │   └── requirements.txt      # Heavy ML dependencies (Torch, TF, etc.)
│   │
│   └── serving/                  # Serving Code (FastAPI + Redis + FAISS)
│       ├── api/
│       │   ├── routes.py         # FastAPI endpoints (e.g., /predict)
│       │   └── schemas.py        # Pydantic models for request/response
│       ├── core/
│       │   ├── config.py         # App settings (MinIO keys, Redis URLs)
│       │   └── logger.py         # Structured logging (feeds into Promtail/Loki)
│       ├── services/
│       │   ├── model_loader.py   # Pulls .onnx model from MLflow Registry/MinIO
│       │   ├── inference.py      # ONNX Runtime execution logic
│       │   ├── faiss_service.py  # Similarity search logic
│       │   └── cache.py          # Redis caching logic
│       ├── main.py               # FastAPI application entry point
│       └── requirements.txt      # Lightweight serving dependencies (FastAPI, ONNXRuntime)
│
├── infra/                        # Infrastructure as Code & Deployments
│   ├── docker/
│   │   ├── Dockerfile.training   # Image for training jobs
│   │   ├── Dockerfile.serving    # Image for FastAPI serving
│   │   └── Dockerfile.airflow    # Custom Airflow image with DVC/MLflow plugins
│   │
│   ├── nginx/                    # Reverse Proxy for UAV Camera requests
│   │   └── nginx.conf            # Rate limiting, SSL, load balancing
│   │
│   ├── monitoring/               # Monitoring stack configurations
│   │   ├── prometheus/
│   │   │   └── prometheus.yml    # Scrapes FastAPI metrics and Drift metrics
│   │   ├── grafana/
│   │   │   └── provisioning/     # Dashboards as code (System & Data Drift dashboards)
│   │   └── loki/
│   │       └── promtail.yml      # Log collection config
│   │
│   └── docker-compose.yml        # Local testing of the entire stack (MinIO, MLflow, Redis, etc.)
│
├── data/                         # Local data pointers (Ignored in Git, Tracked by DVC)
│   ├── raw.dvc                   # Points to raw MinIO data
│   ├── processed.dvc             # Points to cleaned MinIO data
│   └── .gitignore
│
├── tests/                        # Unit and Integration Tests
│   ├── test_training/
│   ├── test_serving/
│   └── test_faiss_retrieval.py
│
├── Makefile                      # Standardized commands for CI and Developers
├── .pre-commit-config.yaml       # Code formatting (Black, Ruff, MyPy)
├── .gitignore
└── README.md                     # Project documentation & Architecture explanation

```

## 💻 Getting Started

## 📄License 
This project is licensed under the MIT License - see the LICENSE file for details.
