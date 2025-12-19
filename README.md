<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes"/>
  <img src="https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white" alt="Go"/>
  <img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure"/>
  <img src="https://img.shields.io/badge/Istio-466BB0?style=for-the-badge&logo=istio&logoColor=white" alt="Istio"/>
  <img src="https://img.shields.io/badge/FFmpeg-007808?style=for-the-badge&logo=ffmpeg&logoColor=white" alt="FFmpeg"/>
</p>

# 🐝 StreamHive
[![Demo Video](https://img.shields.io/badge/▶️_Watch_Demo-YouTube-red?style=for-the-badge&logo=youtube)](https://youtu.be/j5ETe2vYVw8?si=bzaoTOCsboHxPKr9)

**A production-grade video streaming platform built with microservices architecture**

StreamHive is a cloud-native video streaming solution featuring HLS adaptive bitrate streaming, deployed on a manually configured Kubernetes cluster in Azure. The platform handles the complete video lifecycle—from upload and transcoding to catalog management and playback.

---

## 📐 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CLOUDFLARE                                          │
│                    (DNS, DDoS Protection, Bot Protection, TLS)                   │
└─────────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         AZURE LOAD BALANCER                                      │
└─────────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      KUBERNETES CLUSTER (Azure VMs)                              │
│  ┌───────────────────────────────────────────────────────────────────────────┐  │
│  │                    ISTIO SERVICE MESH (API Gateway)                        │  │
│  └───────────────────────────────────────────────────────────────────────────┘  │
│                                      │                                           │
│    ┌──────────────┬──────────────┬───┴───────┬───────────────┬──────────────┐   │
│    ▼              ▼              ▼           ▼               ▼              ▼   │
│ ┌──────┐    ┌──────────┐   ┌──────────┐ ┌──────────┐  ┌──────────┐  ┌───────┐  │
│ │Nginx │    │ Upload   │   │Transcoder│ │ Video    │  │ Playback │  │Security│ │
│ │(React│    │ Service  │   │ Service  │ │ Catalog  │  │ Service  │  │Service │ │
│ │Static│    │ (Node.js)│   │ (Go)     │ │ (Go)     │  │ (Go)     │  │ (Go)   │ │
│ └──────┘    └────┬─────┘   └────┬─────┘ └────┬─────┘  └────┬─────┘  └───────┘  │
│                  │              │            │             │                    │
│                  └──────────────┴─────┬──────┴─────────────┘                    │
│                                       ▼                                          │
│                              ┌─────────────────┐                                 │
│                              │    RabbitMQ     │                                 │
│                              │ (Message Queue) │                                 │
│                              └─────────────────┘                                 │
│                                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                     │
│  │   PostgreSQL   │  │     Redis      │  │   Prometheus   │                     │
│  │   (Metadata)   │  │   (Caching)    │  │   + Grafana    │                     │
│  └────────────────┘  └────────────────┘  └────────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           AZURE CLOUD SERVICES                                   │
│  ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐            │
│  │ Azure Blob Storage│  │  Azure Key Vault  │  │   Azure DevOps    │            │
│  │(Videos/Thumbnails)│  │    (Secrets)      │  │   (CI/CD)         │            │
│  └───────────────────┘  └───────────────────┘  └───────────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🎬 Video Pipeline Flow

```
┌──────────┐    ┌──────────────┐    ┌───────────────┐    ┌─────────────────┐
│  Client  │───▶│Upload Service│───▶│   Azure Blob  │    │ TranscoderService│
│          │    │              │    │   (Raw Video) │    │                  │
└──────────┘    └──────┬───────┘    └───────────────┘    └────────┬─────────┘
                       │                                          │
                       │ video.uploaded                           │ FFmpeg
                       ▼                                          │ HLS Transcoding
                ┌─────────────┐                                   │
                │  RabbitMQ   │◀──────────────────────────────────┘
                │  Exchange   │     video.transcoded
                └──────┬──────┘
                       │
                       ▼
              ┌─────────────────┐    ┌──────────────┐    ┌──────────┐
              │VideoCatalogServ.│───▶│PlaybackServ. │───▶│  Client  │
              │ (Metadata DB)   │    │ (HLS URLs)   │    │ (Player) │
              └─────────────────┘    └──────────────┘    └──────────┘
```

---

## 🧩 Microservices

| Service | Technology | Description |
|---------|------------|-------------|
| [**Frontend**](https://github.com/Malisha4065/StreamHive-Frontend) | React, Vite, Nginx | Static SPA served via Nginx for maximum performance |
| [**UploadService**](https://github.com/Malisha4065/StreamHive-UploadService) | Node.js, Express | Handles video uploads to Azure Blob Storage |
| [**TranscoderService**](https://github.com/Malisha4065/StreamHive-TranscoderService) | Go, FFmpeg | Transcodes videos to HLS adaptive bitrate (1080p/720p/480p/360p) |
| [**VideoCatalogService**](https://github.com/Malisha4065/StreamHive-VideoCatalogService) | Go, GORM, PostgreSQL | CRUD API for video metadata, consumes MQ events |
| [**PlaybackService**](https://github.com/Malisha4065/StreamHive-PlaybackService) | Go, Redis | Generates signed HLS playback URLs with caching |
| [**SecurityService**](https://github.com/Malisha4065/StreamHive-SecurityService) | Go, JWT, bcrypt | Authentication & authorization (signup/login/validate) |
| [**Deployment-Pipelines**](https://github.com/Malisha4065/StreamHive-Pipelines) | Azure DevOps YAML | CI/CD pipelines for building and deploying all services |

---

## 🛡️ Security & Reliability

### Network Security
- **Cloudflare** — DNS, DDoS protection, Bot protection
- **Full (Strict) TLS** — Origin server TLS certificate with Cloudflare verification
- **Istio mTLS** — Service-to-service encryption within the cluster

### Application Security
- **JWT Authentication** — Stateless token-based auth via SecurityService
- **Azure Key Vault** — Secure secret management
- **Rate Limiting** — Protection against abuse

### Resilience
- **Circuit Breakers** — Using [sony/gobreaker](https://github.com/sony/gobreaker) in Go services
- **Health Checks** — Kubernetes liveness and readiness probes
- **Message Queue** — RabbitMQ with retry/DLQ strategy for async processing

---

## 📊 Observability

| Tool | Purpose |
|------|---------|
| **Prometheus** | Metrics collection from all services |
| **Grafana** | Dashboards and alerting |
| **Structured Logging** | Zap logger in Go services |

---

## 🚀 Technology Stack

### Backend
- **Go** — Microservices (Transcoder, VideoCatalog, Playback, Security)
- **Node.js** — Upload Service
- **PostgreSQL** — Primary database for video metadata
- **Redis** — Caching layer for playback URLs
- **RabbitMQ** — Message broker (topic exchange pattern)
- **FFmpeg** — Video transcoding to HLS

### Frontend
- **React** — UI framework
- **Vite** — Build tool
- **Nginx** — Static file server (in production)

### Infrastructure
- **Kubernetes** — Container orchestration (manually configured on Azure VMs)
- **Istio** — Service mesh & API gateway
- **Docker** — Containerization
- **Azure DevOps** — CI/CD pipelines

### Azure Services
- **Azure Blob Storage** — Video segments and thumbnails
- **Azure Key Vault** — Secrets management
- **Azure Load Balancer** — Traffic distribution
- **Azure VMs** — Kubernetes worker nodes

---

## 🏗️ Local Development

### Prerequisites
- Docker & Docker Compose
- Node.js 18+
- Go 1.21+
- FFmpeg

### Quick Start

1. **Clone the repository with submodules:**
   ```bash
   git clone --recursive https://github.com/Malisha4065/StreamHive.git
   cd StreamHive
   ```

2. **Set up environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your Azure Storage credentials
   ```

3. **Start all services:**
   ```bash
   docker-compose up --build
   ```

4. **Access the application:**
   - Frontend: http://localhost:5173
   - Upload API: http://localhost:3001
   - Catalog API: http://localhost:8080
   - Playback API: http://localhost:8090
   - RabbitMQ Management: http://localhost:15672

---

## 📁 Project Structure

```
StreamHive/
├── Frontend/               # React SPA (submodule)
├── UploadService/          # Node.js upload service (submodule)
├── TranscoderService/      # Go FFmpeg transcoder (submodule)
├── VideoCatalogService/    # Go metadata API (submodule)
├── PlaybackService/        # Go playback URL service (submodule)
├── SecurityService/        # Go auth service (submodule)
├── Deployment-Pipelines/   # Azure DevOps YAML pipelines (submodule)
├── docker-compose.yml      # Local development stack
└── README.md
```

---

## 📺 Demo

[![StreamHive Demo](https://img.youtube.com/vi/j5ETe2vYVw8/maxresdefault.jpg)](https://youtu.be/j5ETe2vYVw8?si=bzaoTOCsboHxPKr9)

**▶️ [Watch the full demo on YouTube](https://youtu.be/j5ETe2vYVw8?si=bzaoTOCsboHxPKr9)**

---

## 👥 Contributors

Built with ❤️ by the StreamHive team.

---

## 📄 License

This project is licensed under the MIT License.
