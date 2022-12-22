# daskhub

Daskhub-flavored Jupyterhub deployment configuration for argocd.

## Deploying

Unlike the other deployments in this repository, daskhub requires special attention.

If daskhub is already deployed and managed by ArgoCD, then changes here will need to be synced and deploy into the cluster through https://cd.rhgresearch.com.

If daskhub does not already exist on the cluster, and is not managed by ArgoCD, you will need to manually add the application to ArgoCD. This is unlike other deployments. We need to do initial deployment manually because of the way that daskhub handles secret and sensitive parameters.

First, you need to ensure that you have `argocd` CLI installed and configured for the cluster. If you open a terminal and type `argocd app list` you should see an updated list of live deployments.

Deploy daskhub fresh through `argocd` with:

```bash
argocd app create daskhub \
    --repo https://github.com/rhodiumgroup/rhgresearch-deployments.git \
    --revision "HEAD" \
    --path daskhub \
    --values values.yaml \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_id=$(gcloud secrets versions access latest --secret="jhub_github_client_id" --project rhgresearch) \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_secret=$(gcloud secrets versions access latest --secret="jhub_github_client_secret" --project rhgresearch) \
    --parameter daskhub.jupyterhub.hub.services.dask-gateway.apiToken=$(gcloud secrets versions access latest --secret="daskgateway_api_token" --project rhgresearch) \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace "jhub" \
    --sync-option CreateNamespace=true

```
