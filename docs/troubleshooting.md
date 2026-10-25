# Troubleshooting

## Existing VFkit VM cannot change memory
Observed: starting an existing 4 GB cluster with --memory=8192 warned that the memory size could not be changed. Stop/start retained 4 GB. Before any application deployment, the cluster was deleted and recreated with 8192 MB, then verified with free -h and kubectl get nodes. Do not delete a populated cluster without accounting for resources and VM data.

## Gemma access returned HTTP 401
Observed: model-access check failed with GatedRepoError / 401. A separate token-auth check reported INVALID format before attempting authentication. The Secret contained incorrectly entered text.

Resolution: enter the unchanged zsh read command first, paste the actual token only after its prompt, verify format without displaying the token, update the Secret, then recreate the check Pod. The next run authenticated successfully and downloaded config.json.

Secret DATA 1 and a positive encoded length prove only that a value exists. They do not prove token validity. Pods receiving Secrets through environment variables need recreation to pick up updated values. Never log or commit token values.

## ContainerCreating on first image use
Observed: the image-check Pod initially showed ContainerCreating, then Running and Completed. Inspect Pod events if delayed to distinguish image pull progress from an error. Import success alone does not prove that native inference kernels or model serving work.

## vLLM 0.32.0 CPU serving failures
Initial multiprocess startup downloaded and loaded 1.86 GiB of weights but the container exceeded its 6 GiB limit (OOMKilled, exit 137). Using the uni executor reduced process overhead and reached readiness. However, the first inference request failed with HTTP 500 and a CPU slot-mapping RuntimeError (Boolean value of Tensor with more than one value is ambiguous). This demonstrates why package imports and /health do not prove inference correctness. Deployment was pinned to official v0.18.0-arm64 as a compatibility fallback, retaining the model cache. See verification.md for final status.

## Working compatibility baseline
v0.18.0 reached model loading/warmup but exceeded the 6 GiB limit during final server initialization, including with reduced token buffers. The final working version is v0.30.0-arm64 with an explicit 256 MiB KV cache, uni executor, VLLM_ENABLE_V1_MULTIPROCESSING=0, one OpenMP thread, 256-token context and 128-token batch. It returned an actual chat response and remained Ready with zero restarts. Do not infer that an import or health check alone validates inference. No image source code was patched.
