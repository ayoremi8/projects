# Offline Batch Inference Architecture (GKE, PyTorch, DWS)

## Overview
This project implements distributed offline batch inference using Google Kubernetes Engine (GKE), PyTorch, and Dynamic Workload Scheduler (DWS). Below is an architectural map and component breakdown based on the repository structure.

---

## Architectural Diagram (Text)

```
+-------------------+         +-------------------+         +-------------------+
|   GCS Bucket      | <-----> |   GKE Cluster     | <-----> |   DWS Scheduler   |
| (Input/Output)    |         | (K8s Jobs/Pods)   |         | (GPU Provisioning)|
+-------------------+         +-------------------+         +-------------------+
         ^                             |
         |                             |
         |                             v
         |                  +-------------------+
         |                  | PyTorch DDP Pods  |
         |                  | (Docker Image)    |
         |                  +-------------------+
         |                             |
         +-----------------------------+
```

---


## Component Breakdown (by Folder)

### batch-inference-pytorch/
- `Dockerfile`: Containerizes the PyTorch inference job for distributed batch inference.
- `batch_inference.py`: Main distributed inference script using PyTorch DDP.
- `requirements.torch`: Python dependencies for PyTorch and related tools.
- `upload_images.py`: Script to upload images from `val2017/` to your GCS bucket.
- `val2017/`: Local folder for COCO dataset images (input data for inference).
- `val2017.zip`: Downloaded COCO dataset archive.
- `README.md`: Step-by-step instructions for setup, data upload, job launch, and cleanup.
- `standard/`
  - `batch_inference.yaml`: Kubernetes job manifest for launching distributed batch inference on GKE with DWS and GPU nodes.
  - `setup/`
    - `.env`: Environment variables for GCP, GKE, bucket names, and job configuration.
    - `setup.sh`: Script to initialize environment and resources.
    - `cleanup.sh`: Script to clean up resources after job completion.
    - `kueue.yaml`, `nccl_installer.yaml`, `network_mapping_template.yaml`: Network, RDMA, and scheduler configuration files for advanced cluster setup.

### adk/
- Contains subfolders for different ML orchestration and deployment patterns (e.g., `llama/`, `memory/`, `ray-mcp/`, `vertex/`).

### agentic-llamaindex/
- `rag/`: Example for retrieval-augmented generation workflows.

### autoscale/
- YAML manifests and scripts for GPU autoscaling, pod monitoring, and horizontal pod autoscaler setups.

### blue-green-gateway/
- Blue/green deployment manifests, gateway configs, health policies, and service definitions for safe model rollout.

### ci-resources/
- CI/CD resources, JWT generation, and scheduling examples for automated workflows.

### dws-flex-training-pytorch/
- Example environment, setup scripts, and training utilities for flexible distributed training with DWS and PyTorch.

### finetuning-gemma-3-1b-it-on-l4/
- Dockerfile, fine-tuning scripts, and cloudbuild manifests for Gemma model training.

### flowise/
- Agentflow configuration and Terraform scripts for workflow automation.

### flyte/
- Terraform and environment files for Flyte workflow orchestration.

### hugging-face-tgi/
- Terraform, cloudbuild, and deployment manifests for Hugging Face TGI serving.

### inference-servers/
- Subfolders for different inference server setups (e.g., `checkpoints/`, `jetstream/`, `maxdiffusion/`).

### langchain-chatbot/
- App and deployment manifests for LangChain chatbot, including backend, ingress, and certificate management.

### llamaindex/
- `rag/`: Retrieval-augmented generation example.

### lora-inference-gateway/
- Gateway, health policy, and model pool manifests for LoRA-based inference serving.

### metaflow/
- Metaflow workflow, model serving, and fine-tuning examples with Terraform support.

### mlflow/
- Fine-tuning workflow for Gemma using MLflow.

### models-as-oci/
- Dockerfile and cloudbuild for packaging models as OCI images.

### n8n/
- Workflow automation templates and Terraform scripts for n8n orchestration.

### ray-serve/
- Ray Serve agent, vllm deployment, and Terraform for scalable model serving.

### security/
- Model firewall and armor configurations for securing ML deployments.

### security_test/
- Allowlist and security test resources.

### skippy-chart/
- Helm chart and package for skippy deployment.

### skypilot/
- Launch scripts, training YAMLs, and text classification examples for SkyPilot.

### skypilot-dws-kueue/
- Backend, code, and training manifests for SkyPilot with DWS and Kueue integration.

### storage/
- Backup and recovery scripts for parallelstore.

### workflow-orchestration/
- DWS and multikueue orchestration examples for advanced workflow management.

---

## Workflow
1. **Prepare Data**: Download COCO dataset, upload to GCS
2. **Build Docker Image**: Containerize PyTorch job
3. **Push Image**: Upload to Artifact Registry
4. **Apply Manifest**: Launch job on GKE via DWS
5. **Monitor & Collect Results**: Output CSVs written to GCS

---

## Key Technologies
- Google Cloud Storage (GCS)
- Google Kubernetes Engine (GKE)
- Dynamic Workload Scheduler (DWS)
- PyTorch DistributedDataParallel (DDP)
- Docker
- Kubernetes

---

## Notes
- Each pod processes a shard of data and writes results to GCS
- DWS ensures GPU resources are provisioned only when needed
- The architecture is modular and extensible for other ML workflows
