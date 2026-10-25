# Requirements: AI Inference on Kubernetes with vLLM

## Objective
Reproduce the learning goals of https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment using vLLM and google/gemma-3-1b-it on a local ARM64 CPU Kubernetes cluster. Preserve the upstream project focus and document hardware adaptations.

## Constraints
- No paid infrastructure, paid API or cloud subscription.
- Apple M2 Pro, 16 GB host RAM, 512 GB total host disk.
- Minikube with VFkit; Docker Desktop not required.
- NVIDIA manifest cannot run unchanged. Local vLLM CPU inference is verified on the constrained 0.30.0 configuration.
- Explain each stage before the beginner user executes it.
- Do not substitute another serving engine without discussing a scope change.

## Requirements
1. Run a single-node Kubernetes cluster (verified).
2. Keep vLLM and Gemma 3 1B IT as the target stack.
3. Validate ARM64 CPU image, version, dtype and resource compatibility.
4. Create namespace and Hugging Face token Secret without committing credentials.
5. Save reproducible Deployment and Service YAML, plus storage configuration if needed.
6. Expose a local API through a ClusterIP Service and port forwarding.
7. Verify a real chat-completion response and record evidence without credentials.
8. Document setup, troubleshooting, restart and cleanup.
9. Prepare the repository for GitHub publication, accurately marking unfinished work.

## Milestones
- [x] Cluster recreated with 8 GB RAM; node Ready.
- [x] ARM64 CPU features inspected; asimd and bf16 present.
- [x] User granted access to Gemma and created a Read token.
- [x] Namespace and Secret created and verified (metadata only; model access test pending).
- [x] ARM64 vLLM image and resource settings validated with actual inference.
- [x] Inference returns a complete generated response.
- [ ] Documentation and reproducibility reviewed.
- [ ] GitHub repository published.

## Out of scope
Training models, cloud deployment, NVIDIA GPU setup, public hosting and production reliability. Nginx practice is optional, not an acceptance criterion.
