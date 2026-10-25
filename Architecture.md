# Architecture and Technology Stack

## Target request path
```text
Mac Terminal / curl
  -> localhost port forwarding
  -> Kubernetes ClusterIP Service
  -> Pod running vLLM CPU server
  -> google/gemma-3-1b-it
  -> generated JSON response
```

Minikube manages a Linux ARM64 VM provided by VFkit. Kubernetes uses containerd. Model execution is planned on the VM CPU; Apple GPU acceleration is not assumed.

## Stack and state
| Component | Choice | State |
| --- | --- | --- |
| Host | M2 Pro, 16 GB RAM | Confirmed |
| Local VM | VFkit / NAT | Working |
| Cluster | Minikube, Kubernetes v1.37.0 | Ready in screenshot |
| Runtime | containerd 2.3.4 | Startup output |
| CLI | kubectl in macOS Terminal | Working |
| Model server | vLLM Linux ARM64 CPU image | 0.32.0 image imports verified; inference pending |
| Model | google/gemma-3-1b-it | config.json download verified; weights not downloaded |
| Credentials | hf-secret, key hf_token | Created; screenshot verified |
| Namespace | vllm-example | Created; screenshot verified |
| Exposure | ClusterIP plus local port forwarding | Planned |

## Resources and storage
Cluster recreated with two CPUs and 8192 MB RAM. Screenshot shows 7.8 GiB total and 6.1 GiB available after startup. Previous VM inspection showed a roughly 20 GB virtual disk with 16 GB filesystem space available before recreation; recheck space before downloads.

The root filesystem and /mnt were tmpfs. Persistent disk mounts included /data and /var/lib/containerd. Plan explicit persistent model storage; do not write model weights directly to /mnt. Persistent claims and container images share the underlying VM disk capacity.

## Differences from upstream
Keep vLLM, Gemma, Kubernetes Deployment, Service and local API testing. Replace NVIDIA image/resource assumptions with verified ARM64 CPU settings. GPU-only arguments and environment settings must be reviewed, not copied blindly. Pin a validated image and explicitly budget model context, KV cache and concurrency for available RAM.

## Pending validation
Image availability and disk footprint, ARM kernel compatibility, dtype, model memory, resource requests, startup probes, shared memory, cache storage and actual inference. CPU flags alone do not prove compatibility.

## References
- https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment
- https://docs.vllm.ai/en/latest/getting_started/installation/cpu/
