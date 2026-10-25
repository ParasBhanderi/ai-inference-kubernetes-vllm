# Verification record

## ARM64 image import check — 2026-10-25
User ran an image-only Pod using vllm/vllm-openai-cpu:v0.32.0-arm64. Screenshot shows Completed with zero restarts and these logs:
```text
Architecture: aarch64
PyTorch: 2.13.0+cpu
vLLM: 0.32.0
```
This confirms image execution and package imports on the VM. It does not validate model loading, native inference kernels or generated responses.

## Model access check — pending
From the project root:
```bash
kubectl --context=minikube apply -f manifests/checks/model-access.yaml
kubectl --context=minikube -n vllm-example get pod gemma-access-check -w
```
After Completed or Error, stop the watch with Control+C and run:
```bash
kubectl --context=minikube -n vllm-example logs gemma-access-check
```
Expected: Gemma configuration download: SUCCESS. This downloads only config.json and references the existing Secret without embedding credentials. Model weights and inference are not tested. Failure logs print only error class and HTTP status, not credentials. A 401 or 403 requires checking token and gated-model permissions; network errors need separate diagnosis.

## First Gemma access attempt — 2026-10-25
The gemma-access-check Pod exited with GatedRepoError / HTTP 401. Hugging Face was reachable, but authorization failed; token validity versus model permissions is not yet distinguished. Added manifests/checks/token-auth.yaml to validate token format and call whoami before retrying the configuration download, without logging credentials or account details. Result pending.

## Authentication and gated-model access — PASS, 2026-10-25
After correcting the Secret and recreating hf-auth-check, screenshot shows Completed, zero restarts, and:
```text
Token format: PLAUSIBLE
Token authentication: SUCCESS
Gemma configuration download: SUCCESS
No model weights downloaded.
```
Both authentication and access to Gemma config.json are verified. Model weight downloads, vLLM model initialization and inference are still pending.

## Actual CPU inference — PASS, 2026-10-25
Final deployment: vllm/vllm-openai-cpu:v0.30.0-arm64, digest pinned in manifest. BF16, uni executor, eager execution, 256-token context, 128-token prefill batch, one sequence, 256 MiB KV cache, 6 GiB container memory limit in 8 GiB VM.

- Kubernetes server dry run passed before application.
- PVC Bound, 8 GiB; cache approximately 1.9 GiB.
- Final serving Pod 1/1 Running, zero restarts after request.
- /health returned 200.
- /v1/models returned google/gemma-3-1b-it.
- Chat request via localhost port-forward to the Service returned HTTP success, 16 prompt tokens, 41 completion tokens, finish_reason stop.

Actual response:
> Kubernetes is a powerful platform for automating deployment, scaling, and management of containerized applications. It allows you to efficiently run and manage applications across a cluster of machines, ensuring high availability and resilience.

Saved request/response: examples/chat-request.json, examples/chat-response.json. Model listing: examples/models-response.json. No load benchmark, formal stop/start test or SLO validation was performed. Earlier import-only results and failed model-access attempts above are historical checkpoints, superseded by later successful checks.
