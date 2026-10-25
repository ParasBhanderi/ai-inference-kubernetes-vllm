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
