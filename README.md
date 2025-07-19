# hyperbandit
HyperBandit Implementation Guide: Phase 1Introduction to Part 1: Laying the Cornerstone for Production MLThe initial phase of the HyperBandit project is the most consequential. The architectural decisions and configurations established here form the bedrock upon which the entire platform's stability, scalability, and maintainability will rest. The project vision emphasizes a "production-first mentality," a principle that will guide every action in this foundational stage.1 This section provides an exhaustive guide to architecting a professional local development environment that mirrors production, establishing rigorous and automated code quality standards, and provisioning a secure, scalable, and fully reproducible cloud infrastructure on Google Cloud Platform (GCP) using Infrastructure as Code (IaC). This meticulous approach is designed to de-risk the project by minimizing deployment friction, preventing configuration drift, and ultimately accelerating the entire development lifecycle from data engineering to model serving.Section 1: Architecting the Development EnvironmentA professional development environment is not a matter of convenience; it is a prerequisite for building enterprise-grade software. The objective of this section is to create a consistent, powerful, and automated local setup. This environment will enable rapid, iterative development while programmatically enforcing that all code committed to the repository adheres to the highest standards of quality and consistency.1.1 Local Environment Configuration: Python and Dependency ManagementThe first step is to establish a clean, isolated, and reproducible Python environment. This prevents conflicts with system-level packages and ensures that all developers on the project, as well as the production containers, use the exact same set of dependencies.1.1.1 Python Version ManagementThe project specifies Python 3.11+.1 To manage Python versions without interfering with the system's default Python installation, pyenv is the recommended tool.Step-by-step instructions:Install pyenv following the official instructions for your operating system.Install the target Python version:Bashpyenv install 3.11.9
Set the local Python version for the project directory. This creates a .python-version file that pyenv will automatically use.Bashcd hyperbandit/
pyenv local 3.11.9
1.1.2 Virtual Environment and DependenciesWithin the project's Python version, a virtual environment will be created to isolate dependencies.Step-by-step instructions:Create a virtual environment named .venv using Python's built-in venv module:Bashpython -m venv.venv
Activate the virtual environment. This must be done in every new terminal session.On macOS/Linux:Bashsource.venv/bin/activate
On Windows:..venv\Scripts\activate```Initialize the project's dependency file. All Python packages will be listed here.Bashtouch requirements.txt
1.1.3 Git Ignore ConfigurationTo prevent committing unnecessary or sensitive files to the version control system, a .gitignore file is essential. A comprehensive configuration for a Python project should ignore virtual environments, IDE configuration files, Python bytecode, build artifacts, and local environment files.2Create the .gitignore file in the project root with the following content:Code snippet#.gitignore

# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
*.manifest
*.spec

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
.python-version

# PEP 582; __pypackages__
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak
venv.bak

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# Terraform
.terraform/
.terraform.lock.hcl
*.tfstate
*.tfstate.*
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
1.2 Version Control Strategy: Implementing a Professional Gitflow WorkflowA disciplined branching strategy is crucial for managing complexity in a multi-phase project like HyperBandit. The Gitflow workflow is selected for its robust framework that aligns perfectly with a project involving distinct development, release, and maintenance cycles.4 While some modern web applications favor simpler trunk-based development for high-frequency, small-increment deployments, the phased nature of this project—spanning data engineering, model development, and application deployment—benefits immensely from Gitflow's structured approach.1 It provides clarity and stability by isolating different types of work into dedicated branches.1.2.1 Core Branch InitializationThe workflow is centered around two permanent branches: main and develop.main: This branch contains the official, production-ready release history. Every commit on main is a tagged, deployable version of the software.5develop: This is the primary integration branch where all completed features are merged. It represents the cutting-edge state of development for the next release.6Step-by-step instructions:Initialize the Git repository and make the first commit with the existing project structure.Bashgit init
git add.
git commit -m "Initial commit: Project structure setup"
The default branch is typically named main. Create the develop branch from main.Bashgit branch develop
1.2.2 Gitflow Branching ModelThe following table summarizes the roles and interactions of the different branch types in the Gitflow model.Branch TypeNaming ConventionParent BranchPurpose & LifecycleMainmain-Permanent. Contains only tagged, production-ready release code. 5DevelopdevelopmainPermanent. Main integration branch for all completed features. The "source of truth" for upcoming releases. 6Featurefeature/*developTemporary. Develop new features in isolation. Merged back to develop upon completion. Never interacts with main. 4Releaserelease/*developTemporary. Prepares a new production release. Only bug fixes and release-oriented tasks allowed. Merged to main and develop. 5Hotfixhotfix/*mainTemporary. Addresses critical bugs in a production release. Merged to main and develop. 71.2.3 Common Workflow CommandsStarting a new feature (e.g., adding user authentication):Bash# Ensure develop is up-to-date
git checkout develop
git pull origin develop

# Create the feature branch
git checkout -b feature/user-authentication
Finishing a feature: Once development is complete, the feature/user-authentication branch is pushed to the remote repository, and a Pull Request (PR) is opened to merge it into develop. It should never be merged directly into main.Starting a release: When develop has accumulated enough features for a release (e.g., v1.0.0), a release branch is created.Bashgit checkout -b release/v1.0.0 develop
From this point, only bug fixes and release-specific changes are committed to this branch. The develop branch is now free to accept new features for the next release cycle.Finishing a release: Once the release is stable, it is merged into main and tagged. It is also merged back into develop to ensure any fixes made on the release branch are incorporated into future work.6Bash# Switch to main and merge the release
git checkout main
git merge --no-ff release/v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# Switch to develop and merge the release
git checkout develop
git merge --no-ff release/v1.0.0

# Delete the temporary release branch
git branch -d release/v1.0.0
1.3 Enforcing Code Quality with Pre-Commit HooksTo ensure a high-quality and consistent codebase, automated checks will be run before any code is committed. The pre-commit framework facilitates this by managing a set of hooks that format, lint, and check code for common errors.8Step-by-step instructions:Install the necessary Python packages:Bashpip install pre-commit black isort flake8
Create the pre-commit configuration file .pre-commit-config.yaml in the project root.Install the git hooks into the local .git/ directory. This command needs to be run only once per project clone.Bashpre-commit install
1.3.1 Configuration and Conflict ResolutionA common issue arises from conflicts between the black code formatter and the flake8 linter.10black is an opinionated formatter that enforces a strict style, which sometimes contradicts flake8's default rules (e.g., E203: whitespace before ':'). To resolve this, flake8 will be configured to ignore the rules that black manages, ensuring they work harmoniously.11File: .pre-commit-config.yamlThis file defines the hooks that will run on staged files during git commit.YAML#.pre-commit-config.yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files

-   repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
    -   id: black
        # It is recommended to specify the language_version
        # to ensure consistent formatting across environments.
        language_version: python3.11

-   repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
    -   id: isort
        name: isort (python)
        args: ["--profile", "black"]

-   repo: https://github.com/pycqa/flake8
    rev: 7.1.0
    hooks:
    -   id: flake8
        # Exclude directories that are not part of the source code
        args: ["--config=.flake8"]
        additional_dependencies: [flake8-bugbear]
File: .flake8This configuration file tells flake8 which rules to apply and which to ignore.Ini, TOML#.flake8
[flake8]
# Ignore rules that conflict with black
# E203: whitespace before ':'
# W503: line break before binary operator
ignore = E203, W503
# Set max line length to match black's default
max-line-length = 88
# Set max complexity to encourage simpler functions
max-complexity = 18
# Exclude auto-generated or third-party directories
exclude =.git,__pycache__,.venv,build,dist
Now, every time git commit is run, isort will sort imports, black will format the code, and flake8 will lint for logical errors, all automatically.1.4 Containerization with Docker for Service IsolationContainerization with Docker is fundamental to the project's "production-first" approach. It packages the application and its dependencies into a standardized unit for development, testing, and deployment. This ensures that the application runs the same way everywhere, from a developer's laptop to the production GKE cluster.A multi-stage Dockerfile will be used to create lean, secure, and efficient images. This approach separates the build environment from the final runtime environment, resulting in a smaller image size and reduced attack surface because build tools are not included in the final image.12File: DockerfileThis file will be placed in the project root.Dockerfile# Dockerfile

# ---- Stage 1: Builder ----
# This stage installs all dependencies, including build-time requirements.
FROM python:3.11.9-slim-bookworm AS builder

# Set the working directory
WORKDIR /app

# Prevent python from writing.pyc files
ENV PYTHONDONTWRITEBYTECODE 1
# Ensure python output is sent straight to the terminal without buffering
ENV PYTHONUNBUFFERED 1

# Install system dependencies required for building some Python packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Create and activate a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy and install Python dependencies
# Copying requirements.txt first leverages Docker's layer caching.
# This layer will only be rebuilt if requirements.txt changes.
COPY requirements.txt.
RUN pip install --no-cache-dir -r requirements.txt

# ---- Stage 2: Final Image ----
# This stage creates the final, lean production image.
FROM python:3.11.9-slim-bookworm

# Set the working directory
WORKDIR /app

# Copy the virtual environment from the builder stage
COPY --from=builder /opt/venv /opt/venv

# Make the virtual environment's binaries accessible
ENV PATH="/opt/venv/bin:$PATH"

# Copy the application source code
COPY./src /app/src
COPY./tests /app/tests
# Add other directories as they are developed

# Expose the port the API will run on (to be used by FastAPI)
EXPOSE 8000

# The command to run the application will be specified in docker-compose.yml
# or the Kubernetes deployment configuration. This makes the image more flexible.
# Example command for running the API:
# CMD ["uvicorn", "src.api.main:app", "--host", "0.0.0.0", "--port", "8000"]
1.5 Local Service Orchestration with Docker ComposeTo simulate the entire production environment locally, docker-compose will be used to define and run the multi-service application stack. This includes the application itself, the PostgreSQL database, the Redis cache, and the complete Apache Kafka stack (Zookeeper, Broker, Schema Registry).1 This enables comprehensive, end-to-end testing with a single command: docker-compose up.A critical aspect of configuring Kafka in Docker is managing its network listeners. The Kafka broker must advertise an address that clients can connect to. Inside the Docker network, services communicate via their service names (e.g., broker), while an application running on the host machine needs to connect via localhost. The KAFKA_ADVERTISED_LISTENERS variable allows the broker to broadcast multiple addresses for these different contexts, a common point of failure in local setups if not configured correctly.13File: docker-compose.ymlThis file orchestrates all backend services for local development.YAML# docker-compose.yml
version: '3.8'

services:
  # PostgreSQL Database for MLflow, metadata, etc.
  postgres:
    image: postgres:15-alpine
    container_name: postgres_db
    environment:
      POSTGRES_USER: hyperbandit
      POSTGRES_PASSWORD: password
      POSTGRES_DB: hyperbandit_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test:
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis for Online Feature Store and Caching
  redis:
    image: redis:7-alpine
    container_name: redis_cache
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped
    healthcheck:
      test:
      interval: 10s
      timeout: 5s
      retries: 5

  # Zookeeper for Kafka Coordination
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.1
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  # Kafka Broker for Real-time Event Streaming
  broker:
    image: confluentinc/cp-kafka:7.6.1
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      # Port for external connections from the host machine
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      # Listeners configuration for multi-network access
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

  # Confluent Schema Registry for managing Avro schemas
  schema-registry:
    image: confluentinc/cp-schema-registry:7.6.1
    container_name: schema-registry
    hostname: schema-registry
    depends_on:
      - broker
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'broker:29092'
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8081

volumes:
  postgres_data:
  redis_data:
Section 2: Provisioning Production Infrastructure on GCP with TerraformWith the local environment established, the next step is to define the production cloud infrastructure. Using Terraform, an Infrastructure as Code (IaC) tool, ensures that the cloud environment is version-controlled, reproducible, and can be managed programmatically.15 This eliminates manual configuration errors and provides a reliable path to deploying, updating, or destroying the infrastructure.2.1 GCP Project Setup and AuthenticationBefore writing Terraform code, the GCP project and local authentication must be configured.Step-by-step instructions:Create a new GCP Project. Replace your-unique-project-id with a globally unique ID.Bashgcloud projects create your-unique-project-id --name="HyperBandit Project"
Set the new project as the default for the gcloud CLI:Bashgcloud config set project your-unique-project-id
Link the project to a billing account through the Google Cloud Console.Enable all required APIs. Manually enabling APIs is tedious and error-prone. The following script automates this process idempotently.16Table: GCP Services and API IdentifiersService NameGCP API IdentifierJustification for InclusionCompute Enginecompute.googleapis.comRequired by GKE for node creation and networking.Kubernetes Enginecontainer.googleapis.comCore service for running the GKE cluster.1Cloud SQL Adminsqladmin.googleapis.comTo provision and manage the PostgreSQL database.1Cloud Storagestorage.googleapis.comFor Terraform state backend and future data/model storage.Memorystore for Redisredis.googleapis.comFor online feature store and API caching.1Artifact Registryartifactregistry.googleapis.comTo store Docker container images for deployment to GKE.Cloud Logginglogging.googleapis.comFoundational for monitoring and observability.Cloud Monitoringmonitoring.googleapis.comFoundational for monitoring and observability.**Script: `enable_gcp_apis.sh`**
```bash
#!/bin/bash
set -e

