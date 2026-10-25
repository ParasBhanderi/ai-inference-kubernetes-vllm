# Local setup and model credentials

## Cluster
Use the existing Minikube VFkit cluster (2 CPUs, 8192 MB RAM). Start it with `minikube start` if stopped. Verify with `kubectl --context=minikube get nodes`.

## Namespace
From the repository root:
```bash
kubectl --context=minikube apply -f manifests/namespace.yaml
```
The namespace was initially created imperatively; this manifest records the same resource reproducibly.

## Model access
Sign in at https://huggingface.co/google/gemma-3-1b-it and review/accept the model access conditions yourself. Create a Hugging Face Read token. Do not commit its value.

## Secret creation (macOS zsh)
Run each command separately. The read command must be entered literally; paste the token only AFTER the prompt appears. Input is invisible.
```zsh
read -rs "VLLM_HF_TOKEN?Paste your Hugging Face token, then press Return: "
```
```zsh
printf '%s' "$VLLM_HF_TOKEN" | kubectl --context=minikube -n vllm-example create secret generic hf-secret --from-file=hf_token=/dev/stdin
```
```zsh
unset VLLM_HF_TOKEN
```
```bash
kubectl --context=minikube -n vllm-example get secret hf-secret
```
Expect type Opaque and DATA 1. This verifies resource creation, not token validity. Do not rerun Secret creation if it already exists. Do not export its contents to YAML or show the token in screenshots. Revoke and replace any exposed token.

## CPU image
Public Docker Hub metadata checked 2026-10-25 lists `vllm/vllm-openai-cpu:v0.32.0-arm64`, architecture arm64, with about 1.15 GiB compressed image data. Unpacked layers require additional space. Candidate digest: `sha256:433cbee5e0b689095484090454b6fcdf0c68fe3eabfbb251a5e39d442917a4a6`.

Image pull and imports are verified on this VM: aarch64, PyTorch 2.13.0+cpu, vLLM 0.32.0. The token-auth check also authenticated the Secret and downloaded Gemma config.json successfully. Model weight loading and inference remain untested.

References:
- https://docs.vllm.ai/en/stable/getting_started/installation/cpu/
- https://hub.docker.com/r/vllm/vllm-openai-cpu/tags
- https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

## Verify authentication and gated-model access
```bash
kubectl --context=minikube apply -f manifests/checks/token-auth.yaml
kubectl --context=minikube -n vllm-example get pod hf-auth-check -w
```
After Completed or Error, press Control+C and inspect sanitized logs:
```bash
kubectl --context=minikube -n vllm-example logs hf-auth-check
```
Expected: Token authentication: SUCCESS and Gemma configuration download: SUCCESS. No weights are downloaded.

To update an existing Secret, repeat the hidden-input read above with the actual token pasted AFTER the prompt, then run:
```zsh
printf '%s' "$VLLM_HF_TOKEN" | kubectl --context=minikube -n vllm-example create secret generic hf-secret --from-file=hf_token=/dev/stdin --dry-run=client -o yaml | kubectl --context=minikube apply -f -
unset VLLM_HF_TOKEN
```
Recreate the check Pod to read the updated environment:
```bash
kubectl --context=minikube -n vllm-example delete pod hf-auth-check
kubectl --context=minikube apply -f manifests/checks/token-auth.yaml
```
