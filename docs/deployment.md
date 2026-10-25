# CPU inference deployment

## Files
- manifests/model-cache-pvc.yaml: 8 GiB ReadWriteOnce claim using Minikube standard storage.
- manifests/vllm-deployment.yaml: one vLLM CPU replica, existing hf-secret, persistent Hugging Face cache, shared memory, startup/readiness/liveness probes.
- manifests/vllm-service.yaml: ClusterIP on port 8080.

The digest-pinned ARM64 image is vLLM 0.30.0. Initial limits: BF16 weights, 256-token total context, 128-token prefill batches, one sequence, 256 MiB KV cache, eager execution, uni executor and one OpenMP thread, CPU request 1/limit 2, memory request 4 GiB/limit 6 GiB. These are test settings, subject to measured results. Recreate strategy prevents overlapping model replicas during updates.

## Apply (from repository root)
```bash
kubectl --context=minikube apply -f manifests/namespace.yaml
kubectl --context=minikube apply -f manifests/model-cache-pvc.yaml -f manifests/vllm-deployment.yaml -f manifests/vllm-service.yaml
kubectl --context=minikube -n vllm-example get pods,pvc,svc
kubectl --context=minikube -n vllm-example logs -f deployment/vllm-gemma-deployment
```
First startup downloads model weights into the PVC. Startup probe allows up to 30 minutes. Inspect logs and Pod events rather than assuming timeouts indicate failed downloads.

## Local access
In a separate Terminal:
```bash
kubectl --context=minikube -n vllm-example port-forward service/vllm-service 8080:8080 --address=127.0.0.1
```
Then:
```bash
curl --fail --silent --show-error http://127.0.0.1:8080/v1/models
curl --fail --silent --show-error http://127.0.0.1:8080/v1/chat/completions -H 'Content-Type: application/json' -d '{"model":"google/gemma-3-1b-it","messages":[{"role":"user","content":"Explain Kubernetes in two short sentences."}],"max_tokens":64,"temperature":0}'
```

## Storage lifecycle
The PVC preserves cache across Pod recreation and minikube stop/start. This single-node hostpath storage does not survive minikube delete. An 8 GiB claim is not a dedicated physical disk or a guaranteed hostpath quota; container images and cache share underlying disk capacity. Do not delete the PVC during routine restarts.

## Reference
https://docs.vllm.ai/en/stable/getting_started/installation/cpu/

## Verified result (2026-10-25)
Server-side validation and live deployment passed. PVC Bound, serving Pod 1/1 Running with zero restarts on final configuration. Model cache approximately 1.9 GiB. /v1/models returned the target model, and /v1/chat/completions returned a complete answer (16 prompt tokens, 41 completion tokens, finish_reason stop). Example request and response are in examples/. No GPU or paid service was used. This is a functional test, not a load or production reliability test.

To repeat the recorded request after port forwarding:
```bash
curl --fail --silent --show-error http://127.0.0.1:8080/v1/chat/completions -H 'Content-Type: application/json' --data-binary @examples/chat-request.json
```
Image is pinned by digest. Model revision currently follows the Hugging Face repository default; pinning the model revision is a future reproducibility improvement. Stop the port-forward with Control+C. Use minikube stop/start for routine shutdown/resume; do not delete the cluster or PVC if retaining cache is intended.
