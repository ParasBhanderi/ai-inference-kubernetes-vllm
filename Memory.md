# Project Memory

Updated: 2026-10-24 (Asia/Kolkata)

## Current implementation stage
Cluster setup and model access are verified. Namespace and Secret creation are pending. Verify cluster readiness before proceeding. Keep credentials outside chat and repository files.

## Project identity
Target: AI Inference with vLLM on Kubernetes.
Upstream: https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment
Project name: AI Inference on Kubernetes with vLLM.
Repository created by user: https://github.com/ParasBhanderi/ai-inference-kubernetes-vllm
Local folder remains /Users/parasbhanderi/Desktop/local-ai-kubernetes. Do not rename it automatically.

## User preferences
The project owner is a Senior SRE and DevOps professional learning AI inference and vLLM. Explain AI-specific concepts step by step while building on existing infrastructure experience. Use macOS Terminal. No paid API, cloud subscription or infrastructure. Stay focused on the upstream vLLM project. Keep repository documentation focused on technical progress and decisions, without personal scheduling details.

## Confirmed progress
- M2 Pro, 16 GB RAM, 512 GB total host storage.
- Minikube, kubectl and VFkit installed with Homebrew.
- Original 4 GB VM reported aarch64, 2 CPU cores, asimd/NEON and bf16 flags.
- Prior disk inspection showed about 20 GB virtual disk and 16 GB filesystem space free; /data and /var/lib/containerd on persistent disk; root and /mnt on tmpfs.
- Attempt to resize existing cluster by stop/start failed: Minikube required recreation. Do not repeat that instruction.
- User deleted the old minikube cluster and recreated it with:
  `minikube start --driver=vfkit --network=nat --cpus=2 --memory=8192`
- Screenshot confirms 7.8 GiB total RAM, 6.1 GiB available, node Ready, Kubernetes v1.37.0. Startup showed containerd 2.3.4.
- User accepted Gemma terms; screenshot explicitly confirms access granted to google/gemma-3-1b-it.
- User confirms Hugging Face Read token created. Token value was not shared or stored here.
- Namespace/Secret commands were provided but NOT executed or verified in conversation.
- No application deployment, model download or successful inference yet.
- GitHub repository creation confirmed by screenshot. First documentation commit e01cabf (docs: define project scope and local setup) was pushed to main successfully.
- Codex project registration required user action; available tool could not create a local project, and computer control of Codex was blocked. Registration is not confirmed.

## Scope decisions
Keep vLLM and google/gemma-3-1b-it. Original NVIDIA deployment cannot run unchanged on M2 Pro; assess documented Linux ARM64 CPU adaptation. Do not replace vLLM with another engine without discussion. Nginx is optional practice and not the next project milestone.

## Next implementation steps
1. Check cluster/node status; start existing cluster if stopped.
2. Create namespace vllm-example and Secret hf-secret with key hf_token. Read token silently in Terminal and pipe it to kubectl; never paste token into chat, source files or shell history.
3. Check/pin ARM64 CPU image and disk footprint before pulling it.
4. Prepare manifests with CPU-specific resource, dtype, context/cache, storage and probe settings.
5. Test server startup and actual inference; record only verified results.

## Planned repository structure
Existing: README.md, prd.md, Architecture.md, Memory.md, .gitignore.
Planned: manifests/namespace.yaml, manifests/vllm-deployment.yaml, manifests/vllm-service.yaml; docs/setup.md, docs/verification.md, docs/troubleshooting.md. Additional storage manifest if required.

## References
- https://docs.vllm.ai/en/latest/getting_started/installation/cpu/
- https://huggingface.co/google/gemma-3-1b-it
- https://huggingface.co/docs/hub/security-tokens

## Continuity
Record observed outcomes separately from plans. Never store credentials or private screenshots. Stop/start preserves a cluster; deletion removes cluster resources and VM data. Current cluster runtime status requires verification.

## Version control workflow
Commit and push in meaningful batches after review and relevant verification. First batch: project documentation. Never stage credentials or model weights.
