# AI Inference on Kubernetes with vLLM

Serve Google Gemma 3 1B IT with vLLM on local Kubernetes using an Apple Silicon CPU, without a cloud subscription or paid inference API.

**Status: local inference verified on 2026-10-25.** Gemma weights downloaded to persistent storage; a chat-completion request through the Kubernetes Service returned a complete generated answer. This is a functional lab, not a production deployment or performance benchmark.

Adapted from [AI Inference with vLLM on Kubernetes](https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment). The upstream NVIDIA image is replaced by a digest-pinned ARM64 CPU image; model serving remains vLLM with google/gemma-3-1b-it.

## Working configuration
| Component | Setting |
| --- | --- |
| Host | Apple M2 Pro, 16 GB RAM |
| Cluster | Minikube + VFkit/NAT; 2 CPUs, 8192 MB VM RAM |
| Server | vLLM 0.30.0 ARM64 CPU, BF16, eager execution, uni executor |
| Resources | 1 CPU/4 GiB requested; 2 CPU/6 GiB limits |
| Inference | 256-token context; 128-token batch; one sequence; 256 MiB KV cache |
| Model cache | 8 GiB ReadWriteOnce PVC; approximately 1.9 GiB used by Hugging Face cache |
| Access | ClusterIP port 8080 plus localhost port forwarding |

Docker Desktop is not required. Kubernetes uses containerd inside the Linux VM. Hugging Face model access and a Read token are required.

## Documentation
- [Requirements](prd.md)
- [High-level system design](Design.md)
- [Architecture](Architecture.md)
- [Setup and credentials](docs/setup.md)
- [Deployment and sample request](docs/deployment.md)
- [Verification record](docs/verification.md)
- [Troubleshooting](docs/troubleshooting.md)

## Deploy
Complete [setup](docs/setup.md), including the namespace and token Secret, first. Then from the repository root:
```bash
kubectl --context=minikube apply -f manifests/model-cache-pvc.yaml -f manifests/vllm-deployment.yaml -f manifests/vllm-service.yaml
kubectl --context=minikube -n vllm-example get pods,pvc,svc
```
Initial startup downloads weights; monitor logs as described in the deployment guide.

Forward the Service in one Terminal:
```bash
kubectl --context=minikube -n vllm-example port-forward service/vllm-service 8080:8080 --address=127.0.0.1
```
Send a request in another:
```bash
curl --fail --silent --show-error http://127.0.0.1:8080/v1/chat/completions -H 'Content-Type: application/json' --data-binary @examples/chat-request.json
```
A recorded actual response is in [examples/chat-response.json](examples/chat-response.json).

## Operational findings
vLLM 0.32.0 imported successfully and reached readiness after memory tuning but failed on actual inference with a CPU slot-mapping error. v0.18.0 exceeded the memory limit during startup. The final 0.30.0 configuration passed inference. Details are in troubleshooting.md.

## Storage and credentials
Model weights stay in the PVC and out of Git. Never commit tokens, rendered Secret YAML, kubeconfig or private screenshots. The single-node cache survives Pod replacement and Minikube stop/start, but not cluster deletion. The PVC and container images share the VM disk. Model revision pinning, restart testing, load testing, and SLOs remain future work.

## Stop and resume
```bash
minikube stop
minikube start
```
These commands preserve the existing cluster; deletion is not a routine restart.
