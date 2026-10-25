# Architecture and Technology Stack

## Verified request path
```text
Mac curl -> localhost port-forward -> ClusterIP Service:8080
  -> vLLM CPU Pod -> Gemma 3 1B IT -> generated JSON response
```

Minikube manages a Linux ARM64 VM using VFkit/NAT. Kubernetes runs containers with containerd. Apple GPU acceleration is not used.

## Runtime configuration
- Host: M2 Pro, 16 GB RAM. VM: 2 CPUs, 8192 MB RAM.
- vLLM 0.30.0 ARM64 CPU image pinned by SHA256 in the Deployment.
- Model: google/gemma-3-1b-it, BF16; upstream model revision currently follows its default branch.
- Eager execution; uni executor; VLLM_ENABLE_V1_MULTIPROCESSING=0 avoids the CPU executor auto-switch; OpenMP nobind with one thread.
- 256-token total context, 128-token prefill batches, one sequence, explicit 256 MiB KV cache.
- Container requests 1 CPU/4 GiB RAM, limits 2 CPU/6 GiB RAM.
- Recreate strategy prevents overlapping replicas on this constrained node.
- Startup, readiness and liveness use HTTP /health. Startup allows 30 minutes for initial loading.
- Namespace vllm-example; hf-secret key hf_token supplies HF_TOKEN; no embedded credentials.
- Service is ClusterIP on 8080. Local forwarding binds only 127.0.0.1.

## Model storage
HF_HOME=/model-cache/huggingface on gemma-model-cache, an 8 GiB standard ReadWriteOnce PVC. Cache measured approximately 1.9 GiB after download. /dev/shm uses a memory-backed emptyDir limited to 256 MiB. The local PVC is hostpath storage, shares the VM disk with images, and is not resilient to cluster deletion. Claim size is not a dedicated physical disk or guaranteed hostpath quota.

## Validation
Server-side dry run passed. PVC bound. Final serving Pod became Ready with zero restarts. /v1/models and a complete /v1/chat/completions response passed through the Service. Cache survived image/Pod changes. No production reliability or load claims are made.

## Compatibility decisions
Original example requires NVIDIA hardware unavailable on this Mac. Preserve vLLM and Gemma while using ARM64 CPU serving. Tested 0.32.0 failed with CPU slot mapping during inference; 0.18.0 exceeded the memory budget; 0.30.0 with constrained buffers passed. See docs/troubleshooting.md for observations.

Reference: https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment
