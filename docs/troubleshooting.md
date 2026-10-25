# Troubleshooting

## Existing VFkit VM cannot change memory
Observed: starting an existing 4 GB cluster with --memory=8192 warned that the memory size could not be changed. Stop/start retained 4 GB. Before any application deployment, the cluster was deleted and recreated with 8192 MB, then verified with free -h and kubectl get nodes. Do not delete a populated cluster without accounting for resources and VM data.

## Gemma access returned HTTP 401
Observed: model-access check failed with GatedRepoError / 401. A separate token-auth check reported INVALID format before attempting authentication. The Secret contained incorrectly entered text.

Resolution: enter the unchanged zsh read command first, paste the actual token only after its prompt, verify format without displaying the token, update the Secret, then recreate the check Pod. The next run authenticated successfully and downloaded config.json.

Secret DATA 1 and a positive encoded length prove only that a value exists. They do not prove token validity. Pods receiving Secrets through environment variables need recreation to pick up updated values. Never log or commit token values.

## ContainerCreating on first image use
Observed: the image-check Pod initially showed ContainerCreating, then Running and Completed. Inspect Pod events if delayed to distinguish image pull progress from an error. Import success alone does not prove that native inference kernels or model serving work.