APIS_TO_ENABLE=(
  "compute.googleapis.com"
  "container.googleapis.com"
  "sqladmin.googleapis.com"
  "storage.googleapis.com"
  "redis.googleapis.com"
  "artifactregistry.googleapis.com"
  "logging.googleapis.com"
  "monitoring.googleapis.com"
  "iam.googleapis.com"
)

echo "Enabling required GCP APIs..."
gcloud services enable "${APIS_TO_ENABLE[@]}" --project=$(gcloud config get-value project)
echo "All APIs enabled successfully."
```
Authenticate the local environment for Terraform. Using Application Default Credentials (ADC) is the modern, secure standard, as it avoids the need to download and manage static service account keys.19Bashgcloud auth application-default login
2.2 Infrastructure as Code (IaC) with TerraformThe infrastructure will be defined in a series of modular Terraform files within the infrastructure/ directory.Table: Terraform Module StructureModule NamePurposeKey Terraform Resourcesgcs_backendCreates the GCS bucket for storing remote Terraform state.google_storage_bucket, google_storage_bucket_iam_membernetworkDefines the core networking infrastructure (VPC, subnets, firewall).google_compute_network, google_compute_subnetwork, google_compute_firewallgke_clusterProvisions the GKE Autopilot cluster for container orchestration.google_container_clusterdata_storesProvisions the managed databases for persistence and caching.google_sql_database_instance, google_sql_user, google_redis_instanceiamCreates service accounts and manages IAM bindings.google_service_account, google_project_iam_binding2.2.1 Terraform Backend and Provider ConfigurationThe first step in writing the Terraform code is to configure the provider and set up a secure, remote backend for the state file. Storing the state file (terraform.tfstate) in a GCS bucket is a critical best practice. It provides state locking to prevent concurrent modifications, a durable and versioned history of the infrastructure's state, and a central source of truth for team collaboration.21File: infrastructure/terraform/backend.tfThis file defines the GCS bucket that will store the Terraform state.Terraform# infrastructure/terraform/backend.tf

# This resource creates the GCS bucket that will be used for the remote backend.
# This part is run first with a local backend, then the configuration is migrated.
resource "google_storage_bucket" "tfstate" {
  name          = "hyperbandit-tfstate-bucket-${data.google_project.current.number}"
  location      = "US-CENTRAL1"
  force_destroy = false # Set to true only in non-production for easy cleanup

  # Enable versioning to keep a history of state files, allowing for rollbacks.
  versioning {
    enabled = true
  }

  # Prevent public access to the state file bucket.
  public_access_prevention = "enforced"
}
File: infrastructure/terraform/main.tfThis is the main entry point for the Terraform configuration.Terraform# infrastructure/terraform/main.tf

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }

  # This block is commented out initially. After the bucket is created,
  # this is uncommented and `terraform init -migrate-state` is run.
  # backend "gcs" {
  #   bucket  = "hyperbandit-tfstate-bucket-YOUR_PROJECT_NUMBER" # Replace with your bucket name
  #   prefix  = "terraform/state"
  # }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Data source to get current project details
data "google_project" "current" {}
File: infrastructure/terraform/variables.tfThis file defines input variables for the configuration.Terraform# infrastructure/terraform/variables.tf

variable "project_id" {
  description = "The GCP project ID to deploy resources into."
  type        = string
}

variable "region" {
  description = "The primary GCP region for deployment."
  type        = string
  default     = "us-central1"
}
2.2.2 Core Networking (VPC)A dedicated Virtual Private Cloud (VPC) provides network isolation for the project's resources.File: infrastructure/terraform/network.tfTerraform# infrastructure/terraform/network.tf

resource "google_compute_network" "vpc_network" {
  name                    = "hyperbandit-vpc"
  auto_create_subnetworks = false # Use custom subnets for better control
  project                 = var.project_id
}

resource "google_compute_subnetwork" "main_subnet" {
  name          = "hyperbandit-main-subnet"
  ip_cidr_range = "10.10.0.0/20"
  region        = var.region
  network       = google_compute_network.vpc_network.id
  project       = var.project_id
}

resource "google_compute_firewall" "allow_ssh_iap" {
  name    = "hyperbandit-allow-ssh-iap"
  network = google_compute_network.vpc_network.name
  project = var.project_id

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  # GCP's IAP range for secure SSH access without public IPs
  source_ranges = ["35.235.240.0/20"]
  target_tags   = ["allow-ssh-iap"]
}
2.2.3 Managed Kubernetes (GKE)A Google Kubernetes Engine (GKE) cluster will be provisioned to orchestrate the application's containers. GKE Autopilot is chosen to align with the project's goal of achieving "efficiency gains".1 Autopilot automates node provisioning, scaling, and management, which reduces operational overhead and can lower costs for workloads with variable demand.15File: infrastructure/terraform/gke.tfTerraform# infrastructure/terraform/gke.tf

resource "google_container_cluster" "primary_cluster" {
  name     = "hyperbandit-cluster"
  location = var.region
  project  = var.project_id

  # Use GKE Autopilot for managed node pools and scaling
  enable_autopilot = true

  # Networking configuration
  network    = google_compute_network.vpc_network.id
  subnetwork = google_compute_subnetwork.main_subnet.id

  # Disable legacy metadata endpoints for enhanced security
  node_config {
    metadata = {
      disable-legacy-endpoints = "true"
    }
  }

  # Recommended for production clusters
  deletion_protection = false # Set to true for production environments
}
2.2.4 Data Persistence Layers (Cloud SQL & Memorystore)Managed services for PostgreSQL and Redis will be provisioned for the backend data stores. Cloud SQL provides a reliable, managed relational database, and Memorystore for Redis offers a low-latency, in-memory store ideal for the online feature store and API caching requirements.1File: infrastructure/terraform/data_stores.tfTerraform# infrastructure/terraform/data_stores.tf

# Managed PostgreSQL instance
resource "google_sql_database_instance" "postgres_instance" {
  name             = "hyperbandit-postgres-instance"
  database_version = "POSTGRES_15"
  region           = var.region
  project          = var.project_id

  settings {
    tier              = "db-n1-standard-1" # Choose an appropriate tier
    availability_type = "ZONAL"          # Use REGIONAL for high availability
    disk_autoresize   = true
    ip_configuration {
      ipv4_enabled    = false # Do not assign public IP
      private_network = google_compute_network.vpc_network.id
    }
  }

  deletion_protection = false # Set to true for production
}

# Managed Redis instance for caching and online store
resource "google_redis_instance" "redis_instance" {
  name               = "hyperbandit-redis-instance"
  tier               = "BASIC" # BASIC for dev, STANDARD_HA for prod
  memory_size_gb     = 1
  location_id        = var.region
  connect_mode       = "PRIVATE_SERVICE_ACCESS"
  authorized_network = google_compute_network.vpc_network.id
  project            = var.project_id
}
2.2.5 Identity and Access Management (IAM)Dedicated service accounts will be created to grant specific permissions to different application components, adhering to the principle of least privilege.File: infrastructure/terraform/iam.tfTerraform# infrastructure/terraform/iam.tf

# Service account for GKE nodes to interact with other GCP services
resource "google_service_account" "gke_service_account" {
  account_id   = "gke-node-sa"
  display_name = "Service Account for GKE Nodes"
  project      = var.project_id
}

# Grant the service account permissions to pull container images
resource "google_project_iam_member" "gke_sa_artifact_reader" {
  project = var.project_id
  role    = "roles/artifactregistry.reader"
  member  = "serviceAccount:${google_service_account.gke_service_account.email}"
}

# Grant the service account permissions for logging and monitoring
resource "google_project_iam_member" "gke_sa_monitoring" {
  project = var.project_id
  role    = "roles/monitoring.metricWriter"
  member  = "serviceAccount:${google_service_account.gke_service_account.email}"
}

resource "google_project_iam_member" "gke_sa_logging" {
  project = var.project_id
  role    = "roles/logging.logWriter"
  member  = "serviceAccount:${google_service_account.gke_service_account.email}"
}
Conclusion for Phase 1 This phase has established the complete foundational layer for the HyperBandit project. A professional, automated local development environment has been configured to maximize developer velocity while enforcing high code quality standards from the very first commit. The adoption of the Gitflow branching model provides a structured framework for managing the project's complexity through its lifecycle.Furthermore, the entire cloud infrastructure on GCP has been defined as code using Terraform. This creates a secure, scalable, and reproducible environment, mitigating the risks associated with manual configuration and ensuring a smooth transition from development to production. The strategic selection of managed services like GKE Autopilot, Cloud SQL, and Memorystore aligns with the project's goals of efficiency and operational excellence. With this robust foundation in place, the project is now poised to move into Phase 2: Data Engineering Pipeline, where the focus will shift to building the real-time data processing and feature engineering frameworks.
