# AI Infrastructure Lab

AIインフラストラクチャに関する学習・検証・実験を記録するリポジトリ。

## Learning Topics

### 01. Kubernetes
- Cluster / Node / Pod
- Deployment / Service
- Resource Management
- Scheduling / Scaling

### 02. KubeAI
- Architecture
- Model Operator
- Model Proxy
- Model Deployment
- CPU / GPU Resource Profiles

### 03. LLM Inference
- vLLM
- Ollama
- llama.cpp
- Continuous Batching
- KV Cache

### 04. Distributed Inference
- Data Parallelism
- Tensor Parallelism
- Pipeline Parallelism

### 05. Distributed Training
- PyTorch Distributed
- DistributedDataParallel
- DeepSpeed
- NCCL

### 06. GPU Computing
- GPU Architecture
- CUDA
- GPU Memory
- GPU Communication

## Learning Policy

- Understand the architecture before memorizing commands.
- Verify concepts through hands-on experiments.
- Record experiment results and observations.
- Connect AI infrastructure concepts with distributed computing and data platforms.

## Learning Logs

- [Kubernetes environment setup](docs/kubernetes/00-environment-setup.md)
- [Kubernetes Cluster, Node, and Pod](docs/kubernetes/02-cluster-node-pod.md)
- [Kubernetes Deployment](docs/kubernetes/03-deployment.md)
- [Kubernetes Service](docs/kubernetes/04-service.md)
- [Kubernetes Namespace](docs/kubernetes/05-namespace.md)
- [Kubernetes ConfigMap and Secret](docs/kubernetes/06-configmap-secret.md)
- [Kubernetes CPU and memory requests/limits](docs/kubernetes/07-resource-requests-limits.md)
- [Kubernetes scaling](docs/kubernetes/08-scaling.md)
- [Kubernetes application management](docs/kubernetes/09-application-management.md)
- [KubeAI installation and CPU inference](docs/kubeai/01-install-and-cpu-inference.md)
