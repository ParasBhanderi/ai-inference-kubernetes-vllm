# Project Memory

Updated: 2026-10-25 (Asia/Kolkata)

## Current implementation stage
Namespace, Secret authentication and Gemma configuration download are verified. ARM64 vLLM image imports pass. Next: prepare CPU Deployment, Service and persistent model cache, then validate weight loading and actual inference. No model weights downloaded yet.

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
- Namespace vllm-example and Secret hf-secret created. Screenshot shows Opaque / DATA 1. Temporary token variable cleared. User confirms exposed token revoked and replacement used. Token validity has not been tested against the model.
- No application deployment, model download or successful inference yet.
- GitHub repository creation confirmed by screenshot. First documentation commit e01cabf (docs: define project scope and local setup) was pushed to main successfully.
- Codex project registration required user action; available tool could not create a local project, and computer control of Codex was blocked. Registration is not confirmed.

## Scope decisions
Keep vLLM and google/gemma-3-1b-it. Original NVIDIA deployment cannot run unchanged on M2 Pro; assess documented Linux ARM64 CPU adaptation. Do not replace vLLM with another engine without discussion. Nginx is optional practice and not the next project milestone.

## Next implementation steps
1. Check cluster/node status; start existing cluster if stopped.
2. Namespace and Secret are complete. Do not recreate or disclose the Secret.
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

## CPU image preflight
Docker Hub metadata checked 2026-10-25: v0.32.0-arm64 available, about 1.15 GiB compressed, digest sha256:433cbee5e0b689095484090454b6fcdf0c68fe3eabfbb251a5e39d442917a4a6. Unpacked footprint and VM execution not yet validated. Next: recheck disk space, then an image-only import test before model deployment. Added manifests/namespace.yaml and docs/setup.md; not yet committed.

## Verified image import test — 2026-10-25
Pod vllm-image-check completed successfully with zero restarts. Logs show aarch64, PyTorch 2.13.0+cpu, vLLM 0.32.0. Image execution and imports verified; inference kernels and model execution remain untested. Persistent disk recheck showed about 16 GB free before image pull. User updated Secret via clipboard; encoded value length 264, validity not yet tested. Prepared manifests/checks/model-access.yaml to download only Gemma config.json using existing hf-secret. Next action: run that check and inspect sanitized logs.

## Model access failure — 2026-10-25
Screenshot confirms gemma-access-check Error, GatedRepoError, HTTP 401. No weights downloaded. Token length is not evidence of validity. Next: apply manifests/checks/token-auth.yaml and inspect sanitized logs from hf-auth-check to distinguish invalid format/authentication from model permissions.

## Authentication resolved — 2026-10-25
After Secret correction and check Pod recreation, screenshot confirms Token format PLAUSIBLE, Token authentication SUCCESS, Gemma configuration download SUCCESS, No model weights downloaded. This supersedes earlier pending/401/invalid-format results. Record these checks as passed; do not ask for token values or repeat authentication setup. Next implementation milestone is the vLLM server deployment.
