# Project Memory

Updated: 2026-10-25 (Asia/Kolkata)

## Identity and scope
AI Inference on Kubernetes with vLLM.
Repository: https://github.com/ParasBhanderi/ai-inference-kubernetes-vllm
Local directory: /Users/parasbhanderi/Desktop/local-ai-kubernetes
Upstream: https://github.com/kubernetes/examples/tree/master/AI/vllm-deployment
Keep vLLM and google/gemma-3-1b-it. Adapt NVIDIA assumptions to Linux ARM64 CPU. No paid infrastructure, API or cloud subscription.

## Collaboration
Project owner is a Senior SRE/DevOps professional learning AI inference. Explain AI concepts and material changes; build on existing infrastructure knowledge. Maintain professional technical documentation, without personal scheduling details. Commit and push reviewed changes in meaningful batches. Never request, display or commit credentials.

## Current outcome
Actual inference passed on 2026-10-25. Gemma returned a complete two-sentence Kubernetes explanation via the ClusterIP Service and temporary localhost port-forward. 16 prompt tokens, 41 generated tokens, finish_reason stop. Request and actual response saved under examples/. Final Pod 1/1 Ready with zero restarts. /v1/models also passed.

## Working configuration
M2 Pro host, 16 GB RAM. Minikube VFkit/NAT VM with 2 CPUs and 8192 MB RAM. Kubernetes 1.37.0, containerd 2.3.4. CPU flags asimd and bf16 verified.
Deployment: vllm-gemma-deployment, namespace vllm-example; Service vllm-service port 8080.
Image: vllm/vllm-openai-cpu:v0.30.0-arm64@sha256:09e656588c2dc572240cb62a22f75681dc640a99c61537bbd55e9eb1dc3d4fbc.
BF16, eager, uni executor, VLLM_ENABLE_V1_MULTIPROCESSING=0, OMP_NUM_THREADS=1, thread binding nobind. 256-token context, 128-token batch, one sequence, 256 MiB KV cache. Requests 1 CPU/4 GiB RAM; limits 2 CPU/6 GiB RAM. Recreate strategy and health probes.
PVC gemma-model-cache: 8 GiB standard RWO; HF_HOME=/model-cache/huggingface; cache approximately 1.9 GiB. Secret hf-secret key hf_token, valid authentication and Gemma config download verified. No token values stored in project.

## Findings
- Existing 4 GB VFkit cluster could not resize via stop/start. User recreated with 8 GB before app deployment.
- Earlier incorrect Secret contents caused 401; corrected using hidden input, then recreated check Pod. Authentication and gated config download passed.
- vLLM 0.32.0 import passed. Initial multiprocess serving OOMKilled at 6 GiB. Uni executor reached readiness but inference crashed with CPU slot mapping Tensor Boolean ambiguity.
- v0.18.0 loaded cached weights but exceeded memory during final API initialization, even after reducing token buffers.
- v0.30.0 with small explicit cache passed actual inference. No upstream source patched.
- Downloaded weights remained on PVC across image/Pod replacements. No model redownload needed for compatibility retries.
- /mnt and root are tmpfs; use persistent PVC. PVC and container images share VM disk. Multiple image versions consume space; recheck free disk before more downloads.

## Repository contents
Core docs: README.md, prd.md, Architecture.md, Memory.md.
Manifests: namespace.yaml, model-cache-pvc.yaml, vllm-deployment.yaml, vllm-service.yaml; checks/model-access.yaml and checks/token-auth.yaml.
Guides: docs/setup.md, docs/deployment.md, docs/verification.md, docs/troubleshooting.md.
Examples: chat-request.json, chat-response.json, models-response.json.

## Remaining work
Formal stop/start verification, explicit model revision pinning, optional observability, load tests and SLOs. Functional inference is complete; no production performance or reliability claims. User-accessible port-forward can be started with the command in README. Cluster and server remain running unless explicitly stopped. Old check Pods retain historical completed/error state and do not indicate current server failure.

## Version control
Previous commits: e01cabf, 168734c, e9dc054. Current batch adds validated CPU deployment, persistent cache and real inference evidence. Commit/push after file review. Codex project registration is not confirmed by available tools; do not alter app settings silently.
