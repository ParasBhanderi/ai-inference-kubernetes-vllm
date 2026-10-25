# AI Inference on Kubernetes with vLLM

A work-in-progress lab for serving Google Gemma with vLLM on local Kubernetes, adapting the upstream NVIDIA GPU example to an Apple Silicon Linux ARM64 CPU environment. No cloud subscription or paid API is planned.

Repository: https://github.com/ParasBhanderi/ai-inference-kubernetes-vllm

## Status
Kubernetes setup is verified. The recreated cluster has 8192 MB RAM, two CPU cores, and a Ready node. Gemma access has been granted and the user has created a Hugging Face Read token. Namespace, token authentication and Gemma configuration download are verified. The ARM64 vLLM 0.32.0 image import check completed successfully. Model weights have not been downloaded; local inference is not yet validated.

## Scope
Follow [AI Inference with vLLM on Kubernetes](https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment), keeping vLLM and `google/gemma-3-1b-it`. Document CPU-specific changes explicitly. An Nginx exercise is optional and outside the required project path.

## Stack
Apple M2 Pro / 16 GB host RAM; macOS Terminal; VFkit; Minikube; Kubernetes; containerd; kubectl; planned vLLM ARM64 CPU image; Gemma 3 1B IT; Kubernetes YAML; curl and JSON.

## Documentation
- [Requirements](prd.md)
- [Architecture](Architecture.md)
- [Progress and resume point](Memory.md)

## Cluster setup
For a NEW cluster:
```bash
minikube start --driver=vfkit --network=nat --cpus=2 --memory=8192
kubectl get nodes
minikube ssh -- "free -h"
```
The existing 4 GB cluster could not be resized with stop/start; it was deleted and recreated before any application deployment. Deleting a cluster removes its resources and VM data and is not a routine restart procedure.

For a break:
```bash
minikube stop
```
To resume the existing cluster:
```bash
minikube start
kubectl get nodes
```

## Remaining work
1. Completed: namespace, Secret and gated-model access verification; see [setup](docs/setup.md).
2. Verify and pin an ARM64 CPU image; check its size and CPU compatibility.
3. Prepare deployment, Service, storage and resource configuration for Gemma.
4. Deploy, port-forward and verify an actual generated response.
5. Document troubleshooting, restart behavior and cleanup; publish verified results.

## Credentials and model files
Never commit Hugging Face tokens, Secret manifests containing credentials, kubeconfig, model weights or personal screenshots. Model license acceptance does not prove that inference works locally.

## References
- https://minikube.sigs.k8s.io/docs/drivers/vfkit/
- https://docs.vllm.ai/en/latest/getting_started/installation/cpu/
- https://huggingface.co/google/gemma-3-1b-it
- https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

Verification details: [image and model access checks](docs/verification.md).
